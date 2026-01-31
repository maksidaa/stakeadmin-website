# Ministering Interviews — Feature Specification

**Version:** 1.0  
**Last Updated:** January 13, 2026  
**Status:** Ready for Development

---

## Overview

Tracks leadership interviews conducted by stake presidency members, auxiliary presidents, and other leaders with those they oversee. The system monitors interview frequency, flags overdue interviews, and provides a dashboard for stake council visibility.

### Purpose

- Ensure all leaders receive regular stewardship interviews
- Surface overdue interviews so they don't fall through the cracks
- Provide stake-wide visibility into interview completion rates
- Track interview history over time

### Access

- **Who can see:** All Stake Council members (SP, HC, Auxiliary Presidents, Clerks)
- **Who cannot see:** Bishops, unit auxiliary leaders, regular members
- **Who can add/edit assignments:** PO only
- **Who can record interviews:** All SC members

**See:** `02-permissions.md` for complete permission details.

---

## Data Model

**See:** `01-data-model.md` for table definitions.

**Tables used:**
- `interview_assignments` — Position-based assignment definitions
- `interviews` — Log of actual interviews conducted

**Key design decision:** Assignments are tied to **positions** (e.g., "1 Bastrop Ward EQP"), not people. When someone is released and a new person called, the assignment stays the same.

---

## Interview Frequency Rules

| Frequency | Coming Due | Overdue | Use Case |
|-----------|------------|---------|----------|
| **Monthly** | 21+ days | 30+ days | SP counselors, Bishops, Org Presidents |
| **Quarterly** | 70+ days | 90+ days | HC members, EQPs, Bishopric counselors |
| **Bi-annual** | 150+ days | 180+ days | Patriarch Scribe |
| **As needed** | Not tracked | Not tracked | YW Camp Director/Assistant |

### Who Gets Monthly Interviews

