# Spotlight Feature Specification

## Overview

The Spotlight is a prioritized section at the top of the home screen that surfaces the most important items requiring action. It sits above the existing tiles, providing focused guidance while maintaining access to the full dashboard.

**North Star:** "Here's what matters most right now. One thing at a time."

---

## Core Principles

### From the Vision Doc
- **Shore Principle:** Minister to the minister first. Greeting before asking.
- **No guilt:** Never say "overdue" or "you're behind." Just surface the next thing.
- **Trust but verify:** Tiles stay visible. Spotlight earns trust over time.
- **Chronological honesty:** Time-bound items always in date order.
- **Quick wins float up:** < 30 seconds + no second party = elevated priority.

### Key Rules

1. **Never lie about "caught up"** — Only say it if truly zero items. Otherwise: "Nothing urgent — X items can wait."

2. **Time-bound items in chronological order** — Users assume top = soonest. Never violate this.

3. **Two-phase items have separate effective dates** — Prep phase (2 weeks before) and execution phase (day of) are distinct cards.

4. **Delegation is secretary-owned** — Secretaries configure who handles prep for whom. SP members don't have to configure anything.

5. **Prep is delegated, execution is not** — Secretary handles scheduling; SP member does the interview and marks it complete.

---

## Priority Tiers

### Tier 1: CRITICAL (Today)
- Meeting in < 6 hours
- Sunday unit sustainings (on Sunday, for secretary AND HC)
- Speaking assignment < 3 days with no prep done
- Pre-sustaining bucket not empty (Sunday morning, secretary)
- Any item waiting > 14 days

### Tier 2: HIGH (This Week)
- Meeting in 1-3 days
- Speaking assignment < 1 week (execution phase)
- Speaking assignment 2 weeks out (prep phase — contact bishopric)
- Interview 2 weeks before due (prep phase — schedule it)
- SP/HC votes needed (blocks downstream work)
- Pre-sustaining bucket review (Saturday evening)
- Pending user > 24 hours
- Extend call follow-up (4+ days, no confirmation)
- Quick wins (< 30 sec, single action, no second party)

### Tier 3: MEDIUM (Soon)
- Meeting prep 3-7 days out
- Speaking assignment 2-3 weeks (awareness)
- Interview due this month (execution phase)
- Set apart needed
- Action items with due dates approaching
- Extend call (newly assigned)

### Tier 4: LOW (Background)
- Record in LCR
- Future meeting awareness (> 7 days)
- Speaking assignment > 3 weeks
- Interview not due soon

---

## Two-Phase Items

Many items have a **preparation phase** and an **execution phase** with different owners and timing.

### Speaking Assignments

| Phase | Timing (Effective Date) | Who Sees It | Card |
|-------|-------------------------|-------------|------|
| Prep | Speaking date - 2 weeks | Delegatee (secretary) or SP if not delegated | "Contact Maple Grove bishopric about your talk on Feb 15" |
| Execution | Speaking date | SP member | "You're speaking at Maple Grove today" |

### Interviews

| Phase | Timing (Effective Date) | Who Sees It | Card |
|-------|-------------------------|-------------|------|
| Prep | Due date - 2 weeks | Delegatee (secretary) or SP if not delegated | "Schedule interview with Bishop Martinez (due Feb 17)" |
| Execution | Scheduled date or due date | SP member | "Interview with Bishop Martinez today at 3pm" |

### Set-Aparts

| Phase | Timing (Effective Date) | Who Sees It | Card |
|-------|-------------------------|-------------|------|
| Prep | After call accepted | Delegatee or SP | "Schedule set-apart for Sister Mitchell" |
| Execution | Scheduled date | SP member | "Set apart Sister Mitchell today" |

### Meetings

| Phase | Timing | Who Sees It | Card |
|-------|--------|-------------|------|
| All phases | Meeting date | Relevant roles | "HC Meeting Thursday at 7pm" |

Note: Meetings don't have separate prep effective dates. Related items (votes, agenda review) surface based on meeting proximity.

### Pre-Sustaining Bucket

| Phase | Timing | Who Sees It | Card |
|-------|--------|-------------|------|
| Saturday review | Saturday after 4pm | Secretary | "Review pre-sustaining bucket for tomorrow" |
| Sunday release | Sunday before noon | Secretary | "5 items ready to release to units" |

---

