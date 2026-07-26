---
name: grade-post
description: "Score any draft post against the grading rubric (hook 50%, voice 15%, length 10%, CTA 10%, platform-native 10%, anti-AI 5%). Returns score + top 3 fixes. Pass threshold 80/100. Read-only, never writes the draft."
when_to_use:
  - /grade-post
  - grade this post
  - score my draft
  - is this any good
  - grade post
---

# /grade-post — v0.1.0

Score a draft against the rubric. Returns 0-100 + top 3 fixes. **Read-only — never writes the draft.**

> Be harsh but fair. An 80 is good. A 90 is strong. A 100 doesn't exist. False positives waste more time than honest feedback.

## Inputs

- **Required:** post content — EITHER a file path (e.g., `outputs/social-media-content/2026-05-15-linkedin-foo.md`) OR pasted text
- **Read on every run (lazy-load):**
  - `cowork-social/skills/grade-post/templates/grading-rubric.md` (the rubric — load on every run)
  - `about-me/writing-rules.md` (anti-AI banned-phrase list)
  - `projects/social-media-content/platform-voice.md` (load only the section matching the post's platform)
  - `projects/social-media-content/brand-brief.md` (voice_signature for voice-match check)
  - `_shared/hook-patterns.md` (referenced by the rubric's hook sub-rubric)

## Logic

### Step 1 — Resolve the input

- If the user passed a file path → read it.
- If pasted text → use directly.
- Identify the platform:
  - From filename pattern (`-linkedin-`, `-twitter-`, `-threads-`, `-bluesky-`, `-instagram-`, `-facebook-`, `-tiktok-`)
  - From content cues (length, hashtag pattern, line breaks)
  - If ambiguous → ASK the user which platform.

### Step 2 — Apply the 6-dimension rubric

Walk every dimension of `templates/grading-rubric.md` verbatim. Do NOT drift the weights.

| # | Dimension | Weight | Source check |
|---|---|---|---|
| 1 | **Hook quality** | **50%** | First 1-2 sentences against `_shared/hook-patterns.md` sub-rubric (pattern match, specificity, tension, first-3-words test, standalone-able) |
| 2 | Voice match | 15% | Compare against `platform-voice.md` (platform section) + `brand-brief.md` voice_signature |
| 3 | Length fit | 10% | Platform sweet spot from rubric table |
| 4 | CTA clarity | 10% | One verb-driven CTA matched to platform's reward metric |
| 5 | Platform-native feel | 10% | Hashtag count + link rules + format + algorithm-fit |
| 6 | Anti-AI compliance | 5% | `writing-rules.md` banned-phrase list + rubric's universal checks |

Score each dimension per the rubric. Sum to 0-100.

### Step 3 — Top 3 fixes (always exactly 3)

Pick the **3 highest-ROI fixes** — the changes that gain the most points per word changed. Ranked by point-gain potential.

**Hard rule: Top 3 fixes only. Not 5. Not 7. Forcing prioritization is part of the value.**

For each fix:
1. **What's wrong** — quote the specific line
2. **Why it hurts** — what's the cost (attention, voice, algorithm penalty, etc.)
3. **Specific fix** — exact rewrite (not "make the hook better" — give the replacement line)
4. **Estimated lift** — points gained if applied

### Step 4 — Verdict

| Score | Verdict |
|---|---|
| 80-100 | **PASS** — ship it |
| 60-79 | **REWORK** — apply top 3 fixes + re-grade |
| 0-59 | **REGENERATE** — too many dimensions failing; regenerate from scratch |

### Step 5 — Display the scorecard

Use the rubric's "Scorecard output format" section verbatim:

```
## Post Grade: XX/100 — [PASS / REWORK / REGENERATE]

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

**1. [Issue title] [+X pts]**
- Current: "[exact quote]"
- Why it hurts: [...]
- Fix: [specific rewrite]

**2. [Issue title] [+X pts]**
- Current: "[exact quote]"
- Why it hurts: [...]
- Fix: [specific rewrite]

**3. [Issue title] [+X pts]**
- Current: "[exact quote]"
- Why it hurts: [...]
- Fix: [specific rewrite]
```

## Hard rules

- **READ-ONLY** — NEVER modify the draft being graded. This is a grader, not a writer.
- **Use the rubric VERBATIM** — don't drift weights. Hook is 50%. Period.
- **Top 3 fixes ONLY** — not 5, not 7. Forcing prioritization is part of the value.
- **Top 3 fixes must be ACTIONABLE** — specific replacement text, not "consider strengthening"
- **Don't pad scores** — if the hook is a 25, say 25. A false 85 wastes more time than an honest 60.
- **Don't rewrite the post** — fix instructions only.
- **3rd-4th grade reading level** in user-facing prompts.
- **Plan-then-approve only applies to the Foundation B memory.md append** (not to the grade output itself — the grade displays immediately).

## Voice

Direct, scored, no flattery. The grader's job is to find the weak link, not encourage. If a hook is a 25, the grader says 25 — and tells the user exactly how to push it to 40+.

## Self-improvement close (Foundation B)

See `_shared/foundations.md` → Foundation B. After delivering this skill's main output + the ⚡ NEXT MOVE block, ask the user:

> **"What would've made this 10% better?"**

Then:

1. Append to `projects/social-media-content/memory.md`:
   ```
   <YYYY-MM-DD> | /grade-post | <answer verbatim>
   ```

2. Read `memory.md` and check if any pattern recurs 3+ times for `/grade-post`. Match by:
   - Substring overlap ≥ 60% with prior entries
   - Same keyword (e.g., "hook", "voice", "fix", "platform", "weight", "rubric")

3. If recurrence detected → surface: "I've seen this 3+ times. Want me to update `/grade-post` itself?" → on yes, draft change to `projects/social-media-content/skill-improvements.md`.

4. If no recurrence → silent.

## Actionable close (Foundation C)

See `_shared/foundations.md` → Foundation C. Skill output MUST end with a `⚡ NEXT MOVE` block matching the validation regex:

```
⚡ NEXT MOVE: .+ .+ .+\n   Why: .+
```

The Next Move is verdict-aware. Pick from these three templates:

### If PASS (≥80)
```
⚡ NEXT MOVE: Schedule this <platform> draft via Blotato within the next hour.
   Why: Score is above your usual ceiling — momentum compounds when you ship on a high.
```

### If REWORK (60-79)
```
⚡ NEXT MOVE: Rewrite the hook using the '<pattern>' pattern from hook-patterns.md before scheduling.
   Why: Hook is <X>/50 — pulling it to ≥40 lifts the total score most efficiently.
```

(Substitute the specific hook pattern that would best fit this post's angle, picked from `_shared/hook-patterns.md`: Receipts, Reframe, Reverse, Stolen Lesson, Vulnerable Story, Specific Number, Contrarian Take, Question, Pain Point List, Behind-the-Scenes, Parallel Contrast.)

### If REGENERATE (<60)
```
⚡ NEXT MOVE: Re-run /draft-<platform> on this topic with a tighter angle today.
   Why: Multiple dimensions failing — faster to regenerate than patch.
```

### Validation
The block MUST match: `⚡ NEXT MOVE: .+ .+ .+\n   Why: .+`

If it doesn't match, the output is incomplete — regenerate the close.

## What NOT to do

- Don't rewrite the post (you're a grader, not a writer)
- Don't grade leniently (a false 85 wastes more time than an honest 60)
- Don't return more than 3 fixes (forcing prioritization is the value)
- Don't skip "why it hurts" (that's where the user learns)
- Don't flag style preferences as errors (grade against the rubric, not personal taste)
- Don't pad scores
