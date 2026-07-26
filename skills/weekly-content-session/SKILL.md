---
name: weekly-content-session
description: "The Monday 2-hour batch orchestrator. Reads this week's ideas from ideas/YYYY-MM-DD-ideas.md, walks platform-by-platform through /draft-[platform] + /grade-post + Blotato scheduling, logs to calendar-log.md. Closes the laptop with a week of content shipped. Foundation B + C applied."
when_to_use:
  - /weekly-content-session
  - weekly content session
  - run my content session
  - Monday content session
  - batch my posts
---

# /weekly-content-session — v0.1.0

The Monday 2-hour batch orchestrator. Ideas → draft → grade → schedule → log — for a whole week of content in one sitting. This skill ORCHESTRATES the platform draft skills + `/grade-post` + Blotato. It does NOT draft or grade itself.

> Inspired by Sabrina Ramonov's masterclass framing: **Hour 1 = ideas + drafts. Hour 2 = visuals + scheduling. Close laptop. Week's content is done.**

## Inputs (lazy-load — Foundation A)

Read on every run:

| File | Why |
|---|---|
| `projects/social-media-content/ideas/<latest>-ideas.md` | This week's idea bank (Saturday output) |
| `projects/social-media-content/scheduling-defaults.md` | Per-platform default day + time |
| `projects/social-media-content/calendar-log.md` | What's already scheduled (deduplicate) |
| `projects/social-media-content/asset-index.md` | Image/video lookups for IG/TikTok/FB |
| `projects/social-media-content/brand-brief.md` | Voice + wedge context |
| `_aibos/state-social.md` | `blotato_status` + `selected_platforms` |
| `_shared/foundations.md` | Foundation B + C |

If `ideas/<latest>-ideas.md` is missing → surface:

> "No ideas file found for this week. Run `/generate-weekly-ideas` first (takes ~3 min). Want me to invoke it now?"

If the user says no → halt. The session needs ideas to batch.

---

## The 6-step orchestrator flow

### Step 1 — Read ideas + show pickable list

Resolve the most recent file by date in `ideas/YYYY-MM-DD-ideas.md`. Parse the 10 ideas. Show:

```
This week's ideas (from <YYYY-MM-DD>-ideas.md):

[1]  linkedin   | "I tested 47 candle scents. Only 3 sold."
[2]  twitter    | "Most coaches think the niche is the answer. The enemy is."
[3]  instagram  | "Behind the table: 3am, before launch day."
[4]  tiktok     | "If your funnel converts under 2%, here's why."
...
[10] threads    | "The boring email subject line that doubled my opens."

Pick 5-7 ideas to turn into posts this week.
Reply with numbers (e.g., "1, 3, 4, 7, 9").
```

Wait for user input. If user picks fewer than 5 → confirm ("Only 4 picked — that's fine. Continue?"). If more than 7 → confirm ("8+ is heavy for one session. Want to drop one?").

### Step 2 — Plan-then-approve the full session

For each picked idea:

- **Platform** — use the "platform-fit" tag in the idea; if multi-platform, ask user
- **Day + time** — pull from `scheduling-defaults.md` for that platform, next-occurrence
- **Asset gate** — if platform is IG / TikTok / FB, note "needs asset"

Show the full session plan as a table:

| # | Idea (first 8 words) | Platform | Skill | Scheduled for | Asset? |
|---|---|---|---|---|---|
| 1 | "I tested 47 candle scents..." | linkedin | /draft-linkedin | Tue 9:00am ET | — |
| 3 | "Behind the table: 3am..." | instagram | /draft-instagram | Wed 11:00am ET | needs |
| ... | ... | ... | ... | ... | ... |

Ask: *"Approve this slate? Reply 'go' to start drafting, or edit a row (e.g., 'change #3 to facebook')."*

**Plan-then-approve is non-negotiable.** Do NOT start drafting until the user says go.

### Step 3 — Loop: draft + grade + (rewrite-if-low)

For each row in the approved slate (in order):

1. **Draft.** Invoke `/draft-<platform>` with the idea text as topic. Pass any asset reference from Step 5 below.
2. **Show + approve.** Show the draft to the user. Ask: *"Approve / tweak / skip?"*
   - `approve` → continue to grade
   - `tweak` → relay user's edit request back to `/draft-<platform>`, regenerate
   - `skip` → log the row as `status: skipped` in the session summary, move to next idea
3. **Grade.** Invoke `/grade-post` on the saved draft file.
4. **Show grade report.** Display the rubric scores.
5. **Branch on score:**
   - **≥ 80** → proceed to Step 4 (schedule)
   - **< 80** → offer: *"Score is <X>. The grader's top fix: '<top-3 fix #1>'. Rewrite using that fix, or accept as-is?"*
     - `rewrite` → re-invoke `/draft-<platform>` with the fix instruction; re-grade
     - `accept` → proceed to Step 4
6. **Rewrite cap.** Max **2 rewrites** per idea. After the 2nd rewrite, accept whatever's there + warn user:

   > "Used the 2-rewrite cap on this one. Final score: <X>. Moving on so we keep the batch rhythm."

### Step 4 — Schedule (Blotato-aware branching)

Read `_aibos/state-social.md` → `blotato_status`.

#### Branch A — `blotato_status: connected`

Schedule the draft via the Blotato MCP:

```
mcp__claude_ai_Blotato__blotato_create_post(
  text: <draft body>,
  accountId: <platform's accountId from blotato_list_accounts>,
  scheduledFor: <ISO 8601 with user timezone>,
  mediaUrls: <if asset attached>
)
```

On success:
- Confirm to user: *"Scheduled <platform> post for <day + time>. Blotato ID: <id>."*
- Append calendar-log row (Step 4b below)

