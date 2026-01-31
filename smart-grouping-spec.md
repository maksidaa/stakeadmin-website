# Smart Grouping for Unit Sustaining Presentation

**Status**: Specification
**Priority**: High (solves real pain point from Glide app)
**Last Updated**: January 2025

---

## Problem Statement

When a bishopric reads names for sustaining/releasing in sacrament meeting, the items are currently displayed in submission order (or alphabetically), which creates awkward reading situations:

**Current behavior (bad):**
```
RELEASES:
• Maria Garcia - 2nd Counselor, Relief Society
• John Smith - Sunday School Teacher
• Susan Brown - Relief Society President
• Tom Wilson - Primary Teacher
• Lisa Chen - 1st Counselor, Relief Society
• Amy Johnson - Relief Society Secretary
```

The bishopric member has to mentally reorganize or read names in a jumbled order that doesn't make ecclesiastical sense.

**Desired behavior:**
```
RELEASES:

RELIEF SOCIETY PRESIDENCY:
• Susan Brown - Relief Society President
• Lisa Chen - 1st Counselor
• Maria Garcia - 2nd Counselor
• Amy Johnson - Secretary

OTHER RELEASES:
• John Smith - Sunday School Teacher
• Tom Wilson - Primary Teacher
```

---

## Solution: Smart Grouping + Manual Override

### Phase 1: Automatic Smart Grouping

Parse calling text to detect:
1. **Organization** (Relief Society, Elders Quorum, Primary, etc.)
2. **Position hierarchy** (President → 1st Counselor → 2nd Counselor → Secretary → Other)

Group items by organization, sort by hierarchy within each group.

### Phase 2: Manual Override

Allow PO members to manually reorder items in Pre-Sustaining Bucket before sending to unit, for cases where:
- Calling names don't parse correctly
- Specific presentation order is desired
- Edge cases the algorithm doesn't handle

---

## Organization Detection

### Known Organizations (Stake-Level Tracking)

```typescript
const ORGANIZATIONS = [
  // Stake auxiliaries
  'relief society',
  'elders quorum',
  'young women',
  'young men',
  'primary',
  'sunday school',

  // Stake leadership
  'stake presidency',
  'high council',
  'stake clerk',

  // Ward leadership
  'bishopric',
  'ward clerk',
  'executive secretary',

  // Other common
  'temple',
  'family history',
  'missionary',
  'seminary',
  'institute',
];
```

### Detection Logic

```typescript
function detectOrganization(callingText: string): string {
  const normalized = callingText.toLowerCase();

  for (const org of ORGANIZATIONS) {
    if (normalized.includes(org)) {
      return toTitleCase(org);
    }
  }

  return 'Other';
}
```

---

## Hierarchy Detection

### Position Levels

```typescript
const HIERARCHY_PATTERNS = [
  { pattern: /\bpresident\b/i, level: 1 },
  { pattern: /\b(1st|first)\s+counselor\b/i, level: 2 },
  { pattern: /\b(2nd|second)\s+counselor\b/i, level: 3 },
  { pattern: /\bsecretary\b/i, level: 4 },
  { pattern: /\bclerk\b/i, level: 5 },
  { pattern: /\b(teacher|instructor|leader|advisor)\b/i, level: 10 },
  // Default: level 99
];
```

### Detection Logic

```typescript
function detectHierarchyLevel(callingText: string): number {
  for (const { pattern, level } of HIERARCHY_PATTERNS) {
    if (pattern.test(callingText)) {
      return level;
    }
  }
  return 99; // Unknown/other
}
```

---

## Data Model Changes

### Option A: Computed at Display Time (Recommended for Phase 1)

No database changes. Grouping is computed when the unit sustaining screen loads.

**Pros**: No migration, works with existing data
**Cons**: Can't persist manual overrides without additional work

### Option B: Persist Presentation Order (Required for Manual Override)

```sql
-- Add to calling_submissions
presentation_group TEXT,           -- Detected or manually set organization
presentation_order INTEGER,        -- Order within group (null = auto-sort)
```

Or create a separate table:

```sql
CREATE TABLE unit_sustaining_order (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  submission_id UUID REFERENCES calling_submissions(id) ON DELETE CASCADE,
  unit_id UUID REFERENCES units(id),
  presentation_group TEXT,
  presentation_order INTEGER,
  manually_set BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(submission_id, unit_id)
);
```

---

## Grouping Algorithm

