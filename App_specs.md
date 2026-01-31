# Stake Admin App — Complete Specification

## Document Purpose

This specification provides everything Claude Code needs to build a multi-tenant stake administration app for The Church of Jesus Christ of Latter-day Saints. The app replaces an existing Glide implementation with a native mobile app (iOS/Android) to enable push notifications and a native user experience.

-----

# Part 1: Architecture & Data Model

## 1.1 App Philosophy

**Mission:** “Automate the admin so leaders can minister.”

**UX North Star:** “As easy as Facebook.” Leaders shouldn’t need training — they open the app, see what’s relevant to them, tap things, stuff happens.

**Core Principle:** The app is a workflow engine with a resource layer. Notifications emerge naturally from “notify the next person when the previous step is done.” Permissions are just roles in the workflow.

## 1.2 Multi-Tenancy

- Each stake is a “workspace” — completely isolated data
- Users authenticate via Google or Apple sign-in
- New users land in a review queue; admins approve and assign roles
- Users belong to exactly one stake (edge case: could belong to multiple, but rare)
- Any stake can create their own workspace and invite members

## 1.3 Two-Tier Access Model

The app has two main sections with different access levels:

|Section          |Access                                            |Features                                                                                       |
|-----------------|--------------------------------------------------|-----------------------------------------------------------------------------------------------|
|**Directory**    |All unit leaders (Bishops, EQ Pres, RS Pres, etc.)|Unit info, meeting times, directions, resources, MP ordination submission, speaking assignments|
|**Stake Council**|SP, HC, Stake Council members only                |Calling tracker, action items, agendas, full speaking assignment management                    |

## 1.4 Core Data Model

### Stakes Table

```
stakes
├── id (uuid, primary key)
├── name (string) — "Kyle Texas Stake"
├── logo_url (string, optional) — stake logo image
├── vision_statement (text) — displayed on home screen and agendas
├── zoom_link_hc (string, optional) — High Council meeting Zoom
├── zoom_link_sc (string, optional) — Stake Council meeting Zoom
├── created_at (timestamp)
└── updated_at (timestamp)
```

### Units Table

```
units
├── id (uuid, primary key)
├── stake_id (foreign key → stakes)
├── unit_number (integer) — 1-9 (or up to 20)
├── name (string) — "Bastrop Ward", "San Marcos YSA Branch"
├── unit_type (enum) — "ward" | "branch"
├── building_photo_url (string, optional)
├── address (string)
├── sacrament_time (string) — "12:00 PM"
├── ward_council_schedule (string) — "2nd, 3rd, 4th Sunday 7:45 AM"
├── youth_activities_schedule (string) — "Wednesday at 6 PM"
├── hc_liaison_user_id (foreign key → users, optional) — which HC covers this unit
├── created_at (timestamp)
└── updated_at (timestamp)
```

### Users Table

```
users
├── id (uuid, primary key)
├── stake_id (foreign key → stakes)
├── email (string, unique per stake)
├── name (string)
├── phone (string, optional)
├── auth_provider (enum) — "google" | "apple"
├── auth_provider_id (string) — ID from auth provider
├── calling_id (foreign key → callings_list) — their ONE calling
├── unit_id (foreign key → units) — their home unit
├── is_approved (boolean) — false until admin approves
├── is_admin (boolean) — can edit stake settings
├── can_edit (boolean) — edit permissions beyond their role
├── push_token (string, optional) — for push notifications
├── created_at (timestamp)
└── updated_at (timestamp)
```

### Callings List Table (Reference Data)

```
callings_list
├── id (uuid, primary key)
├── name (string) — "Stake President", "HC#1", "Bishop", etc.
├── category (enum) — "stake_presidency" | "high_council" | "stake_council" | "unit_leadership" | "unit_auxiliary"
├── permission_level (enum) — see Permission Levels below
├── display_order (integer) — for sorting in dropdowns
├── is_stake_level (boolean) — true for stake callings, false for unit callings
└── hc_number (integer, optional) — 1-12 for HC members, null for others
```

### Permission Levels

