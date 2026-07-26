# Phase 7 — First Live Draft (+ optional test schedule)

## What this phase does

The payoff phase. Picks the user's primary platform (`platforms_selected[0]` from Phase 2). Uses `/content-coach` to suggest a topic (or accepts a user-provided one). Invokes the matching `/draft-<platform>` skill. If Blotato is connected, offers to schedule a test post 15 minutes from now — then reminds the user to verify the live post + delete the test.

This is the moment the user goes from "set up" to "shipped a real post." If anything breaks, this is where we catch it before cadence locks in.

## Ask

Three questions, asked in order. Some have follow-ups based on Phase 4 / Phase 5 state.

1. *"Your primary platform is [PRIMARY] (the first one you picked in Phase 2). I'll draft something for [PRIMARY] now. Want me to suggest a topic from your `brand-brief.md`, or do you have one ready?"*

2. (If user wants a suggestion) *"Quick — what's on your mind for [PRIMARY] this week? Even one word works. I'll feed it to `/content-coach` for 3 topic ideas. Or type `pick for me` and I'll grab one from your recent_proof_story or contrarian_belief."*

3. (After draft is ready) Branch on `blotato_status`:
   - **connected**: *"Draft's done. Want me to schedule this as a test — 15 minutes from now — to confirm Blotato's wired up? You'll verify the live post yourself in 15 min and delete it. (yes / no — skip the test)"*
   - **skipped**: *"Draft's done. It's saved to `outputs/social-media-content/`. Copy + paste into [PRIMARY] when you're ready. Want to mark this phase complete?"*

## Why we ask

> *"Day 1 of any tool is when bugs hide. The fastest way to find them is to ship a real post — not a fake test — and watch what happens.*
>
> *I'll draft something for [PRIMARY] based on your brand-brief.md and platform-voice.md. It'll be a real post worth keeping. If you've got Blotato connected, we'll schedule a test 15 minutes out so you see end-to-end working. Then you delete the test and you're done.*
>
> *If you skipped Blotato, no test schedule — just copy the draft into [PRIMARY] when you're ready."*

## Logic

### Step 1 — Read state

```
read _aibos/state-social.md → primary_platform, platforms_selected, blotato_status, brand_brief_path, platform_voice_path, scheduling_defaults_path, asset_index_path, asset_nickname_count
```

Confirm `primary_platform` is set. If not, halt — Phase 2 didn't lock the order. Suggest re-running Phase 2.

### Step 2 — Resolve topic

Three sub-paths:

**A. User provides topic directly:** Skip `/content-coach`, go straight to draft.

**B. User wants a suggestion:** Invoke `/content-coach` with:
- platform = `primary_platform`
- seed = user's one-word/one-line input (or empty if `pick for me`)
- output = top 3 topic candidates

Show the user the 3 candidates. Let them pick one or ask for more.

**C. User says `pick for me`:** Read `brand-brief.md`. Pull `recent_proof_story` (preferred) or `contrarian_belief` (fallback). Convert to a topic phrase. Confirm with user before drafting.

### Step 3 — Asset check (if needed)

If `primary_platform` is `instagram` or `tiktok`:
- Read `asset_index_path`. If `asset_nickname_count == 0`, halt and ask: *"[PRIMARY] needs an image or video to schedule via Blotato. Your asset-index is empty. Three options: (1) nickname an asset right now, (2) draft without an asset (text-only — Blotato will refuse to schedule but you can post manually), (3) skip the live-fire and come back later."*
- If user picks (1), loop back to Phase 5 for one quick nickname mapping, then resume.

### Step 4 — Invoke /draft-<platform>

Build the invocation:

```
/draft-<primary_platform> "<topic>"
```

The draft skill handles its own 12-step pattern (per `_shared/draft-skill-spec.md`). For Phase 7, this wizard's umbrella approval (Step 2 above) covers the draft — the draft skill may still show a plan internally before writing.

Wait for the draft skill to return. Verify the draft file landed at `outputs/social-media-content/<slug>.md`.

### Step 5 — Branch on blotato_status

**If `connected`:**

1. Ask: *"Schedule this as a 15-minute test? (yes / no)"*
2. On yes:
   - Compute scheduled time: `now + 15 minutes` in user's timezone (from Phase 6).
   - Invoke `mcp__claude_ai_Blotato__blotato_create_post` with:
     - content = the draft body
     - platform = `primary_platform`
     - scheduled_at = `now + 15 minutes`
     - asset (if applicable) = resolved from asset-index
   - Save the returned post_id to state under `phase_7_test_post_id`.
3. Tell user: *"Test post scheduled for `[time]`. Set a 15-minute timer. When it fires:*
   - *1. Open [PRIMARY] in your browser, find the post*
   - *2. If it's live → Blotato is working end-to-end. Delete the test post.*
   - *3. If it's NOT live → check the Blotato dashboard for the failed-post reason and re-OAuth that platform if needed.*
   - *Tell me 'test passed' when you've verified + deleted it."*

**If `skipped`:**

1. Tell user: *"Draft saved at `outputs/social-media-content/<slug>.md`. Copy + paste into [PRIMARY] when you're ready. Tell me `done` to move on."*

### Step 6 — Append to calendar-log + memory

Append to `<workspace>/projects/social-media-content/calendar-log.md` (creating from template if missing):
- date, platform, topic, draft_path, scheduled_status (`test-scheduled` / `not-scheduled` / `skipped`)

Append to `<workspace>/projects/social-media-content/memory.md`:
```
<ISO timestamp> — Phase 7 demo: shipped first [PRIMARY] draft on "<topic>", blotato_status: <connected|skipped>
```

### Step 7 — Update state

Append to `_aibos/state-social.md`:

```yaml
phase_7_first_draft: completed_at <ISO timestamp>
first_draft_platform: <primary_platform>
first_draft_topic: <topic>
first_draft_path: outputs/social-media-content/<slug>.md
first_draft_scheduled: <true|false>
phase_7_test_post_id: <blotato post_id|n/a>
```

## Write

Files touched in this phase:

- `<workspace>/outputs/social-media-content/<slug>.md` — the actual draft (written by `/draft-<platform>`)
- `<workspace>/projects/social-media-content/calendar-log.md` — append (one row)
- `<workspace>/projects/social-media-content/memory.md` — append (one line)
- `_aibos/state-social.md` — append the Phase 7 block

If a test post was scheduled, no local file is written for that — it lives in Blotato's queue + the platform itself.

## Resume

After this phase completes, mark `phase_7_first_draft` as `completed_at <ISO timestamp>` in state-social.md and set `next_pending_phase: 8`.

If the user paused after the draft was generated but before the test-schedule answer, on resume re-prompt: *"Your draft is saved at [path]. Want to schedule the 15-min test now, or skip and move on?"*

## Verification before advancing

Phase 7 is complete when ALL of these are true:

- A draft file exists at `outputs/social-media-content/<slug>.md` and is non-empty
- `calendar-log.md` and `memory.md` both have the Phase 7 append
- `state-social.md` reflects `phase_7_first_draft: completed_at` + the metadata block
- If `blotato_status: connected` and user opted in to the test → the user explicitly said `test passed` (no implicit "I guess it worked")
- If `blotato_status: skipped` → the user explicitly said `done` after the draft saved

If the draft skill failed (missing files from earlier phases, malformed brand-brief, etc.) surface the failure clearly. Offer two paths: (a) loop back to the phase that produced the broken input, (b) skip Phase 7 and revisit later (mark `phase_7_first_draft: skipped`). Don't fake a "success."
