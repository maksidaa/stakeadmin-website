# Assistant Roadmap: From Dashboard to Invisible

This document outlines the practical path from where we are today to the invisible assistant vision described in `invisible-assistant-vision.md`.

---

## Where We Are Today

**Current State: Rung 1-2 (Awareness + Prioritization)**

- Dashboard with role-based tiles
- Badge counts showing items needing attention
- Embedded lists with inline actions (partial)
- Some tiles support Center Stage detail mode
- No reflection/recap features
- No nourishment before asking
- Voice is functional, not warm

**What Works:**
- Role-based visibility is solid
- Badge counts update in real-time
- Some tiles have good inline actions
- The data model supports what we need

**What's Missing:**
- One-card experience (Stevie's vision)
- Shore Principle (nourish before asking)
- Reflection features (show what they've done)
- Warm, human voice throughout
- Notification-first actions
- The path to invisibility

---

## Phase 1: The One-Card Foundation

**Goal:** Replace tile grid with single-card experience on home screen

### 1.1 The Priority Algorithm

Build the logic that determines "what's the one thing right now?"

**Factors:**
1. Blocking items (SP votes block HC votes)
2. Time sensitivity (meeting tomorrow > meeting next week)
3. Time waiting (older items surface higher)
4. Quick wins (items completable in one tap)
5. Role relevance (only your items)

**Implementation:**
- New function: `getNextAction(user, stakeId)` in `dashboard.api.ts`
- Returns single most important item with full context
- Falls back gracefully when nothing urgent

### 1.2 The One-Card Component

A single card that shows the next action.

**Structure:**
```
[Warm greeting or context]

[Person's name / Item description]
[Brief details]

[Primary Action]  [Secondary Action]
```

**Example:**
```
Sister Mitchell is ready.

Primary Secretary, Riverside Ward
Submitted 3 days ago

[Extend Call]  [Not Yet]
```

**After action:** Card slides away. Next card appears. Or: "You're caught up."

### 1.3 The Transition

- Keep tiles available via "View All" or swipe gesture
- Default to one-card for users who've used the app before
- New users start with one-card from day one
- Measure: Do users tap "View All"? If rarely, remove it later.

---

## Phase 2: The Shore (Nourish Before Asking)

**Goal:** Minister to the minister before showing tasks

### 2.1 Warm Greetings

Context-aware greetings that feel human.

**Examples:**
- Morning: "Good morning, President."
- Evening: "Hope your day went well."
- Sunday: "Happy Sabbath."
- After absence: "Good to see you back."
- Heavy load: "Busy season. One thing at a time."

**Implementation:**
- Simple time/day logic
- Track last app open for absence detection
- No over-engineering — just a few variants

### 2.2 Inspiring Moments

Brief quotes, scriptures, or thoughts before the task card.

**Rules:**
- Short (one line, maybe two)
- Relevant to leadership/service
- Not every time — maybe 1 in 3 opens
- Skippable but not dismissive

**Examples:**
- "The worth of souls is great."
- "Small and simple things."
- A quote from a recent conference talk

**Implementation:**
- Curated list of 50-100 quotes
- Random selection with category weighting
- Stored locally, refreshed periodically

### 2.3 The Pause

A brief moment before showing the action card.

Not a loading screen. A breath.

"Here's one thing when you're ready."

Then the card appears.

---

## Phase 3: The Reflection

**Goal:** Remind them of the good they've done

### 3.1 Weekly Reflection Card

Shows up once per week (configurable).

**Content:**
- How many items moved forward this week
- Specific names when meaningful ("The Garcias are serving now")
- Stake-wide progress ("12 people sustained this week")

**Voice:**
```
This week:
3 callings extended
2 people set apart
The Hendersons are in Primary now.

You were part of that.
```

**Implementation:**
- Query completed items in last 7 days
- Pull user-specific and stake-wide
- Show once, then dismiss for the week

### 3.2 Monthly Summary

End-of-month reflection available in a dedicated spot.

**Content:**
- Personal contributions
- Stake progress
- Journey highlights ("Remember when this was just a name?")

**Voice:**
```
November

You sat with 8 leaders this month.
14 callings moved through the process.
7 people are serving now who weren't on November 1st.

The work moves forward. Thanks for being part of it.
```

### 3.3 Quarterly/Yearly Recap

Deeper reflection with more context.

**Content:**
- MP ordinations
- Talks given (themes)
- Interviews conducted
- Callings extended/sustained/set apart
- Journey stories

**Voice:**
```
2024 in your stake:

47 brothers received the Melchizedek Priesthood
186 callings extended
312 people sustained
89 talks given — faith, family, service

You were part of every one of these.
```

---

## Phase 4: Voice & Tone Overhaul

**Goal:** Make every word feel human

### 4.1 Audit Current Copy

Go through every screen and identify:
- Corporate/system language
- Guilt-inducing phrases
- Metric-heavy descriptions
- Robotic confirmations

### 4.2 Rewrite Guidelines

**Instead of → Use:**
- "6 items require attention" → "A few things when you're ready"
- "Overdue (5 days)" → "Waiting a bit"
- "Task completed successfully" → "Done."
- "Error: Invalid input" → "That didn't work. Try again?"
- "You have 0 items" → "You're caught up."

### 4.3 Empty States

Every empty state should feel like a gift, not a dead end.

**Instead of:** "No items to display"
**Use:** "Nothing here right now. That's a good thing."

**Instead of:** "No upcoming meetings"
**Use:** "Your calendar's clear. Enjoy."

---

## Phase 5: Notification-First Actions

**Goal:** Handle actions without opening the app

### 5.1 Actionable Notifications

Push notifications with inline actions.

**Examples:**
```
Sister Mitchell accepted her calling.
[Mark Set Apart Needed]  [Open]
```

```
SP vote needed: Brother Garcia
[Sustain]  [View Details]
```

```
HC meeting tomorrow at 7pm
[I'm Ready]  [View Agenda]
```

**Implementation:**
- Expo notification actions
- Backend handlers for each action type
- Confirmation feedback without app open

### 5.2 Lock Screen Completions

User taps action on lock screen → action completes → notification updates → done.

Never needed to unlock. Never opened the app.

---

## Phase 6: Anticipation

**Goal:** Surface things before they're needed

### 6.1 Calendar Awareness

Know what's coming and prepare accordingly.

**Examples:**
- HC meeting in 24 hours → surface HC-related items
- Sunday approaching → surface ward sustaining items
- Quarterly interviews due → surface interview list

### 6.2 Pattern Recognition

Learn when users typically engage.

**Examples:**
- User usually reviews on Saturday evening → prepare items then
- User processes SP votes Monday morning → have them ready
- Quiet notification: "Ready for your Monday review?"

### 6.3 Proactive Preparation

Pre-fill and pre-stage content.

**Examples:**
- Draft email to bishopric about upcoming sustaining
- Compiled interview notes before scheduled meeting
- Agenda summary ready before HC meeting

---

## Phase 7: Toward Invisible

**Goal:** Begin the path to no-app

### 7.1 Auto-Progression

Items move forward automatically when conditions are met.

**Examples:**
- All SP votes cast → auto-move to HC Review
- Ward sustaining date passed → auto-mark sustained
- Set apart confirmed → auto-advance status

**Guardrails:**
- User always sees what happened
- Undo available
- Notifications for auto-actions (initially, reduce over time)

### 7.2 Meeting Integration

Connect to calendar for automatic logging.

**Examples:**
- Calendar shows interview with Bishop → prompt after: "Log this interview?"
- Meeting duration known → pre-fill details
- One tap to confirm

### 7.3 The Invisible Layer (Future)

This is the long-term vision. Not for V1.

- Voice triggers (opt-in): "I extended the calling" → system records
- Location awareness (opt-in): At church → surface relevant items
- Ambient observation: Meeting happened → paperwork updates

---

## Implementation Priority

### Now (V1 Features)

1. **One-Card Component** — Replace tile grid with single focus
2. **Priority Algorithm** — Determine "next most important"
3. **Warm Greetings** — Simple context-aware hellos
4. **Voice Audit** — Rewrite the worst offenders
5. **Weekly Reflection** — Simple "here's what you did" card

### Next (V1.5)

6. **Inspiring Moments** — Quotes/scriptures before action
7. **Monthly Summary** — Deeper reflection
8. **Empty State Overhaul** — Every empty state feels good
9. **Actionable Notifications** — Actions from lock screen

### Later (V2+)

10. **Calendar Awareness** — Meeting-based surfacing
11. **Pattern Recognition** — Learn user habits
12. **Quarterly/Yearly Recap** — Full reflection features
13. **Auto-Progression** — Items move themselves
14. **Proactive Preparation** — Pre-filled content

### Future (V3+)

15. **Meeting Integration** — Calendar → automatic logging
16. **The Invisible Layer** — Voice, location, ambient

---

## Success Metrics (That Actually Matter)

We don't measure:
- App opens
- Time in app
- Features used
- Tasks completed

We measure:
- **Time to action:** How fast from open to completion?
- **Single-session completion:** Did they finish without returning?
- **Return frequency:** Are they opening less often? (Good sign)
- **Reflection engagement:** Do they read the recaps?
- **Qualitative:** Do users report feeling less burdened?

The goal is fewer interactions, not more. Success is when they forget about the app because things just work.

---

## Reference

- Full vision: `docs/invisible-assistant-vision.md`
- Genie (assistant design): `.claude/commands/genie.md`
- Stevie (vision challenges): `.claude/commands/stevie.md`
- Director (Center Stage): `.claude/commands/director.md`
