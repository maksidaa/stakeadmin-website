# Spotlight Tile Integration Guide

This document describes how to add tiles to the Spotlight section with inline dropdown functionality that matches the Center Stage tile behavior.

## What This Pattern Does

When a Spotlight card is tapped:
1. It expands inline (dropdown) - NO navigation away
2. Shows the embedded list component (same as Center Stage tile)
3. Optionally shows detail view inline with a back button
4. Actions refresh the data automatically

## Architecture Overview

```
Spotlight Section
├── Greeting ("Good afternoon, Adam")
├── Pending Users Card (expandable) → PendingUsersListContent
├── HC Pending Card (expandable) → HCReviewListContent
├── HC Vote Card (expandable) → HCReviewListContent
├── To Assign Card (expandable) → ReadyForAssignmentListContent
├── SP Review Card (expandable) → SPInitialReviewListContent
├── Other Cards (meetings, etc.)
└── Debug Summary
```

## Files to Modify

| File | Purpose |
|------|---------|
| `src/api/spotlight.api.ts` | Add items to Spotlight data |
| `src/components/spotlight/SpotlightSection.tsx` | Render expandable cards with inline content |
| `app/(app)/tracker/list/[your-list].tsx` | List component (needs embedded mode support) |
| `app/(app)/tracker/detail/[your-detail].tsx` | Detail component (optional, needs `onDone` callback) |

---

## Currently Implemented Tiles

| Tile Name | Status | Subtitle Keyword | List Component | Detail Component | Who Sees It |
|-----------|--------|------------------|----------------|------------------|-------------|
| SP Review | `sp_initial_review`, `on_hold` | "Awaiting SP review" | `SPInitialReviewListContent` | `SPInitialReviewScreen` | Stake Presidency only |
| HC Vote | `sp_sustaining`, `hc_review` | "Vote needed" | `HCReviewListContent` | `HCReviewDetailScreen` | SP + HC |
| HC Pending | `hc_review` | "HC Pending" | `HCReviewListContent` | `HCReviewDetailScreen` | Exec Sec only |
| To Assign | `ready_for_assignment` | "To Assign" | `ReadyForAssignmentListContent` | `ReadyForAssignmentDetailScreen` | PO users |
| Pending Users | N/A | "users awaiting approval" | `PendingUsersListContent` | None (inline actions) | PO users |
| My Assignments | `assigned` | "My Assignment" | `AssignedListContent` | None (inline actions + nav) | HC members |
| Action Items | N/A | "Action Item" | `ActionsListContent` | None (inline actions) | All users |
| SP Sustaining | `sp_sustaining` | "SP Sustaining" | `SPSustainingListContent` | None (inline actions) | Stake Presidency |
| Ready for HC | `sp_sustaining` (complete) | "Ready for HC" | `ReadyForHCListContent` | None (inline actions) | Exec Sec, PO |
| Pre-Sustain | `pre_sustaining_bucket` | N/A (type-based) | `PreSustainingListContent` | None (inline actions) | Exec Sec, PO |
| Interviews Due | `interview` | "Interview Due" | `InterviewsListContent` | None (inline actions) | SP, HC, SC |
| To Record | `to_record` | N/A (type-based) | `ToBeRecordedListContent` | None (inline actions) | Clerk, Exec Sec |

---

## Step-by-Step: Adding a New Tile

### Step 1: Ensure List Component Supports Embedded Mode

Your list component needs these props:

```typescript
type YourListContentProps = {
  embedded?: boolean;
  allowNavigation?: boolean;
  onSelectSubmission?: (submissionId: string) => void;  // For inline detail view
  onDataChange?: () => void;  // Refresh parent when actions complete
  showSectionHeaders?: boolean;
};
```

Make the card tappable when `onSelectSubmission` is provided:

```typescript
<TouchableOpacity
  style={styles.cardContent}
  activeOpacity={onSelectSubmission ? 0.7 : 1}
  onPress={() => {
    if (onSelectSubmission) {
      onSelectSubmission(item.id);
    }
  }}
  disabled={!onSelectSubmission}
>
  {/* Card content */}
</TouchableOpacity>
```

Add `stopPropagation` to action buttons so they don't trigger card selection:

```typescript
<TouchableOpacity
  onPress={(e) => {
    e.stopPropagation?.();
    handleAction();
  }}
>
```

### Step 2: Ensure Detail Component Has `onDone` Callback (if needed)

