---
name: content-engine-review
description: "Monthly retrospective on the content engine. Reads last 30 days of calendar-log, memory, grade history, and pickup rates. Surfaces what worked, what didn't, proposed tuning. Saves report to reviews/YYYY-MM-review.md. Foundation B + C applied."
when_to_use:
  - /content-engine-review
  - monthly review
  - engine review
  - content retrospective
  - content retro
  - what worked last month
  - review my content engine
---

# /content-engine-review — v0.1.0

Monthly retrospective on the cowork-social engine. Runs ~10 minutes. Surfaces what worked, what didn't, and proposes specific tuning before next month starts.

> **Live skill** (not headless). Walks the user through findings + asks decisions on staged skill changes. `⚡ NEXT MOVE` prints in chat, varies by verdict (3 states).

---

## Inputs (lazy-load — Foundation A)

Read on every run:

| File | Why |
|---|---|
| `projects/social-media-content/calendar-log.md` (last 30 days slice) | Posts shipped, platform breakdown, scheduled vs manual, status |
| `projects/social-media-content/memory.md` (last 30 days slice) | Self-improvement notes, recurring complaints, headless-run logs |
| `projects/social-media-content/skill-improvements.md` | Staged changes waiting on user review |
| `projects/social-media-content/ideas/` (last 4 weekly idea files) | Generated vs picked vs shipped funnel |
| `projects/social-media-content/brand-brief.md` | Current brand state — is the wedge stale? |
| `_aibos/state-social.md` | `blotato_status`, `selected_platforms`, target cadence |
| `_shared/foundations.md` | Foundation B + C |

If `calendar-log.md` has fewer than 5 rows in the last 30 days → surface:

> "Less than 5 posts shipped in the last 30 days. The retrospective will be light. Continue, or block 2 hours for `/weekly-content-session` first?"

If user says "block 2 hours" → halt. If "continue" → proceed with whatever data exists.

---

## The 6-step flow

### Step 1 — Pull the data

Aggregate from each input file:

**From `calendar-log.md` (last 30 days):**
- Total posts shipped (count)
- Per-platform breakdown (count + % of total)
- Status breakdown: scheduled / published / manual / drafted / failed / skipped
- Day-of-week distribution
- If Blotato connected → success rate = published / (scheduled + published + failed)

**From `memory.md` (last 30 days):**
- All self-improvement notes per skill
- All headless-run logs from `/generate-weekly-ideas`
- All failure notes

**From `skill-improvements.md`:**
- All rows where `reviewed: no` — these are staged changes the user hasn't yet decided on

**From `ideas/<last 4 weekly files>`:**
- Ideas generated: total = 40 (4 weeks × 10)
- Ideas picked: cross-reference `calendar-log.md` against idea hooks (substring overlap >= 50%)
- Ideas drafted but not shipped: in `outputs/` but not in `calendar-log.md`
- Ideas shipped: in `calendar-log.md` with `status: scheduled` or `published` or `manual`
- Compute the funnel: generated → picked → drafted → shipped

### Step 2 — Compute patterns

Surface the patterns the user might miss:

**Platform performance:**
- Top platform this month (most posts)
- Lowest platform (least posts) — flag if it's a `selected_platforms` entry
- If grade history is in `calendar-log.md` (Step 3 of `/weekly-content-session` logs grades), compute avg grade per platform

**Hook pattern wins:**
- Which of the 11 hook patterns appeared most in `outputs/social-media-content/` drafts this month
- Correlate hook pattern → grade score (if grades logged) — name the top 2 patterns by avg score

**Cadence adherence:**
- Compare actual posts/week vs target (from `_aibos/state-social.md` or `scheduling-defaults.md`)
- Compute adherence %

**Self-improvement recurrence:**
- For each skill, count how many self-improvement notes appeared in `memory.md`
- Flag any pattern recurring 3+ times that's NOT already in `skill-improvements.md` — auto-stage per Foundation B

**Idea pickup funnel:**
- Generated → Picked %: of 40 ideas, how many became picks?
- Picked → Drafted %: of picks, how many became drafts?
- Drafted → Shipped %: of drafts, how many shipped?
- The narrowest stage = the bottleneck

