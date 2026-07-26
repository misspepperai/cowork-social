---
name: draft-linkedin
description: "Draft a platform-native LinkedIn post in your voice. Reads platform-voice.md + brand-brief.md + business-brain.md. Generates 3 hook candidates, plan-then-approves, writes to outputs/, optionally schedules via Blotato. Follows the 12-step shared draft-skill spec."
when_to_use:
  - /draft-linkedin
  - draft linkedin post
  - linkedin post about
  - write a linkedin post
  - linkedin draft
---

# /draft-linkedin — v0.1.0

Draft a LinkedIn post (hook + body + CTA) tuned to LinkedIn's algorithm and voice. Long-form, comment-driving, no image required.

## Inputs (lazy-load)

Read these files on every run. If any is missing, halt and tell the user which `/onboard-social` phase to re-run.

| File | Purpose |
|---|---|
| `about-me/writing-rules.md` | Universal writing rules |
| `about-me/business-brain.md` | ICP, audience, recent stories, wedge |
| `projects/social-media-content/brand-brief.md` | what_you_sell, primary_cta, recent_proof_story, contrarian_belief, voice_signature |
| `projects/social-media-content/platform-voice.md` | **LinkedIn section only** — length_range, signature_openings, signature_closings, banned_phrases, voice_notes |
| `projects/social-media-content/state-social.md` | `blotato_status` (connected / skipped / awaiting-oauth) |
| `_shared/hook-patterns.md` | 11 hook patterns (Step 4) |

> **Foundation A — lazy-load:** read ONLY the LinkedIn section of `platform-voice.md`. Don't load the whole file.

## The 12-step flow

Follow the 12-step spec in `_shared/draft-skill-spec.md`. Platform-specific deltas below.

## Platform deltas

| Field | LinkedIn value |
|---|---|
| Length | 1200-1900 chars sweet spot (3000 char max) |
| Hook window | First 140 chars (LinkedIn truncates at ~210 before "see more") |
| Image | Optional (text-only often outperforms) |
| Hashtags | 0-3 max, placed at end |
| Line breaks | Preserve aggressively — whitespace is a LinkedIn ranking signal |
| CTA bias | Drive **comments** (LinkedIn algorithm weights comments > likes) |
| Hook pattern bias | Reframe (#1), Receipts (#9), Stolen Lesson (#11) |
| First-3-words test | Must create curiosity within 3 words (see hook-patterns.md) |

## Output location

```
outputs/social-media-content/YYYY-MM-DD-linkedin-<slug>.md
```

Frontmatter:

```yaml
---
date: YYYY-MM-DD
platform: linkedin
slug: <kebab-case-3-5-words>
hook_pattern: <pattern-name>
length_chars: <count>
asset_used: null
status: drafted
---
```

Body = the actual post text, line breaks preserved, ready to copy-paste.

## Schedule step (Step 9-10 of shared spec)

Branch on `projects/social-media-content/state-social.md` → `blotato_status`:

- **`connected`** → Load `scheduling-defaults.md`, find LinkedIn row, get next-occurrence. Ask:
  > "Schedule this for [next Tuesday 8:30am ET]? (y / specific time / skip)"
  On `y`, call `mcp__claude_ai_Blotato__blotato_create_post` with: `text` (the draft), `accountId` (LinkedIn from `blotato_list_accounts`), optional `mediaUrls`, `scheduledFor` (ISO 8601).
- **`skipped` / `awaiting-oauth`** → Save to outputs/ only. Tell user:
  > "Blotato isn't connected — copy/paste the draft into LinkedIn manually at [default time per scheduling-defaults.md]."

On Blotato success: update calendar-log row → `scheduled`, update post frontmatter → `status: scheduled`, return Blotato post ID.

## Hard rules

- Plan-then-approve before any write (Step 5 of shared spec)
- Auto-grade via `/grade-post` internal pass; don't show drafts under 80
- Preserve line breaks (write to disk, not chat-truncated)
- No em dashes, no AI-throat-clearing openers (see Step 2 of shared spec)
- Prompts to user: 3rd-4th grade reading level. The POST itself: voice-tuned per brand-brief + platform-voice.

## Voice guardrails

The LinkedIn post voice is defined in `projects/social-media-content/platform-voice.md` (LinkedIn section) + `brand-brief.md` (voice_signature). Don't reinvent it — load and follow.

## Self-improvement close

See `_shared/foundations.md` → Foundation B. After delivering the main output + the ⚡ NEXT MOVE block, ask: **"What would've made this 10% better?"** Log the answer to `projects/social-media-content/memory.md` as:

```
<YYYY-MM-DD> | /draft-linkedin | <answer verbatim>
```

Run recurrence check. If 3+ matches surface, prompt to update the skill + draft a row in `skill-improvements.md`.

## Next move

See `_shared/foundations.md` → Foundation C. Skill output ends with a `⚡ NEXT MOVE` block matching the validation regex.

### LinkedIn NEXT MOVE priority

1. Just scheduled via Blotato → reply to every comment within first 2 hours
2. Just drafted + Blotato skipped → post manually at 8:30am ET tomorrow
3. Draft scored under 90 → run `/grade-post` after a 10-minute rest
4. Multi-platform topic → run `/draft-twitter` on the same angle

### Examples (the bar)

```
⚡ NEXT MOVE: Schedule the LinkedIn draft for Tuesday 8:30am ET.
   Why: That's your highest-engagement LinkedIn window per scheduling-defaults + the hook is timely this week.
```

```
⚡ NEXT MOVE: Post the LinkedIn draft manually tomorrow at 8:30am ET.
   Why: Blotato isn't connected yet + that's still your peak window for comments.
```

```
⚡ NEXT MOVE: Reply to every comment on this LinkedIn post within 2 hours of going live tomorrow.
   Why: First-2-hours reply velocity is the LinkedIn algo's biggest amplifier signal.
```
