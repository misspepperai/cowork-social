# Phase 2 — Platform Selection

## What this phase does

Asks the user which of the 6 supported platforms they actually post to. Picks 1-6. Locks the selection into `state-social.md` → `platforms_selected`. Every later phase (voice capture, scheduling defaults, first live draft) operates on this subset only.

The 6 supported platforms: **LinkedIn, Twitter/X, Instagram, Facebook, TikTok, Threads.**

## Ask

One question, with a soft recommendation baked in:

> *"Which platforms do you actually post to? Pick from these 6:*
>
> *1. LinkedIn — 2. Twitter/X — 3. Instagram — 4. Facebook — 5. TikTok — 6. Threads*
>
> *Type the numbers, the names, or `all six`. You can pick 1-6.*
>
> *Heads up: picking more than 3 in v0.1 is a lot to keep up with. If you're not posting weekly to a platform already, leave it out — you can add it later by re-running this phase."*

## Why we ask

> *"Every platform has its own length, hook style, and post rhythm. Drafting for 6 platforms isn't 6x the work of drafting for 1 — it's about 1.5x — but only if I know which platforms you actually care about. I'll build the draft skills, the voice capture, and the schedule for the platforms you pick. The others stay as scaffold you can fill later."*

## Logic

### Step 1 — Parse the user's answer

Accept any of these input shapes:

- Numbers: "1, 3, 5"
- Names: "linkedin, instagram, tiktok"
- Mixed: "linkedin + 3 + threads"
- Special words: "all" / "all six" / "all of them" → all 6
- Single answer: "just linkedin" → 1 platform
- Edge case: "none" → halt. Ask: *"cowork-social only makes sense if you're posting somewhere. Want to pause and pick which platform you want to START posting to?"*

Normalize to canonical slugs: `linkedin`, `twitter`, `instagram`, `facebook`, `tiktok`, `threads`.

### Step 2 — Confirm the count

If the user picks 4+, surface a soft warning:

> *"You picked [N] platforms — that's ambitious. Most v0.1 users start with 2-3 and add more as their rhythm settles. Want to trim, or keep all [N]?"*

Accept the user's choice — don't override.

If the user picks just 1, that's fine. No warning. Many users start single-platform.

### Step 3 — Echo back the selection

Show the user the locked list:

```
Locked in:

  Platforms:    [list of picked platforms]
  Count:        [N]
  Order:        [first picked = "primary" for Phase 7 demo + Phase 8 NEXT MOVE]

The order matters — the first one is your "primary" platform.
Phase 7 will draft for the primary, and Phase 8's ⚡ NEXT MOVE
will point at the primary too. Want to reorder?
```

Let the user reorder if they want. The "primary" platform = `platforms_selected[0]`.

### Step 4 — Write to state

Append to `_aibos/state-social.md`:

```yaml
platforms_selected:
  - <slug-1>   # primary
  - <slug-2>
  - <slug-3>
primary_platform: <slug-1>
phase_2_platforms: completed_at <ISO timestamp>
```

## Write

Files touched in this phase:

- `_aibos/state-social.md` — append `platforms_selected` + `primary_platform` + `phase_2_platforms: completed_at`

No file content written yet — `platform-voice.md` is written in Phase 3, not here.

## Resume

After this phase completes, mark `phase_2_platforms` as `completed_at <ISO timestamp>` in state-social.md and set `next_pending_phase: 3`. The `platforms_selected` field MUST be populated and non-empty before advancing.

## Verification before advancing

Phase 2 is complete when ALL of these are true:

- `platforms_selected` is a non-empty list of 1-6 valid slugs in `state-social.md`
- `primary_platform` is set to the first item in that list
- The user explicitly confirmed the list and ordering

If the user wants "all six" but is brand-new to social, gently push back once: *"All six is a heavy lift if you're not already posting weekly to each one. Start with 2-3 you actually post to, add the rest later?"* — but accept their answer either way.
