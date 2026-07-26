---
name: onboard-social
description: "9-phase install wizard for the cowork-social content engine. Reads business-brain.md from cowork-ai-os, auto-derives brand-brief.md, picks 1-6 platforms, calibrates voice, walks Blotato setup, seeds asset-index, runs first draft, schedules cadence. Resumable via state.md. ~15 min."
when_to_use:
  - /onboard-social
  - onboard social
  - set up social
  - install cowork-social
  - start social onboarding
  - begin social setup
---

# /onboard-social — v0.1.0

The setup wizard for cowork-social. 9 phases (Phase 0 through Phase 8). ~15 minutes. Resumable.

You are walking the user through wiring their workspace for social-content drafting + scheduling. By the end, the user has:

- A `brand-brief.md` auto-derived from their `about-me/business-brain.md` (delta-only questions)
- A picked subset of platforms (1-6 of: LinkedIn, Twitter/X, Instagram, Facebook, TikTok, Threads)
- A per-platform `platform-voice.md` with real signature openings + closings + banned phrases
- A Blotato decision locked in (connected / new-install / skipped — all three paths work cleanly)
- An `asset-index.md` mapping nicknames to local image paths
- A `scheduling-defaults.md` with the day + hour + timezone per picked platform
- A first live draft shipped through the matching `/draft-<platform>` skill (and a test scheduled if Blotato connected)
- Weekly + monthly cadence scheduled, plus a 14-day `/audit` calibration check (cowork-ai-os)

## Prereq check

Before Phase 0, run [`checks/prereq-cowork-ai-os.md`](checks/prereq-cowork-ai-os.md). If `about-me/business-brain.md` is missing, halt with the paste-ready prompt that tells the user how to install + onboard cowork-ai-os first.

## State management

Read [`templates/state-social.md.template`](templates/state-social.md.template) and write a fresh state file to `<workspace>/_aibos/state-social.md` if it doesn't exist. If it exists with `install_complete: false`, resume at `next_pending_phase`. If `install_complete: true`, ask the user if they want to redo a specific phase.

## Phase dispatch

Phases run in order, but each can be redone independently. After each phase, pause for user confirmation before advancing.

| # | Phase | File |
|---|---|---|
| 0 | Welcome + prereq check | [`phases/00-welcome.md`](phases/00-welcome.md) |
| 1 | Auto-derive brand brief (delta-only) | [`phases/01-auto-derive-brand-brief.md`](phases/01-auto-derive-brand-brief.md) |
| 2 | Platform selection | [`phases/02-platform-selection.md`](phases/02-platform-selection.md) |
| 3 | Per-platform voice capture | [`phases/03-platform-voice-capture.md`](phases/03-platform-voice-capture.md) |
| 4 | Blotato decision + setup | [`phases/04-blotato-setup.md`](phases/04-blotato-setup.md) |
| 5 | Asset index setup | [`phases/05-asset-index.md`](phases/05-asset-index.md) |
| 6 | Scheduling defaults | [`phases/06-scheduling-defaults.md`](phases/06-scheduling-defaults.md) |
| 7 | First live draft (+ test schedule) | [`phases/07-first-live-draft.md`](phases/07-first-live-draft.md) |
| 8 | Cadence + 14-day calibration + wrap-up | [`phases/08-cadence-and-calibration.md`](phases/08-cadence-and-calibration.md) |

## Pause and resume

If the user types `pause onboarding`: save state. Tell them: *"Phase X complete. You're [N]% through. Resume any time with `/onboard-social`."*

If the user types `continue onboarding` or invokes the skill again: read `state-social.md`, jump to `next_pending_phase`.

## Phase completion protocol

After each phase's verification passes:
1. Update state file: mark phase as `completed`, set `next_pending_phase: <next>`, append timestamped log entry
2. Tell user: *"Phase N complete. You're [%] through. [One-sentence preview of next phase]. Type `continue onboarding` when ready."*

## After all phases complete

The full wrap-up sequence lives in [`phases/08-cadence-and-calibration.md`](phases/08-cadence-and-calibration.md) under "After Phase 8 — wizard wrap-up". The order is load-bearing — `install_complete: true` is the LAST write so a mid-wrap-up crash leaves state as "not complete" and the next invocation resumes the wrap-up. It runs (in order):

1. **Build the ⚡ NEXT MOVE block** — platform-aware, picks one specific action from the user's first picked platform (e.g., LinkedIn → "Reply to one comment on your last LinkedIn post today before 5pm. Why: highest-velocity engagement multiplies your next post's reach.") — must pass the canonical regex `⚡ NEXT MOVE: .+ .+ .+\n   Why: .+`
2. **Append onboarding-complete log line** to `about-me/memory.md` (passive log, not a state flag)
3. **Show the final wrap-up message** — leads with the ⚡ NEXT MOVE, lists all 13 skills the user now has, names the 14-day calibration date in plain English
4. **Self-improvement close** — ask "What would've made this onboarding 10% better?" → append to `projects/social-media-content/memory.md` → flag `/onboard-social` for revision if 3+ recurrence
5. **Mark `install_complete: true`** in `state-social.md` AS THE LAST ACTION

The 14-day calibration check itself is scheduled earlier in Phase 8 (cadence Step 4).

## Hard rules

- **Never re-collect identity** — read `about-me/` files; never overwrite
- **Append-only** on `about-me/connections.md`, `about-me/memory.md`, and `safe-zones.md`
- **Plan-then-approve** for every write outside `safe-zones.md`-declared paths
- **3rd–4th-grade reading level** for every prompt shown to the user (the wizard's voice — NOT the drafts themselves)
- **Resumable** — every phase reads + writes `state-social.md`
- **Blotato-skip path must stay clean** — every later phase works whether `blotato_status` is `connected` or `skipped`
- **No new MCP dependencies** beyond Blotato (already in `catalogs/connectors-social.md`)
- **Lazy-load (Foundation A)** — never load `_shared/foundations.md`, `_shared/hook-patterns.md`, or `_shared/draft-skill-spec.md` at the SKILL.md level; phase files load only what they need
- **Self-improving (Foundation B)** — wrap-up runs the canonical "What would've made this onboarding 10% better?" close. See `_shared/foundations.md`.
- **Actionable output (Foundation C)** — wrap-up ends with a canonical `⚡ NEXT MOVE:` block. See `_shared/foundations.md`.

## Voice

Plain English. Short sentences. One question at a time. Approval gates, not menus. Show your work — when you propose a brand-brief value, cite the line in `business-brain.md` that drove it. When you recommend a default schedule time, name why (e.g., "B2B morning scroll" for LinkedIn Tuesday 9am).

## What this skill never does

- Re-collect identity that already lives in `about-me/` (read-only on those files)
- Write outside paths declared in `safe-zones.md` without explicit approval
- Force the user to use Blotato (skip path is fully supported — 80% of cowork-social value still works)
- Hardcode platform recommendations (the user picks the subset they actually post to)
- Skip the calibration check (the 14-day `/audit` re-run is non-optional — it's how we measure if the wizard actually helped)

## Self-improvement close + Next Move (on wizard completion)

See [`_shared/foundations.md`](../_shared/foundations.md) → Foundation B + Foundation C. The full wrap-up sequence (Phase 8 Step 1-5) implements both. The canonical question is **"What would've made this onboarding 10% better?"** logged to `projects/social-media-content/memory.md`.
