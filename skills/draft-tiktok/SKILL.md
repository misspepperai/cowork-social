---
name: draft-tiktok
description: "Draft a TikTok video script + caption + hashtags. Reads platform-voice.md + brand-brief.md + business-brain.md + asset-index.md. Video MANDATORY. Hook under 3s -> context -> payoff -> CTA. Caption under 150 chars, 3-5 hashtags. Plan-then-approves, outputs/, schedules via Blotato."
when_to_use:
  - /draft-tiktok
  - /draft-tt
  - draft tiktok script
  - tiktok video about
  - tiktok script about
  - write a tiktok
---

# /draft-tiktok — v0.1.0

Draft a TikTok video script + caption + hashtags. Video is required. Completion-rate driving (algorithm rewards watch-through > engagement).

## Inputs (lazy-load)

Read these files on every run. If any is missing, halt and tell the user which `/onboard-social` phase to re-run.

| File | Purpose |
|---|---|
| `about-me/writing-rules.md` | Universal writing rules |
| `about-me/business-brain.md` | ICP, audience, recent stories, wedge |
| `projects/social-media-content/brand-brief.md` | what_you_sell, primary_cta, recent_proof_story, contrarian_belief, voice_signature |
| `projects/social-media-content/platform-voice.md` | **TikTok section only** — length_range, signature_openings, signature_closings, banned_phrases, voice_notes |
| `projects/social-media-content/asset-index.md` | **REQUIRED** — video assets (path or generation prompt) |
| `projects/social-media-content/state-social.md` | `blotato_status` (connected / skipped / awaiting-oauth) |
| `_shared/hook-patterns.md` | 11 hook patterns (Step 4) |

> **Foundation A — lazy-load:** read ONLY the TikTok section of `platform-voice.md`.

## The 12-step flow

Follow the 12-step spec in `_shared/draft-skill-spec.md`. Platform-specific deltas below.

## Platform deltas

