---
type: cowork-social-foundations
plugin: cowork-social
plugin_version: 0.1.0
purpose: The single source for Foundation B (self-improve close) + Foundation C (⚡ NEXT MOVE block). All 13 skills in this plugin reference this file. Do NOT duplicate this content elsewhere.
last_verified: 2026-05-15
---

# Cowork Social — Foundations

> Every skill in `cowork-social` enforces three architectural foundations (locked 2026-05-14):
>
> - **Foundation A: Lazy-load context** — no skill auto-loads. Reference docs, rubrics, hook libraries load only when the skill fires.
> - **Foundation B: Self-improving skills** — every skill ends with a learning capture + recurrence check.
> - **Foundation C: Actionable output** — every skill ends with a `⚡ NEXT MOVE` block (subject + verb + timing).
>
> This file defines B and C. (A is enforced at the skill-loading layer.)

---

## Foundation B — Self-improvement close

> Every skill, every run, after delivering the main output + the `⚡ NEXT MOVE` block, ask:

> **"What would've made this 10% better?"**

Accept a one-line answer. Then:

### 1. Append to `projects/social-media-content/memory.md`

```
<YYYY-MM-DD> | /<skill-name> | <answer verbatim>
```

If `memory.md` doesn't exist, create it with the header:

```
# Cowork Social — Skill Memory

> Append-only log. One row per skill run.

| date | skill | feedback |
|---|---|---|
```

### 2. Recurrence check

Read `memory.md` and check if any pattern recurs **3+ times** for this skill name. Match by:

- Substring overlap ≥ 60% with prior entries
- Same keyword (e.g., "hook", "voice", "CTA", "asset", "schedule", "Blotato", "platform")

### 3. If recurrence detected

Surface to the user:

> "I've seen this 3+ times. Want me to update `/<skill-name>` itself?"

If yes → draft change to `projects/social-media-content/skill-improvements.md`:

```
| /<skill-name> | <pattern> | <first_seen_date> | <recurrence_count> | <suggested_change> | <reviewed: no> |
```

If `skill-improvements.md` doesn't exist, create it with the header:

```
# Cowork Social — Skill Improvement Queue

> Draft changes flagged by recurrence detection. Review weekly; merge approved changes into the SKILL.md files.

| skill | pattern | first_seen | recurrence | suggested_change | reviewed |
|---|---|---|---|---|---|
```

### 4. If no recurrence → silent

No noise. Don't ask the user to confirm anything. The feedback row is logged; that's enough.

---

## Foundation C — `⚡ NEXT MOVE` block

> Every skill output ends with this exact block. The block MUST match the validation regex below — if it doesn't, the skill output is incomplete and must regenerate.

### The format

```
⚡ NEXT MOVE: <Subject> <Verb> <Timing>
   Why: <one-sentence reason>
```

### Validation pattern

```
⚡ NEXT MOVE: .+ .+ .+\n   Why: .+
```

If the block doesn't match, the skill output is incomplete — regenerate.

### Examples (the bar to hit)

- ✅ `⚡ NEXT MOVE: Schedule the LinkedIn draft for Tuesday 9am. Why: That's your scheduling-defaults window + the post is timely for this week.`
- ✅ `⚡ NEXT MOVE: Reply to Sarah's Instagram DM today before 5pm. Why: She asked about pricing twice + the 24-hour window matters for IG conversion.`
- ✅ `⚡ NEXT MOVE: Refresh your platform-voice.md for TikTok tomorrow morning. Why: TikTok section is still scaffold + you're posting there 2x/week now.`

### Counter-examples (REJECT and regenerate)

- ❌ `⚡ NEXT MOVE: Post more on social media.` (no subject, no timing, abstract)
- ❌ `⚡ NEXT MOVE: Try scheduling.` (no specific subject, no when)
- ❌ `⚡ NEXT MOVE: Reply to everyone.` (no specific subject)
- ❌ `⚡ NEXT MOVE: Keep drafting.` (no actionable target)

### When picking the Next Move

Priority order for social-content workflows:

1. Draft just produced + Blotato connected → "Schedule it for [default time]" (today/this-window)
2. Draft just produced + Blotato skipped → "Post it manually to [platform] at [default time]" (today/this-window)
3. Open feedback loop on a post that shipped this week → "Check engagement on [post] tomorrow + log winners"
4. Onboarding/setup skill → "Run /draft-[platform] now to live-fire what you just configured"
5. Review/retro skill → "Apply finding #1 to your next post by [day]"

Always specific. Always timed. Always justified in one sentence.

---

## How skills reference this file

Each `SKILL.md` ends with two short sections that point here — NOT inline copies of the rules:

```md
## Self-improvement close

See `_shared/foundations.md` → Foundation B. After delivering this skill's main output + the ⚡ NEXT MOVE block, ask the user "What would've made this 10% better?" and log to `projects/social-media-content/memory.md`.

## Next move

See `_shared/foundations.md` → Foundation C. Skill output must end with a `⚡ NEXT MOVE` block matching the validation regex.
```

This keeps Foundation B + C **DRY** — change them once here, every skill picks it up.
