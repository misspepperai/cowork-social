---
type: cowork-social-draft-skill-spec
plugin: cowork-social
plugin_version: 0.1.0
purpose: The 12-step pattern all 6 platform draft skills follow — /draft-linkedin, /draft-twitter, /draft-instagram, /draft-facebook, /draft-tiktok, /draft-threads. Each platform skill SKILL.md references this file and provides only the platform-specific deltas (length, hook window, hashtag rules, asset requirements).
last_verified: 2026-05-15
---

# Cowork Social — Shared Draft-Skill Spec

> Every `/draft-<platform>` skill in this plugin follows these 12 steps in order. Per-platform SKILL.md files override only the parts that need platform-specific values (length range, hook character window, hashtag rules, asset requirement).

## The 12 steps

### Step 1: Load context (lazy)

Read these 4 files. If any is missing, halt and tell the user which `/onboard-social` phase to re-run.

| File | What it provides |
|---|---|
| `about-me/writing-rules.md` | User's universal writing rules (apply across every plugin) |
| `about-me/business-brain.md` | Business context, audience, recent stories, wedge |
| `projects/social-media-content/brand-brief.md` | Cowork-social-specific brief (what_you_sell, target_audience, primary_cta, recent_proof_story, contrarian_belief, voice_signature) |
| `projects/social-media-content/platform-voice.md` | **Only the PLATFORM section for this skill** — length_range, signature_openings, signature_closings, banned_phrases, voice_notes |

> **Lazy-load rule:** read ONLY the platform's section of `platform-voice.md`. Don't load the whole file. Foundation A.

### Step 2: Check anti-AI banned-phrase list

Pull the banned-phrase set:

1. Universal banned openers (from `_shared/foundations.md` Anti-AI rules): "in today's world," "let me tell you," "the truth is," "here's the thing," "picture this"
2. Universal filler words: really, very, just, basically, literally, actually, simply
3. Em dashes (zero allowed)
4. Platform-specific `banned_phrases` from `platform-voice.md`

Hold this set in working memory for Step 4 + Step 7.

### Step 3: Parse user input

Extract from the user's invocation:

- **Topic** — what to post about (required)
- **Asset reference** — optional nickname matching a row in `projects/social-media-content/asset-index.md`
- **Override hints** — e.g. "make it short," "use the receipts pattern," "schedule for Friday"

If topic is missing → ask once: "What's the post about?" Don't fill in a default.

### Step 4: Generate 3 hook candidates

Load `_shared/hook-patterns.md`. For this topic + the user's `contrarian_belief` from `brand-brief.md`, pick the 3 best-matching patterns. **Prefer high-virality patterns** (Receipts #9, Reverse #10, Stolen Lesson #11) when the topic could fit them.

Draft 3 hook variations using different patterns. Each hook:

- Specific number, name, or moment in the first 10 words
- Creates a question the reader needs answered
- Could stand alone as a tweet
- Passes the first-3-words test (does it create curiosity, surprise, or emotional pull?)
- No banned phrases from Step 2

### Step 5: Plan-then-approve the draft

Before writing the full draft, show the user:

```
**3 hook candidates** (pick one, or say "different angle"):

1. [Pattern name] — [hook text]
2. [Pattern name] — [hook text]
3. [Pattern name] — [hook text]

**Body outline:** [3-bullet outline of where the post is heading]

**CTA candidate:** [the verb-driven action from brand-brief.md, adapted to platform]

**Target length:** [from platform-voice.md length_range]
```

Wait for the user to pick a hook (or request a different angle) before writing the full draft. **Plan-then-approve is non-negotiable** — never dump a finished 1,500-char LinkedIn draft on the user without showing the hook + outline first.

### Step 6: Write the full draft

On hook approval, write the full post into:

```
outputs/social-media-content/YYYY-MM-DD-<platform>-<slug>.md
```

Where `<slug>` is a 3-5-word kebab-case summary of the topic (e.g. `2026-05-15-linkedin-most-people-miss-this.md`).

Frontmatter format:

```yaml
---
date: 2026-05-15
platform: linkedin
slug: most-people-miss-this
hook_pattern: reverse
length_chars: 1342
asset_used: null
status: drafted
---
```

Body = the actual post text, ready to copy-paste.

### Step 7: Self-check against anti-AI rules

Re-read the draft. Run the Step 2 banned-phrase set against every line. If any banned phrase or filler word appears:

1. Flag the line + the violation
2. Regenerate ONLY that section (not the whole post)
3. Re-check until clean

