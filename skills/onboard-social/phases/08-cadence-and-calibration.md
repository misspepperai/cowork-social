# Phase 8 — Cadence + 14-day Calibration + Wizard Wrap-up

## What this phase does

Sets up the long-term rhythm so cowork-social stays useful past day 1, then runs the wizard's full wrap-up sequence:

1. **Weekly content session** — default Monday 9am — user's batch-drafting time
2. **Saturday `/generate-weekly-ideas`** — default Saturday 8am — auto-scan trend sources + suggest next week's posts
3. **Monthly `/content-engine-review`** — first Sunday of each month — review what shipped, what landed, what to tune
4. **14-day calibration check** — re-runs cowork-ai-os's `/audit` skill 14 days from today to measure 4 Cs score movement (non-optional in v0.1)

Then the wrap-up:

5. Build the ⚡ NEXT MOVE block (Foundation C — platform-aware)
6. Append onboarding-complete log line to `about-me/memory.md`
7. Show the final wrap-up message (lists all 13 skills + 14-day calibration date + NEXT MOVE leading)
8. Self-improvement close (Foundation B — "What would've made this onboarding 10% better?")
9. Mark `install_complete: true` — **THIS IS THE LAST WRITE** (crash safety)

The order is load-bearing. Do not re-shuffle.

## Ask

Four cadence questions, one per cadence. Then the canonical self-improvement question at Step 8 of wrap-up.

1. *"Schedule your weekly batch session? Default is Monday 9am — that's the slot where you'll come back to draft a week's worth of content in one sitting. Yes / change time / no."*

2. *"Schedule the Saturday content-ideas scan? Default is Saturday 8am — I'll auto-scan your `trend-sources.md` and suggest next week's post topics. Yes / change time / no (you can run `/generate-weekly-ideas` manually any time)."*

3. *"Schedule the monthly content-engine review? First Sunday of each month — I'll walk what shipped, what landed, what to tune. Yes / change time / no."*

4. *"Schedule the 14-day calibration check? This is non-optional in v0.1 — it's how we measure if this wizard actually helped. The check runs `/audit` (cowork-ai-os) on `[date 14 days from today]` and shows your 4 Cs score change."*

Q4 has no opt-out in v0.1 — explain why if pushed: *"It's the only way to know if cowork-social is doing its job. If after 14 days nothing improved, we know the wizard needs tuning. Trust the loop."*

## Why we ask

> *"Day 1 setup is the easy part. The real value of cowork-social shows up over weeks — your weekly batch session keeps content shipping without thinking, your Saturday scan keeps topics fresh, your monthly review keeps you honest about what's actually landing, and the 14-day check makes sure I'm earning my keep.*
>
> *Four short questions. Defaults are sensible — most people just hit yes."*

## Logic

### Step 1 — Schedule the weekly batch session

If yes (default time): write to `_aibos/state-social.md`:
```yaml
weekly_content_session:
  enabled: true
  cron: "0 9 * * 1"   # Monday 9am local
  next_run: <ISO date for next Monday>
  skill_to_invoke: /weekly-content-session
```

If user wanted a different time: parse + convert to cron, write that.

If no: write `weekly_content_session: { enabled: false }`.

### Step 2 — Schedule the Saturday ideas scan

If yes: write:
```yaml
weekly_ideas_scan:
  enabled: true
  cron: "0 8 * * 6"   # Saturday 8am local
  next_run: <ISO date for next Saturday>
  skill_to_invoke: /generate-weekly-ideas
```

If no: write `weekly_ideas_scan: { enabled: false }`.

### Step 3 — Schedule the monthly review

If yes: write:
```yaml
monthly_engine_review:
  enabled: true
  cron: "0 9 1-7 * 0"   # 9am on the first Sunday of the month
  next_run: <ISO date for next first-Sunday>
  skill_to_invoke: /content-engine-review
```

If no: write `monthly_engine_review: { enabled: false }`.

### Step 4 — Schedule the 14-day calibration check (mandatory)

Compute the date: `today + 14 days`. Write:
```yaml
calibration_check_14_day:
  enabled: true
  scheduled_date: <ISO date>
  audit_skill: cowork-ai-os /audit
  baseline_4cs_score: <pull from state if available, else "unknown — first audit will set baseline">
```

Tell the user the exact date in plain English: *"I've scheduled the 14-day calibration check for `[Tuesday, May 28, 2026]`. On that day, run `/audit` from cowork-ai-os (or just type `continue calibration` and I'll trigger it). It'll compare your 4 Cs score now vs. then, and we'll see what cowork-social actually moved."*

