---
name: draft-twitter
description: "Draft a platform-native Twitter/X post or thread in your voice. Reads platform-voice.md + brand-brief.md + business-brain.md. 280-char enforcement per tweet, thread support up to 12 tweets. Plan-then-approves, writes to outputs/, optionally schedules via Blotato."
when_to_use:
  - /draft-twitter
  - /draft-x
  - draft twitter post
  - twitter thread
  - x post about
  - tweet about
  - write a twitter post
---

# /draft-twitter — v0.1.0

Draft a Twitter/X tweet or thread. Native cowork-social skill — Blotato's post-writer doesn't cover X. Reply-driving, no hashtags, 280-char enforcement.

## Inputs (lazy-load)

Read these files on every run. If any is missing, halt and tell the user which `/onboard-social` phase to re-run.

| File | Purpose |
|---|---|
| `about-me/writing-rules.md` | Universal writing rules |
| `about-me/business-brain.md` | ICP, audience, recent stories, wedge |
| `projects/social-media-content/brand-brief.md` | what_you_sell, primary_cta, recent_proof_story, contrarian_belief, voice_signature |
| `projects/social-media-content/platform-voice.md` | **Twitter section only** — length_range, signature_openings, signature_closings, banned_phrases, voice_notes |
| `projects/social-media-content/state-social.md` | `blotato_status` (connected / skipped / awaiting-oauth) |
| `_shared/hook-patterns.md` | 11 hook patterns (Step 4) |

> **Foundation A — lazy-load:** read ONLY the Twitter section of `platform-voice.md`.

## The 12-step flow

Follow the 12-step spec in `_shared/draft-skill-spec.md`. Platform-specific deltas below.

## Platform deltas

| Field | Twitter value |
|---|---|
| Length per tweet | **280 chars HARD limit** — validate before save, regenerate if over |
| Thread support | Up to 12 tweets via Blotato `additionalPosts` array |
| Image | Optional |
| Hashtags | **0-2 max** (X engagement drops with hashtags — prefer 0) |
| Emojis | Sparing (≤1 per tweet) |
| First-3-words test | First 5-7 words decide whether anyone reads |
| CTA bias | Drive **replies + retweets** |
| Hook pattern bias | Contrarian Take (#6), Specific Number (#3), Reframe (#1) |

### Thread-specific rules (insert between Step 6 and Step 7 of shared spec)

If the topic needs >280 chars, draft as a thread:

1. Ask the user: "Single tweet or thread?" (default: single if topic fits)
2. Thread = 3-7 tweets typical, 12 max
3. Tweet 1 = hook (no preamble, no "thread incoming")
4. Each tweet a complete thought — readers can land mid-thread
5. Last tweet = CTA
6. Validate each tweet ≤280 chars. Regenerate any that exceed.

## Output location

```
outputs/social-media-content/YYYY-MM-DD-twitter-<slug>.md
```

Frontmatter:

```yaml
---
date: YYYY-MM-DD
platform: twitter
slug: <kebab-case-3-5-words>
hook_pattern: <pattern-name>
length_chars: <count>
type: single | thread
tweet_count: <N>
asset_used: null
status: drafted
---
```

### Single-tweet body

Just the tweet text. One block.

### Thread body format

```
## Tweet 1 (hook)
<tweet text> [<char_count>/280]

## Tweet 2
<tweet text> [<char_count>/280]

...

## CTA tweet (last)
<tweet text> [<char_count>/280]

---
Type: thread (<N> tweets)
Total chars: <total>
```

## Schedule step (Step 9-10 of shared spec)

Branch on `projects/social-media-content/state-social.md` → `blotato_status`:

- **`connected`** → Load `scheduling-defaults.md`, find Twitter row, get next-occurrence. Ask:
  > "Schedule this for [today 11am ET]? (y / specific time / skip)"
  On `y`, call `mcp__claude_ai_Blotato__blotato_create_post` with: `text` (Tweet 1), `accountId` (Twitter from `blotato_list_accounts`), optional `mediaUrls`, `additionalPosts` (array of remaining tweets if thread), `scheduledFor` (ISO 8601).
- **`skipped` / `awaiting-oauth`** → Save to outputs/ only. Tell user:
  > "Blotato isn't connected — copy/paste the tweet(s) into X manually at [default time per scheduling-defaults.md]."

On success: update calendar-log row → `scheduled`, update post frontmatter → `status: scheduled`, return Blotato post ID.

## Hard rules

- 280-char per-tweet limit enforced — regenerate any tweet that exceeds
- Plan-then-approve before any write
- Auto-grade via `/grade-post` (grader scores Tweet 1 as the primary hook)
- No em dashes, no AI-throat-clearing openers
- No hashtags unless user explicitly asks for one
- Prompts to user: 3rd-4th grade reading level. The TWEET itself: voice-tuned per brand-brief + platform-voice.

## Voice guardrails

Twitter post voice is defined in `projects/social-media-content/platform-voice.md` (Twitter section) + `brand-brief.md` (voice_signature). Don't reinvent it.

## Self-improvement close

See `_shared/foundations.md` → Foundation B. After delivering the main output + the ⚡ NEXT MOVE block, ask: **"What would've made this 10% better?"** Log to `projects/social-media-content/memory.md`:

```
<YYYY-MM-DD> | /draft-twitter | <answer verbatim>
```

Run recurrence check.

## Next move

See `_shared/foundations.md` → Foundation C. Skill output ends with a `⚡ NEXT MOVE` block matching the validation regex.

### Twitter NEXT MOVE priority

1. Just scheduled via Blotato → "Check replies on this tweet/thread tomorrow morning + reply to top 3 within 30 minutes of seeing them"
2. Just drafted + Blotato skipped → "Post the tweet manually within 2 hours of any trending topic in your niche today"
3. Thread drafted → "Pin the thread for 48 hours after posting + reply to your own Tweet 1 with a 'bookmark this' CTA"
4. Draft scored under 90 → "Run `/grade-post` on Tweet 1 — the hook is doing 80% of the work"

### Examples (the bar)

```
⚡ NEXT MOVE: Schedule the tweet for today 11am ET.
   Why: That's your peak Twitter window per scheduling-defaults + the contrarian take rides this morning's news cycle.
```

```
⚡ NEXT MOVE: Pin the thread for 48 hours after posting tomorrow.
   Why: Pinned threads compound on replies + your follower base sees Tweet 1 first when they visit your profile.
```

```
⚡ NEXT MOVE: Run `/draft-linkedin` on the same angle tonight.
   Why: This contrarian take has LinkedIn-length depth + repurposing now keeps the hook timely on both platforms.
```