```
permission_levels (enum):
- "admin" — Full access, can edit stake settings and users
- "stake_presidency" — SP, SP1, SP2: see everything, approve callings
- "executive_secretary" — Exec Sec, Asst Exec Secs: manage workflow, record in LCR
- "high_council" — HC#1-12: see after SP approval, extend callings, conduct business
- "stake_council" — Auxiliary presidents: attend SC meetings, speaking assignments
- "stake_council_counselor" — Auxiliary counselors/secretaries: limited SC access
- "unit_leader" — Bishops, Branch Presidents: submit callings, track their unit
- "unit_auxiliary" — EQ Pres, RS Pres, etc.: Directory access only
- "clerk" — Stake Clerk, Asst Clerks: recording access
```

-----

# Part 2: Calling & Release Tracker

## 2.1 Overview

The calling tracker is the heart of the app. A calling flows through distinct stages, with visibility expanding as it progresses.

## 2.2 Submission Types

Three types of submissions flow through related but distinct workflows:

1. **Calling** — Someone being called to a new position
1. **Release** — Someone being released from a position
1. **Calling with Release** — Same person releasing from one calling, receiving another (single submission)

## 2.3 Calling/Release Submission Table

```
calling_submissions
├── id (uuid, primary key)
├── stake_id (foreign key → stakes)
├── submission_type (enum) — "calling" | "release" | "calling_with_release"
├── person_name (string) — name of person being called/released
├── unit_id (foreign key → units) — their home unit
├── 
├── # For callings:
├── new_calling_id (foreign key → callings_list, optional)
├── calling_type (enum) — "stake" | "ward" | "branch"
├── 
├── # For releases:
├── current_calling_id (foreign key → callings_list, optional)
├── 
├── # Workflow state:
├── status (enum) — see Status Flow below
├── 
├── # SP Review stage:
├── sp_reviewer_id (foreign key → users, optional) — who did initial review
├── sp_review_date (timestamp, optional)
├── sp1_sustained (boolean, default false)
├── sp2_sustained (boolean, default false)
├── sp3_sustained (boolean, default false)
├── sp_notes (text, optional) — only visible to SP
├── 
├── # HC Review stage:
├── hc_review_complete (boolean, default false)
├── hc_notes (text, optional) — visible to SP and HC
├── 
├── # Assignment:
├── assigned_to_user_id (foreign key → users, optional)
├── assigned_date (timestamp, optional)
├── 
├── # Extension:
├── calling_accepted (boolean, optional) — null until extended
├── calling_declined (boolean, optional)
├── release_extended (boolean, optional)
├── extension_date (timestamp, optional)
├── 
├── # Sustaining tracking:
├── in_pre_sustaining_bucket (boolean, default false)
├── released_to_units (boolean, default false) — SP released batch to units
├── home_unit_sustained (boolean, default false)
├── 
├── # Completion:
├── set_apart_complete (boolean, default false)
├── set_apart_by_user_id (foreign key → users, optional)
├── set_apart_date (timestamp, optional)
├── recorded_in_lcr (boolean, default false)
├── recorded_by_user_id (foreign key → users, optional)
├── recorded_date (timestamp, optional)
├── 
├── # Bishop visibility:
├── allow_bishop_to_track (boolean, default false)
├── 
├── # Metadata:
├── submitted_by_user_id (foreign key → users)
├── created_at (timestamp)
└── updated_at (timestamp)
```

## 2.4 Status Flow (enum)

```
calling_status:
1. "sp_initial_review" — New submission, waiting for SP review
2. "sp_sustaining" — SP reviewed, waiting for all 3 SP to sustain
3. "hc_review" — SP sustained, HC reviewing
4. "ready_for_assignment" — HC sustained, ready to assign who extends
5. "assigned" — Assigned to SP/HC member to extend
6. "pre_sustaining_bucket" — Extended and accepted, waiting for batch release
7. "sustaining_in_progress" — Released to units for sustaining
8. "to_be_set_apart" — Home unit sustained, ready for setting apart
9. "to_be_recorded" — Set apart complete, needs LCR recording
10. "complete" — Fully recorded, archived
11. "declined" — Person declined the calling
```

