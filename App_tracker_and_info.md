# Calling Tracker — Complete Specification

**Last Updated:** January 8, 2026  
**Status:** Workflow Fully Documented

-----

## Overview

The Calling Tracker is the heartbeat of stake administration. It manages the complete lifecycle of:

- **Callings** — new calling requests
- **Releases** — releasing someone from a current calling
- **Combined Calling/Release** — simultaneous release from old + calling to new
- **Melchizedek Priesthood Ordinations** — separate but related workflow

Each submission flows through a multi-step workflow with role-based visibility and actions.

-----

## Complete Workflow (End-to-End)

```
1. SUBMISSION
   └─→ Stake council member creates new calling/release request
   └─→ Visible to: PO (Presidency Office)

2. SP REVIEW
   └─→ SP configures settings, adds notes
   └─→ Assigns who extends calling/release
   └─→ Sets who will set apart (SP vs HC vs Bishopric)

3. SP SUSTAINING
   └─→ All 3 SP members tap to sustain
   └─→ "Send to High Council for Sustaining"

4. HC SUSTAINING
   └─→ 12 HC members each tap to sustain
   └─→ Progress bar shows X/12
   └─→ "Sustained By High Council" when complete

5. READY FOR ASSIGNMENT
   └─→ Item in assigned SP/HC member's queue

6. EXTENSION
   └─→ Assigned person extends calling/release
   └─→ Track: Release Extended? Calling Accepted? Calling Declined?

7. DECLINED BUCKET (if declined)
   └─→ Goes to Declined Bucket
   └─→ SP reviews and deletes
   └─→ New submission created if trying different person

8. PRE-SUSTAINING BUCKET (if accepted)
   └─→ Staging before ward sustaining
   └─→ Allows batching sustainings across stake on same day
   └─→ "Send to Unit for Sustaining"

9. WARD SUSTAINING
   └─→ Appears on ward's sacrament meeting screen
   └─→ Bishop/counselor reads script from app
   └─→ Congregation votes
   └─→ Tap "Sustained" or "Released" for each item

10. TO BE SET APART OR ORDAINED
    └─→ Routed to SP, HC, or Ward based on configuration
    └─→ Toggle "Set Apart?" when complete

11. TO BE RECORDED
    └─→ Toggle "Release Recorded in LCR?"
    └─→ Toggle "Sustaining Recorded in LCR?"

12. COMPLETE (archived)
```

-----

## Terminology

|Term                      |Members                                   |Notes                                  |
|--------------------------|------------------------------------------|---------------------------------------|
|**SP**                    |Stake President (one person)              |                                       |
|**SP1**                   |1st Counselor                             |                                       |
|**SP2**                   |2nd Counselor                             |                                       |
|**Stake Presidency**      |SP + SP1 + SP2                            |The three presiding officers           |
|**PO (Presidency Office)**|SP + SP1 + SP2 + Exec Sec + Asst Exec Secs|Full admin team with elevated access   |
|**HC**                    |High Council                              |12 members, numbered HC#1 through HC#12|

**Key Visibility Rule:** PO can see the full calling/release/MP tracker at all times.

-----

## Calling Tracker Home Screen

**Header:** “Calling and Release Tracker”

### Components

