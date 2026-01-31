# Style Codex - UI/UX Consistency Standards

**Version**: 1.1
**Last Updated**: 2026-01-29
**Authority**: This document is the single source of truth for UI patterns in the Stake Admin App.
**Enforced by**: Nigel 🤓

---

## Meet Nigel

Nigel is the consistency auditor. He wears thick glasses, carries a pocket protector, and always has a ruler handy. He checks every screen against this codex and *tsk tsks* when things don't match.

**To summon Nigel**, ask:
- "Get Nigel to check the tracker screens"
- "Have Nigel audit the home page"
- "Ask Nigel if this matches the codex"

Nigel checks for:
- Missing back buttons
- Inconsistent screen structures
- Padding/spacing violations
- Touch target sizes
- Labeling format inconsistencies
- Navigation pattern deviations
- Any screen that "feels different" from the rest

---

## The Philosophy

> "The user should never have to think about how to use the interface. Every screen should feel like the same app."

This codex establishes patterns based on what's **already working well** in the app, validated against Apple HIG, Material Design 3, and React Native best practices (2025/2026).

---

## Foundation: Design Tokens

All values come from `src/constants/colors.ts`. **Never use magic numbers.**

### Spacing (8-point grid)
```typescript
spacing.xs   = 4    // Tight gaps, icon-to-text
spacing.sm   = 8    // Between related elements
spacing.md   = 16   // Standard padding, between cards
spacing.lg   = 24   // Section separation
spacing.xl   = 32   // Major sections
spacing.xxl  = 48   // Screen-level padding bottom
```

### Border Radius
```typescript
borderRadius.xs   = 6    // Small badges
borderRadius.sm   = 8    // Buttons, inputs, small cards
borderRadius.md   = 12   // Cards, modals
borderRadius.lg   = 16   // Large cards, bottom sheets
borderRadius.xl   = 20   // Prominent containers
borderRadius.full = 9999 // Pills, circular badges
```

### Font Sizes
```typescript
fontSize.xs   = 12   // Badges, tertiary labels
fontSize.sm   = 14   // Secondary text, unit names
fontSize.md   = 16   // Body text, primary labels (MINIMUM for body)
fontSize.lg   = 18   // Section headers
fontSize.xl   = 20   // Screen titles
fontSize.xxl  = 24   // Hero text
fontSize.xxxl = 32   // Large display
```

### Font Weights
```typescript
fontWeight.normal   = '400'
fontWeight.medium   = '500'
fontWeight.semibold = '600'  // Primary labels, names
fontWeight.bold     = '700'  // Headers, emphasis
```

---

## Navigation Patterns (Nigel's Priority)

### Back Button Rules

**MANDATORY**: Every screen that is NOT a root tab screen MUST have a back button.

```typescript
// In _layout.tsx for nested routes
<Stack.Screen
  name="detail"
  options={{
    headerShown: true,
    headerTitle: "Screen Title",
    headerBackTitle: "Back",  // iOS
    headerLeft: () => (       // Custom back if needed
      <TouchableOpacity onPress={() => router.back()}>
        <Ionicons name="arrow-back" size={24} color={colors.text} />
      </TouchableOpacity>
    ),
  }}
/>
```

**Root Tab Screens** (NO back button needed):
- `(tabs)/home.tsx`
- `(tabs)/tracker.tsx`
- `(tabs)/actions.tsx`
- `(tabs)/directory.tsx`
- `(tabs)/more.tsx`

**All Other Screens** (MUST have back button):
- Any screen in `/tracker/...`
- Any screen in `/directory/...`
- Any screen in `/meetings/...`
- Any screen in `/speaking-assignments/...`
- Modal screens (use close/X or Cancel)

### Header Structure

#### Standard Detail Screen Header
```typescript
headerStyle: {
  backgroundColor: colors.surface,
}
headerTitleStyle: {
  fontSize: fontSize.lg,
  fontWeight: fontWeight.semibold,
  color: colors.text,
}
headerShadowVisible: false,
headerBackTitleVisible: false,  // iOS: hide "Back" text, just show arrow
```

#### Screen with Custom Header (Inside Screen)
When using a custom header inside the screen (not in _layout):
```typescript
customHeader: {
  flexDirection: 'row',
  alignItems: 'center',
  paddingHorizontal: spacing.md,
  paddingVertical: spacing.sm,
  backgroundColor: colors.surface,
  borderBottomWidth: 1,
  borderBottomColor: colors.border,
}
backButton: {
  padding: spacing.sm,
  marginRight: spacing.sm,
  minWidth: 44,
  minHeight: 44,
}
headerTitle: {
  flex: 1,
  fontSize: fontSize.lg,
  fontWeight: fontWeight.semibold,
  color: colors.text,
}
```