On failure:
- Surface the error
- Fall back to manual: write draft to `outputs/social-media-content/` if not already there + tell user to post manually
- Append calendar-log row with `status: failed`

#### Branch B — `blotato_status: skipped` (or `awaiting-oauth`)

Tell user:

> "Blotato not connected. Draft saved to `outputs/social-media-content/<filename>.md`. Copy-paste into <platform> manually at <day + time>. Log when you actually post."

Append calendar-log row with `status: manual`.

#### Step 4b — Calendar-log append (always, every post)

Append ONE row to `projects/social-media-content/calendar-log.md`:

```
| <ISO date> | <platform> | <post_slug> | <scheduled_time or "manual"> | <scheduled | manual | failed | skipped> |
```

**Append-only.** Never rewrite past rows. Even on `failed` or `skipped` — log it so the session retro can see what happened.

### Step 5 — Visual attach gate (IG / TikTok / FB only)

**Before Step 4 fires for an asset-mandatory platform**, verify the draft has an asset path resolved.

- IG → required image (or carousel) per draft-skill-spec
- TikTok → required video
- FB → optional but strongly recommended

If the draft has no asset:

1. Pause Step 4 for that idea
2. Open `asset-index.md` and show 3-5 matching nicknames (search by topic keywords from the idea)
3. Ask: *"Pick an asset (nickname), upload a new one, or skip the visual?"*
4. On pick → resolve nickname → path → pass to `/draft-<platform>` to update the draft frontmatter + re-save
5. On upload → walk user through adding it to `asset-index.md` first → then resolve
6. On skip → for IG/TikTok, halt and warn ("IG won't publish without media — skip this post or add an asset"). For FB, continue without asset.

Once asset is resolved → resume Step 4 for this idea.

### Step 6 — Session wrap-up summary

After the last idea is processed, show:

```
SESSION COMPLETE — Week of <YYYY-MM-DD>
─────────────────────────────────
Drafted:     <N> posts
Scheduled:   <N> (Blotato) / <N> (manual) / <N> (skipped) / <N> (failed)
Grades:      avg <X>/100, range <min>-<max>
Time:        <wall time of session>

Calendar:
| platform | when | status |
|---|---|---|
| linkedin   | Tue 9:00am   | scheduled |
| twitter    | Tue 10:30am  | manual    |
| instagram  | Wed 11:00am  | scheduled |
| ...        | ...          | ...       |

Next ideas: Saturday <next-Saturday date> 8am — run /generate-weekly-ideas.
```

Then apply Foundation B + C.

---

## Foundation B — Self-improvement close

See `_shared/foundations.md` → Foundation B. After the session summary + the `⚡ NEXT MOVE` block, ask:

> "What would've made this 10% better?"

Append the answer to `projects/social-media-content/memory.md`:

```
<YYYY-MM-DD> | /weekly-content-session | <answer verbatim>
```

**Recurrence patterns to watch** for this skill specifically:

- "grading too slow" → consider auto-accept on score ≥ 85
- "too many platforms in one session" → suggest dropping to 5 picks default
- "scheduling failed on <platform>" → flag platform-specific Blotato auth issue
- "asset gate broke flow" → consider asset-pre-resolve in Step 2 plan

If any of those patterns hits 3+ → flag in `skill-improvements.md` per Foundation B rules.

## Foundation C — `⚡ NEXT MOVE` block

See `_shared/foundations.md` → Foundation C. The skill output ends with exactly one block matching the validation regex. **Pick by session state:**

### State 1 — All 5-7 ideas scheduled successfully (Blotato connected, no skips)

```
⚡ NEXT MOVE: Reply to comments on your last LinkedIn post for 15 minutes today.
   Why: Engagement on this week's batch compounds when you show up in the comment threads first.
```

### State 2 — Mixed (some scheduled, some manual because Blotato skipped/failed)

```
⚡ NEXT MOVE: Copy/paste the <N> manual drafts into <platform> by <day>.
   Why: Drafts saved to outputs/ but won't go live without you posting them.
```

### State 3 — Session aborted early (only some ideas drafted)

```
⚡ NEXT MOVE: Resume /weekly-content-session tomorrow morning.
   Why: <N> ideas still un-drafted — the rhythm only works if you finish the batch.
```

### State 4 — All drafted but all manual (Blotato never connected)

```
⚡ NEXT MOVE: Run /onboard-social Phase 4 to wire up Blotato before next Monday.
   Why: Manual copy-paste burns 30+ minutes per week — Blotato eliminates that.
```

Validate the block against the regex in `_shared/foundations.md`. If invalid → regenerate.

---

## Hard rules

- **Orchestrate, don't reimplement.** This skill invokes `/draft-<platform>` and `/grade-post`. It does NOT generate hooks, drafts, or scores itself.
- **Plan-then-approve at Step 2.** Show the whole session plan as a table BEFORE any draft fires. User must say "go."
- **Calendar log is append-only.** Never rewrite past entries. Even `failed` and `skipped` get rows.
- **Blotato branching is mandatory.** Both `connected` and `skipped` paths must complete the session. Never strand the user.
- **Visual attach gate runs BEFORE schedule** for IG/TikTok/FB. No "I'll add the image later."
- **Rewrite cap = 2.** After two rewrites on the same idea, accept and move on. Protects the 2-hour session length.
- **3rd-4th grade reading level in user-facing prompts.** Match Sabrina's "close the laptop" energy.
- **Reference `_shared/foundations.md` for B + C.** Don't duplicate the rules inline.

---

## Voice

The orchestrator's prompts to the user: short, direct, 3rd-4th grade reading level. Each invoked draft skill keeps its own voice (per `platform-voice.md`).

When in doubt: imagine Sabrina narrating Monday morning. Crisp, no fluff, "go go go."
