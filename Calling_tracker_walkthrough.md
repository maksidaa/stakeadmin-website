# Calling Tracker Documentation

**Last Updated:** January 8, 2026  
**Status:** In Progress — Walkthrough Partially Complete

-----

## Overview

The Calling Tracker is the heartbeat of stake administration. It manages the complete lifecycle of:

- **Callings** — new calling requests
- **Releases** — releasing someone from a current calling
- **Combined Calling/Release** — simultaneous release from old + calling to new
- **Ordinations** — Melchizedek Priesthood ordinations (not yet documented)

Each submission flows through a multi-step workflow with role-based visibility and actions.

-----

## Workflow Stages

Based on the Tracker Home screen, submissions move through these stages:

### Stage 1: Initial Review By Stake Presidency

- New submissions land here
- SP member reviews, adds notes, configures settings
- Assigns who will extend the calling/release
- SP sustains (all 3 members tap to sustain)
- Advances to HC review

### Stage 2: To Be Sustained By Stake Presidency

- (Appears to be a separate queue — need clarification on when this is used vs Initial Review)

### Stage 3: To Be Reviewed By High Council

- All 12 HC members see the submission
- Each taps to sustain individually
- Progress bar shows count (e.g., 5/12)
- Once an HC member sustains, item disappears from their personal list
- When all 12 sustain, advances to next stage

### Stage 4: Ready For Assignment

- HC sustaining complete
- Ready for calling to be extended
- (Further steps not yet documented — extension, acceptance, set apart, recording)

### Stake Presidency Assignments

- Separate queues showing items assigned to each SP member
- Stake President, 1st Counselor, 2nd Counselor each have their own count

-----

## Calling Tracker Home Screen

**Header:** “Calling and Release Tracker”

**Components:**

1. **Search All Submissions** — button at top
1. **Workflow Buckets** — cards showing counts at each stage
1. **SP Assignments** — individual queues per SP member
1. **HC Assignments** — (assumed, not yet shown)

-----

## Submission Detail Screen (SP View)

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

### SP-Only Fields

|Field                   |Description                                             |
|------------------------|--------------------------------------------------------|
|Stake Presidency Note   |Text area for SP internal notes                         |
|Assign Calling to       |Dropdown — which SP/HC member extends the calling       |
|Assign Release to       |Dropdown — which SP/HC member extends the release       |
|Assign Denied Request To|Dropdown — who handles if declined (needs clarification)|

### Sustaining Location

|Field       |Options                                            |
|------------|---------------------------------------------------|
|Sustain in….|“All Stake Units” (stake callings) or specific unit|
|Release In….|“All Stake Units” or specific unit                 |

### Configuration Toggles

|Toggle                              |Purpose                                     |
|------------------------------------|--------------------------------------------|
|Allow Bishop to Track?              |Gives bishop visibility into this submission|
|To Be Set Apart By Stake Presidency?|SP member will set apart                    |
|Can Be Set Apart By High Councilor? |HC member authorized to set apart           |
|Place on hold                       |Pauses the workflow                         |

### SP Sustaining

Three buttons with hand icons:

- 👋 Stake President
- 👋 1st Counselor
- 👋 2nd Counselor

Each SP member taps their button to sustain. Once all 3 are tapped, the “Send to High Council for Sustaining” button appears/activates.

**Improvement Opportunity:** The SP member who does the initial review should have their button auto-tapped so they don’t have to do it separately.

### Actions

- **“Send to the High Council for Sustaining”** — advances workflow (appears after all 3 SP sustain)
- **“Delete Request”** — removes submission

-----

## HC Sustaining View

When an HC member opens a submission from “To Be Reviewed”:

### Simplified View (compared to SP)

Shows only:

- Basic submission info (type, person, unit, calling, notes)
- **High Council Notes** — text area for HC notes
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

|Action          |Who Can Use                                     |
|----------------|------------------------------------------------|
|“Set All To Yes”|PO only (SP, SP1, SP2, Exec Sec, Asst Exec Secs)|

This bulk action marks all 12 HC as sustained — used during HC meeting after live voice vote.

### Completion

When all 12 HC have sustained:

- **“Sustained By High Council”** button appears
- Progress bar shows 12/12
- Tapping advances to “Ready For Assignment”