---

## Screen Structure Patterns (Nigel's Checklist)

Every screen should follow ONE of these patterns:

### Pattern A: List Screen
```
┌─────────────────────────────┐
│ [Back]  Screen Title        │  ← Header (from layout or custom)
├─────────────────────────────┤
│                             │
│  ┌───────────────────────┐  │
│  │ Card 1                │  │  ← Cards with consistent structure
│  └───────────────────────┘  │
│  ┌───────────────────────┐  │
│  │ Card 2                │  │
│  └───────────────────────┘  │
│                             │
│              OR             │
│                             │
│     (Empty State Icon)      │  ← EmptyState component if no items
│     No items to show        │
│                             │
│                        [+]  │  ← FAB if screen allows adding
└─────────────────────────────┘
```

### Pattern B: Detail Screen
```
┌─────────────────────────────┐
│ [Back]  Detail Title        │  ← Header
├─────────────────────────────┤
│                             │
│  Person Name                │  ← Primary info at top
│  Unit Name                  │
│                             │
│  ─────────────────────────  │  ← Divider or section break
│                             │
│  Section Header             │
│  ┌───────────────────────┐  │
│  │ Info Card / Form      │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │ [Primary Action]      │  │  ← Action buttons at bottom
│  └───────────────────────┘  │
│  [Secondary Action]         │
│                             │
└─────────────────────────────┘
```

### Pattern C: Form Screen
```
┌─────────────────────────────┐
│ [Cancel]  Form Title [Save] │  ← Header with Cancel/Save
├─────────────────────────────┤
│                             │
│  Label                      │
│  ┌───────────────────────┐  │
│  │ Input                 │  │
│  └───────────────────────┘  │
│                             │
│  Label                      │
│  ┌───────────────────────┐  │
│  │ Input                 │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │ [Submit Button]       │  │  ← Primary action, full width
│  └───────────────────────┘  │
│                             │
└─────────────────────────────┘
```

### Pattern D: Dashboard/Home Screen
```
┌─────────────────────────────┐
│        App Title            │  ← No back button (root tab)
├─────────────────────────────┤
│                             │
│  Section Header             │
│  ┌───────────────────────┐  │
│  │ Summary Card          │  │  ← Aggregate info
│  └───────────────────────┘  │
│                             │
│  Section Header      [See All]
│  ┌─────────┐ ┌─────────┐    │
│  │ Item 1  │ │ Item 2  │    │  ← Preview items
│  └─────────┘ └─────────┘    │
│                             │
└─────────────────────────────┘
```

---

## Cross-Screen Consistency (Nigel's Pet Peeves)

### Same Data = Same Appearance
If the same type of data appears on multiple screens, it MUST look identical:

| Data Type | Appearance | Where Used |
|-----------|------------|------------|
| Person name | fontSize.md, semibold, colors.text | All tracker screens, detail screens |
| Unit name | fontSize.sm, colors.textSecondary | All tracker screens, detail screens |
| Calling label | "Calling: [name]" with green icon | List screens, detail screens |
| Release label | "Release: [name]" with red icon | List screens, detail screens |
| Date display | "Mon, Jan 15" or relative ("Today", "Tomorrow") | All screens with dates |
| Status badge | Badge component with consistent variants | All status displays |

### Same Action = Same Button
| Action | Button Style | Icon |
|--------|--------------|------|
| Edit item | Tinted background (primary+15%), pencil icon | `pencil` |
| Delete item | Red text or button, confirmation required | `trash-outline` |
| Add new | FAB or primary button | `add` |
| Navigate to detail | Card is tappable + chevron | `chevron-forward` |
| Close/Cancel | Text button, left side of header | `close` or text |
| Save/Done | Primary text, right side of header | text only |

### Same Empty = Same Message
| Screen Type | Empty State Title | Icon |
|-------------|-------------------|------|
| Action items | "All Caught Up!" | `checkmark-circle-outline` |
| Submissions | "No Submissions" | `document-outline` |
| Search results | "No Results" | `search-outline` |
| Assignments | "No Assignments" | `clipboard-outline` |

---

## Screen Structure

### Standard Screen Container
```typescript
container: {
  flex: 1,
  backgroundColor: colors.background,
}
content: {
  padding: spacing.md,        // 16px - STANDARD for all screens
  paddingBottom: spacing.xxl, // 48px - room for FAB
}
```

**VIOLATION**: `sp-initial-review.tsx` uses `padding: spacing.sm`. Should be `spacing.md`.

