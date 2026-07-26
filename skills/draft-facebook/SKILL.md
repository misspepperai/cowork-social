---
name: draft-facebook
description: "Draft a platform-native Facebook post in your voice. Reads platform-voice.md + brand-brief.md + business-brain.md. 40-80 word sweet spot. Link previews supported. Plan-then-approves, writes to outputs/, optionally schedules via Blotato. Reads pageId from state-social.md."
when_to_use:
  - /draft-facebook
  - /draft-fb
  - draft facebook post
  - fb post about
  - write a facebook post
---

# /draft-facebook — v0.1.0

Draft a Facebook post. Share-driving, link-preview-aware, short-form sweet spot. Reads `pageId` for the Facebook subaccount captured during `/onboard-social`.

## Inputs (lazy-load)

Read these files on every run. If any is missing, halt and tell the user which `/onboard-social` phase to re-run.

| File | Purpose |
|---|---|
| `about-me/writing-rules.md` | Universal writing rules |
| `about-me/business-brain.md` | ICP, audience, recent stories, wedge |
| `projects/social-media-content/brand-brief.md` | what_you_sell, primary_cta, recent_proof_story, contrarian_belief, voice_signature |
| `projects/social-media-content/platform-voice.md` | **Facebook section only** — length_range, signature_openings, signature_closings, banned_phrases, voice_notes |
| `projects/social-media-content/state-social.md` | `blotato_status` + **`facebook_page_id`** (FB subaccount) |
| `projects/social-media-content/asset-index.md` | Optional images |
| `_shared/hook-patterns.md` | 11 hook patterns (Step 4) |

> **Foundation A — lazy-load:** read ONLY the Facebook section of `platform-voice.md`.

## The 12-step flow

Follow the 12-step spec in `_shared/draft-skill-spec.md`. Platform-specific deltas below.

## Platform deltas

| Field | Facebook value |
|---|---|
| Length | **40-80 words optimal** (80-300 acceptable for story posts) |
| Hook window | First line — must hook in 1 sentence |
| Image | Optional |
| Link previews | If topic involves a link, put the URL on its own line — FB generates a rich preview that outperforms inline links |
| Hashtags | 0-2 max (FB doesn't reward them) |
| Video | If user attaches video, `mediaType` MUST be `"reel"` per Blotato MCP — regular feed video no longer supported |
| CTA bias | Drive **shares** (FB algorithm weights shares > likes/comments) |
| Hook pattern bias | Vulnerable Story (#5), Receipts (#9) |

## Output location

```
outputs/social-media-content/YYYY-MM-DD-facebook-<slug>.md
```

Frontmatter:

```yaml
---
date: YYYY-MM-DD
platform: facebook
slug: <kebab-case-3-5-words>
hook_pattern: <pattern-name>
length_words: <count>
length_chars: <count>
asset_used: <asset-name-or-null>
link_preview_url: <url-or-null>
video_type: <reel-or-null>
status: drafted
---
```

Body = the actual post text. If a link is included, place it on its own line so FB triggers the preview.

## Schedule step (Step 9-10 of shared spec)

Branch on `projects/social-media-content/state-social.md` → `blotato_status`:

- **`connected`** → Load `scheduling-defaults.md`, find Facebook row. Read `facebook_page_id` from state-social (this is the FB page subaccount captured during `/onboard-social` via `blotato_list_accounts`). Ask:
  > "Schedule this for [Thursday 1pm ET] on your `<page_name>` page? (y / specific time / skip)"
  On `y`, call `mcp__claude_ai_Blotato__blotato_create_post` with: `text` (the draft), `accountId` (Facebook from `blotato_list_accounts`), `pageId` (from state-social), optional `mediaUrls`, `scheduledFor` (ISO 8601). If video → set `mediaType: "reel"`.
- **`skipped` / `awaiting-oauth`** → Save to outputs/ only. Tell user:
  > "Blotato isn't connected — copy/paste the post into Facebook manually at [default time per scheduling-defaults.md]."
- **`connected` but `facebook_page_id` missing** → Halt scheduling. Tell user: "I don't have your Facebook page ID. Re-run `/onboard-social` Phase X to capture it via `blotato_list_accounts`."

On Blotato success: update calendar-log row, update post frontmatter → `status: scheduled`.

## Hard rules

- If video attached, validate `mediaType: "reel"` — refuse to schedule plain feed video
- Facebook scheduling requires `pageId` (FB subaccount) — block scheduling if missing
- Plan-then-approve before any write
- Auto-grade via `/grade-post`; don't show drafts under 80
- No em dashes, no AI-throat-clearing openers
- Prompts to user: 3rd-4th grade reading level. The POST itself: voice-tuned per brand-brief + platform-voice.

## Voice guardrails

Facebook post voice is defined in `projects/social-media-content/platform-voice.md` (Facebook section) + `brand-brief.md` (voice_signature). Don't reinvent it.

## Self-improvement close

See `_shared/foundations.md` → Foundation B. After delivering the main output + the ⚡ NEXT MOVE block, ask: **"What would've made this 10% better?"** Log to `projects/social-media-content/memory.md`:

```
<YYYY-MM-DD> | /draft-facebook | <answer verbatim>
```

Run recurrence check.

## Next move

See `_shared/foundations.md` → Foundation C. Skill output ends with a `⚡ NEXT MOVE` block matching the validation regex.

### Facebook NEXT MOVE priority

1. Facebook page ID missing → "Re-run `/onboard-social` Phase X today to capture your FB pageId via `blotato_list_accounts`"
2. Just scheduled via Blotato → "Check shares + comments on this FB post 24 hours after posting + log winning hook pattern to memory.md"
3. Just drafted + Blotato skipped → "Post the FB draft manually Thursday at 1pm ET — that's your peak share window"
4. Story post with link → "Pin the post for 48 hours after publishing — link-preview posts compound on shares"

### Examples (the bar)

```
⚡ NEXT MOVE: Schedule the Facebook post for Thursday 1pm ET on the Miss Pepper AI page.
   Why: Thursday afternoon is your peak share window + the Receipts hook fits FB story-pattern preference.
```

```
⚡ NEXT MOVE: Re-run /onboard-social Phase 6 today to capture your Facebook pageId via blotato_list_accounts.
   Why: FB scheduling is blocked until pageId is in state-social.md.
```

```
⚡ NEXT MOVE: Check shares + comments on this FB post 24 hours after publishing.
   Why: FB rewards shares heavier than likes + this tells you which hook pattern works for your audience.
```
