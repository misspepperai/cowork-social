# Phase 3 — Per-Platform Voice Capture

## What this phase does

For each platform the user picked in Phase 2, walks them through a 5-field voice capture: length range (use template default), 3 signature openings, 3 signature closings, banned phrases, voice notes. Writes the full result to `projects/social-media-content/platform-voice.md`. Unpicked platforms stay as scaffold — the draft skills only ever load their own section.

This phase is voice the way *that platform* expects it. LinkedIn-you sounds different from TikTok-you.

## Ask

For each picked platform, ask 5 questions in this order. One at a time. Don't batch.

1. *"For [PLATFORM], what 3 ways do you naturally open a post? Real lines you've used or would use — not made-up examples. Type 3 short openings, one per line. Skip if you want me to use defaults."*

2. *"What 3 ways do you naturally close a post on [PLATFORM]? Your sign-off pattern, the question you tend to ask, the CTA wording you favor. 3 lines, one per line. Skip for defaults."*

3. *"What words or phrases should I NEVER use in your [PLATFORM] posts? Common ones people block: 'literally', 'just', 'simply', 'game-changer', em dashes, hashtag spam. List them comma-separated. Skip if none."*

4. *"Anything else I should know about your voice on [PLATFORM]? Tone notes, things you do differently here vs. other platforms, formatting quirks. One line. Skip if covered already."*

Length range is NOT asked — the template's industry-baseline defaults are good (e.g., LinkedIn 1,200-1,500 chars). The user can edit `platform-voice.md` directly later.

For reference: `_shared/hook-patterns.md` documents the 10 hook patterns the draft skills use. Voice capture here feeds those patterns at draft time.

## Why we ask

> *"Your LinkedIn voice isn't your TikTok voice. Same brain, different rhythm. Drafts get 10x better when I know how YOU open and close on each platform — because that's the texture that makes a post feel like you instead of feeling like AI.*
>
> *Five short questions per platform. Most of them you can skip if you don't have a strong preference — the draft skills fall back to your `brand-brief.md` voice signature when a platform section is blank.*
>
> *Reminder: only the platforms you picked in Phase 2 need answers. The rest stay as scaffold. I'll loop through your [N] picked platforms one at a time."*

## Logic

### Step 1 — Read the template + state

```
read templates/platform-voice.md.template
read _aibos/state-social.md → platforms_selected
```

Confirm `platforms_selected` is non-empty. If it is empty, halt — Phase 2 didn't complete cleanly.

### Step 2 — Loop per picked platform

For each platform in `platforms_selected` (in order):

1. Show: *"Capturing voice for [PLATFORM]. 4 short questions. ~90 seconds. Ready?"*
2. Ask the 4 questions one at a time. Collect answers (or `skip`).
3. For each answer, store under the canonical template variable:
   - `{{<platform>_opening_1}}`, `{{<platform>_opening_2}}`, `{{<platform>_opening_3}}` (split user's 3 lines)
   - `{{<platform>_closing_1}}`, `{{<platform>_closing_2}}`, `{{<platform>_closing_3}}` (split user's 3 lines)
   - `{{<platform>_banned}}` (comma-separated list as-is)
   - `{{<platform>_voice_notes}}` (one line, as-is)
4. If the user skips a field, leave the template variable as the original placeholder (e.g., `{{linkedin_opening_1}}` stays literal). The draft skills detect literal `{{...}}` markers and fall back to brand-brief voice rules.

### Step 3 — For unpicked platforms

Leave all their template variables as-is (literal `{{...}}` placeholders). The draft skills never load these sections, so the placeholders don't matter — but the file MUST be complete so re-running this phase later (when the user adds a platform) just fills in the gaps.

### Step 4 — Build + write the file

Interpolate every captured variable. Set `{{date}}` to today's ISO date.

Show the user the rendered file as a preview, focused on the picked platforms:

```
Plan: write projects/social-media-content/platform-voice.md.

[show only the sections for picked platforms — show "[scaffold, will fill if you add this platform later]" for the others]

Approve? (yes / edit a section / restart a platform)
```

On approval, write the full file (picked + scaffold sections). On "edit a section", let the user name the section and re-do those 4 questions.

### Step 5 — Update state

Append to `_aibos/state-social.md`:

```yaml
platform_voice_captured_for:
  - <slug-1>
  - <slug-2>
  - <slug-3>
platform_voice_path: projects/social-media-content/platform-voice.md
phase_3_voice: completed_at <ISO timestamp>
```

## Write

Files touched in this phase:

- `<workspace>/projects/social-media-content/platform-voice.md` — created from template, picked platforms filled, others left as scaffold
- `_aibos/state-social.md` — append `platform_voice_captured_for` + `platform_voice_path` + `phase_3_voice: completed_at`

## Resume

After this phase completes, mark `phase_3_voice` as `completed_at <ISO timestamp>` in state-social.md and set `next_pending_phase: 4`. The `platform_voice_captured_for` list MUST match (or be a subset of) `platforms_selected`.

If the user pauses mid-loop (e.g., after 2 of 3 platforms), state-social.md should reflect partial progress: `platform_voice_captured_for: [linkedin, twitter]` even if `platforms_selected: [linkedin, twitter, instagram]`. On resume, the wizard picks up at the next un-captured platform.

## Verification before advancing

Phase 3 is complete when ALL of these are true:

- `projects/social-media-content/platform-voice.md` exists
- Every platform in `platforms_selected` has either filled values OR explicit user-confirmed skips
- `state-social.md` reflects `phase_3_voice: completed_at` and `platform_voice_captured_for` is at least as large as `platforms_selected`

If the user skips ALL 4 questions for ALL picked platforms, that's fine — the drafts will fall back to brand-brief voice. Note in state: `phase_3_voice_quality: minimal` so Phase 8's wrap-up can surface a NEXT MOVE hint to refresh this later.