**Brand-brief staleness:**
- Check `brand-brief.md` `last_updated` field
- If older than 90 days OR no posts referenced `contrarian_belief` in their hook pattern → flag

### Step 3 — Plan-then-approve the report write

Before writing the report file, show the user a summary table + ask approval:

```
Last 30 days at a glance:
─────────────────────────
Posts shipped:    <N> across <M> platforms
Top platform:     <X> (<N> posts, avg grade <X>)
Worst platform:   <Y> (<N> posts, avg grade <X>) — flag for tuning
Top hook pattern: <name> (<count> uses, avg grade <X>)
Cadence:          <actual>/week vs <target>/week target → <adherence %>
Idea funnel:      40 generated → <N> picked (<X%>) → <N> drafted → <N> shipped
Staged tuning:    <N> skill-improvements rows waiting on you

Approve writing the full report to reviews/<YYYY-MM>-review.md?
Reply 'go' to write, or 'skip' to keep this conversation-only.
```

**Plan-then-approve is non-negotiable for the file write.** If user says skip → don't write the file, but still apply Foundation B + C in chat.

### Step 4 — Write the report

If approved, resolve path: `projects/social-media-content/reviews/<YYYY-MM>-review.md`.

Create `reviews/` directory if missing.

Write:

```markdown
---
type: cowork-social-monthly-review
plugin: cowork-social
plugin_version: 0.1.0
review_month: <YYYY-MM>
generated_date: <ISO>
posts_shipped: <N>
adherence_pct: <X>
source_skill: /content-engine-review
---

# Content Engine Review — <Month Year>

## Summary

- **Posts published:** <N>
- **Platform breakdown:** linkedin <N>, twitter <N>, instagram <N>, tiktok <N>, threads <N>, facebook <N>
- **Avg posts/week:** <X>
- **Target posts/week:** <X>
- **Adherence:** <X>%
- **Blotato success rate:** <X>% (if connected) | not connected — manual posting can't be measured

## What worked

- **Top hook pattern:** #<N> <pattern_name> — <count> uses, avg grade <X>/100
- **Top platform:** <platform> — <count> posts, avg grade <X>/100, <highest engagement signal if any>
- **Standout posts (top 3 by score):**
  - <post_slug> — <platform>, grade <X> — <1-line why it worked>
  - <post_slug> — <platform>, grade <X> — <1-line why it worked>
  - <post_slug> — <platform>, grade <X> — <1-line why it worked>

## What needs tuning

- **<platform> underperformed.** <count> posts, avg grade <X>. <Specific reason from grade reports.>
- **<skill_name> flagged improvements.** <N> entries in `skill-improvements.md` waiting review (see Staged section below).
- **Cadence gap:** <platform> missed <N> scheduled slots. <Reason if surface-able.>
- **Idea funnel bottleneck:** <stage>. <X>% of <upstream> never became <downstream>. <One specific reason.>
- **Brand-brief staleness:** Last updated <date>. <Recommendation if stale.>

## Staged skill improvements (review + decide)

<List from skill-improvements.md where reviewed=no — render as table:>

| skill | pattern | first_seen | recurrence | suggested_change |
|---|---|---|---|---|
| /<skill> | <pattern> | <date> | <N> | <change> |

## Recommendations for next month

1. **<Specific tuning action>** — <one-line rationale + deadline>
2. **<Specific tuning action>** — <one-line rationale + deadline>
3. **<Specific tuning action>** — <one-line rationale + deadline>

---

Generated by `/content-engine-review` on <ISO>.
```

### Step 5 — Walk staged improvements with user

For EACH row in `skill-improvements.md` where `reviewed: no`, ask the user:

> "Apply this change to `/<skill>`? Pattern: '<pattern>' recurred <N> times. Suggested change: '<suggested_change>'. Reply Y / N / defer."

