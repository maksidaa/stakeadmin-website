# Stake Admin App - Notification System Plan

## Overview

This notification system is designed for efficient administrative collaboration. The core principle is **actionable inline content** - users can see and act on items directly within the Notification Center without navigating to separate screens.

---

## Part 1: In-App Notification Center

### Design Philosophy

The Notification Center is NOT a list of links. It's an **action hub** where users:
1. See grouped notifications by category
2. Tap a category to expand and see the **actual cards** (same UI as tracker)
3. Take actions directly (vote, assign, mark complete, etc.)
4. Expand individual cards for more detail
5. All within a centered modal/overlay - no screen navigation

### Visual Layout

```
┌─────────────────────────────────────────────────────────────────┐
│  ╔═══════════════════════════════════════════════════════════╗  │
│  ║                   Notification Center              [X]    ║  │
│  ╠═══════════════════════════════════════════════════════════╣  │
│  ║                                                           ║  │
│  ║  ┌─────────────────────────────────────────────────────┐  ║  │
│  ║  │ 🔴 SP Initial Review                        (3) ▼   │  ║  │
│  ║  └─────────────────────────────────────────────────────┘  ║  │
│  ║                                                           ║  │
│  ║  ┌─────────────────────────────────────────────────────┐  ║  │
│  ║  │ 🟡 SP Sustaining                           (2) ▼   │  ║  │
│  ║  │ ┌───────────────────────────────────────────────┐   │  ║  │
│  ║  │ │ John Smith                        [pencil]    │   │  ║  │
│  ║  │ │ Maple Ward                                    │   │  ║  │
│  ║  │ │ ──────────────────────────────────────────    │   │  ║  │
│  ║  │ │ [+] Calling: Sunday School President          │   │  ║  │
│  ║  │ │ ──────────────────────────────────────────    │   │  ║  │
│  ║  │ │ SP Votes: 1/3         [expand ▼] [Sustain]    │   │  ║  │
│  ║  │ └───────────────────────────────────────────────┘   │  ║  │
│  ║  │ ┌───────────────────────────────────────────────┐   │  ║  │
│  ║  │ │ Jane Doe                          [pencil]    │   │  ║  │
│  ║  │ │ Oak Branch                                    │   │  ║  │
│  ║  │ │ ──────────────────────────────────────────    │   │  ║  │
│  ║  │ │ [-] Release: Primary Teacher                  │   │  ║  │
│  ║  │ │ ──────────────────────────────────────────    │   │  ║  │
│  ║  │ │ SP Votes: 2/3         [expand ▼] [Sustain]    │   │  ║  │
│  ║  │ └───────────────────────────────────────────────┘   │  ║  │
│  ║  └─────────────────────────────────────────────────────┘  ║  │
│  ║                                                           ║  │
│  ║  ┌─────────────────────────────────────────────────────┐  ║  │
│  ║  │ 🟢 Action Items Due                        (5) ▼   │  ║  │
│  ║  └─────────────────────────────────────────────────────┘  ║  │
│  ║                                                           ║  │
│  ║  ┌─────────────────────────────────────────────────────┐  ║  │
│  ║  │ 📅 Meeting Tomorrow                        (1) ▼   │  ║  │
│  ║  └─────────────────────────────────────────────────────┘  ║  │
│  ║                                                           ║  │
│  ╚═══════════════════════════════════════════════════════════╝  │
└─────────────────────────────────────────────────────────────────┘
```

### Interaction Flow

1. **User taps bell icon** → Notification Center modal opens (centered overlay)
2. **User sees categories** → Each category shows count badge
3. **User taps category header** → Category expands to show actual cards
4. **Cards are identical to tracker** → Same components, same styling, same actions
5. **User taps action button** → Action executes inline (with loading state)
6. **Item resolved** → Card animates away, count decrements
7. **User taps [X] or outside** → Modal closes

### Key Principle: Reuse Existing Card Components