### Step 5 — Append cadence summary to memory + connections

Append to `<workspace>/projects/social-media-content/memory.md`:

```markdown
## <ISO timestamp> — Phase 8 cadence scheduled
- Weekly content session: <enabled / disabled>, runs <day + time>
- Saturday ideas scan: <enabled / disabled>, runs Saturday 8am
- Monthly engine review: <enabled / disabled>, runs first Sunday
- 14-day calibration check: scheduled for <date>
```

Append to `<workspace>/about-me/connections.md`:

```markdown
## <ISO timestamp> — added by cowork-social /onboard-social Phase 8
- Cadence scheduled: weekly_session=<bool>, weekly_ideas=<bool>, monthly_review=<bool>, 14_day_calibration=<date>
```

## Write (Phase 8 cadence portion)

- `_aibos/state-social.md` — append the 4 cadence blocks AND mark `phase_8_cadence: completed_at <ISO timestamp>`
- `<workspace>/projects/social-media-content/memory.md` — append cadence summary
- `<workspace>/about-me/connections.md` — append (append-only, never overwrite)

## Verification before wrap-up

Cadence portion of Phase 8 is complete when ALL of these are true:

- The 14-day calibration check is scheduled with a concrete `scheduled_date` (mandatory — non-optional)
- The weekly session + weekly ideas + monthly review are EITHER scheduled with valid cron OR explicitly marked `enabled: false`
- `memory.md` and `connections.md` both have the Phase 8 append
- `state-social.md` reflects all 4 cadence blocks + `phase_8_cadence: completed_at`

---

## After Phase 8 — wizard wrap-up

This is the final phase. Run the wrap-up in this exact order. **Do not re-shuffle.** The order is load-bearing: `install_complete: true` is the LAST action so a crash anywhere mid-wrap-up correctly leaves state as "not complete" and the next `/onboard-social` invocation resumes the wrap-up.

### Step 6 — Build the ⚡ NEXT MOVE block (Foundation C — platform-aware)

See [`../../_shared/foundations.md`](../../_shared/foundations.md) → Foundation C for the canonical format + validation regex.