## 2.5 Unit Sustaining Tracking

```
unit_sustainings
├── id (uuid, primary key)
├── submission_id (foreign key → calling_submissions)
├── unit_id (foreign key → units)
├── sustained (boolean, default false)
├── sustained_by_user_id (foreign key → users, optional)
├── sustained_date (timestamp, optional)
└── is_home_unit (boolean) — true if this is person's home unit
```

For stake-level callings, a record is created for each unit. Setting apart can proceed after home unit sustains; other units continue in parallel.

## 2.6 Visibility Rules

|Status                               |Who Can See              |
|-------------------------------------|-------------------------|
|sp_initial_review                    |SP only                  |
|sp_sustaining                        |SP only                  |
|hc_review                            |SP + HC                  |
|ready_for_assignment → complete      |SP + HC + Assigned person|
|Any status (if allow_bishop_to_track)|+ Bishop of that unit    |

## 2.7 Workflow Screens

### Submit Calling or Release (Bishops, authorized leaders)

- Select: Calling / Release / Both
- Person name (free text)
- Home unit (dropdown)
- For calling: Calling type (stake/ward), specific calling (dropdown)
- For release: Current calling (dropdown)
- Submit → notification to SP

### Calling Tracker Home (SP view)

Buckets showing counts:

- Initial Review By Stake Presidency (X)
- To Be Sustained By Stake Presidency (X)
- To Be Reviewed By High Council (X)
- Ready For Assignment (X)
- [Per SP member] Stake President Danny German (X)
- [Per SP member] 1st Counselor David Allen (X)
- [Per SP member] 2nd Counselor Adam Goodwin (X)
- [Per HC member] HC#1 Chris Hernandez (X)
- … HC#2-12 …
- Declined Callings (X)
- Pre-Sustaining Bucket (Releases: X | Callings: X)
- To Be Sustained or Released
- To Be Set Apart (X)
- To Be Recorded in LCR (X)

### Calling Detail Screen

Shows all fields, appropriate actions based on status and user role:

- SP can: Review, send for sustaining, approve, assign
- HC can: Review, sustain, change assignment (if theirs)
- Assignee can: Mark accepted/declined/extended, add notes
- Exec Sec can: Mark set apart, mark recorded

### Assignment Screen (HC/SP member’s personal queue)

- Assigned Callings to Extend (list)
- Assigned Releases to Extend (list)
- Each item shows: Name, Calling, ⋯ menu (Change Assignment, Edit)

### Stake Callings and Releases By Unit

- List of all units + “All Stake Business” option
- Tap unit → see all pending sustainings/releases for that unit
- Grouped: Releases | Sustainings | Priesthood Sustainings | To Be Set Apart
- Pre-written scripts for conducting
- ⋯ menu to mark each as “Sustained” or “Released”

-----

# Part 3: Melchizedek Priesthood Ordinations

## 3.1 Overview

MP ordinations have a separate submission flow, accessible from the Directory page by unit leaders.

## 3.2 MP Ordination Table

```
mp_ordinations
├── id (uuid, primary key)
├── stake_id (foreign key → stakes)
├── person_name (string)
├── unit_id (foreign key → units)
├── current_office (enum) — "deacon" | "teacher" | "priest" | "none"
├── proposed_office (enum) — "elder" | "high_priest"
├── 
├── # Pre-submission requirements:
├── bishop_interview_complete (boolean)
├── lcr_record_created (boolean)
├── 
├── # Workflow:
├── status (enum) — "submitted" | "approved" | "sustained" | "ordained" | "recorded"
├── to_be_ordained_by (string, optional) — name of who will ordain
├── target_ordination_date (date, optional)
├── 
├── # Completion:
├── ordained (boolean, default false)
├── ordained_date (timestamp, optional)
├── 
├── # Metadata:
├── submitted_by_user_id (foreign key → users)
├── bishop_comments (text, optional)
├── created_at (timestamp)
└── updated_at (timestamp)
```

## 3.3 Submission Form Fields

