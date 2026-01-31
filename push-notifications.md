# Push Notifications & Reminders System

## Overview

This document covers the push notification and reminder system implementation. This is separate from the in-app Notification Center (see `NOTIFICATION_PLAN.md`).

**Purpose**: Allow stake leaders to send reminders about meetings and notify users about their assignments.

---

## Architecture

### Database Tables (`058_push_notifications.sql`)

```sql
-- Push tokens: stores Expo push tokens for each user
push_tokens (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(id),
  token TEXT NOT NULL,           -- ExponentPushToken[...]
  platform TEXT NOT NULL,        -- 'ios', 'android', 'web'
  UNIQUE(user_id)
)

-- Scheduled notifications: queue for meeting/assignment reminders
scheduled_notifications (
  id UUID PRIMARY KEY,
  stake_id UUID NOT NULL,
  notification_type TEXT,        -- 'meeting_reminder', 'assignment_reminder', 'agenda_sent', 'custom'
  meeting_type TEXT,             -- 'presidency', 'high_council', 'stake_council'
  target_user_id UUID,           -- For individual notifications
  target_audience TEXT,          -- 'po', 'hc', 'sc', 'all' for group notifications
  title TEXT NOT NULL,
  body TEXT NOT NULL,
  data JSONB,                    -- Navigation data (screen, meetingType, etc.)
  scheduled_for TIMESTAMPTZ,
  sent_at TIMESTAMPTZ,
  status TEXT,                   -- 'pending', 'sent', 'failed', 'cancelled'
  reference_type TEXT,           -- What triggered this (e.g., 'assignment')
  reference_id UUID
)

-- Notification preferences: user-specific settings
notification_preferences (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  meeting_reminder_enabled BOOLEAN DEFAULT TRUE,
  meeting_reminder_hours INTEGER DEFAULT 24,
  meeting_reminder_day_of BOOLEAN DEFAULT TRUE,
  assignment_reminder_enabled BOOLEAN DEFAULT TRUE,
  assignment_reminder_hours INTEGER DEFAULT 24,
  agenda_notification_enabled BOOLEAN DEFAULT TRUE,
  quiet_hours_enabled BOOLEAN DEFAULT FALSE,
  quiet_hours_start TIME DEFAULT '22:00',
  quiet_hours_end TIME DEFAULT '07:00'
)

-- Notification log: audit trail
notification_log (
  id UUID PRIMARY KEY,
  stake_id UUID,
  user_id UUID,
  title TEXT,
  body TEXT,
  status TEXT,                   -- 'sent', 'failed', 'delivered'
  error_message TEXT,
  sent_at TIMESTAMPTZ
)
```

### Database Function

```sql
-- Get users by meeting type for targeted notifications
get_meeting_audience(p_stake_id UUID, p_meeting_type TEXT)
-- Returns: user_id, user_name, calling, push_token
-- Filters by:
--   presidency: SP, SP1, SP2, Exec Sec
--   high_council: PO + HC#1-12
--   stake_council: PO + HC + SC (auxiliary presidencies, clerks)
```

---

## API Layer (`src/api/notifications.api.ts`)

### Key Functions

#### Audience Targeting

```typescript
// Get users who should receive notifications for a meeting type
getMeetingAudience(stakeId: string, meetingType: MeetingType): Promise<MeetingAudience[]>

// Get users with emails for sending agenda emails
getMeetingAudienceWithEmails(stakeId: string, meetingType: MeetingType): Promise<MeetingAudience[]>
```

#### Push Notifications

```typescript
// Send push notifications via Expo Push API
sendPushNotifications(
  stakeId: string,
  tokens: string[],
  title: string,
  body: string,
  data?: Record<string, unknown>
): Promise<{ success: number; failed: number }>

// Send meeting reminder to all attendees
sendMeetingReminder(
  stakeId: string,
  meetingType: MeetingType,
  meetingTitle: string,
  meetingDate: string,
  zoomLink?: string
): Promise<{ sent: number; failed: number }>

// Send individual assignment reminder
sendAssignmentReminder(
  stakeId: string,
  userId: string,
  assignmentType: 'opening_prayer' | 'closing_prayer' | 'spiritual_thought' | 'training',
  meetingTitle: string,
  meetingDate: string
): Promise<boolean>
```

#### Scheduled Notifications

```typescript
// Schedule a future meeting reminder
scheduleMeetingReminder(
  stakeId: string,
  meetingType: MeetingType,
  meetingTitle: string,
  meetingDate: string,
  scheduledFor: Date,
  createdBy: string,
  zoomLink?: string
): Promise<ScheduledNotification>

// Get pending scheduled notifications
getPendingNotifications(stakeId: string): Promise<ScheduledNotification[]>

// Cancel a scheduled notification
cancelScheduledNotification(notificationId: string): Promise<void>
```

#### Auto Assignment Reminders