The notification center does NOT create new card designs. It imports and renders the **exact same card components** used in:
- `app/(app)/tracker/list/sp-initial-review.tsx`
- `app/(app)/tracker/list/sp-sustaining.tsx`
- `app/(app)/tracker/list/hc-review.tsx`
- `app/(app)/tracker/list/ready-for-assignment.tsx`
- etc.

This ensures:
- Consistent UI
- No duplicate code
- Users recognize familiar patterns
- Actions work identically

---

## Part 2: Notification Categories by Role

### Stake President (SP)

| Category | Badge Shows | Cards Displayed | Available Actions |
|----------|-------------|-----------------|-------------------|
| **New Submissions** | Count in `sp_initial_review` | SP Initial Review cards | Configure, assign HC, add notes, move to sustaining |
| **SP Sustaining** | Count awaiting SP vote | SP Sustaining cards | Vote to sustain, view who's voted |
| **Declined Callings** | Count in `declined` status | Declined cards | Review, resubmit, archive |
| **User Approvals** | Count with `is_approved=false` | User approval cards | Approve, assign calling, reject |
| **Stale Items** | Count of items 3+ days in review | Stale review cards | Same as initial review |

### SP Counselors (SP1, SP2)

| Category | Badge Shows | Cards Displayed | Available Actions |
|----------|-------------|-----------------|-------------------|
| **SP Sustaining** | Count awaiting their vote | SP Sustaining cards | Vote to sustain |
| **Declined Callings** | Count in `declined` status | Declined cards (read-only) | View only |

### Executive Secretary

Same as SP, plus:
| Category | Badge Shows | Cards Displayed | Available Actions |
|----------|-------------|-----------------|-------------------|
| **Meeting Prep** | Meetings within 48 hours | Meeting summary cards | View agenda, see assignments |

### High Council Members

| Category | Badge Shows | Cards Displayed | Available Actions |
|----------|-------------|-----------------|-------------------|
| **HC Sustaining** | Count in `hc_review` | HC Review cards | Vote to sustain, view vote count |
| **My Assignments** | Count assigned to them in `assigned` | Assignment cards | Mark extended, record response |
| **Ready to Extend** | Count in `ready_for_assignment` assigned to them | Ready cards | Begin extension process |
| **Speaking Assignments** | Upcoming within 7 days | Speaking cards | View details |
| **Meeting Reminders** | Meetings within 24 hours | Meeting cards | View agenda, Zoom link |

### Stake Council Members

| Category | Badge Shows | Cards Displayed | Available Actions |
|----------|-------------|-----------------|-------------------|
| **Action Items** | Open items assigned to them | Action item cards | Mark complete, add notes |
| **Speaking Assignments** | Upcoming within 7 days | Speaking cards | View details |
| **Meeting Reminders** | Meetings within 24 hours | Meeting cards | View agenda |

### Stake Clerk

| Category | Badge Shows | Cards Displayed | Available Actions |
|----------|-------------|-----------------|-------------------|
| **Ready for Recording** | Count in `to_be_recorded` | Recording cards | Mark recorded in LCR |
| **MP Ordinations to Record** | Ordained but not recorded | Ordination cards | Mark recorded |

### Unit Leaders (Bishops/Branch Presidents)

| Category | Badge Shows | Cards Displayed | Available Actions |
|----------|-------------|-----------------|-------------------|
| **Ward Sustainings Needed** | Count needing their unit's sustaining | Sustaining cards | Record sustained/opposed |
| **My Submissions** | Their submissions with updates | Status cards (read-only) | View progress |

---

## Part 3: Card Components to Expose

These existing card patterns from the tracker need to be extracted into reusable components:

### 1. CallingSubmissionCard (base component)

```typescript
interface CallingSubmissionCardProps {
  submission: CallingSubmission;
  variant: 'sp-review' | 'sp-sustaining' | 'hc-review' | 'assigned' | 'recording';
  onAction?: (action: string, data?: any) => void;
  expandable?: boolean;
  showActions?: boolean;
}
```

