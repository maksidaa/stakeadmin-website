# Stake Admin App Design System

**Purpose**: Define consistent, compact, readable UI patterns that work for users aged 30-70 with varying tech comfort and vision.

**Core Principle**: Readable but compact. Scannable but complete.

---

## Typography

### Scale

| Name | Size | Weight | Line Height | Use For |
|------|------|--------|-------------|---------|
| `h1` | 24px | 700 | 32px | Screen titles only |
| `h2` | 20px | 600 | 28px | Section headers |
| `h3` | 17px | 600 | 24px | Card titles, person names |
| `body` | 15px | 400 | 22px | Primary reading text |
| `bodySmall` | 13px | 400 | 18px | Secondary info, descriptions |
| `label` | 12px | 500 | 16px | Field labels, badge text |
| `caption` | 11px | 400 | 14px | Timestamps, hints (minimum size) |

### Rules

1. **Minimum readable size**: 15px for body text, 11px absolute minimum
2. **Line height**: ~1.4x font size for comfortable reading
3. **Hierarchy through weight**: Use bold (600-700) for emphasis, not larger sizes
4. **Never use ALL CAPS** for more than 2 words (badges OK, sentences not)

### Implementation

```typescript
// src/theme/typography.ts
export const typography = {
  h1: { fontSize: 24, fontWeight: '700', lineHeight: 32 },
  h2: { fontSize: 20, fontWeight: '600', lineHeight: 28 },
  h3: { fontSize: 17, fontWeight: '600', lineHeight: 24 },
  body: { fontSize: 15, fontWeight: '400', lineHeight: 22 },
  bodySmall: { fontSize: 13, fontWeight: '400', lineHeight: 18 },
  label: { fontSize: 12, fontWeight: '500', lineHeight: 16 },
  caption: { fontSize: 11, fontWeight: '400', lineHeight: 14 },
};
```

---

## Colors

### Primary Palette

| Name | Hex | Use For |
|------|-----|---------|
| `primary` | #6366F1 | Primary buttons, links, active states |
| `primaryLight` | #818CF8 | Hover states, secondary emphasis |
| `primaryDark` | #4F46E5 | Pressed states |

### Secondary Palette

| Name | Hex | Use For |
|------|-----|---------|
| `secondary` | #8B5CF6 | Secondary actions, accents |

### Semantic Colors

| Name | Hex | Use For |
|------|-----|---------|
| `success` | #10B981 | Success states, completed items |
| `warning` | #F59E0B | Warnings, pending items |
| `error` | #EF4444 | Errors, danger actions, declined |
| `info` | #3B82F6 | Informational, in-progress |

### Neutral Colors

| Name | Hex | Use For |
|------|-----|---------|
| `background` | #F3F4F6 | Screen backgrounds |
| `surface` | #FFFFFF | Cards, modals |
| `border` | #E5E7EB | Dividers, card borders |
| `textPrimary` | #111827 | Primary text |
| `textSecondary` | #6B7280 | Secondary text, labels |
| `textDisabled` | #9CA3AF | Disabled states |

---

## Spacing

### Scale

| Name | Value | Use For |
|------|-------|---------|
| `xs` | 4px | Tight spacing, icon gaps |
| `sm` | 8px | Between related elements |
| `md` | 16px | Standard padding, section gaps |
| `lg` | 24px | Between sections |
| `xl` | 32px | Large gaps, screen padding |
| `xxl` | 48px | Major section separation |

### Consistent Application

- **Card padding**: 16px (md) all sides
- **List item padding**: 12px vertical, 16px horizontal
- **Between list items**: 1px border or 8px gap
- **Section spacing**: 24px (lg) between sections
- **Screen padding**: 16px horizontal

---

## Components

### List Item (SubmissionCard)

**Target height**: 60-72px

```
┌────────────────────────────────────────────────────────────┐
│ 12px                                                       │
│ ┌────────────────────────────────────────────────────────┐ │
│ │ Richard Crow                        ┌──────────┐       │ │  ← h3 (17px bold)
│ │ Elders Quorum President             │ SP Review│       │ │  ← body (15px)
│ │ Bastrop Ward                        └──────────┘       │ │  ← bodySmall (13px, secondary color)
│ └────────────────────────────────────────────────────────┘ │
│ 12px                                                       │
└────────────────────────────────────────────────────────────┘
```

**Fields to show (max 4-5):**
1. Person name (required)
2. Calling (required)
3. Unit (required)
4. Status badge (required)
5. Assigned to (optional, context-dependent)

**Fields to hide (show on detail only):**
- Submission date
- Submitted by
- Request notes
- All boolean flags
- Internal IDs

### Status Badge

**Size**: 12px text, 4px vertical padding, 8px horizontal padding, 4px border radius

**Colors by status:**

| Status | Background | Text |
|--------|------------|------|
| SP Review | `#DBEAFE` (blue-100) | `#1E40AF` (blue-800) |
| SP Sustaining | `#E0E7FF` (indigo-100) | `#3730A3` (indigo-800) |
| HC Review | `#EDE9FE` (violet-100) | `#5B21B6` (violet-800) |
| Assigned | `#FEF3C7` (amber-100) | `#92400E` (amber-800) |
| Pre-Sustaining | `#CFFAFE` (cyan-100) | `#155E75` (cyan-800) |
| Sustaining | `#D1FAE5` (green-100) | `#065F46` (green-800) |
| Set Apart | `#D1FAE5` (green-100) | `#065F46` (green-800) |
| Recording | `#FEE2E2` (red-100) | `#991B1B` (red-800) |
| Complete | `#F3F4F6` (gray-100) | `#374151` (gray-700) |
| Declined | `#FEE2E2` (red-100) | `#991B1B` (red-800) |
| On Hold | `#FEF3C7` (amber-100) | `#92400E` (amber-800) |