1. Date (auto-filled)
1. MP Ordination Submission? (toggle, default ON)
1. Your Name (auto-filled from logged-in user)
1. Person You Are Recommending (required, text)
1. Ward or Branch this person currently attends (dropdown)
1. Proposed Office (dropdown: Elder, High Priest)
1. Current Office (dropdown)
1. Bishop’s Interview Complete? (required, dropdown) + link to instructions
1. MP Ordination Record Created and Submitted in LCR? (required) + link to LCR
1. To Be Ordained By (text)
1. Target Ordination Date (date picker)
1. Bishop’s Comments (text area)

## 3.4 MP Tracker (Stake Council view)

Shows ordinations by status, allows SP to manage approval and track completion.

-----

# Part 4: Action Items

## 4.1 Overview

Three separate action item lists based on scope:

- **SP Action Items** — Created by SP, visible to SP only
- **HC Action Items** — Created by SP or HC, visible to SP + HC
- **SC Action Items** — Created by anyone on SC/HC/SP, visible to all council

## 4.2 Action Items Table

```
action_items
├── id (uuid, primary key)
├── stake_id (foreign key → stakes)
├── scope (enum) — "sp" | "hc" | "sc"
├── title (string) — brief description
├── description (text) — full details
├── source_meeting_tag (string, optional) — "[SMC]", "[HC]", etc.
├── due_date (date)
├── 
├── # Assignment (can have multiple assignees):
├── follow_up_user_id (foreign key → users, optional)
├── 
├── # Status:
├── status (enum) — "open" | "complete"
├── completed_date (timestamp, optional)
├── 
├── # Notes:
├── notes (text, optional)
├── 
├── # Metadata:
├── created_by_user_id (foreign key → users)
├── created_at (timestamp)
└── updated_at (timestamp)
```

### Action Item Assignees (junction table for multiple assignees)

```
action_item_assignees
├── id (uuid, primary key)
├── action_item_id (foreign key → action_items)
└── user_id (foreign key → users)
```

## 4.3 Action Item Screens

### List View

- Search bar + filter icon + Add button
- Grouped by due date
- Each item shows: Assignee(s), description, due date, ⋯ menu
- Bottom actions: Completed Items, Sort By Assignee, Email All, Copy Outstanding

### Add/Edit Form

- Assigned to (dropdown, multi-select)
- Assignment (text)
- Complete By Date (date picker)
- Follow up (dropdown)
- Notes or Report (text area)

-----

# Part 5: Speaking Assignments

## 5.1 Overview

Two distinct speaking assignment systems:

1. **Stake Council Speaking Assignments** — SC members speak in wards (monthly)
1. **Branch Speaking Assignments** — Wards provide speakers to branches (monthly rotation)

## 5.2 Stake Council Speaking Assignments Table

```
speaking_assignments
├── id (uuid, primary key)
├── stake_id (foreign key → stakes)
├── date (date)
├── topic (text, optional) — with scripture references
├── created_at (timestamp)
└── updated_at (timestamp)
```

### Speaking Assignment Units (per-unit speaker for each date)

```
speaking_assignment_units
├── id (uuid, primary key)
├── speaking_assignment_id (foreign key → speaking_assignments)
├── unit_id (foreign key → units)
├── speaker_user_id (foreign key → users) — who is speaking
└── reminder_sent (boolean, default false)
```

## 5.3 Branch Speaking Assignments Table

```
branch_speaking_assignments
├── id (uuid, primary key)
├── stake_id (foreign key → stakes)
├── branch_id (foreign key → units) — the branch receiving speaker
├── providing_ward_id (foreign key → units) — the ward providing speaker
├── month (integer) — 1-12
├── year (integer)
├── speaker_name (string, optional) — if known
└── created_at (timestamp)
```

## 5.4 Speaking Assignment Screens

### Stake Council Speaking Assignments List

- Edit All Speaking Assignments button (opens spreadsheet view)
- Speaking Calendar button
- Add (+) button
- List of dates with topics

### Spreadsheet View (Desktop/Web)

Matrix: Rows = dates, Columns = units
Each cell = dropdown to select speaker from eligible users (SP, HC, SC members)

### Day Detail View