```typescript
type YourDetailScreenProps = {
  submissionId?: string;
  onDone?: () => void;
};

export default function YourDetailScreen({ submissionId, onDone }: Props) {
  const id = submissionId || params.id;  // Support both prop and route param

  const handleDone = useCallback(() => {
    if (onDone) {
      onDone();
    } else {
      router.back();
    }
  }, [onDone, router]);

  // Call handleDone() after save/action completes
}
```

### Step 3: Add Items to Spotlight API

In `src/api/spotlight.api.ts`, inside `getSpotlightItems()`:

```typescript
// ========================================
// YOUR TILE NAME (status description)
// ========================================
if (userCanSeeThis) {
  const { data: yourItems } = await supabase
    .from('calling_submissions')
    .select(`
      id,
      person_name,
      submission_type,
      new_calling_text,
      current_calling_text,
      created_at,
      linked_submission_id,
      unit:units!calling_submissions_unit_id_fkey(name)
    `)
    .eq('stake_id', stakeId)
    .eq('status', 'your_status')
    .order('created_at', { ascending: true });

  // Filter out linked releases to count linked pairs as 1 item
  const filtered = (yourItems || []).filter(item =>
    !(item.linked_submission_id && item.submission_type === 'release')
  );

  for (const item of filtered) {
    const createdAt = new Date(item.created_at || now);
    const daysOld = daysBetween(createdAt, now);
    const tier: SpotlightTier = daysOld > 5 ? 'critical' : 'high';

    if (isSnoozed('calling_submission', item.id)) continue;

    allItems.push({
      id: item.id,
      type: 'calling_submission',
      phase: null,
      tier,
      effectiveDate: createdAt,
      title: item.submission_type === 'release'
        ? `Release: ${item.current_calling_text || ''}`
        : `Calling: ${item.new_calling_text || ''}`,
      subtitle: `${item.person_name} · Your Keyword`,  // IMPORTANT: This keyword is used for grouping
      actions: [...],
      data: item,
      indicator: daysOld > 5 ? 'red' : 'orange',
      quickWin: true,
      daysOld: Math.floor(daysOld),
    });
  }
}
```

**IMPORTANT**: The `subtitle` must contain a unique keyword that will be used for grouping (e.g., "HC Pending", "To Assign", "Vote needed").

### Step 4: Add Grouping in SpotlightSection

In `src/components/spotlight/SpotlightSection.tsx`, in `groupSimilarItems()`:

```typescript
// Group your items together
else if (item.type === 'calling_submission' && item.subtitle?.includes('Your Keyword')) {
  const key = `your_key_${item.tier}`;
  if (!grouped.has(key)) grouped.set(key, []);
  grouped.get(key)!.push(item);
}
```

Then in the grouped items conversion:

```typescript
else if (key.startsWith('your_key')) {
  result.push({
    type: 'grouped',
    tier,
    items: groupItems,
    groupTitle: count === 1 ? '1 your item' : `${count} your items`,
    groupSubtitle: 'Your description',
    groupRoute: '/your/fallback/route',  // Only used if expand fails
    groupIcon: 'your-icon-name',
  });
}
```

### Step 5: Add State for Detail Mode (if needed)

```typescript
// State for inline detail views within Your Tile dropdown
const [yourTileMode, setYourTileMode] = useState<'list' | 'detail'>('list');
const [selectedYourItemId, setSelectedYourItemId] = useState<string | null>(null);
```

### Step 6: Add Import for Components

```typescript
import { YourListContent } from '../../../app/(app)/tracker/list/your-list';
import YourDetailScreen from '../../../app/(app)/tracker/detail/your-detail';
```

### Step 7: Add Detection in `renderGroupedItem`

```typescript
const isYourGroup = group.groupTitle?.includes('your keyword');
const canExpandInline = isVotesGroup || isHCPendingGroup || isToAssignGroup || isPendingUsersGroup || isYourGroup;
```

### Step 8: Add to handleGroupAction

```typescript
} else if (isYourGroup) {
  // Reset to list mode when collapsing
  if (isExpanded) {
    setYourTileMode('list');
    setSelectedYourItemId(null);
  }
  toggleExpand(groupKey);
}
```

### Step 9: Add Expanded Content