| Field | TikTok value |
|---|---|
| Script length | 100-300 words (30-60 seconds spoken) |
| Caption length | ≤150 chars (caption is SECONDARY — script is primary content) |
| Video | **MANDATORY** — fail clean if no asset matches |
| Hook window | **First 3 seconds** decide watch-through (~15 spoken words rule of thumb) |
| Hashtags | **3-5 niche-specific** (AVOID `#fyp` / `#foryou` — TikTok flags as low-signal) |
| CTA bias | Drive **completion rate** (algorithm rewards watch-through, then engagement) |
| Hook pattern bias | Specific Number (#3), Vulnerable Story (#5), Receipts (#9) |

### Step 6 override: body is the script (timestamped sections, not prose)

Output the body as 4 timestamped sections:

```
[0:00-0:03] Hook: <hook line — ≤15 words, deliverable in 3 seconds>
[0:03-0:10] Context: <why the viewer should care — 1-2 sentences>
[0:10-0:30] Payoff: <the substance — specific, scannable, no fluff>
[0:30+] CTA: <one line — comment / follow / save>
```

### Video-mandatory step (insert between Step 5 and Step 6 of shared spec)

After hook approval, BEFORE writing the full script:

1. Read `projects/social-media-content/asset-index.md`
2. Match a video asset (path or generation prompt) to the topic
3. If match found → surface to user: "Using video asset: `<name>` (`<path-or-prompt>`) — confirm or pick different? (y / nominate another)"
4. If no match → ask: "Which video will you use? Paste a path, describe a video to shoot, or say 'skip' to draft script-only with the video gap flagged."
5. If user can't provide → WARN: "This TikTok draft is incomplete — no video, can't ship via Blotato. Script saved with `video_used: PENDING` and won't schedule until video is attached."

### Step 7 override: caption is the short feed text

Caption is ≤150 chars and complements the video — don't restate the script. Validate count before write.

## Output location

```
outputs/social-media-content/YYYY-MM-DD-tiktok-<slug>.md
```

Frontmatter:

```yaml
---
date: YYYY-MM-DD
platform: tiktok
slug: <kebab-case-3-5-words>
hook_pattern: <pattern-name>
script_word_count: <count>
caption_chars: <count>
video_used: <asset-name-or-PENDING>
video_path: <path-or-prompt-or-null>
hashtags: [<list-of-3-to-5>]
status: drafted
---
```

Body format:

```
## Script

[0:00-0:03] Hook: ...
[0:03-0:10] Context: ...
[0:10-0:30] Payoff: ...
[0:30+] CTA: ...

## Caption

<caption text — ≤150 chars>

## Hashtags

<#hashtag1> <#hashtag2> <#hashtag3> <#hashtag4> <#hashtag5>
```

## Schedule step (Step 9-10 of shared spec)

Branch on `projects/social-media-content/state-social.md` → `blotato_status`:

- **`connected` + video attached** → Load `scheduling-defaults.md`, find TikTok row. Ask:
  > "Schedule this for [today 7pm ET]? (y / specific time / skip)"
  On `y`, call `mcp__claude_ai_Blotato__blotato_create_post` with: `text` (caption), `accountId` (TikTok from `blotato_list_accounts`), **`mediaUrl` REQUIRED** (the video), `privacyLevel` (`PUBLIC_TO_EVERYONE` default), boolean flags (`disableComment: false`, `disableDuet: false`, `disableStitch: false` — defaults), `scheduledFor` (ISO 8601).
- **`connected` + video MISSING** → Refuse to schedule. Tell user: "Can't schedule — TikTok requires a video. Add the asset to `asset-index.md`, then re-run scheduling."
- **`skipped` / `awaiting-oauth`** → Save to outputs/ only. Tell user:
  > "Blotato isn't connected — record the video, upload to TikTok manually at [default time per scheduling-defaults.md], use the script + caption above."

On Blotato success: update calendar-log row, update post frontmatter → `status: scheduled`.

## Hard rules

- **Video is REQUIRED** — never ship a no-video TikTok draft to Blotato
- Hook must be deliverable in ≤3 seconds (≤15 spoken words)
- Caption ≤150 chars (validate)
- 3-5 hashtags (validate count), no `#fyp` / `#foryou`
- Plan-then-approve before any write
- Auto-grade via `/grade-post` (grader scores the script section, not caption)
- No em dashes, no AI-throat-clearing openers
- Prompts to user: 3rd-4th grade reading level. The SCRIPT itself: voice-tuned per brand-brief + platform-voice, written for spoken delivery.

## Voice guardrails

TikTok script voice is defined in `projects/social-media-content/platform-voice.md` (TikTok section) + `brand-brief.md` (voice_signature). Written for spoken delivery — short sentences, contractions, energy.

## Self-improvement close

See `_shared/foundations.md` → Foundation B. After delivering the main output + the ⚡ NEXT MOVE block, ask: **"What would've made this 10% better?"** Log to `projects/social-media-content/memory.md`:

```
<YYYY-MM-DD> | /draft-tiktok | <answer verbatim>
```

Run recurrence check.

## Next move

See `_shared/foundations.md` → Foundation C. Skill output ends with a `⚡ NEXT MOVE` block matching the validation regex.

### TikTok NEXT MOVE priority

1. No video attached → "Record the video today before 5pm — the script is ready to read off your phone screen"
2. Just scheduled via Blotato → "Check 3-second watch-through rate 24 hours after post + log winning hook pattern to memory.md"
3. Just drafted + Blotato skipped → "Record + post manually tonight before 7pm ET — that's your peak active-window"
4. Draft scored under 90 → "Run `/grade-post` on the first 3 seconds of the script — that's 80% of the watch-through battle"

### Examples (the bar)

```
⚡ NEXT MOVE: Record the TikTok today before 5pm.
   Why: The hook is ≤3 seconds + posting before 7pm ET catches your peak audience window.
```

```
⚡ NEXT MOVE: Add the missing reel-video asset to asset-index.md today before scheduling.
   Why: TikTok won't ship via Blotato without a video + the script is finished otherwise.
```

```
⚡ NEXT MOVE: Check 3-second watch-through rate 24 hours after the TikTok posts tomorrow.
   Why: Watch-through is the TikTok algo's primary signal + the lowest-scoring 3 seconds gets rewritten next time.
```