### Screen with ScrollView
```typescript
<ScrollView
  style={styles.container}
  contentContainerStyle={styles.content}
  refreshControl={<RefreshControl refreshing={refreshing} onRefresh={handleRefresh} />}
>
```

---

## Cards

### Standard Card (List Items)
**Source of Truth**: `sp-initial-review.tsx:itemCard`

```typescript
itemCard: {
  backgroundColor: colors.surface,
  borderRadius: borderRadius.md,      // 12px
  padding: spacing.md,                // 16px internal
  marginBottom: spacing.sm,           // 8px between cards
  // Shadow - standard elevation
  shadowColor: '#000',
  shadowOffset: { width: 0, height: 2 },
  shadowOpacity: 0.08,
  shadowRadius: 4,
  elevation: 2,
}
```

### Card Content Layout
```typescript
cardContent: {
  flexDirection: 'row',
  alignItems: 'center',
}
cardInfo: {
  flex: 1,
}
cardActions: {
  flexDirection: 'row',
  alignItems: 'center',
  gap: spacing.sm,  // 8px between action items
}
```

### Grouped Cards (Same Organization)
When cards are grouped by organization (e.g., multiple items from same ward):
```typescript
groupedCardContainer: {
  backgroundColor: colors.primary + '15',  // Light tint
  marginHorizontal: -spacing.md,           // Bleed to edges
  paddingHorizontal: spacing.md,
  paddingTop: spacing.sm,
  paddingBottom: spacing.sm,
  borderRadius: borderRadius.lg,           // Rounded container
}
// Cards inside: marginBottom: 0 (touch each other)
// Use borderBottomWidth: 1, borderBottomColor: colors.border
```

---

## Person/Item Cards

### Name + Unit Pattern (THE STANDARD)
```typescript
personName: {
  fontSize: fontSize.md,          // 16px
  fontWeight: fontWeight.semibold,
  color: colors.text,
}
unitName: {
  fontSize: fontSize.sm,          // 14px
  color: colors.textSecondary,
  marginTop: 2,
}
```

### Detail Row (Calling/Release Info)
```typescript
detailRow: {
  flexDirection: 'row',
  alignItems: 'center',
  gap: spacing.xs,    // 4px between icon and text
}
detailText: {
  fontSize: fontSize.sm,
  color: colors.text,
  flex: 1,
}
```

### Detail Icon Colors
- **Informational screens** (SP Review, HC Review): `colors.textTertiary` (gray)
- **Action screens** (Pre-Sustaining, Unit Sustaining): Colored icons help distinguish
  - Calling: `colors.success` (green) with `add-circle-outline`
  - Release: `colors.error` (red) with `remove-circle-outline`
  - Ordination: `colors.success` with `arrow-up-circle-outline`

### Labeling Format (MANDATORY)
```
Calling: [calling name]     // Green plus icon
Release: [calling name]     // Red minus icon
Ordain to [Elder/High Priest]  // Green up-arrow icon
```

**NEVER** use variants like:
- "Release from: [name]"
- Just "[calling name]" without prefix
- "New Calling: [name]"

---

## Touch Targets

### Minimum Sizes (Apple HIG + Material Design)
```typescript
// Minimum touch target: 44x44 pt
touchTarget: {
  minWidth: 44,
  minHeight: 44,
}
```

### Edit Button
```typescript
editButton: {
  padding: spacing.sm,                    // 8px (NOT xs!)
  backgroundColor: colors.primary + '15', // 15% opacity
  borderRadius: borderRadius.sm,
  minWidth: 44,                           // Touch target
  minHeight: 44,
  alignItems: 'center',
  justifyContent: 'center',
}
// Icon: pencil, size 14, color: colors.primary
```

**VIOLATION**: Many screens use `padding: spacing.xs` (4px). Should be `spacing.sm`.

### Chevron for Navigation
```typescript
// Size 20, color: colors.textTertiary
<Ionicons name="chevron-forward" size={20} color={colors.textTertiary} />
```

---

## Buttons

### Primary Button (Full Width)
```typescript
primaryButton: {
  backgroundColor: colors.primary,
  borderRadius: borderRadius.md,
  paddingVertical: spacing.md,
  paddingHorizontal: spacing.lg,
  minHeight: 48,
  alignItems: 'center',
  justifyContent: 'center',
}
primaryButtonText: {
  fontSize: fontSize.md,
  fontWeight: fontWeight.semibold,
  color: colors.textInverse,
}
```