### Buttons

**Primary button:**
- Background: primary (#6366F1)
- Text: white, 15px, 600 weight
- Height: 48px (touch-friendly)
- Width: full width (100%)
- Border radius: 8px
- Position: bottom of screen

**Secondary button:**
- Background: transparent
- Border: 1px solid border color
- Text: textPrimary, 15px, 500 weight
- Height: 44px
- Position: below primary or in header

**Danger button:**
- Background: error (#EF4444)
- Text: white
- Only for destructive actions
- Always requires confirmation modal

### Form Fields

**Input field:**
- Height: 48px
- Border: 1px solid border
- Border radius: 8px
- Padding: 12px horizontal
- Label: 12px, above field, secondary color
- Placeholder: 15px, textDisabled

**Toggle/Switch:**
- Width: 48px
- Height: 28px
- Track: border color (off), primary (on)
- Thumb: white, 24px

---

## Page Layouts

### List Page Template

```
┌─────────────────────────────────────────────────────────────┐
│ HEADER (56px)                                               │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ [← Back]     Screen Title              [Search] [Menu]  │ │
│ └─────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│ FILTERS (optional, 48px)                                    │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ [All] [Pending] [Complete]  or  [Filter chips...]       │ │
│ └─────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│ LIST (scrollable)                                           │
│                                                             │
│   [Card 1 - 64px]                                           │
│   ─────────────────────────────────────────────────────     │
│   [Card 2 - 64px]                                           │
│   ─────────────────────────────────────────────────────     │
│   [Card 3 - 64px]                                           │
│   ...                                                       │
│                                                             │
│   (Empty state if no items)                                 │
│   (Pull to refresh)                                         │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ FAB (optional, bottom-right)                                │
│                                        [+]                  │
└─────────────────────────────────────────────────────────────┘
```

### Detail Page Template

```
┌─────────────────────────────────────────────────────────────┐
│ HEADER (56px)                                               │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ [← Back]     Person Name                    [Edit/⋯]    │ │
│ └─────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│ CONTENT (scrollable)                                        │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ IDENTITY SECTION (always same)                          │ │
│ │ ┌─────────────────────────────────────────────────────┐ │ │
│ │ │ Status: [Badge]                                     │ │ │
│ │ │ New Calling: Elders Quorum President                │ │ │
│ │ │ Current: Executive Secretary                        │ │ │
│ │ │ Unit: Bastrop Ward                                  │ │ │
│ │ └─────────────────────────────────────────────────────┘ │ │
│ └─────────────────────────────────────────────────────────┘ │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ METADATA SECTION (collapsible)                          │ │
│ │ ┌─────────────────────────────────────────────────────┐ │ │
│ │ │ Submitted: Jan 15, 2025 by Adam Goodwin             │ │ │
│ │ │ Notes: "Bishop recommended this change..."          │ │ │
│ │ └─────────────────────────────────────────────────────┘ │ │
│ └─────────────────────────────────────────────────────────┘ │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ STAGE SECTION (varies per workflow stage)               │ │
│ │ ┌─────────────────────────────────────────────────────┐ │ │
│ │ │ [Stage-specific content]                            │ │ │
│ │ │ - SP Review: Configuration fields                   │ │ │
│ │ │ - HC Review: Sustaining grid (12 buttons)           │ │ │
│ │ │ - Assigned: Extension toggles                       │ │ │
│ │ └─────────────────────────────────────────────────────┘ │ │
│ └─────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│ ACTIONS (fixed at bottom)                                   │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │           [ Primary Action - 48px ]                     │ │
│ │           [ Secondary Action - 44px ]                   │ │
│ │           [ Danger Action - if applicable ]             │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## Information Density Guidelines

### What to Show Where

| Information | List Card | Detail Page |
|-------------|-----------|-------------|
| Person name | ✅ Primary | ✅ Header |
| Calling | ✅ | ✅ |
| Unit | ✅ | ✅ |
| Status | ✅ Badge | ✅ Badge |
| Submitted by | ❌ | ✅ |
| Submitted date | ❌ | ✅ |
| Request notes | ❌ | ✅ |
| SP notes | ❌ | ✅ (if PO) |
| HC notes | ❌ | ✅ |
| Assigned to | ⚠️ Context | ✅ |
| Timestamps | ❌ | ✅ |
| Boolean flags | ❌ | ✅ (as toggles) |
| Vote counts | ❌ | ✅ (as grid) |

### Anti-Patterns to Avoid

1. **More than 5 lines on a list card** → Simplify
2. **Multiple status badges on one card** → Combine into one
3. **Timestamps on every card** → Move to detail
4. **Form fields without sections** → Group and label
5. **Competing action buttons** → One primary max
6. **Small tap targets** → Minimum 44x44px
7. **Technical jargon** → Plain English ("Send for Approval" not "Advance Status")

---

## Accessibility

### Minimum Requirements

- Touch targets: 44x44px minimum
- Color contrast: 4.5:1 for normal text, 3:1 for large text
- Don't rely on color alone (use icons + color for status)
- Support dynamic type (respect system font size preference)
- Screen reader labels on all interactive elements

### Testing Checklist

- [ ] Can complete all flows with VoiceOver/TalkBack
- [ ] Readable with system font size at 150%
- [ ] Understandable with color blindness simulation
- [ ] Tap targets are finger-friendly (no precision required)