```typescript
{/* Expanded inline Your Tile */}
{isExpanded && isYourGroup && (
  <View style={styles.expandedContainer}>
    {yourTileMode === 'list' && (
      <YourListContent
        embedded
        showSectionHeaders={false}
        allowNavigation={false}
        onSelectSubmission={(id) => {
          setSelectedYourItemId(id);
          setYourTileMode('detail');
        }}
        onDataChange={() => {
          loadSpotlight();
          onDataChange?.();
        }}
      />
    )}
    {yourTileMode === 'detail' && selectedYourItemId && (
      <View>
        <Pressable
          style={styles.backButton}
          onPress={() => {
            setYourTileMode('list');
            setSelectedYourItemId(null);
          }}
        >
          <Ionicons name="arrow-back" size={18} color={colors.primary} />
          <Text style={[styles.backButtonText, { color: colors.primary }]}>Back to list</Text>
        </Pressable>
        <YourDetailScreen
          submissionId={selectedYourItemId}
          onDone={() => {
            setYourTileMode('list');
            setSelectedYourItemId(null);
            loadSpotlight();
            onDataChange?.();
          }}
        />
      </View>
    )}
  </View>
)}
```

---

## Common Gotchas

1. **Card doesn't expand**: Check that your detection condition matches the groupTitle exactly
2. **Navigates away instead of expanding**: Make sure the expand condition is checked BEFORE the `group.groupRoute` fallback
3. **Detail view doesn't show**: Verify state is being set and the conditional render checks both mode AND selected ID
4. **Actions don't refresh parent**: Ensure `onDataChange` is called after actions complete
5. **Buttons trigger card selection**: Add `e.stopPropagation?.()` to button `onPress` handlers
6. **Count mismatch with dashboard tile**: Filter out linked releases so linked pairs count as 1 item
7. **Wrong users see the tile**: Check permission conditions in spotlight.api.ts (e.g., `isPOUser`, `isExecSecUser`, `canDoSPReview`)

---

## Complete Tile List (Dashboard → Spotlight)

All tiles from the upper dashboard tile rows, with Spotlight implementation status:

| Tile ID | Label | Who Sees | Status Field | Spotlight Done? |
|---------|-------|----------|--------------|-----------------|
| `pending_users` | Pending Users | PO users | N/A | ✅ YES |
| `sp_review` | SP Review | Stake Presidency | `sp_initial_review`, `on_hold` | ✅ YES |
| `sp_sustaining` | SP Sustaining | Stake Presidency | `sp_sustaining` | ✅ YES |
| `hc_sustainings` | HC Sustainings | High Council | `hc_review` (needs their vote) | ✅ YES (as "HC Vote") |
| `hc_pending` | HC Pending | Exec Sec | `hc_review` (tracking) | ✅ YES |
| `ready_for_assignment` | To Assign | Exec Sec, PO | `ready_for_assignment` | ✅ YES |
| `ready_for_hc` | Ready for HC | Exec Sec, PO | `sp_sustaining` complete | ✅ YES |
| `pre_sustaining_bucket` | Pre-Sustain | Exec Sec, PO | `pre_sustaining_bucket` | ✅ YES |
| `my_assignments` | Assignments | HC members | `assigned` to user | ✅ YES |
| `action_items` | Action Items | All users | Custom action items | ✅ YES |
| `interviews` | Interviews Due | SP, HC, SC | Overdue interviews | ✅ YES |
| `to_be_recorded` | To Record | Stake Clerk, Exec Sec | Various completed statuses | ✅ YES |

### Implementation Priority

**High Priority (common workflows):**
1. `my_assignments` - HC members need to see items assigned to them to extend
2. `action_items` - All users see their tasks

**Medium Priority:**
3. `sp_sustaining` - SP voting
4. `ready_for_hc` - Items waiting to go to HC
5. `pre_sustaining_bucket` - Pre-sustain items

**Lower Priority:**
6. `interviews` - Interview tracking
7. `to_be_recorded` - Clerk recording

---

## Debug Mode

Currently enabled for development:
- Shows tier letter (C/H/M/L) instead of icon
- Shows `[tier]` in subtitle
- Shows `[DEBUG] X total` summary
- No display limit (shows all items)

To disable debug mode, revert these changes in SpotlightSection.tsx:
- Restore icons in grouped cards
- Remove `[tier]` prefix from subtitles
- Restore `getSummaryText()` to show "X more items can wait"
- Restore `displayItems.slice(0, limit)` in spotlight.api.ts