1. **Search All Submissions** — button at top
1. **Workflow Buckets** — cards showing counts at each stage:
- Initial Review By Stake Presidency
- To Be Sustained By Stake Presidency
- To Be Reviewed By High Council
- Ready For Assignment
1. **Stake Presidency Assignments** — individual queues per SP member showing their assigned items
1. **High Council Assignments** — individual queues per HC member (HC#1 through HC#12)

-----

## Submission Form (Creating New Requests)

**Header:** “Calling or Release Request”

### Form Fields (Progressive Disclosure)

The form uses conditional fields — fields appear/hide based on previous selections.

#### Always Shown

|Field        |Type       |Required|Notes                                                               |
|-------------|-----------|--------|--------------------------------------------------------------------|
|**Date**     |Date picker|✓       |Auto-populated with current date/time                               |
|**Your Name**|Dropdown   |✓       |Searchable list of all stake council members (SP, SP1, SP2, HC#1-12)|

#### Submission Type

|Field                                              |Type      |Required|Notes                 |
|---------------------------------------------------|----------|--------|----------------------|
|**Calling and/or Release? (select all that apply)**|Checkboxes|✓       |Can select one or both|

Options:

- ☐ Calling
- ☐ Release

This creates three possible submission types:

1. Calling only
1. Release only
1. Calling + Release (both checked)

#### If “Calling” is checked

|Field                                                                                                        |Type      |Required|Notes                                                         |
|-------------------------------------------------------------------------------------------------------------|----------|--------|--------------------------------------------------------------|
|**Person You Are Requesting or Releasing**                                                                   |Text input|✓       |Free text for person’s name                                   |
|**Have you counseled with the Bishop? (If no then leave blank)**                                             |Dropdown  |✓       |Options: blank, “Yes” — GATE: blocks submission if not “Yes”  |
|**If this individual is currently serving in a calling, have you counseled with the organization president?**|Dropdown  |        |Options: “—”, “Yes”                                           |
|**Calling Type**                                                                                             |Dropdown  |✓       |See options below                                             |
|**Calling You Are Requesting Them For**                                                                      |Text input|        |Free text for new calling name                                |
|**This Person’s Current Calling**                                                                            |Text input|        |Context for what they’re leaving                              |
|**Ward or Branch this person currently attends**                                                             |Dropdown  |✓       |Stake-specific list configured by admin                       |
|**Request notes**                                                                                            |Text area |        |Placeholder: “(Please add any notes relevant to this request)”|

#### If “Release” only is checked

|Field                                           |Type      |Required|Notes                           |
|------------------------------------------------|----------|--------|--------------------------------|
|**Person You Are Requesting or Releasing**      |Text input|✓       |                                |
|**This Person’s Current Calling**               |Text input|        |What they’re being released from|
|**Ward or Branch this person currently attends**|Dropdown  |✓       |                                |
|**Request notes**                               |Text area |        |                                |

### Calling Type Options

|Option                                                  |Description                                            |
|--------------------------------------------------------|-------------------------------------------------------|
|Stake Calling (calling within the Stake)                |Stake-level callings (stake auxiliary presidents, etc.)|
|Ward or Branch Calling (a calling presided over by the…)|Ward/branch callings that require stake approval       |
|Stake Assignment (assignment to assist within the Sta…) |Less formal stake assignments                          |
|Ward or Branch Release (a ward or branch calling pres…) |Ward releases that need stake awareness                |

### Ward/Branch Dropdown

**Configured by stake admin** during initial app setup. Example (Kyle Texas Stake):

1. 1 Bastrop Ward
1. 2 Blanco Vista Branch
1. 3 Buda Ward
1. 4 Cedar Creek Ward
1. 5 Kyle Ward
1. 6 Lockhart Branch
1. 7 Plum Creek Ward
1. 8 San Marcos Ward
1. 9 San Marcos YSA Branch

### Form Footer

- Validation message: “IF THE SUBMIT BUTTON IS GRAYED OUT, PLEASE MAKE SURE ALL REQUIRED FIELDS ARE COMPLETED.”
- **Submit** button — disabled until all required fields are filled

-----

## SP Review Screen

When an SP member opens a submission for review:

### Read-Only Submission Info

|Field                                 |Description                                     |
|--------------------------------------|------------------------------------------------|
|Date                                  |When submitted                                  |
|Submitted By                          |Who created the submission (calling + name)     |
|Calling or Release                    |Type: “Calling”, “Release”, or “Calling,Release”|
|Person You Are Requesting or Releasing|The member’s name                               |
|Calling Type                          |“Stake Calling” or “Ward Calling”               |
|Calling You Are Requesting Them For   |The new calling                                 |
|This Person’s Current Calling         |Their existing calling (for context/release)    |
|Request notes                         |Submitter’s notes                               |
|Home Unit                             |The member’s ward/branch                        |
|Bishop is aware?                      |Yes/No                                          |

### SP-Only Editable Fields

|Field                       |Description                                               |
|----------------------------|----------------------------------------------------------|
|**Stake Presidency Note**   |Text area for SP internal notes (visible only to SP)      |
|**Assign Calling to**       |Dropdown — which SP/HC member extends the calling         |
|**Assign Release to**       |Dropdown — which SP/HC member extends the release         |
|**Assign Denied Request To**|Dropdown — who handles notification if calling is declined|

### Sustaining Location

|Field       |Options                                            |
|------------|---------------------------------------------------|
|Sustain in….|“All Stake Units” (stake callings) or specific unit|
|Release In….|“All Stake Units” or specific unit                 |

### Configuration Toggles

|Toggle                                  |Purpose                                                                               |
|----------------------------------------|--------------------------------------------------------------------------------------|
|**Allow Bishop to Track?**              |Gives bishop visibility so they don’t have to text “where are we at?”                 |
|**To Be Set Apart By Stake Presidency?**|SP member will perform setting apart                                                  |
|**Can Be Set Apart By High Councilor?** |HC member authorized to set apart (decided per-submission by SP)                      |
|**Place on hold**                       |Pauses the workflow — used for complex calling chains where one change triggers others|

### SP Sustaining Section

Three buttons with hand icons:

- 👋 Stake President
- 👋 1st Counselor
- 👋 2nd Counselor

Each SP member taps their button to sustain. Once all 3 are tapped, the “Send to High Council for Sustaining” button appears/activates.

### Actions

- **“Send to the High Council for Sustaining”** — advances workflow (appears after all 3 SP sustain)
- **“Delete Request”** — removes submission

-----

## HC Sustaining Screen

When an HC member opens a submission from “To Be Reviewed By High Council”:

### Simplified View (compared to SP)

Shows:

- Basic submission info (type, person, unit, calling, notes)
- **High Council Notes** — text area for HC notes (visible to SP + all HC members)
- **High Council Sustaining Total** — progress bar showing X/12
- **List of all 12 HC members** with sustain buttons

### Sustaining List

Each HC member shown with position number and name:

- HC#1 Chris Hernandez 👋
- HC#2 Andrew Pinkston 👋
- … through HC#12

When an HC member taps their own button:

- Their row shows “Yes”
- The submission disappears from their personal “To Be Reviewed” list
- Progress bar updates

### PO-Only Actions

|Action          |Who Can Use|Purpose                                        |
|----------------|-----------|-----------------------------------------------|
|“Set All To Yes”|PO only    |Bulk action after live voice vote in HC meeting|

### Completion

When all 12 HC have sustained:

- **“Sustained By High Council”** button appears
- Progress bar shows 12/12
- Tapping advances to “Ready For Assignment”

-----

## Extension Screen (Assigned SP/HC Member View)

**Header:** “Assigned to [SP/HC Position]”

### Two Sections

1. **Assigned Callings to Extend** — list of callings to extend
1. **Assigned Releases to Extend** — list of releases to extend

Even for combined “Calling,Release” submissions, the calling and release appear as separate items (may be extended at different times or by different people).

### Extension Detail View

**Header:** “Calling,Release”

#### Read-Only Info

- Calling or Release type
- Person name
- Home Unit
- Calling Type
- Requested calling
- Current calling
- Request notes

#### Action Toggles

- **Release Extended?** — toggle when release has been extended
- **Calling Accepted** — toggle when person accepts the calling
- **Calling Declined** — toggle if person declines

#### High Council Note

- Text field for notes

### Outcomes

- **If Accepted:** Item moves to Pre-Sustaining Bucket
- **If Declined:** Item moves to Declined Bucket → SP reviews → Deletes → New submission if retrying with different person

### Change Assignment

- SP or HC can reassign items to a different person
- Notification fires to newly assigned person

-----

## Pre-Sustaining Bucket

**Purpose:** Staging area for items that are approved and accepted, waiting for coordinated sustaining day.

**Why it exists:** The stake tries to do all sustainings and releases across all units on the same day, batching them together.

### Sections

- **Releases** — grouped by ward
- **Sustainings** — grouped by ward

### Actions

|Action                         |Description                                |
|-------------------------------|-------------------------------------------|
|**Send to Unit for Sustaining**|Pushes item to the ward’s sustaining screen|
|**Edit**                       |Modify the item                            |

-----

## Ward Sustaining Screen

**Header:** “[Ward Name]” (e.g., “1 Bastrop Ward”)

**Purpose:** Used by bishopric during sacrament meeting to conduct sustaining votes.

### Instructions

“After the individual has been sustained or released, click the three dots to the right of each name and select ‘Released’ or ‘Sustained’”

### Sections with Scripts

#### Stake Business — Releases

Items with script: *“These individuals have been released from the following positions. Those who would like to express thanks for their service may show it by the uplifted hand.”*

#### Stake Business — Sustainings

Items with script: *“These individuals have been called to the positions as stated. Those who can sustain these individuals in these callings please show it by the uplifted hand. Those opposed, if any, may show it by the uplifted hand.”*

#### Priesthood Sustainings

Script: *“We propose that the following individual(s) receive the Melchizedek Priesthood and be ordained to the office as stated. Those in favor may show it by the uplifted hand. Those opposed, if any, may also show it.”*

#### Priesthood Ratifications

Script: *“We propose for ratification the ordination of the following individual(s) to the office(s) of the Melchizedek Priesthood, as previously performed.”*

### Item Display

Each item shows:

- Ward name (blue label)
- Person name
- Calling/position
- ⋯ menu with “Sustained” or “Released” action

### After Vote

Tap ⋯ → “Sustained” (or “Released”) to mark complete and advance to next stage.

-----

## Stake Business Screen (PO-Only View)

**Header:** “Stake Business”

**Who sees it:** PO members only (via “View All Stake Business” button)

Shows ALL stake business across ALL units, including:

- Releases (all wards)
- Sustainings (all wards)
- Priesthood Sustainings
- Priesthood Ratifications

Allows PO to:

- Review what’s pending across the entire stake
- Make edits/corrections
- See full picture vs single ward view

-----

## To Be Set Apart Or Ordained Screen

**Header:** “To Be Set Apart Or Ordained”

### Organization

Items routed based on **who performs the setting apart**:

|Who             |Purpose                             |
|----------------|------------------------------------|
|Stake Presidency|Stake callings (SP sets apart)      |
|[Ward Name]     |Ward callings (bishopric sets apart)|

Each entry shows count of pending items.

### Set Apart Detail View

**Info displayed:**

- Ward
- Person name
- Notes
- Calling Type
- Sustained Time Stamp

**Action:**

- **Set Apart?** — toggle when setting apart is complete

Once toggled, item advances to “To Be Recorded” stage.

-----

## To Be Recorded Screen

**Header:** “Calling To Be Recorded”

### Summary Info (Read-only)

|Field                 |Value Example                           |
|----------------------|----------------------------------------|
|Ward                  |BASTROP WARD                            |
|Person                |Test name                               |
|Notes                 |Test                                    |
|Calling or Release    |Calling,Release                         |
|Calling Type          |Stake Calling (calling within the Stake)|
|New Calling           |Test                                    |
|Sustained Time Stamp  |January 8, 2026 at 1:24 PM              |
|Current Calling       |Test                                    |
|Released in Home Unit?|✓                                       |

### Action Toggles

- **Release Recorded in LCR?** — toggle when release is entered in Leader and Clerk Resources
- **Sustaining Recorded in LCR?** — toggle when sustaining is entered in LCR

Once both are toggled ON, item is complete and archived.

-----

## Permissions Summary

### PO (Presidency Office) Members

**Members:** Stake President, SP 1st Counselor, SP 2nd Counselor, Executive Secretary, Assistant Executive Secretaries

**Capabilities:**

- View all submissions across all units at all times
- “Set All To Yes” bulk HC sustaining
- “View All Stake Business”
- Configure all submission settings
- Assign submissions to others
- Access all workflow stages

### High Council Members

**Capabilities:**

- View submissions in “To Be Reviewed By High Council” queue
- Sustain only their own vote (not others’)
- Add High Council Notes
- View their assigned extension items
- Change assignment on items assigned to them
- Cannot use PO bulk actions
- Items disappear from their list once sustained

### Bishops / Ward Leaders

- **“Allow Bishop to Track”** toggle controls their visibility
- When enabled, bishop can see where submission is in workflow
- Prevents need to text/call for status updates

-----

## Melchizedek Priesthood Ordination Flow

MP Ordinations follow a **separate but parallel workflow**:

```
1. SUBMISSION (by Bishop)
   └─→ Bishop completes interview with candidate
   └─→ Bishop creates ordination record in LCR (prerequisite)
   └─→ Bishop submits via app
   └─→ Visible to: PO

2. SP REVIEW & ASSIGNMENT
   └─→ SP reviews and assigns to SP/SP1/SP2 for stake interview
   └─→ Sets sustaining location (home unit or stake conference)
   └─→ Assigns HC member for ordination oversight

3. STAKE PRESIDENCY INTERVIEW
   └─→ Assigned SP member conducts interview
   └─→ Uses "Instructions for Interview" guide
   └─→ Marks "Stake Interview Complete"

4. HC REVIEW & SUSTAINING
   └─→ 12 HC members sustain (same as callings)

5. SUSTAINING
   └─→ Home unit only (between stake conferences), OR
   └─→ Whole stake (at stake conference)

6. ORDINATION
   └─→ Assigned HC member oversees
   └─→ Records ordination details in app:
       - Who performed ordination
       - Date ordained
       - Ordainer's info (record number, birthdate, priesthood office)

7. RECORDING
   └─→ Ordination Recorded in LCR?
   └─→ Record Signed By Stake Presidency Member?
   └─→ Certificate Printed and Given to Individual?

8. RATIFICATION (if needed)
   └─→ If only sustained in home unit, name flagged for ratification at next stake conference
```

### MP Submission Form Fields

**Part 1:**

- Date
- MP Ordination Submission? (toggle)
- Your Name (auto-filled)
- Person You Are Recommending (Required)
- Ward or Branch dropdown
- Proposed Office (Elder, High Priest)
- Current Office

**Part 2:**

- Bishop’s Interview Complete? (Required gate)
- “Instructions for Bishop’s Interview” button
- Melchizedek Priesthood Ordination Record Created and Submitted in LCR? (Required gate)
- “Ordination Record Link (Sign into LDS.Org first)” button
- To Be Ordained By

**Part 3:**

- Target Ordination Date
- Bishop’s Comments

### Key Differences: Callings vs MP Ordinations

|Aspect             |Callings                 |MP Ordinations                  |
|-------------------|-------------------------|--------------------------------|
|Initial visibility |PO sees all              |PO sees all                     |
|SP stage           |Review + 3-way sustaining|Review + assign interview       |
|Interview          |None                     |Required SP interview           |
|Sustaining options |Always in units          |Home unit OR stake conference   |
|Ratification       |Not applicable           |Required if home unit only      |
|Completion tracking|Set apart + LCR          |LCR + SP signature + certificate|

-----

## Data Model (Draft)

### calling_submissions

```
id (uuid, primary key)
stake_id (foreign key → stakes)
created_at (timestamp)
updated_at (timestamp)

# Submission basics
submitted_by_user_id (foreign key → users)
submission_type (enum: "calling" | "release" | "calling_release")
person_name (string)
person_member_id (string, optional)
home_unit_id (foreign key → units)

# Calling details
calling_type (enum: "stake_calling" | "ward_calling" | "stake_assignment" | "ward_release")
requested_calling (string)
current_calling (string, optional)
request_notes (text)
bishop_is_aware (boolean)
org_president_consulted (boolean)

# SP configuration
sp_notes (text)
assign_calling_to_user_id (foreign key → users)
assign_release_to_user_id (foreign key → users)
assign_denied_to_user_id (foreign key → users)
sustain_in (enum: "all_units" | "home_unit")
release_in (enum: "all_units" | "home_unit")
allow_bishop_to_track (boolean)
set_apart_by_sp (boolean)
can_set_apart_by_hc (boolean)
on_hold (boolean)

# SP sustaining
sp_sustained (boolean)
sp1_sustained (boolean)
sp2_sustained (boolean)
sp_sustaining_complete (boolean)
sp_sustaining_complete_at (timestamp)

# HC sustaining
hc1_sustained (boolean)
hc2_sustained (boolean)
... through hc12_sustained (boolean)
hc_sustaining_complete (boolean)
hc_sustaining_complete_at (timestamp)
hc_notes (text)

# Extension stage
release_extended (boolean)
release_extended_at (timestamp)
calling_accepted (boolean)
calling_accepted_at (timestamp)
calling_declined (boolean)
calling_declined_at (timestamp)

# Ward sustaining
sent_to_unit_for_sustaining (boolean)
sent_to_unit_at (timestamp)
sustained_in_unit (boolean)
sustained_in_unit_at (timestamp)

# Set apart
set_apart_by (enum: "stake_presidency" | "high_councilor" | "bishopric")
set_apart_complete (boolean)
set_apart_at (timestamp)
set_apart_by_user_id (foreign key → users)

# Recording
release_recorded_in_lcr (boolean)
release_recorded_at (timestamp)
sustaining_recorded_in_lcr (boolean)
sustaining_recorded_at (timestamp)
recorded_by_user_id (foreign key → users)

# Workflow status
status (enum: "submitted" | "sp_review" | "sp_sustaining" | "hc_review" | 
        "ready_for_assignment" | "extension" | "declined" | "pre_sustaining" | 
        "ward_sustaining" | "set_apart" | "recording" | "complete")
```

### mp_ordinations

```
id (uuid, primary key)
stake_id (foreign key → stakes)
created_at (timestamp)
updated_at (timestamp)

# Submission basics
submitted_by_user_id (foreign key → users)
person_name (string)
home_unit_id (foreign key → units)
proposed_office (enum: "elder" | "high_priest")
current_office (string)

# Prerequisites
bishop_interview_complete (boolean)
lcr_record_created (boolean)
to_be_ordained_by (string)
target_ordination_date (date)
bishop_comments (text)

# SP assignment
assigned_to_interview_user_id (foreign key → users)
sustain_in (enum: "home_unit" | "stake_conference")
assigned_to_oversee_user_id (foreign key → users)
sp_comments (text)

# Interview
stake_interview_complete (boolean)
stake_interview_date (date)
stake_interview_by_user_id (foreign key → users)

# HC sustaining (same pattern as callings)
hc1_sustained through hc12_sustained (boolean)
hc_sustaining_complete (boolean)

# Sustaining
sustained (boolean)
sustained_at (timestamp)
sustained_in (enum: "home_unit" | "stake_conference")

# Ordination
ordained (boolean)
date_ordained (date)
ordained_by_name (string)
ordained_by_record_number (string)
ordained_by_birthdate (date)
ordained_by_priesthood_office (string)
stake_representative_user_id (foreign key → users)

# Recording
ordination_recorded_in_lcr (boolean)
record_signed_by_sp (boolean)
certificate_printed (boolean)

# Ratification (if home unit only)
needs_ratification (boolean)
ratified (boolean)
ratified_at (timestamp)

# Status
status (enum: "submitted" | "sp_review" | "interview_assigned" | "interview_complete" |
        "hc_review" | "awaiting_sustaining" | "sustained" | "ordained" | 
        "recording" | "complete" | "awaiting_ratification")
```

-----

## Design Improvement Opportunities

### Already Identified

1. **Auto-sustain for reviewer** — SP member reviewing should have their sustain button auto-tapped
1. **Confusing toggle logic** — “Calling Accepted” AND “Calling Declined” as separate toggles is error-prone. Should be single choice.
1. **Missing “Extended” step for callings** — There’s “Release Extended?” but no “Calling Extended?” step before acceptance
1. **No date tracking on many actions** — When was it extended? When did they respond?
1. **No “stuck” indicators** — Items awaiting response for weeks should surface
1. **“TRUE” bug** — Boolean displaying instead of ward name in some views
1. **Free text for calling names** — Should be dropdown from valid calling list
1. **Free text for person names** — Could integrate with member directory
1. **“If no then leave blank” is confusing** — Contradictory UX on bishop counseling field

### Recommended Improvements for Rebuild

#### State Machine for Extension

Replace toggles with proper status:

```
Extension Status:
- Not Yet Extended
- Extended (date: ___)
- Awaiting Response
- Accepted (date: ___)
- Declined (date: ___, reason: ___)
```

#### Timeline/History View

Show audit trail of who did what when for each submission.

#### Calling Dependencies/Chains

Link related submissions (e.g., “new bishop” triggers release of old bishop, counselors, HC backfill).

#### Better Combined Calling/Release Flow

When both selected, show explicit connection:

```
Releasing from: [current calling]
Calling to: [new calling]
```

#### Structured Data

- Person name: Autocomplete from member directory
- Calling names: Dropdown filtered by calling type
- Current calling: Pull from church records if available

#### Meeting Date Association

“This item will appear on [Ward]’s sustaining list for [Date]”

#### Batch Actions

“Mark all releases as complete” / “Mark all sustainings as sustained”

#### Opposition Handling

“Was there an opposition?” → triggers different workflow (escalate to SP)

-----

-----

## Meetings & Agendas

Two main meeting types with full agenda management:

- **High Council Meeting**
- **Stake Council Meeting**

### Meeting List

- Shows upcoming and past meetings
- ⋯ menu: **Go to Agenda**, **Edit**, **Delete**

### Agenda View

#### Header Actions

- **Email Agenda to [HC/SC]** — sends to all attendees
- **Copy Agenda to Clipboard** — for pasting elsewhere
- **Join Meeting Via Zoom** — direct link
- **Edit Zoom Link** — configure meeting URL

#### Agenda Fields

- **Date** picker
- **Opening Prayer** dropdown (select from attendees)
- **Hymn** dropdown
- **Counsel Discussion Leader** dropdown
- **Discussion Topic** text field
- **Ministering One-On-One Experience** dropdown
- **Closing Prayer** dropdown
- **Vision Statement** (displayed)

#### Integration Buttons

- **“Review Callings and Releases”** — links to calling tracker
- **“Review [HC/SC] Action Items”** — links to action items

#### Agenda Content Block

Formatted text showing structured agenda:

- Presiding/Conducting
- Ministering Moment (5 min)
- Action Item Review (5 min)
- Stake Callings & Releases (5 min)
- Council Discussion
- Closing Testimony

#### Additional Sections

- **Minutes** text field for meeting notes
- **Parking Lot Items** — numbered list of items for future discussion
- **Minutes From Previous Meeting** — carried forward

#### Bottom Actions

- **Archive This Meeting**
- **Add Action Item**

### Attendance Tracking

Tap-to-select list showing all stake council members:

- Stake Presidency (3)
- Stake Clerk
- Executive Secretary + Assistants
- HC #1-12
- Stake RS Presidency
- Stake YW Presidency
- Stake YM Presidency
- Stake Primary Presidency
- Stake Sunday School
- Stake Patriarch + Scribe

**Chips/pills UI** — tap to mark present

-----

## Action Items

**Three separate scopes:**

- SP Action Items (Presidency only)
- HC Action Items (High Council)
- SC Action Items (Stake Council)

### Action Item List View

- **Search bar** + **filter icon** + **”+” add button**
- Items grouped by due date
- Each item shows:
  - **Assignee(s)** in blue header
  - **[Tag]** — source meeting (e.g., [SMC] for Stake Mission Committee)
  - **Description** — the task
  - **Complete By date**
  - **⋯ menu** for actions

### Bottom Actions

- **“Completed Action Items”** — view archive
- **“Sorted By Assignee”** toggle — alternate view
- **“Email Out All Assignments”** — weekly reminder to all assignees
- **“Copy Outstanding Action Items”** — clipboard

### Add Action Item Form

|Field               |Type       |Notes             |
|--------------------|-----------|------------------|
|**Assigned to**     |Dropdown   |Select person(s)  |
|**Assignment**      |Text       |Task description  |
|**Complete By Date**|Date picker|Due date          |
|**Follow up**       |Dropdown   |Who follows up    |
|**Notes or Report** |Text area  |Additional details|

**Note:** Items can have multiple assignees (e.g., “ALL STAKE PRESIDENCY”).

-----

## Speaking Assignments

There are **two separate systems** for speaking assignments:

### 1. Stake Council Speaking Assignments

**Purpose:** Stake council members (SP, HC, auxiliary presidents) speak in wards on assigned dates.

#### List View

- **“Edit All Speaking Assignments”** button — opens spreadsheet view
- **“Speaking Calendar”** button
- **”+” add button** — create new date
- List of dates with topics (e.g., “Jan 18, 2026 — Following promptings of the Spirit…”)

#### Spreadsheet View (Desktop/Web)

Matrix layout for bulk editing:

|Date  |Topic                |1 Bastrop          |2 Blanco Vista|3 Buda        |4 Cedar Creek |5 Kyle        |6 Lockhart    |7 Plum Creek|8 San Marcos      |9 YSA     |
|------|---------------------|-------------------|--------------|--------------|--------------|--------------|--------------|------------|------------------|----------|
|Jan 18|Following promptings…|HC#1 Chris         |Stake YW Sec  |HC#2 Andrew   |Stake YM 1st C|HC#3 Dennis   |Stake YW 1st C|HC#4 Chad   |Stake Primary Pres|HC#6 Devin|
|Feb 15|                     |Stake Primary 1st C|HC#7 Gene     |Stake SS 1st C|HC#8 Stuart   |Stake YW 2nd C|Stake RS 1st C|HC#10 Nick  |…                 |…         |

Each cell has a **dropdown selector** to pick from eligible speakers (SP, HC, Stake Council members).

#### Day Detail View

- Date header
- Topic with scripture references
- **“Email Reminder to Upcoming Speakers”** button — bulk email
- List of all units with assigned speaker
- Individual email icon per speaker — one-tap reminder

#### Edit Form

- **Date** picker
- **Topic** text field (can include scripture references)
- **Per-unit dropdowns** — select speaker for each unit from user list

### 2. Branch Speaking Assignments

**Purpose:** Wards provide speakers to branches on a monthly rotation.

**Different from Stake Council assignments** — this is about wards helping branches, not stake council members speaking.

#### List View

- Organized by month (January through December)

#### Month Detail View

Shows rotation:

|Branch Name        |Ward to Provide Speaker|
|-------------------|-----------------------|
|Blanco Vista Branch|Buda Ward              |
|Lockhart Branch    |Kyle Ward              |

-----

## Directory Tab (Main Hub)

The Directory tab serves as the **main navigation hub** of the app — a grid of feature tiles.

**Header:** Stake logo (customized per stake, e.g., “Kyle Texas Stake”)

### Feature Tiles

|Tile                                            |Purpose                                   |
|------------------------------------------------|------------------------------------------|
|**High Council Members**                        |View/manage HC roster                     |
|**High Council/Stake Council Meetings**         |Meeting schedules & agendas               |
|**Stake Council Speaking Assignments**          |SC members speaking in wards              |
|**Branch Speaking Assignments**                 |Wards providing speakers to branches      |
|**General Resources**                           |Resource library (training, guides, links)|
|**Info Change Request**                         |Request updates to personal info          |
|**Temple Recommend Interviews Questions**       |Interview question reference              |
|**Melchizedek Priesthood Ordination Submission**|MP ordination submission form             |
|**Melchizedek Priesthood Tracker**              |MP ordination workflow tracker            |
|**Calling and Release Tracker**                 |Main calling tracker (full-width tile)    |
|**Priesthood Ordinances**                       |Ordinance reference/tracking              |
|**Missionary Interview Questions**              |Mission interview reference               |
|**Training Documents**                          |Training materials                        |

-----

## Edit Users Tab

### User List Screen

**Header:** “Users”

**Components:**

- **Search bar** — Search users by name
- **+ button** — Add new user (top right)
- **Filter/sort button** — Organize user list

### User List Display

Each user card shows:

- **Calling** (blue label) — e.g., “STAKE YOUNG MEN 1ST COUNSELOR”
- **Name** — e.g., “Corey Humrich”
- **Ward** — e.g., “Bastrop Ward”
- **⋯ menu** — Actions

### User Detail/Edit Screen

**Header:** “[User Name]”

#### User Info (Top)

- Name
- Current calling
- Email button (envelope icon) to contact user

#### Editable Fields

|Field                                   |Type    |Notes                       |
|----------------------------------------|--------|----------------------------|
|**Ward or Branch**                      |Dropdown|User’s home unit            |
|**Calling**                             |Dropdown|Determines app permissions  |
|**If High Council Member, What Number?**|Dropdown|HC#1-12 (only if HC calling)|

#### Permission Toggles

|Toggle                  |Purpose                                                   |
|------------------------|----------------------------------------------------------|
|**Allow Access to App?**|User can log in and use the app                           |
|**Can Edit App Data?**  |Admin-level editing privileges (typically PO members only)|

#### Status Indicator (Read-only)

- **Receive Push Notifications Turned On?** — Shows whether user has enabled notifications on their device

### Permission Model

1. **Calling-based identity** — User’s calling determines their permissions throughout the app
1. **Ward association** — Each user belongs to a ward/branch
1. **HC number tracking** — If High Council, their number (1-12) is tracked separately
1. **“Can Edit App Data?”** — Admin flag for PO members, grants ability to edit data across the app

-----

## Share This App Tab

**Header:** “Share This App”

### Components

1. **QR Code** — Large, scannable QR code with stake logo embedded in center
1. **Instructions:** “SCAN THIS QR CODE TO OPEN THE [STAKE NAME] DIRECTORY APP”
1. **Copy App Link** button — Copies shareable link to clipboard
1. **Share Link Via Email** button — Opens email composer with app link

### Purpose

Allows stake council members to easily share the app with others who need access:

- New leaders being onboarded
- Ward council members who need the app
- Anyone authorized to use the stake app

**Note:** Sharing the link/QR code just gets someone to the app — they still need to be added as a user and granted access via Edit Users.

-----

## Admin Configuration (Stake Setup)

When a new stake sets up the app, admin configures:

1. **Units list** — All wards and branches in the stake (populates all dropdowns)
1. **Stake council members** — SP, SP1, SP2, Exec Sec, Asst Exec Secs, HC#1-12, auxiliary presidents
1. **User roles/permissions** — Who gets what access level
1. **Stake branding** — Stake logo for Directory header and QR code

### User Database

- **Standalone system** — No connection to LCR/church records
- **Manual management** — Admins add/edit users via Edit Users tab
- **Calling dropdown** — Pre-configured list of valid callings

-----

## Notifications

### Notification Triggers

|Event                                           |Recipients                   |Notes                                                |
|------------------------------------------------|-----------------------------|-----------------------------------------------------|
|**Action Items**                                |                             |                                                     |
|Action item created                             |Assignee(s)                  |                                                     |
|Action item edited                              |Assignee(s)                  |                                                     |
|Weekly action item reminder                     |All assignees with open items|Manual trigger via “Email Out All Assignments” button|
|**Calling/Release/MP Ordination**               |                             |                                                     |
|New submission created                          |SP, SP1, SP2                 |Any new calling, release, or MP ordination submission|
|SP member reviews and approves for SP sustaining|SP, SP1, SP2                 |Notifies other SP members to sustain                 |
|Sent to High Council for review/sustaining      |All 12 HC members            |First time HC sees the submission                    |
|Assignment given or changed                     |Assignee (SP/HC member)      |Who will extend the calling/release                  |

### Notification Delivery

**Current (Glide):** Email only

**Target (Native App):**

- Push notifications (primary)
- Email (secondary/digest option)

### Not Currently Triggering Notifications

- Calling accepted (no notification needed per Adam)
- Set apart complete (no notification needed)
- Recorded in LCR (no notification needed)
- Bishop tracking updates (not currently built — potential future enhancement)

-----

## Still To Document

### Other Features (from Directory tiles)

- High Council Members screen
- General Resources (content details)
- Info Change Request flow
- Temple Recommend Interview Questions
- Priesthood Ordinances
- Missionary Interview Questions
- Training Documents
- Unit Info screens (building photos, addresses, meeting times)

### Edge Cases

- What happens if HC member is unavailable for extended period?
- How to handle emergency callings that need to bypass normal flow?
- What if bishop position is being changed (who approves)?

-----

## Onboarding Flow

### Step 1: Invitation

- Existing user shares app via **QR code** or **link** (from “Share This App” tab)
- New person scans QR or taps link to open app

### Step 2: Account Creation

New user creates account using one of:

- **Email + Password**
- **Sign in with Google**
- **Sign in with Apple**

Then completes registration form:

|Field   |Type      |Required|
|--------|----------|--------|
|**Name**|Text input|✓       |
|**Unit**|Dropdown  |✓       |

**Note:** Email is captured from auth provider (Google/Apple) or entered directly. Unit dropdown is pre-populated with stake’s configured wards/branches.

### Step 3: Pending State

After registration, user sees a message:

> “The admins have been notified and will approve your account within 24 hours. If you still don’t have access after that time, please contact:
> 
> - Executive Secretary: [name/contact]
> - Stake Clerk: [name/contact]”

User cannot access app features until approved.

### Step 4: Admin Notification

- Admins (PO members with “Can Edit App Data?”) receive notification
- New registrant appears in **“New Sign-Ups to Review”** queue on Directory page

### Step 5: Admin Action

Admin reviews the new sign-up and can:

**Approve:**

1. Verify the person’s identity/legitimacy
1. Assign a **Calling** from the dropdown
1. Set **“Allow Access to App?”** to ON
1. Optionally set **“Can Edit App Data?”** if they need admin access
1. If High Council, set **HC number** (1-12)

**Reject/Delete:**

- Remove the registration if unrecognized or unauthorized

**Leave Pending:**

- Keep in queue for later review

### Step 6: Access Granted (if approved)

- User can now log in and use the app
- Their **calling determines what they see** throughout the app
- Permissions are role-based from that point forward

### Key Design Points

1. **No automatic access** — Every new user must be manually approved by an admin
1. **Calling = Permissions** — The calling assigned during approval determines all app access
1. **Stake-specific** — Each stake’s app is isolated; users register to a specific stake
1. **Clear pending state** — User knows what to expect and who to contact if delayed
1. **Fallback contact info** — Exec Sec and Stake Clerk contact displayed for issues

### Improvement Opportunities

1. **Approval notification to user** — When approved, user should get push notification or email: “Your access has been approved”
1. **Rejection notification** — If rejected, should user be notified? Or just silently removed? (Consider: could be awkward if someone wasn’t supposed to have access)
1. **Duplicate detection** — Flag if someone registers with same email or name as existing user
1. **Configurable contact info** — Exec Sec and Stake Clerk contact info should be editable in stake settings

-----

## Appendix: Screen Reference

|Screen                     |Purpose                                        |Who Sees It                |
|---------------------------|-----------------------------------------------|---------------------------|
|**Directory (Main Hub)**   |Navigation grid to all features                |All users                  |
|Calling Tracker Home       |Overview of all workflow stages and assignments|PO, HC                     |
|Submission Form            |Create new calling/release request             |All stake council          |
|SP Review                  |Configure and approve submissions              |SP, SP1, SP2               |
|HC Sustaining              |Individual HC member sustaining vote           |HC members, PO             |
|Assigned to [Position]     |Extension queue for assigned SP/HC member      |Assigned person            |
|Pre-Sustaining Bucket      |Staging area before ward sustaining            |PO                         |
|Ward Sustaining ([Ward])   |Sacrament meeting sustaining script            |Ward bishopric             |
|Stake Business             |All-units view of stake business               |PO only                    |
|To Be Set Apart Or Ordained|Queue organized by who sets apart              |SP, HC, Bishoprics         |
|Calling To Be Recorded     |Final recording in LCR                         |Clerks, PO                 |
|**Edit Users - List**      |Search and manage all users                    |PO (with Can Edit App Data)|
|**Edit Users - Detail**    |Edit individual user settings/permissions      |PO (with Can Edit App Data)|
|**Share This App**         |QR code and links to share app                 |All users                  |