Also run the `grade-post` rubric (load `skills/grade-post/templates/grading-rubric.md`) as an internal scoring pass. Target **80+**. If under 80, identify the lowest-scoring dimension and regenerate just that part (typically the hook).

### Step 8: Append to calendar log

Append one row to `projects/social-media-content/calendar-log.md`:

```
| YYYY-MM-DD | <platform> | <post_slug> | -- | drafted |
```

The `scheduled_time` column stays `--` until Step 10 (if scheduling happens).

### Step 9: Offer scheduling (if Blotato connected)

Check `projects/social-media-content/state-social.md` → `blotato_status`.

- If `connected`: load `scheduling-defaults.md`, find the row for this platform, get the next-occurrence of `day_of_week + hour`. Ask:

  > "Schedule this for [next Tuesday 9am ET]? (y / specific time / skip)"

- If `skipped` or `awaiting-oauth`: skip Step 9 + Step 10. Tell user "Blotato not connected — post manually at [default time per scheduling-defaults.md]."

### Step 10: Schedule via Blotato (if user said yes)

On `y` or a specific time, call `mcp__claude_ai_Blotato__blotato_create_post` with platform-correct fields:

| Platform | Required field shape |
|---|---|
| linkedin | `text`, `accountId` (linkedin account from `blotato_list_accounts`), optional `mediaUrls` |
| twitter | `text`, `accountId`, optional `mediaUrls` |
| instagram | `text`, `accountId`, **required `mediaUrls`** (IG posts need media) |
| facebook | `text`, `accountId`, optional `mediaUrls` |
| tiktok | `text`, `accountId`, **required video `mediaUrl`** |
| threads | `text`, `accountId`, optional `mediaUrls` |

Include `scheduledFor` (ISO 8601 with user's timezone). On success:

1. Update the calendar-log row → `blotato_status` = `scheduled`, `scheduled_time` = ISO time
2. Update the post's frontmatter → `status: scheduled`
3. Tell user: "Scheduled — Blotato post ID `<id>`. View at [Blotato dashboard link]."

On Blotato API failure: log status `failed`, surface the error, offer "Retry, schedule manually, or skip?"

### Step 11: Foundation B — Self-improve close

Apply per `_shared/foundations.md` → Foundation B.

Ask: **"What would've made this 10% better?"**

Append the answer to `projects/social-media-content/memory.md`. Run recurrence check. If 3+ matches, prompt the user to update the skill itself + draft a row in `skill-improvements.md`.

### Step 12: Foundation C — `⚡ NEXT MOVE` block

Apply per `_shared/foundations.md` → Foundation C.

Output ends with:

```
⚡ NEXT MOVE: <Subject> <Verb> <Timing>
   Why: <one-sentence reason>
```

Validate against the regex. If invalid, regenerate.

**Priority order for the Next Move on a draft skill:**

1. Just scheduled via Blotato → "Check engagement on this post in 48 hours + log winners to memory"
2. Just drafted + Blotato skipped → "Post the draft manually at [scheduled_time] today" or "Post to [platform] at [default time] tomorrow"
3. Draft scored under 90 → "Run /grade-post on this draft after you let it rest 10 minutes"
4. Multi-platform topic → "Run /draft-[next-platform] on the same topic — repurpose the angle"

---

## Per-platform delta cheat sheet

Each `/draft-<platform>/SKILL.md` defines:

| Field | Source |
|---|---|
| Length range | `platform-voice.md` section + this spec |
| Hook character window | LinkedIn: 140 (before "See more"). IG: 125. Twitter: ~50. Others: full first line |
| Hashtag rules | LinkedIn 3-5 (optional), IG 3-5 (required niche), Twitter/Threads/FB 0, TikTok ≤ 5 |
| Asset requirement | IG: required. TikTok: required (video). Others: optional |
| CTA style | LinkedIn/Twitter/Threads: comment-driving. IG/FB: save/share/DM-driving. TikTok: completion-driving |
| Algorithm-fit | LinkedIn = comments. IG = saves/shares. Twitter = replies. FB = shares. TikTok = completion. Threads = replies. |

---

## What NOT to do

- Don't skip Step 5 (plan-then-approve). Even if the user says "just write it" — show the hook candidates first.
- Don't load the whole `platform-voice.md`. Just the platform section. Foundation A.
- Don't write below an 80 grade. Regenerate the failing dimension.
- Don't return without the `⚡ NEXT MOVE` block. Validation regex must match.
- Don't ask the self-improve question BEFORE the Next Move. Order is: main output → ⚡ NEXT MOVE → 10%-better question.