- Date header
- Topic with scripture references
- “Email Reminder to Upcoming Speakers” button
- List of all units with assigned speaker + individual email button

### Branch Speaking Assignments

- List by month (January - December)
- Drill into month → shows branch name + ward providing speaker

-----

# Part 6: Meetings & Agendas

## 6.1 Overview

Two main meeting types with full agenda management:

- **High Council Meeting**
- **Stake Council Meeting**

## 6.2 Meetings Table

```
meetings
├── id (uuid, primary key)
├── stake_id (foreign key → stakes)
├── meeting_type (enum) — "high_council" | "stake_council" | "presidency" | "other"
├── date (date)
├── 
├── # Agenda fields:
├── opening_prayer_user_id (foreign key → users, optional)
├── closing_prayer_user_id (foreign key → users, optional)
├── hymn (string, optional) — hymn number and name
├── discussion_leader_user_id (foreign key → users, optional)
├── discussion_topic (text, optional)
├── ministering_experience_user_id (foreign key → users, optional)
├── 
├── # Content:
├── agenda_content (text) — rich text/markdown for agenda
├── minutes (text, optional) — notes from meeting
├── parking_lot_items (text, optional) — items for future discussion
├── 
├── # Status:
├── is_archived (boolean, default false)
├── 
├── created_at (timestamp)
└── updated_at (timestamp)
```

### Meeting Attendance

```
meeting_attendance
├── id (uuid, primary key)
├── meeting_id (foreign key → meetings)
├── user_id (foreign key → users)
└── attended (boolean, default false)
```

## 6.3 Agenda Screens

### Agenda View

Header actions:

- Email Agenda to [HC/SC]
- Copy Agenda to Clipboard
- Join Meeting Via Zoom
- Edit Zoom Link

Content sections:

- Vision Statement (displayed)
- Date picker
- Hymn dropdown
- Opening Prayer dropdown (from attendee list)
- Counsel Discussion Leader dropdown
- Discussion Topic text field
- Ministering One-On-One Experience dropdown

Integration buttons:

- Review Callings and Releases (links to tracker)
- Review [HC/SC] Action Items (links to action items)

Agenda block:

- Presiding / Conducting
- Structured agenda items with times
- Council Discussion items

Footer:

- Minutes text area
- Closing Prayer dropdown
- Archive This Meeting button
- Add Action Item button

Additional sections:

- Parking Lot Items (numbered list with links)
- Minutes From Previous Meeting
- Attendance (tap-to-select all eligible members)

-----

# Part 7: Resources & Directory

## 7.1 Directory Page Structure

Main tiles (visible based on role):

- High Council Members
- High Council/Stake Council Meetings
- Stake Council Speaking Assignments
- Branch Speaking Assignments
- General Resources
- Info Change Request
- Temple Recommend Interview Questions
- Melchizedek Priesthood Ordination Submission
- Melchizedek Priesthood Tracker
- Calling and Release Tracker
- Priesthood Ordinances
- Missionary Interview Questions
- Training Documents

Admin sections (bottom):

- New Sign-Ups to Review (X)
- Info Change Submissions to Review (X)

## 7.2 Unit Info Screens

### Unit List

- Grid of unit cards with building photos
- Unit name below each photo

### Unit Detail

- Large building photo header
- Unit name
- Address
- Edit button (if authorized)
- Meeting times (Sacrament, Ward Council, Youth Activities, etc.)
- Get Directions button (opens maps)

## 7.3 Resources Table

```
resources
├── id (uuid, primary key)
├── stake_id (foreign key → stakes)
├── category (enum) — "stake_council_tools" | "high_council_tools" | "general" | "training"
├── title (string)
├── description (text, optional)
├── url (string, optional) — external link
├── content (text, optional) — inline content
├── display_order (integer)
├── created_at (timestamp)
└── updated_at (timestamp)
```

-----

# Part 8: Permissions Matrix

## 8.1 Feature Access by Role