**Used in**: SP Initial Review, SP Sustaining, HC Review, Ready for Assignment, Assigned, To Be Recorded

### 2. ActionItemCard

```typescript
interface ActionItemCardProps {
  item: ActionItem;
  onComplete: () => void;
  onEdit: () => void;
  expandable?: boolean;
}
```

**Used in**: Action items tab, Notification Center

### 3. MeetingReminderCard

```typescript
interface MeetingReminderCardProps {
  meeting: Meeting;
  userAssignments?: ('prayer' | 'discussion' | 'experience')[];
  showZoomLink?: boolean;
}
```

**Used in**: Meeting reminders, Notification Center

### 4. SpeakingAssignmentCard

```typescript
interface SpeakingAssignmentCardProps {
  assignment: SpeakingAssignment;
  showDate: boolean;
  showUnit: boolean;
}
```

**Used in**: Speaking assignments, Notification Center

### 5. UserApprovalCard

```typescript
interface UserApprovalCardProps {
  user: PendingUser;
  callings: Calling[];
  units: Unit[];
  onApprove: (callingId: string, unitId: string) => void;
  onReject: () => void;
}
```

**Used in**: User management, Notification Center

### 6. UnitSustainingCard

```typescript
interface UnitSustainingCardProps {
  submission: CallingSubmission;
  unit: Unit;
  onSustain: (sustained: boolean) => void;
}
```

**Used in**: Unit sustainings, Notification Center for bishops

---

## Part 4: Push Notification Triggers

Push notifications alert users to open the app. The in-app Notification Center then shows the details.

### When Push Notifications Are Sent

#### Immediate (High Priority)

| Trigger | Recipients | Push Message |
|---------|------------|--------------|
| Calling declined | SP, assigned HC | "Calling declined: [Person] declined [Calling]" |
| New user pending | SP, Exec Sec | "New user awaiting approval: [Name]" |
| Meeting in 1 hour | Attendees | "[Meeting Type] meeting in 1 hour" |

#### Immediate (Normal Priority)

| Trigger | Recipients | Push Message |
|---------|------------|--------------|
| New submission | SP, Exec Sec | "New submission: [Person] for [Calling]" |
| Assigned to you | HC member | "New assignment: Extend [Calling] to [Person]" |
| Your vote needed | SP/HC member | "[X] items need your sustaining vote" |
| Ward sustaining needed | Bishop | "Sustaining needed: [Person] as [Calling]" |
| Speaking reminder (1 day) | Speaker | "Tomorrow: Speaking at [Unit]" |
| Action item assigned | Assignee | "New action item: [Title]" |

#### Batched (Every 15 minutes max)

| Trigger | Recipients | Push Message |
|---------|------------|--------------|
| Multiple items ready for HC | All HC | "[X] items ready for HC sustaining" |
| Multiple items ready for recording | Clerk | "[X] items ready for LCR recording" |

#### Daily Digest (Morning)

| Trigger | Recipients | Push Message |
|---------|------------|--------------|
| Stale items reminder | SP, Exec Sec | "[X] items need attention (3+ days old)" |
| Overdue action items | Assignees | "[X] action items overdue" |
| Items stuck in assigned | SP (escalation) | "[X] callings not extended after 7+ days" |

#### Scheduled Reminders

| Trigger | Recipients | Push Message |
|---------|------------|--------------|
| Meeting 24 hours | Attendees | "[Meeting Type] meeting tomorrow" |
| Speaking 7 days | Speaker | "Speaking assignment in 1 week: [Unit]" |
| Action item due tomorrow | Assignee | "Due tomorrow: [Title]" |

### Push Notification → App Deep Link

When user taps a push notification:
1. App opens
2. Notification Center modal opens automatically
3. Relevant category is pre-expanded
4. User sees the actual cards and can take action

---

## Part 5: Calling Tracker Status Notifications

