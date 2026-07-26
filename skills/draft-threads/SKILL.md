---
name: draft-threads
description: "Draft a platform-native Meta Threads post or reply chain in your voice. Reads platform-voice.md + brand-brief.md + business-brain.md. Up to 500 chars per post, reply chain up to 10. Conversational tone (Instagram-but-text). Plan-then-approves, writes to outputs/, schedules via Blotato."
when_to_use:
  - /draft-threads
  - draft threads post
  - meta threads post
  - threads about
  - write a threads post
---

# /draft-threads — v0.1.0

Draft a Meta Threads post or reply chain. Native cowork-social skill — Blotato's post-writer doesn't cover Threads. Reply-driving, conversational, sparing hashtags.

## Inputs (lazy-load)

Read these files on every run. If any is missing, halt and tell the user which `/onboard-social` phase to re-run.

| File | Purpose |
|---|---|
| `about-me/writing-rules.md` | Universal writing rules |
| `about-me/business-brain.md` | ICP, audience, recent stories, wedge |
| `projects/social-media-content/brand-brief.md` | what_you_sell, primary_cta, recent_proof_story, contrarian_belief, voice_signature |
| `projects/social-media-content/platform-voice.md` | **Threads section only** — length_range, signature_openings, signature_closings, banned_phrases, voice_notes |
| `projects/social-media-content/state-social.md` | `blotato_status` (connected / skipped / awaiting-oauth) |
| `projects/social-media-content/asset-index.md` | Optional image/video |
| `_shared/hook-patterns.md` | 11 hook patterns (Step 4) |

> **Foundation A — lazy-load:** read ONLY the Threads section of `platform-voice.md`.

## The 12-step flow

Follow the 12-step spec in `_shared/draft-skill-spec.md`. Platform-specific deltas below.

## Platform deltas

| Field | Threads value |
|---|---|
| Length per post | **500 chars HARD limit** — validate before save |
| Reply chain | Up to 10 posts via Blotato `additionalPosts` |
| Image | Optional |
| Hashtags | **0-1 max** (Threads is text-first, hashtags feel inorganic) |
| Tone | Conversational, slightly more long-form than X, less polished than LinkedIn — "Instagram-but-text" |
| CTA bias | Drive **replies** (replies are the Threads algo multiplier) |
| Hook pattern bias | Question Hook (#4), Contrarian Take (#6) |

### Reply chain rules (insert between Step 6 and Step 7 of shared spec)

If single post exceeds 500 chars OR topic naturally extends:

1. Ask user: "Single post or reply chain?" (default: single if topic fits)
2. Chain = 2-5 posts typical, 10 max
3. Post 1 = hook (no preamble)
4. Each post stands alone — readers may land mid-chain
5. Last post = CTA (reply-driving)
6. Validate each post ≤500 chars

## Output location

```
outputs/social-media-content/YYYY-MM-DD-threads-<slug>.md
```

Frontmatter:

```yaml
---
date: YYYY-MM-DD
platform: threads
slug: <kebab-case-3-5-words>
hook_pattern: <pattern-name>
length_chars: <count>
type: single | chain
post_count: <N>
asset_used: <asset-name-or-null>
status: drafted
---
```

### Single-post body

Just the post text. One block.

### Reply chain body format

```
## Post 1 (hook)
<post text> [<char_count>/500]

## Post 2
<post text> [<char_count>/500]

...

## CTA post (last)
<post text> [<char_count>/500]

---
Type: chain (<N> posts)
Total chars: <total>
```

## Schedule step (Step 9-10 of shared spec)

Branch on `projects/social-media-content/state-social.md` → `blotato_status`:

- **`connected`** → Load `scheduling-defaults.md`, find Threads row. Ask:
  > "Schedule this for [tomorrow 9am ET]? (y / specific time / skip)"
  On `y`, call `mcp__claude_ai_Blotato__blotato_create_post` with: `text` (Post 1), `accountId` (Threads from `blotato_list_accounts`), optional `mediaUrls`, `additionalPosts` (array of remaining posts if chain), `scheduledFor` (ISO 8601).
- **`skipped` / `awaiting-oauth`** → Save to outputs/ only. Tell user:
  > "Blotato isn't connected — copy/paste the post(s) into Threads manually at [default time per scheduling-defaults.md]."

On Blotato success: update calendar-log row, update post frontmatter → `status: scheduled`.

## Hard rules

- 500-char per-post limit enforced — regenerate any post that exceeds
- Plan-then-approve before any write
- Auto-grade via `/grade-post`
- No em dashes, no AI-throat-clearing openers
- 0-1 hashtags max (default to 0)
- Prompts to user: 3rd-4th grade reading level. The POST itself: conversational, voice-tuned per brand-brief + platform-voice.

## Voice guardrails

Threads post voice is defined in `projects/social-media-content/platform-voice.md` (Threads section) + `brand-brief.md` (voice_signature). Conversational — write like you'd reply to a friend. Don't reinvent it.

## Self-improvement close

See `_shared/foundations.md` → Foundation B. After delivering the main output + the ⚡ NEXT MOVE block, ask: **"What would've made this 10% better?"** Log to `projects/social-media-content/memory.md`:

```
<YYYY-MM-DD> | /draft-threads | <answer verbatim>
```

Run recurrence check.

## Next move

See `_shared/foundations.md` → Foundation C. Skill output ends with a `⚡ NEXT MOVE` block matching the validation regex.

### Threads NEXT MOVE priority

1. Just scheduled via Blotato → "Check reply rate on this Threads post within 4 hours of posting + reply to top 3 replies that day"
2. Just drafted + Blotato skipped → "Post the Threads draft manually tomorrow 9am ET + reply to your own first post with a follow-up question to seed engagement"
3. Reply chain drafted → "Pin the chain for 48 hours after posting + reply to your first hook with the most-engaging follow-up"
4. Draft scored under 90 → "Run `/grade-post` on Post 1 — the hook is doing 80% of the work in Threads' feed"

### Examples (the bar)

```
⚡ NEXT MOVE: Schedule the Threads post for tomorrow 9am ET.
   Why: That's your peak Threads window per scheduling-defaults + the question hook is built to drive replies.
```

```
⚡ NEXT MOVE: Post the Threads chain manually tonight + reply to your own Post 1 with a follow-up question to seed engagement.
   Why: Blotato isn't connected yet + Threads' algo amplifies posts with self-reply activity.
```

```
⚡ NEXT MOVE: Check the reply rate on this Threads post within 4 hours of posting tomorrow.
   Why: Replies are the Threads algo multiplier + responding to top 3 within the same window doubles reach.
```