Branch:
- **Y** → mark row `reviewed: yes — approved <ISO>`. Open the target `SKILL.md` for editing (or surface the file path for the user to edit). Append the approval to `memory.md`.
- **N** → mark row `reviewed: yes — declined <ISO> | reason: <user's one-liner>`.
- **defer** → leave row as `reviewed: no`. It'll show up in next month's review.

If 5+ rows are pending, batch: ask the user "5+ staged. Want to walk all of them now (~5 min) or defer this and only walk the top 3 by recurrence count?"

### Step 6 — Brand-brief refresh check

If brand-brief staleness flagged in Step 2, ask:

> "Brand-brief was last updated <date> — <X> days ago. <If 3+ self-improvement notes mentioned voice mismatch:> Multiple voice-mismatch notes this month suggest the brief drifted. Run `/brand-brief` now to refresh? Y / N / defer."

If Y → hand off to `/brand-brief` (the user's next skill invocation).
If N or defer → log in `memory.md` and move on.

---

## Foundation B — Self-improvement close

See `_shared/foundations.md` → Foundation B. After delivering the report + the `⚡ NEXT MOVE` block, ask the user:

> "What would've made this review 10% better?"

Append the answer to `projects/social-media-content/memory.md`:

```
<YYYY-MM-DD> | /content-engine-review | <answer verbatim>
```

**Recurrence patterns to watch** for this skill specifically:

- "review took too long" → consider trimming Step 5 walkthrough
- "didn't find the bottleneck" → consider deeper funnel analysis in Step 2
- "recommendations were generic" → consider more business-brain.md context pull
- "missed grade trends" → consider better grade-history aggregation

If any pattern hits 3+ → flag in `skill-improvements.md` per Foundation B rules.

---

## Foundation C — `⚡ NEXT MOVE` block (verdict-aware, 3 variants)

See `_shared/foundations.md` → Foundation C. Pick the variant based on the month's verdict:

### Verdict 1 — Shipping consistently (>= 10 posts/month) AND grade trend flat-or-up

```
⚡ NEXT MOVE: Schedule a 30-minute /brand-brief refresh this week.
   Why: You're shipping consistently — time to refresh the angle so the next 30 days don't recycle the same hooks.
```

### Verdict 2 — Shipping is the bottleneck (< 5 posts/month)

```
⚡ NEXT MOVE: Block 2 hours Monday for /weekly-content-session.
   Why: Last month's shipping rate is below the engine's break-even — habit needs a calendar slot.
```

### Verdict 3 — Grade trend dropping (avg grade lower than prior month by >= 5 points)

```
⚡ NEXT MOVE: Re-run /brand-brief on the recent_proof_story field this week.
   Why: Grade trend is dropping — voice may have drifted from your real proof, hooks are getting generic.
```

### Picking rule (when multiple verdicts apply)

Priority order:
1. **Verdict 2 wins if shipping < 5/month** (no point fixing voice if nothing's going out).
2. **Verdict 3 wins if grades dropped >= 5 points** (voice drift kills the engine even at high volume).
3. **Verdict 1 is the default** when shipping + grades are both healthy.

**Validation pattern (same as canonical):** `⚡ NEXT MOVE: .+ .+ .+\n   Why: .+`

If the block doesn't match → regenerate before printing.

---

## Hard rules

- **Plan-then-approve before writing the report file.** Step 3 must complete before Step 4 fires.
- **Don't auto-apply skill changes.** Step 5 walkthrough — always confirm each row.
- **Reviews directory:** `projects/social-media-content/reviews/` — create if missing, never overwrite past months.
- **3 verdict-aware NEXT MOVE variants** — never a generic catch-all.
- **Apply 3 foundations every run** — Foundation A (lazy-load) is automatic; B + C are explicit close.
- **Reference `_shared/foundations.md`** rather than duplicating foundation text.
- **3rd-4th grade reading level** in user-facing prompts.
- **Surface patterns the user might miss** — that's the whole point of the retrospective. Don't flatter, don't hedge.

---

## Voice

The retrospective's voice: direct, opinionated, willing to tell the user a platform isn't working. Not flattery, not hype.

When in doubt: imagine a senior strategist sitting across the table on the first of the month. They've read the data. They have an opinion. They give it in plain words.