### Full Workflow Notification Map

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CALLING TRACKER WORKFLOW                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────┐                                               │
│  │  SUBMISSION      │  → PUSH: SP + Exec Sec                        │
│  │  CREATED         │    "New submission: [Person] for [Calling]"   │
│  └────────┬─────────┘                                               │
│           ▼                                                         │
│  ┌──────────────────┐                                               │
│  │  SP INITIAL      │  → IN-APP: "New Submissions" category         │
│  │  REVIEW          │    Full card with configure/assign actions    │
│  │                  │                                               │
│  │  If stale (3d)   │  → PUSH: SP + Exec Sec (daily digest)         │
│  │                  │    "[X] items awaiting initial review"        │
│  └────────┬─────────┘                                               │
│           ▼                                                         │
│  ┌──────────────────┐                                               │
│  │  SP SUSTAINING   │  → PUSH: SP, SP1, SP2                         │
│  │                  │    "Your vote needed: [Person] as [Calling]"  │
│  │                  │                                               │
│  │                  │  → IN-APP: "SP Sustaining" category           │
│  │                  │    Card with vote button + vote count         │
│  │                  │                                               │
│  │  When all 3 vote │  → (No notification - auto-advances)          │
│  └────────┬─────────┘                                               │
│           ▼                                                         │
│  ┌──────────────────┐                                               │
│  │  HC REVIEW       │  → PUSH: All HC (batched if multiple)         │
│  │                  │    "[X] items ready for HC sustaining"        │
│  │                  │                                               │
│  │                  │  → IN-APP: "HC Sustaining" category           │
│  │                  │    Card with vote button + vote count (X/12)  │
│  │                  │                                               │
│  │  If stale (5d)   │  → PUSH: HC who haven't voted                 │
│  │                  │    "Awaiting your vote on [X] items"          │
│  └────────┬─────────┘                                               │
│           ▼                                                         │
│  ┌──────────────────┐                                               │
│  │  READY FOR       │  → PUSH: Assigned HC member                   │
│  │  ASSIGNMENT      │    "Ready to extend: [Calling] to [Person]"   │
│  │                  │                                               │
│  │                  │  → IN-APP: "Ready to Extend" category         │
│  │                  │    Card with assignment info                   │
│  └────────┬─────────┘                                               │
│           ▼                                                         │
│  ┌──────────────────┐                                               │
│  │  ASSIGNED        │  → PUSH: Assigned HC member                   │
│  │                  │    "Assigned: Extend [Calling] to [Person]"   │
│  │                  │                                               │
│  │                  │  → IN-APP: "My Assignments" category          │
│  │                  │    Card with extend/response actions          │
│  │                  │                                               │
│  │  If stale (7d)   │  → PUSH: Assigned HC                          │
│  │                  │    "Reminder: Extend calling to [Person]"     │
│  │                  │                                               │
│  │  If stale (14d)  │  → PUSH: SP (escalation)                      │
│  │                  │    "Escalation: [Person] not extended (14d)"  │
│  └────────┬─────────┘                                               │
│           │                                                         │
│           ├──────────────────────────────────────────────────────┐  │
│           │ If DECLINED                                          │  │
│           │                                                      ▼  │
│  ┌────────┴─────────┐                              ┌─────────────┐  │
│  │  CALLING         │                              │  DECLINED   │  │
│  │  EXTENDED        │  → No push (info only)       │             │  │
│  │                  │                              │  → PUSH:    │  │
│  │  Awaiting        │                              │    SP, HC   │  │
│  │  response...     │                              │    "DECLINED│  │
│  └────────┬─────────┘                              │    [Person] │  │
│           │                                        │    declined │  │
│           │ If ACCEPTED                            │    [Calling]│  │
│           ▼                                        └─────────────┘  │
│  ┌──────────────────┐                                               │
│  │  PRE-SUSTAINING  │  → No push (batch with ward sustaining)      │
│  │  BUCKET          │                                               │
│  └────────┬─────────┘                                               │
│           ▼                                                         │
│  ┌──────────────────┐                                               │
│  │  SENT TO UNIT    │  → PUSH: Bishop/Branch President              │
│  │  FOR SUSTAINING  │    "Sustaining needed: [Person] as [Calling]" │
│  │                  │                                               │
│  │                  │  → IN-APP: "Ward Sustainings Needed"          │
│  │                  │    Card with sustained/opposed buttons        │
│  └────────┬─────────┘                                               │
│           ▼                                                         │
│  ┌──────────────────┐                                               │
│  │  TO BE SET       │  → PUSH: Designated setter-aparter            │
│  │  APART           │    "Ready for set apart: [Person]"            │
│  │                  │                                               │
│  │                  │  → IN-APP: Card with set apart action         │
│  └────────┬─────────┘                                               │
│           ▼                                                         │
│  ┌──────────────────┐                                               │
│  │  TO BE           │  → PUSH: Stake Clerk (batched)                │
│  │  RECORDED        │    "[X] items ready for LCR recording"        │
│  │                  │                                               │
│  │                  │  → IN-APP: "Ready for Recording" category     │
│  │                  │    Card with recording checkboxes             │
│  └────────┬─────────┘                                               │
│           ▼                                                         │
│  ┌──────────────────┐                                               │
│  │  COMPLETE        │  → PUSH: Original submitter                   │
│  │                  │    "Complete: [Person] now [Calling]"         │
│  │                  │                                               │
│  │                  │  → IN-APP: No notification (archived)         │
│  └──────────────────┘                                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Part 6: Action Items Notifications