### Secondary Button (Outline)
```typescript
secondaryButton: {
  backgroundColor: 'transparent',
  borderWidth: 1,
  borderColor: colors.primary,
  borderRadius: borderRadius.md,
  paddingVertical: spacing.sm,
  paddingHorizontal: spacing.md,
  minHeight: 44,
}
secondaryButtonText: {
  fontSize: fontSize.sm,
  fontWeight: fontWeight.medium,
  color: colors.primary,
}
```

### Action Chip (Inline Actions)
```typescript
actionChip: {
  flexDirection: 'row',
  alignItems: 'center',
  gap: 4,
  paddingVertical: spacing.xs,
  paddingHorizontal: spacing.sm,
  borderRadius: borderRadius.full,
  backgroundColor: colors.primary + '15',
  minHeight: 28,
}
actionChipText: {
  fontSize: fontSize.xs,
  fontWeight: fontWeight.medium,
  color: colors.primary,
}
```

### Sustain Button
```typescript
sustainButton: {
  flexDirection: 'row',
  alignItems: 'center',
  justifyContent: 'center',
  gap: 4,
  paddingVertical: spacing.xs,
  paddingHorizontal: spacing.md,
  borderRadius: borderRadius.md,
  backgroundColor: colors.primary,
  minWidth: 90,
  minHeight: 32,
}
```

---

## Floating Action Button (FAB)

### Primary FAB
```typescript
fab: {
  position: 'absolute',
  bottom: spacing.xl,    // 32px from bottom
  right: spacing.md,     // 16px from right
  width: 56,
  height: 56,
  borderRadius: 28,
  backgroundColor: colors.primary,
  alignItems: 'center',
  justifyContent: 'center',
  // Shadow
  shadowColor: '#000',
  shadowOffset: { width: 0, height: 4 },
  shadowOpacity: 0.15,
  shadowRadius: 8,
  elevation: 6,
}
// Icon: size 24, color: colors.textInverse
```

### Secondary FAB (Smaller)
```typescript
fabSecondary: {
  width: 44,
  height: 44,
  borderRadius: 22,
  backgroundColor: colors.surface,
  borderWidth: 1,
  borderColor: colors.border,
}
```

---

## Section Headers

### Organization Group Header (Inside Cards)
```typescript
sectionHeader: {
  fontSize: fontSize.sm,
  fontWeight: fontWeight.semibold,
  color: colors.textSecondary,
  textTransform: 'uppercase',
  letterSpacing: 0.5,
  marginBottom: spacing.xs,
}
```

### Major Section Header
```typescript
majorSectionHeader: {
  fontSize: fontSize.md,
  fontWeight: fontWeight.bold,
  color: colors.text,
  marginTop: spacing.lg,
  marginBottom: spacing.sm,
}
```

---

## Status Badges

### Count Badge (Header/Inline)
```typescript
countBadge: {
  minWidth: 24,
  height: 24,
  borderRadius: 12,
  backgroundColor: colors.surface,
  justifyContent: 'center',
  alignItems: 'center',
  paddingHorizontal: spacing.sm,
}
countBadgeText: {
  fontSize: fontSize.sm,
  fontWeight: fontWeight.bold,
  color: colors.primary,
}
```

### Status Badge (Component)
Use the `Badge` component from `src/components/ui/Badge.tsx`.
Variants: `default`, `success`, `warning`, `error`, `info`

---

## Empty States

Use the `EmptyState` component consistently:
```typescript
<EmptyState
  title="All Caught Up!"
  message="No items pending review."
  showQuote
  quoteCategory="leadership"
/>
```

---

## Loading States

### Full Screen Loading
```typescript
<LoadingSpinner fullScreen message="Loading..." />
```

### Inline Loading
```typescript
<ActivityIndicator size="small" color={colors.primary} />
```

---

## Modals

### Standard Modal Container
```typescript
modalOverlay: {
  flex: 1,
  backgroundColor: 'rgba(0, 0, 0, 0.5)',
  justifyContent: 'center',
  alignItems: 'center',
  padding: spacing.lg,
}
modalContent: {
  backgroundColor: colors.surface,
  borderRadius: borderRadius.lg,
  padding: spacing.lg,
  width: '100%',
  maxWidth: 400,
}
modalHeader: {
  flexDirection: 'row',
  justifyContent: 'space-between',
  alignItems: 'center',
  marginBottom: spacing.md,
}
modalTitle: {
  fontSize: fontSize.lg,
  fontWeight: fontWeight.semibold,
  color: colors.text,
}
```

---

## Form Inputs

Use the `Input` component from `src/components/ui/Input.tsx`.

