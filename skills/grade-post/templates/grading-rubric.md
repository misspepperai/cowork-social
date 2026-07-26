---
type: cowork-social-grading-rubric
plugin: cowork-social
plugin_version: 0.1.0
purpose: The scoring rubric used by `/grade-post` (standalone) and the internal scoring pass inside every `/draft-*` skill. Total = 100 points. Pass threshold = 80. Hook dominates at 50%.
last_verified: 2026-05-15
---

# Grading Rubric — Cowork Social

> Be harsh but fair. An 80 is good. A 90 is strong. A 95 means almost nothing needs fixing. A 100 doesn't exist. False positives waste more time than honest feedback.

## How to use this file

- `/grade-post` reads this file when invoked standalone (user pastes a draft, asks "is this good?").
- Every `/draft-*` skill loads this rubric internally at Step 7 (self-check pass) before writing the final draft.
- Output format is the **scorecard** + **top 3 fixes** in the order defined at the bottom of this file.

## Total score

**100 points.** Pass = **80/100**. Below 80 → top 3 fixes get returned and (if inside a draft skill) the failing section regenerates.

## Dimensions + weights

| # | Dimension | Weight | What it measures |
|---|---|---|---|
| 1 | **Hook quality** | **50%** | Does the first line stop the scroll? Specifically — would someone reading the first 3-5 words keep reading? |
| 2 | Voice match | 15% | Does it sound like the user's voice as defined in `brand-brief.md` + `platform-voice.md`? |
| 3 | Length fit | 10% | Within platform sweet spot? Hook within platform-specific char window? |
| 4 | CTA clarity | 10% | One clear next action. Verb-driven. Matches the platform's reward metric. |
| 5 | Platform-native feel | 10% | Hashtag count, format, links, on-platform conventions all correct. |
| 6 | Anti-AI compliance | 5% | Passes the banned-phrase + filler-word + em-dash audit. |

> Hook = 50% is opinionated and load-bearing. A 50/50 hook with mediocre everything else scores ~75. A 25/50 hook with perfect everything else maxes at ~70. If a post comes back under 80, the fix is almost always "rewrite the hook."

## Pass threshold

**80/100.**

- 80-89: ship it
- 90-95: strong work
- 96-100: don't aim here on a draft pass — diminishing returns

## Dimension 1: Hook quality sub-rubric (the 50%)

The hook score breaks down further. Score each sub-criterion 0-10. Sum, then map to the 0-50 hook scale (sum * 1.0 = hook points, capped at 50).

| Sub-criterion | What to check | Score 0-10 |
|---|---|---|
| **Pattern match** | Does the hook use a recognized pattern from `_shared/hook-patterns.md`? Reframe, Receipts, Reverse, Stolen Lesson, Vulnerable Story, Specific Number, Contrarian Take, Question, Pain Point List, Behind-the-Scenes, Parallel Contrast. Pattern-less hooks tank here. | |
| **Specificity** | Real number, real name, real moment in the first 10 words? "I tested 47" passes. "Many people" fails. | |
| **Tension / curiosity gap** | Does it create a question the reader needs answered? Force a side to be picked? Open a loop the body closes? | |
| **First 3 words test** | Read the first 3 words alone. Do they create curiosity, surprise, or emotional pull? "Here's what I" — fail. "I almost shut" — pass. "Most people think" — pass. | |
| **Standalone-able** | Could the hook work alone as a tweet, with no body? If yes — strong. If it needs the body to make sense — weak. | |

**Hook scoring red flags (each one knocks the hook score down by at least 5):**
- Opens with "In today's world," "Let me tell you," "Here's the thing," "The truth is," "Picture this"
- Opens with a generic question every business asks ("Want more leads?")
- Opens with the brand name or business name
- Opens with a definition or industry-explainer phrase

## Dimension 2: Voice match (15%)

Compare the draft against:

1. `projects/social-media-content/brand-brief.md` → voice_signature
2. `projects/social-media-content/platform-voice.md` → the platform-specific section (length_range, signature_openings, banned_phrases, voice_notes)

Score:

- 13-15: sounds unmistakably like this user, on this platform
- 10-12: passes but generic — could be them, could be someone else
- 7-9: drifts from voice in 1-2 specific lines (call them out)
- 0-6: doesn't sound like them at all

If `platform-voice.md` is empty for this platform → score against `brand-brief.md` only and note "platform voice not yet captured" in the output.

## Dimension 3: Length fit (10%)

Check against the platform's sweet spot (from `_shared/draft-skill-spec.md` + `platform-voice.md` length_range):