|Feature                    |Admin|SP|Exec Sec|HC|SC|SC Counselor|Unit Leader|Unit Aux|Clerk|
|---------------------------|-----|--|--------|--|--|------------|-----------|--------|-----|
|**Stake Council Section**  |✓    |✓ |✓       |✓ |✓ |✓           |✗          |✗       |✓    |
|Submit Calling/Release     |✓    |✓ |✓       |✓ |✓ |✓           |✓          |✗       |✓    |
|View Calling Tracker       |✓    |✓ |✓       |✓ |✗ |✗           |◐          |✗       |✓    |
|SP Review/Sustain          |✓    |✓ |✗       |✗ |✗ |✗           |✗          |✗       |✗    |
|HC Review/Sustain          |✓    |✓ |✗       |✓ |✗ |✗           |✗          |✗       |✗    |
|Assign Callings            |✓    |✓ |✓       |✗ |✗ |✗           |✗          |✗       |✗    |
|Change Assignment          |✓    |✓ |✓       |✓ |✗ |✗           |✗          |✗       |✗    |
|Mark Set Apart             |✓    |✓ |✓       |✓ |✗ |✗           |✗          |✗       |✗    |
|Record in LCR              |✓    |✓ |✓       |✗ |✗ |✗           |✗          |✗       |✓    |
|SP Action Items            |✓    |✓ |✗       |✗ |✗ |✗           |✗          |✗       |✗    |
|HC Action Items            |✓    |✓ |✓       |✓ |✗ |✗           |✗          |✗       |✗    |
|SC Action Items            |✓    |✓ |✓       |✓ |✓ |✓           |✗          |✗       |✗    |
|Meeting Agendas            |✓    |✓ |✓       |✓ |✓ |◐           |✗          |✗       |✗    |
|Speaking Assignments (edit)|✓    |✓ |✓       |✗ |✗ |✗           |✗          |✗       |✗    |
|Speaking Assignments (view)|✓    |✓ |✓       |✓ |✓ |✓           |✓          |✓       |✓    |
|MP Ordination Submit       |✓    |✓ |✓       |✓ |✗ |✗           |✓          |✗       |✗    |
|MP Ordination Tracker      |✓    |✓ |✓       |✓ |✗ |✗           |◐          |✗       |✓    |
|**Directory Section**      |✓    |✓ |✓       |✓ |✓ |✓           |✓          |✓       |✓    |
|Unit Info                  |✓    |✓ |✓       |✓ |✓ |✓           |✓          |✓       |✓    |
|Resources                  |✓    |✓ |✓       |✓ |✓ |✓           |✓          |✓       |✓    |
|Edit Users                 |✓    |✓ |✗       |✗ |✗ |✗           |✗          |✗       |✗    |
|Approve New Users          |✓    |✓ |✗       |✗ |✗ |✗           |✗          |✗       |✗    |
|Edit Stake Settings        |✓    |✓ |✗       |✗ |✗ |✗           |✗          |✗       |✗    |

Legend: ✓ = Full access, ◐ = Limited/conditional access, ✗ = No access

## 8.2 Conditional Access Notes

- **Unit Leader Calling Tracker:** Only sees callings for their unit when “Allow Bishop to Track” is enabled
- **Unit Leader MP Tracker:** Only sees ordinations for their unit
- **SC Counselor Agendas:** Can view but not edit

-----

# Part 9: Notification System

## 9.1 Notification Architecture

**Current:** Email only
**Target:** Push notifications (primary) + Email (secondary/digest)

## 9.2 Notification Triggers

### Calling/Release Tracker

|Event                 |Recipients     |Message                                               |
|----------------------|---------------|------------------------------------------------------|
|New submission        |SP, SP1, SP2   |“New calling submission: [Name] for [Calling]”        |
|Sent for SP sustaining|SP, SP1, SP2   |“Calling ready for sustaining: [Name]”                |
|Sent to High Council  |All HC members |“Calling ready for HC review: [Name]”                 |
|Assignment made       |Assigned person|“You’ve been assigned to extend: [Name] for [Calling]”|
|Assignment changed    |New assignee   |“Assignment transferred to you: [Name]”               |

### Other Notifications