### Standard Input Container
```typescript
inputContainer: {
  marginBottom: spacing.md,
}
inputLabel: {
  fontSize: fontSize.sm,
  fontWeight: fontWeight.medium,
  color: colors.text,
  marginBottom: spacing.xs,
}
```

### TextInput Style
```typescript
textInput: {
  backgroundColor: colors.surfaceSecondary,
  borderRadius: borderRadius.sm,
  padding: spacing.sm,
  fontSize: fontSize.md,
  color: colors.text,
  minHeight: 44,  // Touch target
}
```

---

## Animations (Reanimated 3 / Moti)

### Approved Animation Types
- Opacity fades: `withTiming({ duration: 200 })`
- Scale transforms: `withSpring({ damping: 15 })`
- Layout animations: `LayoutAnimation.Presets.easeInEaseOut`

### Avoid
- Animating width/height/margin (causes layout thrashing)
- More than 100 simultaneous animations on low-end devices

---

## Icon Standards

### Icon Library
Use `@expo/vector-icons` with `Ionicons`.

### Standard Sizes
- Detail icons: 14-16
- Button icons: 16-18
- Navigation icons: 20
- FAB icons: 24
- Empty state icons: 48

### Common Icons
| Purpose | Icon Name |
|---------|-----------|
| Calling (add) | `add-circle-outline` |
| Release (remove) | `remove-circle-outline` |
| Ordination (up) | `arrow-up-circle-outline` |
| Edit | `pencil` |
| Navigate | `chevron-forward` |
| Expand | `chevron-down` / `chevron-up` |
| Close | `close` |
| Add | `add` |
| People | `people` |
| Calendar | `calendar-outline` |
| Notifications | `notifications-outline` |

---

## Violations Log

### Current Issues to Fix

1. **Content Padding**: `sp-initial-review.tsx` uses `spacing.sm` instead of `spacing.md`
2. **Touch Targets**: Edit buttons across screens use `spacing.xs` padding - should be `spacing.sm` minimum
3. **Labeling**: Some screens don't prefix with "Calling:" or "Release:"

---

## Nigel's Audit Checklist

Before merging any new screen, Nigel checks:

### Structure & Navigation
- [ ] Screen follows one of the 4 patterns (List, Detail, Form, Dashboard)
- [ ] Back button present (unless root tab screen)
- [ ] Header style matches other screens of same type
- [ ] Cancel/Save buttons in correct positions for forms

### Spacing & Layout
- [ ] Uses `spacing.md` for content padding
- [ ] Cards use standard shadow (0 2px, opacity 0.08)
- [ ] Touch targets minimum 44x44
- [ ] Consistent gap between elements (spacing.sm between cards)

### Typography & Labels
- [ ] Person name: fontSize.md, fontWeight.semibold
- [ ] Unit name: fontSize.sm, colors.textSecondary
- [ ] Calling/Release labeled with correct prefix
- [ ] Section headers use consistent style

### Components
- [ ] Uses EmptyState component for empty lists
- [ ] Uses LoadingSpinner for loading states
- [ ] Uses Badge component for status (not custom)
- [ ] Uses Card component (not custom TouchableOpacity with styles)
- [ ] Edit button has proper padding and background

### Visual Consistency
- [ ] All colors from design tokens, no magic hex values
- [ ] FAB positioned correctly if present
- [ ] Icons from Ionicons, correct sizes per context
- [ ] Same data looks same as on other screens

### Cross-Screen Check
- [ ] If this data appears elsewhere, does it match?
- [ ] If this action exists elsewhere, is the button the same?
- [ ] Would a user recognize this as the same app?

---

## Violations Log

### Current Issues to Fix

1. **Content Padding**: `sp-initial-review.tsx` uses `spacing.sm` instead of `spacing.md`
2. **Touch Targets**: Edit buttons across screens use `spacing.xs` padding - should be `spacing.sm` minimum
3. **Labeling**: Some screens don't prefix with "Calling:" or "Release:"

*Nigel will update this log as he finds more violations.*

---

## How to Request a Nigel Audit

Ask Claude to:
1. **Audit a specific screen**: "Get Nigel to check tracker/detail/index.tsx"
2. **Audit a feature area**: "Have Nigel audit all speaking assignment screens"
3. **Compare two screens**: "Ask Nigel if the HC review and SP review screens are consistent"
4. **Full app audit**: "Nigel, do a full consistency check"

Nigel will report:
- ✅ What's correct
- ❌ Violations with file:line references
- 🔧 Specific fixes needed

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-29 | Initial codex based on audit |
| 1.1 | 2026-01-29 | Added Nigel, navigation patterns, screen structure patterns, cross-screen consistency rules |