### Action Item Lifecycle

```
┌──────────────────┐
│  CREATED         │  → PUSH: Assignee(s)
│                  │    "New action item: [Title]"
│                  │
│                  │  → IN-APP: "Action Items" category
│                  │    Full card with complete action
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  EDITED          │  → PUSH: Assignee(s) (if due date changed)
│                  │    "Action item updated: [Title] - now due [Date]"
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  DUE TOMORROW    │  → PUSH: Assignee(s) (scheduled)
│                  │    "Due tomorrow: [Title]"
└────────┬─────────┘
         │
         ├─────────────────────────────────────────┐
         │                                         │
         ▼                                         ▼
┌──────────────────┐                  ┌──────────────────┐
│  COMPLETED       │                  │  OVERDUE         │
│                  │                  │                  │
│  → PUSH: Creator │                  │  Day 1: Assignee │
│    (if different │                  │    "Overdue: [T]"│
│    from completer│                  │                  │
│    "[User]       │                  │  Day 3: Assignee │
│    completed:    │                  │    + Creator     │
│    [Title]"      │                  │    "3 days over" │
│                  │                  │                  │
│  → IN-APP:       │                  │  Day 7: SP       │
│    Removed from  │                  │    (escalation)  │
│    list          │                  │                  │
└──────────────────┘                  └──────────────────┘
```

---

## Part 7: Meeting & Speaking Notifications

### Meetings