|Event                      |Recipients        |Message                                         |
|---------------------------|------------------|------------------------------------------------|
|Speaking assignment created|Assigned speaker  |“Speaking assignment: [Date] at [Unit]”         |
|Speaking reminder (manual) |Individual speaker|“Reminder: Speaking assignment [Date] at [Unit]”|
|Action item assigned       |Assignee(s)       |“New action item: [Title]”                      |
|MP ordination submitted    |SP                |“New MP ordination submission: [Name]”          |
|New user sign-up           |Admins            |“New user awaiting approval: [Name]”            |

### Bulk/Scheduled Notifications

|Type                         |Trigger                |Recipients                    |
|-----------------------------|-----------------------|------------------------------|
|Action item weekly digest    |Manual trigger (button)|All assignees with open items |
|Speaking assignment reminders|Manual trigger (button)|All speakers for upcoming date|

## 9.3 Notification Preferences (Future)

Consider allowing users to configure:

- Push vs email preference
- Digest frequency (immediate, daily, weekly)
- Notification categories to receive

-----

# Part 10: UX Recommendations

## 10.1 Design Principles

1. **Mobile-first** — Optimize for phone use in hallways and cars
1. **Progressive disclosure** — Simple surface, depth when needed
1. **Context-aware** — Show relevant info based on role
1. **No chasing** — Notifications drive workflow, not manual checking
1. **Workflow invisible** — Users “do the next thing,” system handles the rest

## 10.2 Recommended Improvements Over Current Glide App

### Home Dashboard

**Current:** 10+ buttons competing for attention
**Recommended:** Priority-based task list

```
┌─────────────────────────────────────┐
│  🔔 You have 5 action items         │
├─────────────────────────────────────┤
│  NEEDS YOUR ATTENTION               │
│  • 2 callings awaiting review       │
│  • 1 calling to extend              │
│  • 7 to be set apart                │
├─────────────────────────────────────┤
│  UPCOMING                           │
│  Jan 18 · Speaking Assignment       │
│  Jan 21 · High Council Meeting      │
├─────────────────────────────────────┤
│  [ + Submit Calling/Release ]       │
└─────────────────────────────────────┘
```

### Calling Tracker

**Current:** Long scrolling list of buckets
**Recommended:** Kanban-style pipeline or filtered views

- Default view: “My Items” (only what’s assigned to me)
- Toggle to: “All Items” (full pipeline view)
- Visual status indicators (color-coded stages)
- Swipe actions for common operations

### Related Submissions

**Current:** Separate submissions for release + calling
**Recommended:** Single combined submission

When submitting a calling that involves a release:

- Capture both in one form
- Show both toggles on extend screen
- Link for “How to extend callings” guide

### Agenda Builder

**Current:** Very long scrolling form
**Recommended:** Tabbed or collapsible sections

Tabs: Setup | Agenda | Minutes | Attendance
Or collapsible sections that expand on tap

### Attendance Tracking

**Current:** Long scrolling list of tap targets
**Recommended:** Compact multi-select with checkboxes

Group by category (SP, HC, Auxiliary Presidents, etc.)
“Select All” / “Deselect All” options

### Auto-Sustain for Reviewer

**Current:** SP reviewer must open separate screen to sustain
**Recommended:** Auto-mark reviewer as sustained when they send for sustaining

## 10.3 Navigation Structure

**Bottom tabs:**

1. **Home** — Dashboard with tasks and upcoming items
1. **Tracker** — Calling pipeline (role-appropriate view)
1. **Calendar** — Meetings, speaking assignments
1. **Directory** — Units, resources, people

**Hamburger menu:**

- Settings
- User management (admin only)
- Stake settings (admin only)
- Share This App
- Help/Feedback

-----

# Part 11: Technical Implementation Notes

## 11.1 Technology Recommendations

**Framework:** React Native (or Flutter)

- Single codebase for iOS and Android
- Large community and ecosystem
- Good performance for this type of app

**Backend:** Supabase or Firebase

- Built-in auth (Google, Apple)
- Real-time database
- Push notification support
- Row-level security for multi-tenancy
- Generous free tier

**Push Notifications:** Firebase Cloud Messaging