-----

## Stake Sustainings View

Where ward members sustain stake callings.

### PO-Only Feature

|Feature                         |Who Can Use|
|--------------------------------|-----------|
|“View All Stake Business” button|PO only    |

Shows all stake business items across all units. Allows PO to:

- Review what’s pending sustaining across the stake
- Make edits/corrections
- See full picture vs single ward view

-----

## Permissions Summary

### PO (Presidency Operations) Members

- Stake President
- Stake Presidency 1st Counselor
- Stake Presidency 2nd Counselor
- Executive Secretary
- Assistant Executive Secretaries

**PO Capabilities:**

- View all submissions across all units
- “Set All To Yes” bulk HC sustaining
- “View All Stake Business”
- Configure all submission settings
- Assign submissions to others

### High Council Members

- Can view submissions in “To Be Reviewed By High Council” queue
- Can sustain only their own vote (not others)
- Can add High Council Notes
- Cannot use bulk actions
- Items disappear from their list once sustained

### Bishops / Ward Leaders

- “Allow Bishop to Track” toggle controls their visibility
- Details of what they see TBD

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
person_member_id (string, optional — link to church records?)
home_unit_id (foreign key → units)

# Calling details
calling_type (enum: "stake" | "ward")
requested_calling (string)
current_calling (string, optional)
request_notes (text)
bishop_is_aware (boolean)

# SP configuration
sp_notes (text)
assign_calling_to_user_id (foreign key → users)
assign_release_to_user_id (foreign key → users)
assign_denied_to_user_id (foreign key → users)
sustain_in (enum: "all_units" | "home_unit" | specific unit?)
release_in (enum: "all_units" | "home_unit" | specific unit?)
allow_bishop_to_track (boolean)
set_apart_by_sp (boolean)
can_set_apart_by_hc (boolean)
on_hold (boolean)

# SP sustaining
sp_sustained (boolean)
sp1_sustained (boolean)
sp2_sustained (boolean)
sp_sustaining_complete (boolean)

# HC sustaining
hc1_sustained (boolean)
hc2_sustained (boolean)
... through hc12_sustained (boolean)
hc_sustaining_complete (boolean)
hc_notes (text)

# Workflow status
status (enum: "sp_review" | "sp_sustaining" | "hc_review" | "ready_for_assignment" | ...)
```

-----

## Still Unknown / Needs Documentation

### Workflow

1. What happens after “Ready For Assignment”?
1. Extension flow — how is it tracked?
1. Acceptance/Decline handling — what triggers “Assign Denied Request To”?
1. Set Apart tracking
1. Recording/completion
1. Release-only submissions — different flow?

### Sustaining

1. Ward sustaining flow — who sustains ward callings and where?
1. “Sustain in… All Stake Units” — how does multi-ward sustaining actually work?

### Ordinations

1. Full MP ordination workflow (mentioned but not shown)

### Permissions

1. What does an HC member’s home screen look like?
1. What does a Bishop see with “Allow Bishop to Track” enabled?
1. Ward council member access?

### Screens

1. How are submissions created? (submission form)
1. Directory tab
1. Share This App tab
1. Edit Users tab

### Notifications

1. Who gets notified at each step?
1. What do notifications say?
1. How are they delivered currently (Glide)?

-----

## Design Improvement Opportunities

### Already Identified

1. **Auto-sustain for reviewer** — SP member reviewing should have their sustain button auto-tapped

### Potential Improvements (to discuss)

1. **Progress visualization** — Could show more clearly where a submission is in the overall flow
1. **Batch operations** — If multiple submissions for same person (calling + release), could they be linked more explicitly?
1. **Mobile optimization** — Long scrolling lists of 12 HC members could be condensed
1. **Async vs Sync sustaining** — Clearer distinction between “I’m reviewing between meetings” vs “we’re in the meeting now”
1. **Audit trail** — Who did what when? Currently implicit in the button states

-----

## Next Steps for Documentation

1. Continue walkthrough of remaining calling tracker screens
1. Document the submission creation flow
1. Document post-assignment workflow (extension → set apart → record)
1. Document ordination workflow
1. Capture notification triggers and content
1. Review other tabs (Directory, Share, Edit Users)