```
┌──────────────────────────────────────────────────────────────────┐
│                         MEETING NOTIFICATIONS                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  MEETING CREATED/UPDATED                                         │
│  ─────────────────────                                           │
│  If user has assignment (prayer, discussion, experience):        │
│    → PUSH: "You've been assigned [type] for [Meeting] on [Date]" │
│    → IN-APP: Card shows their specific assignment highlighted    │
│                                                                   │
│  MEETING IN 24 HOURS                                             │
│  ──────────────────                                              │
│    → PUSH: All expected attendees                                │
│      "[Meeting Type] meeting tomorrow at [Time]"                 │
│      Include: agenda item count, Zoom link if virtual            │
│    → IN-APP: "Meeting Tomorrow" category                         │
│      Card shows agenda summary, prayer assignments, Zoom link    │
│                                                                   │
│  MEETING IN 1 HOUR                                               │
│  ─────────────────                                               │
│    → PUSH: All expected attendees (HIGH PRIORITY)                │
│      "[Meeting Type] meeting in 1 hour"                          │
│    → IN-APP: Same card, Zoom link prominent                      │
│                                                                   │
│  ATTENDEES BY MEETING TYPE                                       │
│  ─────────────────────────                                       │
│  • high_council: PO + All HC (1-12)                              │
│  • stake_council: PO + HC + All SC members                       │
│  • presidency: SP, SP1, SP2, Executive Secretary                 │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### Speaking Assignments

```
┌──────────────────────────────────────────────────────────────────┐
│                    SPEAKING ASSIGNMENT NOTIFICATIONS              │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ASSIGNMENT CREATED                                              │
│  ─────────────────                                               │
│    → PUSH: Speaker                                               │
│      "Speaking assignment: [Unit] on [Date]. Topic: [Topic]"     │
│    → IN-APP: "Speaking Assignments" category                     │
│      Card shows date, unit, topic, sacrament time                │
│                                                                   │
│  7 DAYS BEFORE                                                   │
│  ────────────                                                    │
│    → PUSH: Speaker                                               │
│      "Reminder: Speaking at [Unit] next [Day]. Topic: [Topic]"   │
│    → IN-APP: Card moves to prominent position                    │
│                                                                   │
│  1 DAY BEFORE                                                    │
│  ───────────                                                     │
│    → PUSH: Speaker (HIGH PRIORITY)                               │
│      "Tomorrow: Speaking at [Unit]. Sacrament at [Time]"         │
│    → IN-APP: Card highlighted with urgency styling               │
│                                                                   │
│  ASSIGNMENT CHANGED                                              │
│  ─────────────────                                               │
│    → PUSH: Previous speaker                                      │
│      "Your speaking assignment on [Date] has been reassigned"    │
│    → PUSH: New speaker                                           │
│      "Speaking assignment: [Unit] on [Date]. Topic: [Topic]"     │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## Part 8: User Management Notifications

```
┌──────────────────────────────────────────────────────────────────┐
│                    USER MANAGEMENT NOTIFICATIONS                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  NEW USER REGISTERS                                              │
│  ─────────────────                                               │
│    → PUSH: SP + Executive Secretary                              │
│      "New user awaiting approval: [Name] ([Email])"              │
│    → IN-APP: "User Approvals" category (SP only)                 │
│      Card with user info, calling dropdown, unit dropdown        │
│      [Approve] and [Reject] buttons inline                       │
│                                                                   │
│  USER APPROVED                                                   │
│  ─────────────                                                   │
│    → PUSH: The approved user                                     │
│      "Welcome! Your account has been approved."                  │
│    → IN-APP: User sees full app (no notification category)       │
│                                                                   │
│  USER CALLING CHANGED                                            │
│  ────────────────────                                            │
│    → PUSH: The user                                              │
│      "Your calling has been updated to [New Calling]"            │
│    → IN-APP: No notification category (informational only)       │
│                                                                   │
│  USER REJECTED                                                   │
│  ─────────────                                                   │
│    → No push (user is removed)                                   │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## Part 9: MP Ordination Notifications

```
┌──────────────────────────────────────────────────────────────────┐
│                    MP ORDINATION NOTIFICATIONS                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  SUBMISSION CREATED                                              │
│    → PUSH: SP + Exec Sec                                         │
│      "New ordination request: [Person] to [Office]"              │
│    → IN-APP: Included in "New Submissions" category              │
│                                                                   │
│  INTERVIEW ASSIGNED                                              │
│    → PUSH: Assigned SP member                                    │
│      "Interview assigned: [Person] for [Office] ordination"      │
│    → IN-APP: "My Assignments" category                           │
│                                                                   │
│  READY FOR HC SUSTAINING                                         │
│    → PUSH: All HC (batched with calling submissions)             │
│      (Included in "[X] items ready for HC sustaining" count)     │
│    → IN-APP: "HC Sustaining" category (mixed with callings)      │
│                                                                   │
│  READY FOR ORDINATION                                            │
│    → PUSH: SP + assigned overseer                                │
│      "[Person] sustained - ready for ordination"                 │
│    → IN-APP: "Ordinations Ready" category                        │
│                                                                   │
│  ORDAINED - READY FOR RECORDING                                  │
│    → PUSH: Stake Clerk                                           │
│      "Record ordination: [Person] ordained [Office]"             │
│    → IN-APP: "Ready for Recording" category                      │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## Part 10: Notification Center UI Components