- Stake Presidency Counselors (from SP)
- All Bishops/Branch Presidents (from SP)
- Stake RS/YW/Primary Presidents (from SP or SP counselor)
- HC members serving as Organization Presidents (e.g., HC#4 as Stake YM President)

---

## Complete Assignment List

### Interviewed by Area 70
| Position | Frequency |
|----------|-----------|
| Stake President | Quarterly |

### Interviewed by Stake President
| Position | Frequency |
|----------|-----------|
| SP 1st Counselor | Monthly |
| SP 2nd Counselor | Monthly |
| Stake RS President | Monthly |
| Stake Clerk | Quarterly |
| Stake Executive Secretary | Quarterly |
| Stake Patriarch | Quarterly |
| Patriarch Scribe | Bi-annual |
| All Bishops (9 units) | Monthly |

### Interviewed by SP 1st Counselor
| Position | Frequency |
|----------|-----------|
| HC#3, HC#7, HC#8, HC#10, HC#11 | Quarterly |
| Stake Asst Executive Secretary 1 | Quarterly |
| Units 2, 3, 6, 8, 9: EQP | Quarterly |
| Units 2, 3, 6, 8, 9: Bishopric 1st & 2nd Counselors | Quarterly |

### Interviewed by SP 2nd Counselor
| Position | Frequency |
|----------|-----------|
| HC#1, HC#2, HC#4, HC#5, HC#9, HC#12 | Quarterly |
| HC#4 (as YM President) | Monthly |
| Stake Asst Executive Secretary 2 | Quarterly |
| Stake YW President | Monthly |
| Stake Primary President | Monthly |
| Units 1, 4, 7, 10: EQP | Quarterly |
| Units 1, 4, 7, 10: Bishopric 1st & 2nd Counselors | Quarterly |

### Interviewed by Stake Clerk
| Position | Frequency |
|----------|-----------|
| Stake Assistant Clerk -- Finance | Quarterly |

### Interviewed by Auxiliary Presidents
| Interviewer | Interviewees | Frequency |
|-------------|--------------|-----------|
| Stake RS President | RS 1st/2nd Counselor, Secretary, Activity Leader | Quarterly |
| Stake YW President | YW 1st/2nd Counselor, Secretary | Quarterly |
| Stake YW President | YW Camp Director, Camp Assistant | As needed |
| Stake Primary President | Primary 1st/2nd Counselor, Secretary | Quarterly |
| HC#4 (YM President) | YM 1st/2nd Counselor, Secretary | Quarterly |
| HC#10 | SS 1st/2nd Counselor, Secretary | Quarterly |

---

## Screens

### 1. Dashboard (Main Screen)

**Entry:** Stake Council tab → "SC Ministering Interviews" button

**Header:** "Ministering Interviews"

**Components:**

```
┌─────────────────────────────────────────────────┐
│ Percent Of Interviews Up to Date    49.4%       │
│ ████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░        │
├─────────────────────────────────────────────────┤
│ [View All Interviews]  [+ Add Interview] (PO)   │
├─────────────────────────────────────────────────┤
│ Overdue Quarterly Interviews            🔽      │
│ ┌─────────────────────────────────────────────┐ │
│ │ 7 Lockhart Branch Bishopric 2nd Coun...     │ │
│ │ 🔴 196.64 days                          ⋯   │ │
│ └─────────────────────────────────────────────┘ │
│ [Search]                              [Filter]  │
├─────────────────────────────────────────────────┤
│ Overdue Monthly Interviews              🔽      │
├─────────────────────────────────────────────────┤
│ Coming Due                              🔽      │
│ QUARTERLY                                       │
│ 1 Bastrop Ward Bishopric 1st Counselor         │
│ 🟡 86.15 days                                  │
└─────────────────────────────────────────────────┘
```

**Status Indicators:**
- 🔴 Red = Overdue
- 🟡 Yellow = Coming Due
- 🟢 Green = Current (not shown on dashboard, only in grid)

**⋯ Menu:** Edit, Delete (PO only for delete)

### 2. Filter Panel

Bottom sheet with multi-select checkboxes:

**"To Be Interviewed By"**
- Area 70
- Stake President Danny German
- Stake Presidency 1st Counselor David Allen
- Stake Presidency 2nd Counselor Adam Goodwin
- Stake Clerk Kevin Hodges
- Stake RS President Tina Roudebush
- Stake YW President Jessica Humrich
- Stake Primary President Leslie Green
- HC#4 Chad Moss
- HC#10 Nick D'Alessio

**Footer:** [Clear all] [Done]

### 3. All Interviews Grid (Desktop)

Spreadsheet-style view showing all assignments with monthly completion.

| WHO TO BE INTERVIEWED | TO BE INTERVIEWED BY | FREQ | J | F | M | A | M | J | J | A | S | O | N | D |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| Stake President | Area 70 | Q | ✗ | ✗ | ✗ | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| SP 1st Counselor | Stake President | M | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |

**Features:**
- Search bar
- Filter dropdown
- "+ Add" button (PO only)
- Click row → opens detail

### 4. Interview Detail Screen

```
┌─────────────────────────────────────────────────┐
│ ←                                    [Edit]     │
├─────────────────────────────────────────────────┤
│ To Be Interviewed By                            │
│ Stake Presidency 2nd Counselor Adam Goodwin     │
├─────────────────────────────────────────────────┤
│ Most Recent Interview Date     July 1, 2025    │
│ Days Since                     🔴 196.64 days  │
│ Monthly or Quarterly?          Quarterly        │
├─────────────────────────────────────────────────┤
│ January                                    ✗    │
│ February                                   ✗    │
│ March                                      ✗    │
│ ...                                             │
│ December                                   ✗    │
├─────────────────────────────────────────────────┤
│ Do Not Show On Coming Due or Overdue List  ○   │
├─────────────────────────────────────────────────┤
│           [✓ Record Interview]                  │
├─────────────────────────────────────────────────┤
│ Notes                                           │
│ ┌─────────────────────────────────────────────┐ │
│ │ Write a note...                             │ │
│ └─────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────┤
│              [🗑 Delete] (PO only)              │
└─────────────────────────────────────────────────┘
```

### 5. Record Interview

**Tap "Record Interview"** → Bottom sheet:

- **Date:** [Date picker, defaults to today, can backdate]
- **Notes:** [Optional text field]
- **[Save]** button

On save:
- Creates interview record
- Updates dashboard/grid
- Shows success toast

---

## Business Logic

### Status Calculation

```typescript
type Frequency = 'monthly' | 'quarterly' | 'biannual' | 'as_needed';
type Status = 'overdue' | 'coming_due' | 'current' | 'never' | 'not_tracked' | 'excluded';

const thresholds = {
  monthly: { overdue: 30, comingDue: 21 },
  quarterly: { overdue: 90, comingDue: 70 },
  biannual: { overdue: 180, comingDue: 150 }
};

function calculateStatus(frequency: Frequency, daysSince: number | null, excluded: boolean): Status {
  if (frequency === 'as_needed') return 'not_tracked';
  if (excluded) return 'excluded';
  if (daysSince === null) return 'never';
  
  const t = thresholds[frequency];
  if (daysSince > t.overdue) return 'overdue';
  if (daysSince > t.comingDue) return 'coming_due';
  return 'current';
}
```

### Percent Up to Date

```typescript
function calculatePercentUpToDate(assignments: Assignment[]): number {
  const tracked = assignments.filter(a => 
    !['not_tracked', 'excluded'].includes(a.status)
  );
  const current = tracked.filter(a => a.status === 'current');
  return (current.length / tracked.length) * 100;
}
```

### Monthly Grid

Determine which months have interviews for the current year:

```typescript
function getMonthlyGrid(interviews: Interview[], year: number): boolean[] {
  return Array.from({ length: 12 }, (_, i) => 
    interviews.some(int => {
      const d = new Date(int.interview_date);
      return d.getFullYear() === year && d.getMonth() === i;
    })
  );
}
```

---

## API Endpoints

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| GET | `/stakes/:id/interview-assignments` | List all with status | SC |
| GET | `/interview-assignments/:id` | Single with history | SC |
| POST | `/interview-assignments` | Create assignment | PO |
| PUT | `/interview-assignments/:id` | Update assignment | PO |
| DELETE | `/interview-assignments/:id` | Delete assignment | PO |
| POST | `/interview-assignments/:id/interviews` | Record interview | SC |
| GET | `/interview-assignments/:id/interviews` | Get history | SC |
| PUT | `/interviews/:id` | Edit interview | SC |
| DELETE | `/interviews/:id` | Delete interview | PO |

### Query Parameters for List

- `interviewer` — Filter by interviewer position
- `status` — Filter: overdue, coming_due, current
- `frequency` — Filter: monthly, quarterly, biannual

### Response Shape

```json
{
  "assignments": [...],
  "stats": {
    "total": 81,
    "current": 40,
    "coming_due": 15,
    "overdue": 26,
    "percent_up_to_date": 49.38
  }
}
```

---

## UX Improvements (vs Glide)

1. **Color-coded badges** instead of emoji in text strings
2. **Quick filter chips:** "My Interviews" | "All" | "Overdue Only"
3. **Interview history timeline** on detail screen
4. **Bulk record option** for post-meeting batch recording
5. **Smart default sort:** Most overdue first
6. **Searchable** across all position names

---

## Migration from Glide

| Glide Column | New Location |
|--------------|--------------|
| `Who To Be Interviewed` | `interview_assignments.interviewee_position` |
| `To Be Interviewed By` | `interview_assignments.interviewer_position` |
| `Monthly or Quarterly Interview?` | `interview_assignments.frequency` |
| `Most Recent Interview Date` | Create `interviews` record |
| `Notes` | `interview_assignments.notes` |
| `January` - `December` | Derived from `interviews` records |

### Migration Steps

1. Create `interview_assignments` for each unique position
2. For rows with `Most Recent Interview Date`, create `interviews` record
3. Optionally: Create historical interview records from monthly checkmarks
