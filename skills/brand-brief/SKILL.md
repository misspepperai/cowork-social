---
name: brand-brief
description: "Capture or refresh the brand brief that powers every content skill. Auto-derives 5 of 6 fields from about-me/business-brain.md; asks delta questions only. Saves to projects/social-media-content/brand-brief.md. Foundation B + C applied."
when_to_use:
  - /brand-brief
  - refresh my brand brief
  - update brand brief
  - capture brand brief
---

# /brand-brief — v0.1.0

Capture or refresh the brand brief used by every content-drafting skill in this plugin. Auto-derives most fields from `about-me/business-brain.md` and only asks the user for the delta the vault can't answer.

## When to fire

- User types `/brand-brief`, "refresh my brand brief", "update brand brief", or any trigger above
- Another skill (e.g., `/content-coach`, `/draft-linkedin`) needs the brief but the file is missing → invoke this skill silently, then continue

## Inputs (lazy-load — read only when this skill fires)

- `about-me/about-me.md` (user identity)
- `about-me/business-brain.md` (**primary** — most fields derive from here)
- `projects/social-media-content/brand-brief.md` (if exists — refresh mode)
- `projects/social-media-content/memory.md` (past self-improvement notes)
- `skills/brand-brief/templates/brand-brief.md.template` (output shape)

## Logic

### Step 1 — Detect mode

- If `projects/social-media-content/brand-brief.md` does NOT exist → **CREATE mode**
- If it exists → **REFRESH mode** (show the user the current brief one section at a time, ask "still right?")

### Step 2 — Auto-derive from `business-brain.md`

Map the vault to the brief:

| Brief field | Pull from `business-brain.md` |
|---|---|
| `what_you_sell` | product / service description ("What I sell") |
| `target_audience` | ICP / customer persona section |
| `primary_cta` | primary call-to-action / primary offer |
| `voice_signature` | brand-voice / tone descriptors section (if present) |
| `contrarian_belief` | "strong opinions" or differentiator section (if present) |

Show your work to the user in plain words. Example:

> "I read your business-brain. Here's what I'm pulling in:
> - **What you sell:** AI marketing coaching for solopreneurs
> - **Who buys:** coaches who feel stuck swapping time for money
> - **Primary CTA:** book a free consult
>
> Still right? (yes / fix something)"

If a field is missing or stale in `business-brain.md`, flag it for Step 3.

### Step 3 — Ask delta questions only

Never re-ask what the vault already answers. The brief has six fields; the vault usually fills 3-5 of them. Ask only what's missing, one question at a time. Max three questions per run.

Canonical delta questions (use these exact words at 3rd-4th grade reading level):

1. **Recent proof story** — "What's a recent client or project win we can pull from?"
2. **Contrarian belief** — "What's a belief you hold about your industry that most people would push back on?"
3. **Voice signature** — "Three words that describe how you sound when you write?"

If the user is stuck on the contrarian belief, push gently:
> "Pick one. What's a habit other [industry] people have that you think is a mistake? What's common advice you ignore?"

If still stuck, flag in the brief as `TBD` and tell them: "We'll come back to this. Watch competitor content this week — note what bothers you."

### Step 4 — Plan-then-approve write

Assemble the full brief in memory. Show it to the user. Ask:

> "Save this to `projects/social-media-content/brand-brief.md`? (yes / fix something)"

Do not write before approval.

### Step 5 — Write

On approval, write the file using the shape in `templates/brand-brief.md.template`. Replace `{{date}}` with today's ISO date.

If a vault file is missing the derived value, fill the brief field with `TBD` rather than making something up.

### Step 6 — Index the run

Append one line to `projects/social-media-content/memory.md`:

```
<ISO timestamp> | /brand-brief | <captured / refreshed>
```

If `memory.md` doesn't exist, create it with the canonical header from `_shared/foundations.md` → Foundation B.

### Step 7 — Self-improvement close + ⚡ NEXT MOVE

See **Self-improvement close** + **Next move** sections below.

## Output

- `projects/social-media-content/brand-brief.md` (the brief — professional voice, scannable)
- `projects/social-media-content/memory.md` (one-line index append)

## Hard rules

- **Lazy-load** — never read these inputs on session start; only when this skill fires
- **Don't re-ask the vault** — read `business-brain.md` first, then ask only the delta
- **Plan-then-approve** before any write
- **No new MCP dependencies** — Read + Write only
- **3rd-4th grade reading level** in the wizard's prompts to the user
- **Professional voice** in the brief file itself (it's the user's deliverable)

## Voice for the brief itself

Short paragraphs. Scannable. No marketing-speak in the Voice section — use the user's actual words.

## Self-improvement close

See [`_shared/foundations.md`](../_shared/foundations.md) → Foundation B. After delivering the brief + the `⚡ NEXT MOVE` block, ask the user:

> **"What would've made this 10% better?"**

Append the answer to `projects/social-media-content/memory.md` in the canonical row format. Run the 60%-overlap / 3+ recurrence check. If a pattern recurs, surface it and offer to stage a draft change in `projects/social-media-content/skill-improvements.md`.

## Next move

See [`_shared/foundations.md`](../_shared/foundations.md) → Foundation C. End every run with the canonical block. The block MUST match the validation regex:

```
⚡ NEXT MOVE: .+ .+ .+\n   Why: .+
```

If it doesn't match, regenerate.

### Examples to hit the bar

- ✅ `⚡ NEXT MOVE: Run /draft-linkedin on your new contrarian belief tomorrow morning. Why: it's the sharpest angle in the brief and LinkedIn rewards strong opinions.`
- ✅ `⚡ NEXT MOVE: Run /content-coach today before lunch. Why: you now have a brief, so the coach can generate 5 brand-tied ideas instead of generic ones.`
- ❌ `⚡ NEXT MOVE: Use the brief.` (no subject, no timing — regenerate)