- Free
- Works for both iOS and Android
- Easy integration with React Native

## 11.2 Multi-Tenancy Implementation

Use row-level security (RLS) policies:

- Every table has `stake_id` column
- RLS policy: users can only see/modify rows where `stake_id` matches their stake
- Auth token includes stake_id claim

## 11.3 Offline Considerations

For users in areas with poor connectivity:

- Cache key data locally (their assignments, upcoming meetings)
- Queue actions when offline, sync when connected
- Show clear offline indicator

## 11.4 App Store Considerations

**iOS:**

- Apple Developer account ($99/year)
- Religious app category
- No IAP needed (free app)

**Android:**

- Google Play Developer account ($25 one-time)
- Similar categorization

-----

# Part 12: Implementation Order

## Phase 1: Foundation (MVP)

1. Authentication (Google/Apple sign-in)
1. Multi-tenant stake setup
1. User management and role assignment
1. Basic permissions system
1. Unit/Directory management

## Phase 2: Core Workflow

1. Calling/Release submission
1. SP review and sustaining flow
1. HC review flow
1. Assignment system
1. Basic notifications (push)

## Phase 3: Completion Features

1. Pre-sustaining bucket
1. Unit sustaining tracking
1. Set apart and recording
1. Bishop visibility option

## Phase 4: Supporting Features

1. Action items (all three scopes)
1. Meeting agendas
1. Speaking assignments
1. MP ordination tracker

## Phase 5: Polish

1. Resources/training documents
1. Offline support
1. Notification preferences
1. Analytics/reporting

-----

# Appendix A: Complete Callings List

## Stake Leadership

- Stake President
- Stake Presidency 1st Counselor
- Stake Presidency 2nd Counselor
- Stake Executive Secretary
- Stake Assistant Executive Secretary 1
- Stake Assistant Executive Secretary 2
- Stake Clerk
- Stake Assistant Clerk

## High Council

- HC#1 through HC#12

## Stake Auxiliary Presidents

- Stake Relief Society President
- Stake Relief Society 1st Counselor
- Stake Relief Society 2nd Counselor
- Stake Relief Society Secretary
- Stake Young Women President
- Stake Young Women 1st Counselor
- Stake Young Women 2nd Counselor
- Stake Young Women Secretary
- Stake Young Men President (if applicable)
- Stake Young Men 1st Counselor
- Stake Young Men 2nd Counselor
- Stake Primary President
- Stake Primary 1st Counselor
- Stake Primary 2nd Counselor
- Stake Sunday School President
- Stake Sunday School 1st Counselor
- Stake Sunday School 2nd Counselor
- Stake Sunday School Secretary

## Other Stake Callings

- Stake Patriarch
- Patriarch Scribe

## Unit Leadership

- Bishop
- Bishop 1st Counselor
- Bishop 2nd Counselor
- Branch President
- Branch President 1st Counselor
- Branch President 2nd Counselor
- Ward/Branch Executive Secretary
- Ward/Branch Clerk

## Unit Auxiliary (for Directory access)

- Elders Quorum President
- Relief Society President
- Young Women President
- Primary President
- Sunday School President
- Ward Mission Leader

-----

# Appendix B: Glossary

|Term             |Definition                                                  |
|-----------------|------------------------------------------------------------|
|**Calling**      |A volunteer position in the church organization             |
|**Release**      |Formally ending service in a calling                        |
|**Sustaining**   |Congregational vote to support someone in a calling         |
|**Setting Apart**|Blessing given when beginning a new calling                 |
|**LCR**          |Leader and Clerk Resources — church’s official record system|
|**SP**           |Stake Presidency (President + 2 Counselors)                 |
|**HC**           |High Council (12 members)                                   |
|**SC**           |Stake Council (SP + HC + Auxiliary Presidents)              |
|**MP**           |Melchizedek Priesthood                                      |
|**Ward**         |Standard local congregation                                 |
|**Branch**       |Smaller congregation (fewer members than ward)              |
|**Unit**         |Generic term for ward or branch                             |
|**Stake**        |Geographic region containing multiple wards/branches        |

-----

*End of Specification*