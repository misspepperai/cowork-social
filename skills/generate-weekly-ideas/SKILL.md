---
name: generate-weekly-ideas
description: "Saturday 8am scheduled task. Reads trend-sources.md + brand-brief + recent calendar-log entries; produces 10 ranked post ideas to ideas/YYYY-MM-DD-ideas.md. Headless: NEXT MOVE block embedded in the file footer."
when_to_use:
  - /generate-weekly-ideas
  - generate weekly ideas
  - weekly ideas
  - Saturday ideas
  - saturday cron ideas
---

# /generate-weekly-ideas — v0.1.0

Saturday 8am scheduled task. Reads the user's trend-sources + brand-brief + recent posts, then drops 10 ranked ideas into `projects/social-media-content/ideas/YYYY-MM-DD-ideas.md` so Monday's `/weekly-content-session` has a ready idea bank.

> **Headless skill.** Runs on cron — no live user. No `AskUserQuestion` calls. No chat output beyond a one-line "Saved <N> ideas to <path>" confirmation. The `⚡ NEXT MOVE` block lives in the FILE FOOTER (per the headless exception in `_shared/foundations.md`), not in chat — the user reads it Monday morning when they open the file.

---

## Inputs (lazy-load — Foundation A)

Read on every run:

| File | Why |
|---|---|
| `projects/social-media-content/trend-sources.md` | RSS feeds, sites, keywords, accounts to monitor |
| `projects/social-media-content/brand-brief.md` | Voice + wedge + contrarian belief for ranking |
| `about-me/business-brain.md` | Business context for relevance scoring |
| `projects/social-media-content/calendar-log.md` | Last 30 days of posts — to NOT re-suggest |
| `_aibos/state-social.md` | `selected_platforms` list for platform-fit tagging |
| `_shared/hook-patterns.md` | Hook pattern library for scoring + assignment |
| `_shared/foundations.md` | Foundation B + C (headless variants) |

If `trend-sources.md` is missing or empty → write a one-line failure note to `projects/social-media-content/memory.md` and halt:

```
<ISO> | /generate-weekly-ideas | headless run — FAILED: trend-sources.md missing or empty. Run /onboard-social Phase 4.
```

The user will see the failure on next manual invocation of any cowork-social skill that reads `memory.md`.

---

## Headless behavior contract

- **No `AskUserQuestion` calls.** Not at any step.
- **No interactive plan-then-approve.** Scheduled task — `plan-then-approve: OFF` for this skill only.
- **No chat preamble.** Final stdout = `Saved <N> ideas to <path>`. Nothing more.
- **Failures append to `memory.md`** instead of halting visibly. The user finds out at the next manual run.
- **`⚡ NEXT MOVE` lives in the OUTPUT FILE footer.** Not in chat. The user opens the file Monday morning and the file itself tells them what to do next.

---

## The 5-step flow

### Step 1 — Pull from each trend source

Walk `trend-sources.md` category by category. For each entry:

| Source category | How to query (in priority order) |
|---|---|
| RSS feeds | Perplexity: "what's new in <feed name> this week?" — fall back to direct fetch if available |
| Sites to check | Firecrawl (if available + auth): scrape latest article titles + summaries — fall back to Perplexity |
| Keywords to monitor | Perplexity: "what's trending around <keyword> this week?" |
| Accounts to watch | VidIQ (for YouTube): channel outliers — IG: vidiq_ig_profile_reels — others: Perplexity "what did <handle> post recently?" |
| Podcasts | Skip in v0.1 (deferred — note in output if user had podcasts listed) |

Collect raw signals as a flat list: `{source, title, snippet, url}`.

### Step 2 — Synthesize 10 candidate ideas

Mix angles to match `/content-coach`'s 5 categories. Allow duplicate angles across the 10 (we want range, but the strongest angle can repeat if signals support it):

| Angle | Target count | Pull from |
|---|---|---|
| Contrarian belief / polarizing opinion | 2 | `brand-brief.md` wedge + contrarian signals |
| Proof story / receipts | 2 | `brand-brief.md` recent_proof_story + user data |
| Trend commentary | 2 | Step 1 trend signals |
| Personal experience / vulnerable | 2 | Suggested topics user could share |
| Customer transformation / "most people get this wrong" | 2 | Wedge + customer story patterns |

