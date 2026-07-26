# Phase 1 — Auto-derive Brand Brief (delta-only)

## What this phase does

Reads `about-me/business-brain.md` (from cowork-ai-os) and auto-fills the cowork-social brand brief. Then asks the user ONLY the delta questions — the 3-4 things `business-brain.md` doesn't cover. Writes the result to `projects/social-media-content/brand-brief.md`.

This is the moment the user feels Foundation A pay off: no re-collecting identity. The wizard reads what's already there and asks only what's missing.

## Ask

Three delta questions, asked one at a time. Skip any field that's already strong in `business-brain.md`.

1. *"What's a real, recent thing that happened in your business that you could tell a story about? A customer win, a mistake you fixed, a milestone, a strange moment. One or two sentences — I'll use it as raw material for the 'Vulnerable Story' hook pattern."*

2. *"What's a strong opinion you hold that most people in your industry would push back on? Your contrarian belief. This is the single biggest viral fuel in this whole file — hooks like 'Reframe', 'Contrarian Take', and 'Reverse' all pull from it."*

3. *"In 5 words or fewer — how would you describe your voice? (e.g., 'direct, dry, slightly profane' or 'warm, generous, professorial'.) Skip if you already wrote this in `about-me/voice.md` — I'll find it."*

If any of the 5 auto-derivable fields look weak after the read, ask one extra question for that field too.

## Why we ask

> *"Your `business-brain.md` already covers what you sell, who buys it, and where you want them to go. I read all that. I'm not making you write it again.*
>
> *But three things matter MORE for social than they do for general business writing — a recent story, your contrarian belief, and your voice signature. Those drive the hooks. I'll ask you about those, then save the whole brief to `projects/social-media-content/brand-brief.md`."*

## Logic

### Step 1 — Read business-brain.md + voice.md

```
read <workspace>/about-me/business-brain.md
read <workspace>/about-me/voice.md   (if exists)
```

If `business-brain.md` is empty or under 100 characters, halt: *"Your `business-brain.md` looks empty or unfinished. Run `/onboard` from cowork-ai-os to fill it in, then come back."*

### Step 2 — Auto-derive the 5 brief fields from business-brain.md

The brief template has 6 fields. Map them like this:

| brand-brief.md field | Source in business-brain.md | Fallback |
|---|---|---|
| `what_you_sell` | "What I sell" / "Offer" / "Product" section | Ask: "What do you sell, in plain words?" |
| `target_audience` | "Customer persona" / "Ideal client" / "Audience" section | Ask: "Describe one specific human who buys from you." |
| `primary_cta` | "Primary CTA" / "Main offer link" / "Call to action" section | Ask: "What ONE action do you want a reader to take after seeing a post?" |
| `recent_proof_story` | Scan for "win", "case study", "milestone", "story" | **Always ask** (this is delta question 1) |
| `contrarian_belief` | Scan for "wedge", "contrarian", "I believe", "opinion" | **Always ask** (this is delta question 2) |
| `voice_signature` | Read `about-me/voice.md` if exists; else scan business-brain.md for "voice", "tone", "style" | Ask delta question 3 |

For each auto-derivable field, pull the exact line(s) from `business-brain.md` that fill it. Cache the citations for Step 3.

### Step 3 — Show the auto-derived draft, then ask deltas

Show the user:

```
Here's what I pulled from your business-brain.md:

  What you sell:        [auto-derived value]
    (from: "[cited line from business-brain.md]")

  Target audience:      [auto-derived value]
    (from: "[cited line]")

  Primary CTA:          [auto-derived value]
    (from: "[cited line]")

  Voice signature:      [auto-derived value OR "TBD — I'll ask in a sec"]

Now I need 3 short things business-brain.md doesn't cover...
```

Then ask the delta questions one at a time. Accept short answers. Don't push for paragraphs.

### Step 4 — Build the brief

Read [`../templates/brand-brief.md.template`](../templates/brand-brief.md.template). Interpolate:

- `{{what_you_sell}}` → auto-derived value
- `{{target_audience}}` → auto-derived value
- `{{primary_cta}}` → auto-derived value
- `{{recent_proof_story}}` → user's delta answer #1
- `{{contrarian_belief}}` → user's delta answer #2
- `{{voice_signature}}` → from voice.md if found, else delta answer #3
- `{{date}}` → today's ISO date

If any field is still blank after the deltas, write `TBD` — the draft skills tolerate `TBD` but degrade quality.

### Step 5 — Plan-then-approve the write

Show the user the rendered brief as a preview:

```
Plan: write this to projects/social-media-content/brand-brief.md.

[show the rendered brief — full content, with TBDs called out]

Approve? (yes / edit a field / start over)
```

On approval, write the file. On "edit a field", let the user pick a field by name and replace just that section.

## Write

Files touched in this phase:

- `<workspace>/projects/social-media-content/brand-brief.md` — created from template, fully rendered
- `_aibos/state-social.md` — append `phase_1_brand_brief: completed_at <ISO timestamp>`, set `brand_brief_path: projects/social-media-content/brand-brief.md`

No writes to `about-me/` files. No safe-zones changes.

## Resume

After this phase completes, mark `phase_1_brand_brief` as `completed_at <ISO timestamp>` in state-social.md and set `next_pending_phase: 2`. The `brand_brief_path` field MUST be populated before advancing — every later draft skill reads it.

## Verification before advancing

Phase 1 is complete when ALL of these are true:

- `projects/social-media-content/brand-brief.md` exists and is non-empty
- All 6 fields are filled (or explicitly `TBD` if the user skipped)
- `state-social.md` reflects `phase_1_brand_brief: completed_at` and `brand_brief_path` is set
- The user explicitly approved the rendered brief

If the user keeps rejecting the auto-derived values, that's a signal `business-brain.md` is stale. Suggest: *"Your `business-brain.md` might be out of date — want to refresh it in cowork-ai-os first, or keep going with what we have?"*
