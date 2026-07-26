---
type: cowork-social-hook-patterns
plugin: cowork-social
plugin_version: 0.1.0
purpose: The 11 hook patterns every /draft-* skill picks from at Step 4. Three are tagged HIGH-VIRALITY — prefer those when the topic could fit. Adapted from public templates used by top short-form creators.
last_verified: 2026-05-15
---

# Hook Patterns

> The hook is 50% of the score. Pick the pattern that fits the topic AND the user's `contrarian_belief` from `brand-brief.md`. When ambiguous, default to a high-virality pattern (#9, #10, #11).

## How to use this file

- `/draft-<platform>` Step 4 loads this file lazily (only when drafting).
- For a given topic, pick the 3 best-fitting patterns and draft one hook per pattern.
- Show the 3 hooks at Step 5 (plan-then-approve). User picks one.

## The 11 patterns

### 1. The Reframe (one-liner flip)

> Common belief, flipped in one sentence.

- **Why it works:** instant cognitive dissonance — the reader has to keep reading to resolve it.
- **When to pick:** industry critique, contrarian-take topics, any post pulling from `contrarian_belief`.
- **Example:** "Your 9-5 isn't killing your dreams. Wasting your 5-9 is."

### 2. The Parallel Contrast

> [Group A] does X. [Group B] does Y.

- **Why it works:** forces the reader into one of the two groups — they pick a side, then keep reading.
- **When to pick:** comparison posts, "us vs. them" thinking, value-system contrasts.
- **Example:** "Rich people buy time. Poor people buy stuff."

### 3. The Specific Number Hook

> Lead with a concrete, surprising number.

- **Why it works:** specificity beats generality every time — the reader trusts the post is real.
- **When to pick:** case studies, data, "I tried X times" posts.
- **Example:** "I tested 47 candle scents. Only 3 sold."

### 4. The Question Hook

> Open with a question that makes the reader stop and answer.

- **Why it works:** the reader has to think — and once they're thinking, they're scrolling slower.
- **When to pick:** reflective topics, "what if" angles, audience-recognition plays.
- **Example:** "What's the one thing you'd quit if you weren't scared?"
- **Watch out:** generic questions ("Want more leads?") tank the hook score. Be specific.

### 5. The Vulnerable Story Hook

> Drop the persona. Lead with a real moment.

- **Why it works:** real moments cut through the AI-content sea. Vulnerability triggers trust + curiosity at once.
- **When to pick:** customer stories, founder moments, mistakes-fixed posts. Pull from `brand-brief.md` → recent_proof_story.
- **Example:** "I almost shut down my shop in March. Here's what saved it."

### 6. The Contrarian Take

> State the unpopular position outright.

- **Why it works:** polarity drives comments. Comments drive algorithm boost.
- **When to pick:** strong-opinion posts. Pull from `contrarian_belief`.
- **Example:** "Most marketing advice is wrong for businesses under $10k/month."

### 7. The Pain Point List

> Stack 2-3 specific pains the reader recognizes.

- **Why it works:** the reader nods 3 times in a row — by the third nod, they're committed.
- **When to pick:** "you do this" posts, recognition-driven content, problem-aware audience.
- **Example:** "You spend hours writing posts. Nobody comments. You wonder if it's worth it."

### 8. The Behind-the-Scenes Hook

> "Here's what nobody tells you about [thing]."

- **Why it works:** insider-info framing — the reader feels invited into a secret.
- **When to pick:** how-it-really-works posts, industry insider content, "the real story" angles.
- **Example:** "Here's what nobody tells you about handmade businesses."
- **Watch out:** overused on LinkedIn — combine with #3 (Specific Number) for freshness.

### 9. The Receipts Hook ⚡ HIGH-VIRALITY

> "I [did specific thing] for [time period]. Here's what happened."
> Or: "I tested [N] [things]. Only [smaller N] worked."

- **Why it works:** specific numbers + earned right to speak + curiosity gap = the trifecta.
- **When to pick:** any post with real data, tested results, or a personal experiment.
- **Examples:**
  - "I tested 47 candle scents. Only 3 sold."
  - "I posted on Instagram every day for 90 days. Engagement dropped 60%."
  - "I called 100 cold prospects. 7 booked. Here's what the 7 had in common."

### 10. The Reverse Hook ⚡ HIGH-VIRALITY

> "Most people think [common belief]. Here's why they're wrong."
> Or: "[Audience] keeps telling me [X]. They're missing the point."

- **Why it works:** forces the reader to pick a side. Polarity → comments → algorithm boost. Pull the wedge from `brand-brief.md` → contrarian_belief.
- **When to pick:** any post tied to the user's wedge — this is what the wedge field exists for.
- **Examples:**
  - "Most candle makers think scent is the product. The product is calm."
  - "Every coach tells you to find your niche. Find your enemy first."

### 11. The Stolen Lesson Hook ⚡ HIGH-VIRALITY

> "I copied [specific thing]. Here's what happened."
> Or: "[Famous person/brand] does [specific thing]. I tried it. Result: [outcome]."

- **Why it works:** borrowed credibility + tested-by-me proof + the reader can copy it too = all 3 reasons to share.
- **When to pick:** tactical posts, how-to content, any moment you applied someone else's idea.
- **Examples:**
  - "I copied Apple's product page format. Sales went up 23%."
  - "Hormozi gives away his entire course for free. I copied that. My email list 4x'd."

---

## Pattern-picking guide (Step 4 cheat sheet)

| Topic shape | Best 2-3 patterns |
|---|---|
| Customer story or moment | #5 (Vulnerable Story), #9 (Receipts) |
| Strong opinion / wedge | #10 (Reverse), #1 (Reframe), #6 (Contrarian) |
| Numbers / data / case study | #9 (Receipts), #3 (Specific Number) |
| Industry critique | #1 (Reframe), #6 (Contrarian), #10 (Reverse) |
| Tip / how-to | #11 (Stolen Lesson), #7 (Pain Point List) |
| Recognition / "you do this" | #7 (Pain Point List), #5 (Vulnerable Story) |
| Behind-the-scenes / insider | #8 (Behind-the-Scenes), #9 (Receipts) |
| Value-system contrast | #2 (Parallel Contrast), #1 (Reframe) |
| Reflective / "what if" | #4 (Question), #5 (Vulnerable Story) |

**When in doubt:** prefer #9, #10, or #11 — they have the highest virality ceiling.

---

## The first-3-words test (apply to every hook)

Before showing the 3 candidates at Step 5, read each one's first 3 words aloud. Ask:

- Do they create curiosity, surprise, or emotional pull?
- Or do they sound like AI throat-clearing?

Pass:
- "I tested 47"
- "Most people think"
- "I almost shut"
- "Rich people buy"

Fail:
- "In today's world"
- "Here's what I"
- "Let me tell"
- "The truth is"

If any candidate fails the test, regenerate it with a different pattern before showing the user.