For each candidate, write:

- 1-sentence description ("What this post is about")
- 1-line draft hook (apply `_shared/hook-patterns.md`)
- Suggested hook pattern (number + name from the 11 patterns)
- Platform-fit tags (which of the user's `selected_platforms` it works best on)
- Source URL or "user wedge" / "brand-brief" if pulled from internal context

### Step 3 — Rank by virality score (0-100)

Score each idea on five dimensions. Total = sum, ceiling 100:

| Dimension | Max points | What earns full marks |
|---|---|---|
| Wedge alignment | 30 | Pulls directly from `brand-brief.md` → contrarian_belief or recent_proof_story |
| Specificity | 20 | Concrete numbers, names, dollar amounts, dates in the hook |
| Recency | 20 | Trend is fresh THIS week (from Step 1 signals); evergreen ideas cap at 10 |
| Hook ceiling | 15 | Fits a HIGH-VIRALITY pattern (#9 Receipts, #10 Reverse, #11 Stolen Lesson) — partial credit for others |
| Recognition trigger | 15 | User's audience will nod on the first sentence (pain point or shared belief) |

Rank 1-10 by total score. Break ties by Wedge alignment, then Specificity.

### Step 4 — De-duplicate against the last 30 days

Read `calendar-log.md`. For each candidate idea, compare against `post_slug` rows from the last 30 days:

- Substring overlap >= 50% on the slug or first 6 words of the hook → SKIP that candidate, pull the next-highest unused signal from Step 1
- If after de-dup you fall below 10 ideas → re-pull from Step 1 with looser keyword matching until you hit 10

Final list = exactly 10 ideas. Not more, not less.

### Step 5 — Write the output file

Resolve the output path: `projects/social-media-content/ideas/<today's-YYYY-MM-DD>-ideas.md`.

If the directory doesn't exist, create it.

Write this exact structure:

```markdown
---
type: cowork-social-weekly-ideas
plugin: cowork-social
plugin_version: 0.1.0
generated_date: <YYYY-MM-DD>
generated_time: <HH:MM ISO>
week_starting: <next Monday's YYYY-MM-DD>
total_ideas: 10
source_skill: /generate-weekly-ideas
---

# Weekly Ideas — Week of <Monday date>

> Generated by `/generate-weekly-ideas` on <ISO timestamp>. Read by `/weekly-content-session` on Monday.

## The 10 ideas (ranked)

| # | Score | Platform fit | Hook | Angle | Source |
|---|---|---|---|---|---|
| 1 | 92 | linkedin, threads | "<hook text>" | Contrarian belief | <url or "user wedge"> |
| 2 | 88 | twitter, threads | "<hook text>" | Proof story | <url or "brand-brief"> |
| 3 | 85 | instagram | "<hook text>" | Trend commentary | <url> |
| 4 | 83 | linkedin | "<hook text>" | Vulnerable | brand-brief |
| 5 | 80 | tiktok | "<hook text>" | Customer transformation | <url> |
| 6 | 78 | linkedin, facebook | "<hook text>" | Trend commentary | <url> |
| 7 | 75 | twitter | "<hook text>" | Most-people-wrong | user wedge |
| 8 | 72 | threads | "<hook text>" | Proof story | brand-brief |
| 9 | 68 | instagram, tiktok | "<hook text>" | Vulnerable | brand-brief |
| 10 | 65 | facebook | "<hook text>" | Recognition pain | <url> |

## Idea detail

### Idea 1 (92/100) — <short title>

- **What:** <1-sentence description>
- **Why it'll go viral:** <1 sentence — name the dimension that earned highest score>
- **Suggested platform(s):** <from platform-fit column>
- **Suggested hook pattern:** #<N> — <pattern_name>
- **Source:** <url or internal reference>

### Idea 2 (88/100) — <short title>

... (repeat for all 10)

---

## How to use this file

Monday morning, run `/weekly-content-session`. It'll read this file, show you the 10 ideas, and walk you through picking 5-7 to draft + schedule for the week.

---

⚡ NEXT MOVE: Run /weekly-content-session Monday at 10am to draft idea #1.
   Why: Idea #1 scored 92/100 — highest-virality candidate this week + the trend signal is fresh, the window closes by Friday.
```

The `⚡ NEXT MOVE` block is the FILE FOOTER — not chat output. The user reads it when they open the file.

---

## Foundation B — Self-improvement close (headless variant)

See `_shared/foundations.md` → Foundation B. Headless adaptation:

1. **Skip the live question.** No user present — can't ask "what would've made this 10% better?"
2. **Log the run.** Append to `projects/social-media-content/memory.md`:

   ```
   <ISO> | /generate-weekly-ideas | headless run — <N> ideas generated, top score <X>, sources used <count>
   ```

3. **Recurrence detection across headless runs.** Each Monday's `/weekly-content-session` reads the picks from the user. After 4 weeks, if the user has consistently picked 0 ideas from the generated files (compare picks in `calendar-log.md` against rows in the last 4 `ideas/*.md` files), flag for revision.

   Append to `projects/social-media-content/skill-improvements.md`:

   ```
   | /generate-weekly-ideas | low-pickup rate across 4 weeks | <first_seen_date> | 4 | re-tune ranking algorithm or trend-source coverage | reviewed: no |
   ```

   If `skill-improvements.md` doesn't exist, create it with the canonical header from `_shared/foundations.md`.

4. **Silent flag for live skills to surface.** On the next manual run of any cowork-social skill that reads `memory.md`, the headless failure or low-pickup flag becomes visible.

---

## Foundation C — `⚡ NEXT MOVE` block (file-footer variant)

See `_shared/foundations.md` → Foundation C. Headless adaptation:

- **Block location: FILE FOOTER**, not chat. The headless task has no live chat to print into.
- **Picking rule:** Name the #1 ranked idea by score, name the next concrete action (`/weekly-content-session` Monday morning), justify in one sentence tied to the highest scoring dimension.
- **Validation regex (same as canonical):** `⚡ NEXT MOVE: .+ .+ .+\n   Why: .+` — applied to the file footer text.

**Examples by top-idea state:**

- ✅ `⚡ NEXT MOVE: Run /weekly-content-session Monday at 10am to draft idea #1. Why: Idea #1 scored 92/100 — highest-virality candidate this week + the trend signal is fresh, the window closes by Friday.`
- ✅ `⚡ NEXT MOVE: Run /weekly-content-session Monday morning to start with idea #1. Why: Idea #1 hits all 3 high-virality hook patterns — pulling from your contrarian wedge with a fresh proof point.`
- ❌ `⚡ NEXT MOVE: Pick some ideas later.` (no specific subject, no timing — REGENERATE)

If the block fails the validation regex → regenerate before writing the file.

---

## Hard rules

- **10 ideas exactly.** Not 9, not 11 — Monday's session depends on this count.
- **De-duplicate against last 30 days** in `calendar-log.md`. No repeat slugs.
- **Headless = no chat noise.** Stdout = one line confirming the save path.
- **⚡ NEXT MOVE goes in the file footer.** Validated against the regex even though it's a file write, not a chat print.
- **Failures append to `memory.md`.** Never crash silently — the next live skill surfaces the failure.
- **Foundation B logs every run** (success or failure) so recurrence detection has signal.
- **Plan-then-approve: OFF** for this skill only. Scheduled task can't approve itself.
- **3rd-4th grade reading level** in the file's user-facing prose (the "How to use this file" + the NEXT MOVE).
- **Reference `_shared/foundations.md` for B + C.** Don't duplicate the rules.

---

## Voice

The output file's prose: short, direct, Monday-morning-friendly. The user is opening this groggy, with coffee. Lead them straight to the action.

The hooks themselves: full creator-voice per `_shared/hook-patterns.md`. Match the wedge, match the brand-brief — don't AI-flatten them.