| Platform | Sweet spot |
|---|---|
| Twitter / X | 60-100 chars |
| Threads | 200-350 chars |
| Bluesky | 200-280 chars |
| LinkedIn | 1,200-1,500 chars; hook ≤ 140 chars before "See more" |
| Instagram | Hook ≤ 125 chars; full caption ≤ 2,200 |
| Facebook | 40-80 chars optimal |
| TikTok caption | ≤ 150 chars |

- 10: inside sweet spot
- 7-9: outside sweet spot but inside hard limit
- 0-6: over hard limit OR wildly under (e.g. 12-char LinkedIn post)

## Dimension 4: CTA clarity (10%)

- 10: one specific verb-driven CTA matched to the platform's reward metric (comments for LinkedIn/Twitter/Threads, saves/shares for IG/FB, watch-time for TikTok)
- 7-9: clear CTA but generic ("What do you think?")
- 0-6: no CTA, multiple competing CTAs, or "Like and share!" / "Click link in bio"

## Dimension 5: Platform-native feel (10%)

| Check | Pass | Fail |
|---|---|---|
| Hashtag count | Within platform limit (0 Twitter/Threads/LinkedIn/FB, 3-5 IG, ≤5 TikTok) | Over the limit |
| External links | Honors platform rules (no body links on LinkedIn/IG; OK on Twitter/Threads) | Drops a link where the algorithm penalizes it |
| Format | Line breaks where the platform parses them; on-screen text noted for video posts | Wall-of-text on a platform that breaks on line breaks |
| Algorithm-fit | Invites the metric the platform rewards | Begs the wrong metric ("Like this!" on Twitter) |

- 10: all 4 checks pass
- 7-9: 1 fail
- 0-6: 2+ fails

## Dimension 6: Anti-AI compliance (5%)

Each check is pass / fail. Each fail subtracts 1 point (max -5).

| Check | Pass = |
|---|---|
| Em dashes | Zero em dashes (`—`) anywhere in the post |
| Contractions | `don't` over `do not`, `you've` over `you have`, etc. |
| Numbers as digits | `5 tips` not `five tips` |
| Active voice | No "was created by," "is being done," "has been built" |
| Filler words | None of: really, very, just, basically, literally, actually, simply |
| Filler openers | No: "in today's world", "let me tell you", "the truth is", "here's the thing", "picture this" |

If `projects/social-media-content/platform-voice.md` lists additional banned_phrases for this platform, fold those in.

## Top 3 fixes output format

Return the 3 changes that would raise the score most. For each:

1. **What's wrong** — quote the specific line from the draft
2. **Why it hurts** — what's the cost (less attention, sounds generic, loses the reader, algorithm penalty)
3. **Specific fix** — exact rewrite or instruction. Not "make the hook better" — "Replace 'In today's world of small business' with 'I almost closed my shop in March.'"

## Scorecard output format

```
## Post Grade: [XX]/100 — [PASS / NEEDS FIX]

### Score Breakdown

| Dimension | Weight | Score | Note |
|---|---|---|---|
| Hook quality | 50% | XX/50 | [1-line note if under 40] |
| Voice match | 15% | XX/15 | [...] |
| Length fit | 10% | XX/10 | [...] |
| CTA clarity | 10% | XX/10 | [...] |
| Platform-native feel | 10% | XX/10 | [...] |
| Anti-AI compliance | 5% | XX/5 | [...] |

### Anti-AI Audit

| Check | Pass/Fail | Violation |
|---|---|---|
| Em dashes | ... | ... |
| Contractions | ... | ... |
| Numbers as digits | ... | ... |
| Active voice | ... | ... |
| Filler words | ... | ... |
| Filler openers | ... | ... |

### Top 3 Fixes (ranked by impact)

**1. [Issue title]**
- Current: "[exact quote]"
- Why it hurts: [...]
- Fix: [specific rewrite]

**2. [Issue title]**
- Current: "[...]"
- Why it hurts: [...]
- Fix: [...]

**3. [Issue title]**
- Current: "[...]"
- Why it hurts: [...]
- Fix: [...]
```

## What NOT to do

- Don't grade leniently. A false 85 wastes more time than an honest 60.
- Don't rewrite the entire post. You're a grader — specific fix instructions only.
- Don't flag style preferences as errors. Grade against this rubric, not personal taste.
- Don't skip "why it hurts." That's where the user learns. "Move this clause" without context teaches nothing.
- Don't pad scores. If the hook is a 25, say 25.
