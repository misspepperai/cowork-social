---
name: content-coach
description: "Generate 5 platform-aware post ideas tied to brand-brief.md + business-brain.md. Entry point for \"I don't know what to post.\" Falls back to /brand-brief if no brief exists. Foundation B + C applied."
when_to_use:
  - /content-coach
  - give me post ideas
  - I don't know what to post
  - what should I post about
  - content ideas
---

# /content-coach — v0.1.0

The front door for "I don't know what to post." Generates 5 SPECIFIC post ideas tied to the user's brand, each with a hook, a platform fit, and the draft skill to invoke next.

## When to fire

- User types `/content-coach`, "give me post ideas", "I don't know what to post", "what should I post about", or any trigger above
- User shows up vague — no topic, no platform, just wants to post
- If they give you a specific topic AND platform, skip to `/draft-<platform>` directly
- If they paste an existing draft, skip to `/grade-post`

## Inputs (lazy-load — read only when this skill fires)

- `projects/social-media-content/brand-brief.md` (**required** — if missing, invoke `/brand-brief` first)
- `about-me/business-brain.md` (extra context: ICP, offers, terminology)
- `projects/social-media-content/calendar-log.md` (last 30 days — don't repeat recent topics)
- `projects/social-media-content/trend-sources.md` (optional — exists if `/generate-weekly-ideas` Phase 6 ran)
- `skills/_shared/hook-patterns.md` (hook library)
- `projects/social-media-content/memory.md` (past self-improvement notes)

## Logic

### Step 1 — Check for the brand brief

If `projects/social-media-content/brand-brief.md` is missing:

- Tell the user in plain words: "I need your brand brief first. Let's set it up — it takes 2 minutes."
- Invoke `/brand-brief` and walk the user through it. Don't make them run two commands.
- When `/brand-brief` finishes, continue with Step 2.

### Step 2 — Read the brief + recent posts

- Pull `Contrarian belief` + `Recent proof story` + `Target audience` + `Primary CTA` from `brand-brief.md`
- Skim the last 30 days of `calendar-log.md` — note topics + hooks already used so you don't repeat them
- If `trend-sources.md` exists, skim the top 3 trends — one of your 5 ideas can react to one

### Step 3 — Generate 5 ideas (mix of angles)

Produce 5 **SPECIFIC** ideas (not generic). Each idea must clearly tie to one of the brand-brief sections. Cover this mix — at least one of each:

1. **Contrarian belief expansion** — pull from the wedge. Frame as "Most [industry] think X. Here's why they're wrong." Polarity drives comments.
2. **Client / proof story** — pull from `Recent proof story`. Specific person, specific before/after.
3. **Industry trend commentary** — only if `trend-sources.md` exists. React to a current trend with the user's wedge angle. Skip if no trend file.
4. **Personal experience reframed for the ICP** — a small failure, mistake, or realization the user had, reframed as the lesson their ICP needs.
5. **Wedge / controversial-in-niche idea** — something specific to the user's industry that would make peers uncomfortable. Different from #1: more tactical, less philosophical.

If `trend-sources.md` is missing, replace #3 with a second contrarian or a "most people get this wrong" callout — never leave only 4 ideas.

### Step 4 — Format each idea

For each of the 5, output exactly:

```
**Idea N — <one-line title>**

- **Hook:** <one specific hook line, not a topic summary>
- **Why it lands:** <one sentence — which emotion, which metric, why it's tied to the brand>
- **Best platform:** <one of LinkedIn / Instagram / Facebook / Twitter / TikTok / Threads>
- **Next skill:** `/draft-<platform> <idea-title>`
```

Hard rules for ideas:
- **Specific, not generic.** Reject "share a tip about productivity." Accept "Why I stopped time-blocking for clients after 3 months of testing."
- **One concrete idea per post.** Not three stacked.
- **Don't repeat last-30-days topics.** Cross-check `calendar-log.md`.

### Step 5 — Present + ask

Show all 5 as a numbered list with the format above. Then ask in plain words:

> "Which one speaks to you? Pick a number, or tell me to brainstorm 5 more."

If the user picks a weak idea, write it well but flag it:

> "I'll draft this — heads up, it's likely to land softer than #2 because <reason>. Want me to draft this one, or pick a stronger angle?"

### Step 6 — Hand off (or save)

When the user picks a number:
- Auto-invoke `/draft-<platform>` with the idea title as topic. Don't make them re-type it.

If the user says "save these for later" instead:
- Plan-then-approve before writing.
- Append the 5 ideas to `projects/social-media-content/ideas/<YYYY-MM-DD>-ideas.md`. Create the `ideas/` folder if missing.

### Step 7 — Index the run

Append one line to `projects/social-media-content/memory.md`:

```
<ISO timestamp> | /content-coach | <ideas generated / handed off to /draft-X / saved>
```

### Step 8 — Self-improvement close + ⚡ NEXT MOVE

See **Self-improvement close** + **Next move** sections below.

## Output

- 5 ideas presented to the user (numbered list, format above)
- Optional: `projects/social-media-content/ideas/<YYYY-MM-DD>-ideas.md` (if user says save)
- `projects/social-media-content/memory.md` (one-line index append)

## Hard rules

- **Brand-brief required** — if missing, auto-invoke `/brand-brief` (don't tell user to run a second command)
- **Specific, not generic** — reject any idea that could be posted by any business in any industry
- **Mix angles** — at least one contrarian + one proof story in every run
- **Avoid repeats** — cross-check `calendar-log.md` for the last 30 days
- **Plan-then-approve** before any write
- **Lazy-load** — never load these inputs on session start
- **No new MCP dependencies** — Read + Write only
- **3rd-4th grade reading level** in the wizard's prompts to the user

## Voice

Conversational. Coach-like: opinionated, encouraging, willing to push back if the user picks the weakest idea. Don't lecture about algorithms unless asked. Don't dump frameworks on them — apply silently, show the result.

## Self-improvement close

See [`_shared/foundations.md`](../_shared/foundations.md) → Foundation B. After delivering the 5 ideas + the `⚡ NEXT MOVE` block, ask the user:

> **"What would've made this 10% better?"**

Append the answer to `projects/social-media-content/memory.md` in the canonical row format. Run the 60%-overlap / 3+ recurrence check. If a pattern recurs, surface it and offer to stage a draft change in `projects/social-media-content/skill-improvements.md`.

## Next move

See [`_shared/foundations.md`](../_shared/foundations.md) → Foundation C. End every run with the canonical block. The block MUST match the validation regex:

```
⚡ NEXT MOVE: .+ .+ .+\n   Why: .+
```

If it doesn't match, regenerate.

### Examples to hit the bar

- ✅ `⚡ NEXT MOVE: Pick idea #2 and run /draft-linkedin within the hour. Why: it's the proof story — highest-conversion angle given your stated CTA.`
- ✅ `⚡ NEXT MOVE: Run /draft-twitter on idea #1 today before 3pm. Why: the contrarian wedge is sharpest and Twitter rewards polarity in the morning window.`
- ❌ `⚡ NEXT MOVE: Pick an idea and post it.` (no subject, no timing — regenerate)
