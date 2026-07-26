---
name: draft-instagram
description: "Draft a platform-native Instagram caption with required image attachment. Reads platform-voice.md + brand-brief.md + business-brain.md + asset-index.md. Image MANDATORY. 125-2200 char caption, 5-10 hashtags. Plan-then-approves, writes to outputs/, schedules via Blotato."
when_to_use:
  - /draft-instagram
  - /draft-ig
  - draft instagram post
  - instagram post about
  - ig caption
  - write an instagram post
---

# /draft-instagram — v0.1.0

Draft an Instagram caption + attach a required image from `asset-index.md`. Save-driving CTA, hashtag-heavy, first-line hook before "see more" truncation at ~125 chars.

## Inputs (lazy-load)

Read these files on every run. If any is missing, halt and tell the user which `/onboard-social` phase to re-run.

| File | Purpose |
|---|---|
| `about-me/writing-rules.md` | Universal writing rules |
| `about-me/business-brain.md` | ICP, audience, recent stories, wedge |
| `projects/social-media-content/brand-brief.md` | what_you_sell, primary_cta, recent_proof_story, contrarian_belief, voice_signature |
| `projects/social-media-content/platform-voice.md` | **Instagram section only** — length_range, signature_openings, signature_closings, banned_phrases, voice_notes |
| `projects/social-media-content/asset-index.md` | **REQUIRED** — image assets with tags + paths |
| `projects/social-media-content/state-social.md` | `blotato_status` (connected / skipped / awaiting-oauth) |
| `_shared/hook-patterns.md` | 11 hook patterns (Step 4) |

> **Foundation A — lazy-load:** read ONLY the Instagram section of `platform-voice.md`.

## The 12-step flow

Follow the 12-step spec in `_shared/draft-skill-spec.md`. Platform-specific deltas below.

## Platform deltas

| Field | Instagram value |
|---|---|
| Caption length | 125-2200 chars (first 125 visible without "see more") |
| Hook window | **First 125 chars** — hook must land within first 7 words |
| Image | **MANDATORY** — fail clean if no asset matches |
| Hashtags | **5-10** (mix broad + niche), placed at end of caption OR in first comment per user preference in brand-brief |
| Emojis | OK, used sparingly to break up scannable lists |
| Line breaks | Strategic — IG renders them; use double-line for breath |
| CTA bias | Drive **saves** (IG algorithm weights saves highest) + DMs |
| Hook pattern bias | Vulnerable Story (#5), Behind-the-Scenes (#8), Reverse (#10) |

### Image-mandatory step (insert between Step 5 and Step 6 of shared spec)

After hook approval, BEFORE writing the full caption:

1. Read `projects/social-media-content/asset-index.md`
2. Match an asset to the topic by tags/description
3. If match found → surface to user: "Using asset: `<name>` (`<path>`) — confirm or pick different? (y / nominate another)"
4. If no match → ask: "Which image will you use? Paste a path, nominate one to add to asset-index, or say 'skip' to draft caption-only with the image gap flagged."
5. If user can't provide → WARN: "This IG draft is incomplete — no image, can't ship via Blotato. Caption saved with `asset_used: PENDING` and won't schedule until image is attached."

## Output location

```
outputs/social-media-content/YYYY-MM-DD-instagram-<slug>.md
```

Frontmatter:

```yaml
---
date: YYYY-MM-DD
platform: instagram
slug: <kebab-case-3-5-words>
hook_pattern: <pattern-name>
length_chars: <count>
asset_used: <asset-name-or-PENDING>
image_path: <path-or-null>
hashtags: [<list>]
status: drafted
---
```

Body = caption text + hashtag block.

## Schedule step (Step 9-10 of shared spec)

Branch on `projects/social-media-content/state-social.md` → `blotato_status`:

- **`connected` + image attached** → Load `scheduling-defaults.md`, find Instagram row. Ask:
  > "Schedule this for [today 11am ET]? (y / specific time / skip)"
  On `y`, call `mcp__claude_ai_Blotato__blotato_create_post` with: `text` (caption), `accountId` (Instagram from `blotato_list_accounts`), **`mediaUrls` REQUIRED** (the image), `scheduledFor` (ISO 8601).
- **`connected` + image MISSING** → Refuse to schedule. Tell user: "Can't schedule — Instagram requires an image. Add the asset to `asset-index.md`, then re-run scheduling."
- **`skipped` / `awaiting-oauth`** → Save to outputs/ only. Tell user:
  > "Blotato isn't connected — copy/paste the caption + upload the image into Instagram manually at [default time per scheduling-defaults.md]."

On Blotato success: update calendar-log row, update post frontmatter → `status: scheduled`.

## Hard rules

- **Image is REQUIRED** — never ship a no-image IG draft to Blotato
- Plan-then-approve before any write
- Auto-grade via `/grade-post`; don't show drafts under 80
- 5-10 hashtags validated before write
- First 125 chars of caption = the hook (validate)
- No em dashes, no AI-throat-clearing openers
- Prompts to user: 3rd-4th grade reading level. The CAPTION itself: voice-tuned per brand-brief + platform-voice.

## Voice guardrails

Instagram caption voice is defined in `projects/social-media-content/platform-voice.md` (Instagram section) + `brand-brief.md` (voice_signature). Don't reinvent it.

## Self-improvement close

See `_shared/foundations.md` → Foundation B. After delivering the main output + the ⚡ NEXT MOVE block, ask: **"What would've made this 10% better?"** Log to `projects/social-media-content/memory.md`:

```
<YYYY-MM-DD> | /draft-instagram | <answer verbatim>
```

Run recurrence check.

## Next move

See `_shared/foundations.md` → Foundation C. Skill output ends with a `⚡ NEXT MOVE` block matching the validation regex.

### Instagram NEXT MOVE priority

1. No image attached → "Add the missing image to `asset-index.md` today before scheduling — Instagram won't ship without it"
2. Just scheduled via Blotato → "Reply to every IG comment within 2 hours of post going live + log save-rate to memory.md tomorrow"
3. Just drafted + Blotato skipped → "Post the IG draft manually at 11am ET today — that's your peak save window"
4. Draft scored under 90 → "Run `/grade-post` on the caption — first-125-chars hook is doing 80% of the work"

### Examples (the bar)

```
⚡ NEXT MOVE: Schedule the Instagram post for today 11am ET.
   Why: That's your peak save-rate window + the vulnerable-story hook fits IG's algorithm preference.
```

```
⚡ NEXT MOVE: Add the missing studio-photo asset to asset-index.md today before 5pm.
   Why: Instagram won't ship without an image + this draft is finished otherwise.
```

```
⚡ NEXT MOVE: Reply to every IG comment within 2 hours of post going live tomorrow.
   Why: First-2-hours engagement velocity drives IG's save + share amplification.
```