## Delegation System

### Concept

Secretaries configure who handles prep work for each Stake Presidency member. The SP member doesn't need to configure anything — it just works.

### Delegatable Categories

| Category | Prep Phase | Execution Phase |
|----------|------------|-----------------|
| `interview_scheduling` | Schedule the interview | Conduct it, mark complete |
| `speaking_coordination` | Contact bishopric, coordinate | Give the talk |
| `set_apart_scheduling` | Schedule the set-apart | Perform it, mark complete |

### Who Can Configure

- Executive Secretary
- Assistant Executive Secretaries
- (SP members CAN view/edit their own settings, but rarely will)

### Data Model

```sql
CREATE TABLE delegation_settings (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  stake_id UUID NOT NULL REFERENCES stakes(id),
  delegator_id UUID NOT NULL REFERENCES users(id),  -- The SP member
  category TEXT NOT NULL,  -- 'interview_scheduling', 'speaking_coordination', etc.
  delegatee_id UUID NOT NULL REFERENCES users(id),  -- The secretary handling prep
  created_by UUID NOT NULL REFERENCES users(id),    -- Who configured this
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(stake_id, delegator_id, category)
);
```

### Flow

1. Secretary opens "Delegation Settings" (in onboarding or settings)
2. Selects a Stake Presidency member
3. For each category, picks who handles prep (self, another secretary, or "They'll handle it themselves")
4. Saves settings
5. Prep items now route to the configured delegatee's Spotlight
6. Execution items always go to the SP member's Spotlight

---

## Snooze System

### "Not Now" Behavior

When user taps "Not Now" on a Spotlight item:

| Item Tier | Snooze Duration |
|-----------|-----------------|
| Critical | 2 hours |
| High | Until tomorrow |
| Medium | 3 days |
| Low | 1 week |

After snooze expires, item returns to normal priority calculation.

### Data Model

```sql
CREATE TABLE spotlight_snoozes (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id),
  item_type TEXT NOT NULL,  -- 'calling_submission', 'interview', 'speaking_assignment', etc.
  item_id UUID NOT NULL,
  snoozed_until TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, item_type, item_id)
);
```

---

## Prep Status Tracking

### New Fields for Two-Phase Items

**speaking_assignments:**
```sql
ALTER TABLE speaking_assignments ADD COLUMN prep_status TEXT DEFAULT NULL;
-- Values: NULL (not started), 'contacted', 'confirmed'
ALTER TABLE speaking_assignments ADD COLUMN prep_completed_at TIMESTAMPTZ;
ALTER TABLE speaking_assignments ADD COLUMN prep_completed_by UUID REFERENCES users(id);
```

**interview_assignments:**
```sql
ALTER TABLE interview_assignments ADD COLUMN prep_status TEXT DEFAULT NULL;
-- Values: NULL (not started), 'scheduled'
ALTER TABLE interview_assignments ADD COLUMN scheduled_date TIMESTAMPTZ;
ALTER TABLE interview_assignments ADD COLUMN prep_completed_at TIMESTAMPTZ;
ALTER TABLE interview_assignments ADD COLUMN prep_completed_by UUID REFERENCES users(id);
```

**calling_submissions (for set-apart scheduling):**
```sql
ALTER TABLE calling_submissions ADD COLUMN set_apart_scheduled_date TIMESTAMPTZ;
ALTER TABLE calling_submissions ADD COLUMN set_apart_scheduled_by UUID REFERENCES users(id);
```

---

## Spotlight Display

### Layout

```
┌─────────────────────────────────────────┐
│  Good morning, President.               │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  🔴 HC Meeting tonight at 7pm   │    │
│  │     2 votes pending             │    │
│  │                                 │    │
│  │  [Cast Votes]   [View Agenda]   │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  🟠 Sister Mitchell is ready    │    │
│  │     Primary Secretary           │    │
│  │                                 │    │
│  │  [Extend Call]    [Not Now]     │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Nothing else urgent · 4 items can wait │
│                                         │
│  ── Needs Attention ──────────────────  │
│  [SP Review: 2] [Sustaining: 1] ...     │
│                                         │
└─────────────────────────────────────────┘
```

### Components

1. **Greeting** — Time-aware, warm
2. **Spotlight Cards** — Top 2-3 priority items
3. **Summary Line** — "Nothing else urgent · X items can wait" or "You're caught up"
4. **Tiles** — Existing dashboard tiles (unchanged)