```typescript
// Schedule reminder when someone is assigned prayer/thought
scheduleAssignmentReminder(
  stakeId: string,
  userId: string,
  assignmentType: AssignmentType,
  meetingType: MeetingType,
  meetingDate: string,
  meetingTitle: string,
  createdBy?: string
): Promise<ScheduledNotification | null>

// Schedule reminders for all assignments in an agenda
scheduleAgendaAssignmentReminders(
  stakeId: string,
  meetingType: MeetingType,
  meetingDate: string | null,
  meetingTitle: string,
  openingPrayerUserId: string | null,
  closingPrayerUserId: string | null,
  spiritualThoughtLeaderUserId: string | null,
  createdBy?: string
): Promise<{ scheduled: number; skipped: number }>

// Clear reminders when agenda is archived
clearAgendaAssignmentReminders(
  stakeId: string,
  meetingType: MeetingType,
  meetingDate: string
): Promise<void>
```

#### Email Functionality

```typescript
// Format agenda for email (plain text)
formatAgendaForEmail(agenda: AgendaEmailContent): string

// Send agenda via email to meeting attendees
emailAgendaToAttendees(
  stakeId: string,
  meetingType: MeetingType,
  agenda: AgendaEmailContent
): Promise<{ success: boolean; recipientCount: number }>
```

#### Video Link Management

```typescript
// Update zoom/meet link for a meeting type
updateMeetingVideoLink(
  stakeId: string,
  meetingType: MeetingType,
  link: string
): Promise<void>
```

#### User Preferences

```typescript
// Get user's notification preferences
getNotificationPreferences(userId: string): Promise<NotificationPreferences | null>

// Update user's notification preferences
updateNotificationPreferences(
  userId: string,
  preferences: Partial<NotificationPreferences>
): Promise<NotificationPreferences>
```

---

## UI Components

### Home Screen Meeting Actions (`app/(app)/(tabs)/home.tsx`)

The upcoming meetings section includes:

1. **Join Button** - Opens video link or prompts to add one
   - If link exists: Opens in browser
   - If no link: Shows modal to add zoom/meet link

2. **Bell Icon** - Opens action sheet with options:
   - **Send Now**: Immediately sends push notification to meeting audience
   - **Schedule**: Schedule a reminder for later (future enhancement)
   - **Email Agenda**: Opens email composer with formatted agenda

### Agenda Screen Integration (`app/(app)/meetings/agenda/[type].tsx`)

When assignments are saved in the agenda (opening prayer, closing prayer, spiritual thought), the system automatically:

1. Checks if the user has assignment reminders enabled
2. Calculates reminder time (default: 24 hours before meeting)
3. Cancels any existing reminder for that assignment
4. Creates a new scheduled notification

### Notification Preferences Screen (`app/(app)/notifications/preferences.tsx`)

Accessible from More tab → Personalize → Notifications

Settings available:
- **Day-Before Reminder**: Toggle 24-hour meeting reminders
- **Day-Of Reminder**: Toggle 1-hour before meeting reminders
- **Prayer/Thought Assignments**: Toggle assignment reminders
- **Agenda Sent Notification**: Toggle notifications when agendas are shared

---

## Meeting Audience Targeting

| Meeting Type | Who Receives Notifications |
|--------------|---------------------------|
| **Presidency** | SP, SP1, SP2, Exec Sec |
| **High Council** | PO + HC #1-12 |
| **Stake Council** | PO + HC + RS Pres, YW Pres, Primary Pres, YM Pres, SS Pres (and counselors), Stake Clerk, Asst Stake Clerk |

---

## Expo Push Notification Flow

1. **Token Registration** (TODO: implement on app startup)
   - Get push token using `Notifications.getExpoPushTokenAsync()`
   - Store in `push_tokens` table

2. **Sending Notifications**
   - Collect tokens for target audience
   - Filter valid Expo tokens (must start with `ExponentPushToken`)
   - POST to `https://exp.host/--/api/v2/push/send`
   - Handle responses and log results

3. **Scheduled Notifications** (requires backend worker)
   - Store in `scheduled_notifications` with `status='pending'`
   - Backend cron job queries pending notifications where `scheduled_for <= NOW()`
   - Sends notifications and updates `status='sent'`

---

## Email Agenda Format

```
STAKE COUNCIL MEETING
January 28, 2024

Join: https://zoom.us/j/123456789

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

OPENING
• Hymn: #123 - Come, Follow Me
• Prayer: John Smith

SPIRITUAL THOUGHT
• Topic: The Power of Ministering
• Led by: Jane Doe

AGENDA
[Agenda content here]

CLOSING
• Prayer: Bob Johnson

PARKING LOT
1. Item 1
2. Item 2

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Sent from Stake Admin App
```

---

## Future Enhancements

1. **Push Token Registration**: Implement on app startup
2. **Backend Worker**: Cron job to process scheduled notifications
3. **Quiet Hours**: Respect user's do-not-disturb preferences
4. **Notification Center**: In-app notification inbox (see `NOTIFICATION_PLAN.md`)
5. **Read Receipts**: Track when notifications are opened
6. **Custom Notifications**: Admin-authored messages to specific audiences

---

## Files Reference

| File | Purpose |
|------|---------|
| `supabase/migrations/058_push_notifications.sql` | Database schema |
| `src/api/notifications.api.ts` | API functions |
| `app/(app)/(tabs)/home.tsx` | Home screen with bell icon and join button |
| `app/(app)/meetings/agenda/[type].tsx` | Agenda with auto assignment reminders |
| `app/(app)/notifications/preferences.tsx` | User preferences screen |
| `app/(app)/notifications/_layout.tsx` | Navigation layout |