### NotificationCenter.tsx (Main Modal)

```typescript
interface NotificationCenterProps {
  visible: boolean;
  onClose: () => void;
}

// Categories rendered based on user role
const getCategories = (user: User, permissions: Permissions) => {
  const categories: NotificationCategory[] = [];

  if (permissions.isPO()) {
    categories.push({
      id: 'sp-initial-review',
      title: 'New Submissions',
      icon: 'document-text',
      color: colors.error,
      fetchData: () => getSubmissionsByStatus('sp_initial_review'),
      CardComponent: SPInitialReviewCard,
    });
    categories.push({
      id: 'user-approvals',
      title: 'User Approvals',
      icon: 'person-add',
      color: colors.warning,
      fetchData: () => getPendingApprovalUsers(),
      CardComponent: UserApprovalCard,
    });
  }

  if (permissions.canSustainAsSP()) {
    categories.push({
      id: 'sp-sustaining',
      title: 'SP Sustaining',
      icon: 'hand-left',
      color: colors.warning,
      fetchData: () => getSubmissionsNeedingSPVote(user.id),
      CardComponent: SPSustainingCard,
    });
  }

  if (permissions.isHighCouncil()) {
    categories.push({
      id: 'hc-sustaining',
      title: 'HC Sustaining',
      icon: 'hand-left',
      color: colors.primary,
      fetchData: () => getSubmissionsNeedingHCVote(user.hc_number),
      CardComponent: HCSustainingCard,
    });
    categories.push({
      id: 'my-assignments',
      title: 'My Assignments',
      icon: 'briefcase',
      color: colors.success,
      fetchData: () => getAssignedSubmissions(user.id),
      CardComponent: AssignmentCard,
    });
  }

  // ... more categories per role

  return categories;
};
```

### NotificationCategoryRow.tsx

```typescript
interface NotificationCategoryRowProps {
  category: NotificationCategory;
  expanded: boolean;
  onToggle: () => void;
  onItemAction: (itemId: string, action: string) => void;
}

// Renders:
// - Collapsed: Icon + Title + Badge count + Chevron
// - Expanded: Same header + List of actual cards below
```

### Badge in Header

```typescript
// In app header or tab bar
<TouchableOpacity onPress={openNotificationCenter}>
  <Ionicons name="notifications" size={24} />
  {unreadCount > 0 && (
    <View style={styles.badge}>
      <Text style={styles.badgeText}>{unreadCount}</Text>
    </View>
  )}
</TouchableOpacity>
```

---

## Part 11: Database Schema

### notifications table