Build the block now (don't display yet — Step 8 displays it). Pick the single highest-leverage social action the user should take NEXT, based on their `primary_platform` (from Phase 2) and the specifics in `state-social.md` / `brand-brief.md`.

**Action-picking logic by primary platform:**

| primary_platform | Pick | Timing | Why-line anchor |
|---|---|---|---|
| `linkedin` | "Reply to one comment on your last LinkedIn post" | "today before 5pm" | highest-velocity engagement multiplies your next post's reach |
| `twitter` | "Quote-tweet one post from a thought leader in your wedge" | "tomorrow morning" | quote-tweets stack faster than original posts in your first month |
| `instagram` | "Reply to 3 DMs from your last post's comment-replies" | "today before bed" | IG conversion windows close inside 24 hours |
| `facebook` | "Reply to every comment on your last Facebook post" | "today" | Facebook's algorithm rewards reply density |
| `tiktok` | "Watch + comment on 3 videos in your niche" | "today" | TikTok's For You page learns who you are by who you watch |
| `threads` | "Quote-thread one post from your trend-sources accounts" | "today" | Threads rewards real participation, not broadcast |

**Special-case overrides (use instead of the default if applicable):**

- If `phase_7_first_draft.first_draft_scheduled: false` AND `blotato_status: connected` → `⚡ NEXT MOVE: Schedule your Phase 7 draft for [primary's default time] today. Why: Your test draft is still sitting in outputs/ — ship it before the day rolls over.`
- If `blotato_status: skipped` → `⚡ NEXT MOVE: Post your Phase 7 draft manually to [primary] today before [primary's default hour]. Why: You drafted it; shipping it is the difference between cowork-social earning its keep or not.`
- If `phase_3_voice_quality: minimal` (user skipped most voice questions) → `⚡ NEXT MOVE: Refresh your platform-voice.md for [primary] tomorrow morning. Why: Skipped voice fields make every draft 10-20% weaker — fix once, every future draft improves.`

**Format the block exactly like this** (canonical — caps, leading lightning emoji, colon, space):

```
⚡ NEXT MOVE: <Subject> <Verb> <Timing>
   Why: <one-sentence reason tied to user's primary platform>
```

**Examples by primary platform:**

- LinkedIn: `⚡ NEXT MOVE: Reply to one comment on your last LinkedIn post today before 5pm. Why: Highest-velocity engagement multiplies your next post's reach.`
- Twitter: `⚡ NEXT MOVE: Quote-tweet one post from a thought leader in your wedge tomorrow morning. Why: Quote-tweets stack faster than original posts in your first month.`
- Instagram: `⚡ NEXT MOVE: Reply to 3 DMs from your last post's comment-replies today before bed. Why: IG conversion windows close inside 24 hours.`

**Counter-example (REJECT and regenerate):** `⚡ NEXT MOVE: Try the other skills.` (no subject, no timing, no specificity)

**Validation pattern (do not skip):** The block MUST match:
`⚡ NEXT MOVE: .+ .+ .+\n   Why: .+`

If it doesn't match, regenerate the block before advancing to Step 7.

### Step 7 — Append onboarding-complete log line

Append to `<workspace>/about-me/memory.md`:
```
<ISO date>: cowork-social v0.1.0 onboarded for platforms [<list>] — primary [<primary_platform>] — blotato [<connected|skipped>]
```

This is a passive log entry, not a state flag — safe to write before the user sees the wrap-up.

### Step 8 — Show the final wrap-up message

Lead with the ⚡ NEXT MOVE block from Step 6, then the rest:

> *"Wizard done. You're set up.*
>
> *⚡ NEXT MOVE: [Subject Verb Timing from Step 6]*
> *   Why: [why-line from Step 6]*
>
> *You now have 13 cowork-social skills available:*
>
> *Wizard + utility:*
> *- `/onboard-social` — this wizard, re-runnable per phase*
> *- `/brand-brief` — refresh your social brand brief*
> *- `/content-coach` — get 3 topic ideas for any platform*
> *- `/generate-weekly-ideas` — auto-scan trend sources for the week*
> *- `/grade-post` — rate a finished draft against the rubric*
> *- `/weekly-content-session` — Monday batch flow*
> *- `/content-engine-review` — monthly retro*
>
> *Draft skills (one per platform):*
> *- `/draft-linkedin` — long-form B2B*
> *- `/draft-twitter` — short-form thread/single*
> *- `/draft-instagram` — caption + asset*
> *- `/draft-facebook` — community-style*
> *- `/draft-tiktok` — short-video caption + hook*
> *- `/draft-threads` — text-first, conversational*
>
> *Calibration check is locked in for `[date]`. Come back then and we'll see if your content engine is actually working."*

### Step 9 — Self-improvement close (Foundation B — wizard variant)

See [`../../_shared/foundations.md`](../../_shared/foundations.md) → Foundation B for the canonical loop.

The wizard runs once per onboarding, but the self-improve loop still fires once at wrap-up so we learn what the wizard routinely fails at across runs. This fires AFTER the user has seen the wrap-up + NEXT MOVE, matching the gold-standard sequence.

Ask the user this exact question (one line, plain English):

> *"What would've made this onboarding 10% better?"*

Accept a one-line answer. Then:

1. Append to `<workspace>/projects/social-media-content/memory.md`:
   ```
   <YYYY-MM-DD> | /onboard-social | <answer verbatim>
   ```

2. Read `memory.md` and check if any pattern recurs 3+ times for `/onboard-social`. Pattern matching:
   - Substring overlap ≥ 60% with prior entries
   - Same keyword (e.g., "Phase 4", "Blotato", "voice", "platform", "asset", "schedule", "calibration", "demo", "too long")

3. If recurrence detected:
   - Surface: *"I've seen this 3+ times. Want me to update `/onboard-social` itself?"*
   - If yes → draft change to `<workspace>/projects/social-media-content/skill-improvements.md` in this format:
     ```
     | /onboard-social | <pattern> | <first_seen_date> | <recurrence_count> | <suggested_change> | <reviewed: no> |
     ```

4. If no recurrence → silent. No noise.

### Step 10 — Mark install_complete (LAST action — crash safety)

This MUST be the final write. If anything above crashes, state correctly reflects "not complete" and the next `/onboard-social` invocation will resume the wrap-up.

- Mark `install_complete: true` in `state-social.md`
- Confirm `completed_at: <ISO timestamp>` is written at the top of `state-social.md`

The order is non-negotiable: NEXT MOVE built → memory log → wrap-up message shown → self-improve loop → `install_complete: true` LAST.