### Sorting Rules

1. Separate time-bound from non-time-bound items
2. Time-bound: Sort by effective date (chronological, soonest first)
3. Non-time-bound: Sort by priority score (highest first)
4. Display time-bound first, then non-time-bound

### Color Coding

| Tier | Color | Indicator |
|------|-------|-----------|
| Critical | Red | 🔴 |
| High | Orange | 🟠 |
| Medium | Yellow | 🟡 |
| Low | (not shown in Spotlight) | — |

---

## Greeting Logic

### Time-Based

| Time | Greeting |
|------|----------|
| 5am - 12pm | "Good morning, [Name]." |
| 12pm - 5pm | "Good afternoon, [Name]." |
| 5pm - 9pm | "Good evening, [Name]." |
| 9pm - 5am | "Working late, [Name]?" |

### Day-Based Additions

| Day | Addition |
|-----|----------|
| Sunday | "Happy Sabbath." |
| Saturday | (none) |
| Friday | (none) |

### Context Additions (Future)

- After absence (3+ days): "Good to see you back."
- Heavy load (5+ critical items): "Busy day. One thing at a time."

---

## API Design

### getSpotlightItems()

```typescript
interface SpotlightItem {
  id: string;
  type: 'calling_submission' | 'interview' | 'speaking_assignment' | 'meeting' | 'action_item' | 'pending_user' | 'pre_sustaining';
  phase: 'prep' | 'execution' | null;
  tier: 'critical' | 'high' | 'medium' | 'low';
  effectiveDate: Date | null;  // For chronological sorting
  title: string;
  subtitle: string;
  actions: SpotlightAction[];
  data: any;  // The underlying record
}

interface SpotlightAction {
  label: string;
  action: string;  // 'vote', 'extend', 'complete', 'snooze', etc.
  primary: boolean;
}

async function getSpotlightItems(
  user: User,
  stakeId: string,
  limit: number = 3
): Promise<{
  items: SpotlightItem[];
  totalCount: number;
  snoozedCount: number;
}>;
```

### getDelegationSettings()

```typescript
interface DelegationSetting {
  delegatorId: string;
  delegatorName: string;
  category: string;
  delegateeId: string | null;  // null = "they'll handle it themselves"
  delegateeName: string | null;
}

async function getDelegationSettings(stakeId: string): Promise<DelegationSetting[]>;

async function updateDelegationSetting(
  stakeId: string,
  delegatorId: string,
  category: string,
  delegateeId: string | null
): Promise<void>;
```

---

## Implementation Phases

### Phase 1: MVP Spotlight
- [ ] Add greeting component
- [ ] Create `getSpotlightItems()` with basic priority tiers
- [ ] Implement chronological sorting for time-bound items
- [ ] Add Spotlight section to home screen above tiles
- [ ] Quick win detection (hardcoded by item type)
- [ ] Basic "Not Now" (dismisses for session only)

### Phase 2: Two-Phase & Snooze
- [ ] Database migrations for prep status fields
- [ ] Database migration for `spotlight_snoozes` table
- [ ] Two-phase card logic (prep vs execution)
- [ ] Persistent snooze with tier-based duration
- [ ] Prep status UI (mark as scheduled/contacted)

### Phase 3: Delegation
- [ ] Database migration for `delegation_settings`
- [ ] Delegation settings UI for secretaries
- [ ] Route prep items to delegatee
- [ ] Secretary onboarding task for delegation setup

### Phase 4: Refinements
- [ ] "Waiting on response" status
- [ ] Meeting aftermath cards
- [ ] Declined call handling
- [ ] Reflection cards (weekly/monthly)

---

## Success Metrics

**What we measure:**
- Do users tap Spotlight items or skip to tiles?
- Time from app open to first action
- Items completed per session
- Snooze usage (are we surfacing the wrong things?)

**What success looks like:**
- Users engage with Spotlight > 70% of the time
- Average time to first action < 10 seconds
- No reports of "missed items" that should have surfaced

---

## References

- `docs/invisible-assistant-vision.md` — The philosophical foundation
- `docs/assistant-roadmap.md` — The path from dashboard to invisible
- `.claude/commands/genie.md` — Assistant design principles
- `.claude/commands/director.md` — Center Stage architecture