```typescript
interface PresentationGroup {
  organization: string;
  items: PresentationItem[];
}

interface PresentationItem {
  id: string;
  personName: string;
  callingText: string;
  hierarchyLevel: number;
  manualOrder?: number;
}

function groupForPresentation(
  submissions: CallingSubmission[],
  type: 'release' | 'sustaining'
): PresentationGroup[] {
  const groups = new Map<string, PresentationItem[]>();

  for (const sub of submissions) {
    // Use current_calling_text for releases, new_calling_text for sustainings
    const callingText = type === 'release'
      ? sub.current_calling_text
      : sub.new_calling_text;

    const org = detectOrganization(callingText);
    const level = detectHierarchyLevel(callingText);

    if (!groups.has(org)) {
      groups.set(org, []);
    }

    groups.get(org)!.push({
      id: sub.id,
      personName: sub.person_name,
      callingText: callingText,
      hierarchyLevel: level,
      manualOrder: sub.presentation_order ?? undefined,
    });
  }

  // Sort within each group
  for (const items of groups.values()) {
    items.sort((a, b) => {
      // Manual order takes precedence
      if (a.manualOrder !== undefined && b.manualOrder !== undefined) {
        return a.manualOrder - b.manualOrder;
      }
      if (a.manualOrder !== undefined) return -1;
      if (b.manualOrder !== undefined) return 1;

      // Then by hierarchy level
      if (a.hierarchyLevel !== b.hierarchyLevel) {
        return a.hierarchyLevel - b.hierarchyLevel;
      }

      // Then alphabetically
      return a.personName.localeCompare(b.personName);
    });
  }

  // Sort groups: specific orgs first, "Other" last
  const sortedOrgs = [...groups.keys()].sort((a, b) => {
    if (a === 'Other') return 1;
    if (b === 'Other') return -1;
    return a.localeCompare(b);
  });

  return sortedOrgs.map(org => ({
    organization: org,
    items: groups.get(org)!,
  }));
}
```

---

## UI Components

### Unit Sustaining Screen (Read Mode)

```
┌─────────────────────────────────────────────────────────────┐
│              BASTROP WARD - January 19                      │
├─────────────────────────────────────────────────────────────┤
│  STAKE BUSINESS — RELEASES                                  │
│                                                             │
│  "These individuals have been released..."                  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ RELIEF SOCIETY PRESIDENCY                           │    │
│  ├─────────────────────────────────────────────────────┤    │
│  │ Susan Brown          Relief Society President    ⋯  │    │
│  │ Lisa Chen            1st Counselor               ⋯  │    │
│  │ Maria Garcia         2nd Counselor               ⋯  │    │
│  │ Amy Johnson          Secretary                   ⋯  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ OTHER                                               │    │
│  ├─────────────────────────────────────────────────────┤    │
│  │ John Smith           Sunday School Teacher       ⋯  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│              [ Mark All Released ]                          │
└─────────────────────────────────────────────────────────────┘
```

### Pre-Sustaining Bucket (Edit Mode - Manual Override)

```
┌─────────────────────────────────────────────────────────────┐
│  RELEASES READY TO SEND                        [Send All]   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─ RELIEF SOCIETY PRESIDENCY ─────────────────────────┐    │
│  │ [≡] Susan Brown - Relief Society President          │    │
│  │ [≡] Lisa Chen - 1st Counselor                       │    │
│  │ [≡] Maria Garcia - 2nd Counselor                    │    │
│  │ [≡] Amy Johnson - Secretary                         │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  ┌─ OTHER ─────────────────────────────────────────────┐    │
│  │ [≡] John Smith - Sunday School Teacher              │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  [≡] = Drag handle for manual reordering                    │
│                                                             │
│  Drag items between groups or reorder within groups.        │
│  Smart grouping detected these organizations automatically. │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Edge Cases

### 1. Unrecognized Organization
**Scenario**: Calling text is "Addiction Recovery Facilitator"
**Behavior**: Falls into "Other" group
**Manual fix**: User can drag into a custom group or leave in Other

### 2. Multiple Organizations in One Calling
**Scenario**: "Young Women Camp Director" (is it YW or a special calling?)
**Behavior**: First match wins (Young Women)
**Future**: Could allow manual organization override

### 3. Stake-Wide Sustainings Across Multiple Units
**Scenario**: Stake RS President being sustained in all 9 units
**Behavior**: Same grouping logic applies to each unit's view
**Consistency**: Grouping should be identical across all units

### 4. Mixed Releases and New Callings for Same Person
**Scenario**: Richard Crow released as Exec Sec, called as EQ President
**Behavior**: Release appears in releases section, calling in sustainings section
**Grouping**: Release grouped with "Bishopric/Ward Clerk" area, calling with "Elders Quorum"

### 5. Partial Presidency Changes
**Scenario**: Only releasing 1st Counselor (not full presidency)
**Behavior**: Still groups under organization, just shows single item
**Display**: "RELIEF SOCIETY PRESIDENCY" header with only one person is fine

---

## Implementation Phases

### Phase 1: Smart Grouping (MVP)
- Implement `detectOrganization()` and `detectHierarchyLevel()`
- Apply grouping in Unit Sustaining screen
- No database changes
- No manual override yet

### Phase 2: Manual Override
- Add `presentation_order` field or separate table
- Add drag-drop UI in Pre-Sustaining Bucket
- Persist manual ordering when sent to unit

### Phase 3: Enhancements
- Learn from manual overrides to improve detection
- Custom organization labels
- Batch "Mark All Released" per group
- Print-friendly view for bishopric

---

## Testing Scenarios

1. Full presidency release (4 people, same org) → Should group together in hierarchy order
2. Mixed releases (3 orgs, 1-2 people each) → Should create 3 groups
3. Single release → Should work without grouping header (or minimal header)
4. Unknown calling text → Falls to "Other"
5. Manual reorder → Persists after page refresh
6. Stake-wide sustaining → Same grouping in all 9 units