```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stake_id UUID NOT NULL REFERENCES stakes(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,

  -- Content
  type VARCHAR(50) NOT NULL,
  title VARCHAR(255) NOT NULL,
  body TEXT NOT NULL,
  priority VARCHAR(20) DEFAULT 'normal', -- 'low', 'normal', 'high'

  -- Entity linking (for in-app display)
  entity_type VARCHAR(50), -- 'calling_submission', 'action_item', 'meeting', 'user', 'speaking_assignment'
  entity_id UUID,

  -- Status tracking
  push_sent_at TIMESTAMPTZ,
  read_at TIMESTAMPTZ,
  actioned_at TIMESTAMPTZ, -- When user took action on the item

  -- Batching
  batch_key VARCHAR(100), -- Group related notifications
  batch_count INTEGER DEFAULT 1, -- How many items in batch

  created_at TIMESTAMPTZ DEFAULT NOW(),

  CONSTRAINT valid_priority CHECK (priority IN ('low', 'normal', 'high')),
  CONSTRAINT valid_entity_type CHECK (entity_type IN (
    'calling_submission', 'action_item', 'meeting', 'user',
    'speaking_assignment', 'mp_ordination', 'unit_sustaining'
  ))
);

CREATE INDEX idx_notifications_user_unread ON notifications(user_id, read_at) WHERE read_at IS NULL;
CREATE INDEX idx_notifications_user_recent ON notifications(user_id, created_at DESC);
CREATE INDEX idx_notifications_stake ON notifications(stake_id);
CREATE INDEX idx_notifications_entity ON notifications(entity_type, entity_id);
```

### notification_preferences table

```sql
CREATE TABLE notification_preferences (
  user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,

  -- Master toggle
  push_enabled BOOLEAN DEFAULT true,

  -- Category toggles
  calling_tracker BOOLEAN DEFAULT true,
  action_items BOOLEAN DEFAULT true,
  meetings BOOLEAN DEFAULT true,
  speaking_assignments BOOLEAN DEFAULT true,

  -- Timing
  quiet_hours_enabled BOOLEAN DEFAULT false,
  quiet_hours_start TIME DEFAULT '22:00',
  quiet_hours_end TIME DEFAULT '07:00',

  -- Reminder preferences
  meeting_reminder_hours INTEGER DEFAULT 24,
  speaking_reminder_days INTEGER DEFAULT 7,

  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## Part 12: Implementation Checklist

### Phase 1: Notification Center UI
- [ ] Create `NotificationCenter.tsx` modal component
- [ ] Create `NotificationCategoryRow.tsx` expandable component
- [ ] Add bell icon with badge to app header
- [ ] Extract card components from tracker lists into reusable components
- [ ] Wire up category data fetching per role

### Phase 2: Push Notification Infrastructure
- [ ] Create `notifications` and `notification_preferences` tables
- [ ] Set up Expo push notification handlers
- [ ] Create notification creation API (`createNotification()`)
- [ ] Create push sending service (Supabase Edge Function or external)

### Phase 3: Calling Tracker Triggers
- [ ] Trigger on new submission
- [ ] Trigger on status change to `sp_sustaining`
- [ ] Trigger on status change to `hc_review`
- [ ] Trigger on assignment
- [ ] Trigger on declined
- [ ] Trigger on `sent_to_unit_for_sustaining`
- [ ] Trigger on `to_be_recorded`
- [ ] Trigger on complete

### Phase 4: Action Item Triggers
- [ ] Trigger on new action item
- [ ] Trigger on edit (due date change)
- [ ] Scheduled: due tomorrow reminder
- [ ] Scheduled: overdue escalation

### Phase 5: Meeting & Speaking Triggers
- [ ] Trigger on meeting assignment
- [ ] Scheduled: 24-hour meeting reminder
- [ ] Scheduled: 1-hour meeting reminder
- [ ] Trigger on speaking assignment
- [ ] Scheduled: 7-day speaking reminder
- [ ] Scheduled: 1-day speaking reminder

### Phase 6: User Management Triggers
- [ ] Trigger on new user registration
- [ ] Trigger on user approval

### Phase 7: Preferences & Polish
- [ ] Create notification settings screen
- [ ] Implement quiet hours
- [ ] Implement category toggles
- [ ] Add "Mark all as read"
- [ ] Add notification history view

---

## Summary

This notification system is designed around one core principle: **users can act without leaving the Notification Center**. By reusing the same card components from the tracker, we maintain UI consistency while creating an efficient action hub.

The role-based filtering ensures each user only sees what's relevant to their responsibilities, and the push + in-app combination ensures they're alerted at the right time and can take action immediately.
