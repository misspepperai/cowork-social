# Phase 5 — Asset Index Setup

## What this phase does

Sets up the user's local image catalog. Asks where they keep social-content images (default: `~/Pictures/social-content/`). Scans the folder. Walks the user through nickname-mapping 5-10 of their most-reused assets. Writes the result to `projects/social-media-content/asset-index.md`.

The asset index is what makes `/draft-instagram "new launch" --asset headshot` work — `headshot` is a nickname; the index resolves it to the local path. Instagram + TikTok drafts especially need this because those platforms require media to schedule via Blotato.

## Ask

Two questions, asked in order.

1. *"Where do you keep social-content images on this machine? Default is `~/Pictures/social-content/` — type that, or paste your own folder path (e.g., `D:\Dropbox\brand-assets\social\`). I'll scan whatever you give me."*

2. After the scan returns a list: *"I found [N] images / videos in there. Most people nickname 5-10 of the ones they reuse a lot — your headshot, your logo, a team photo, a launch graphic. Pick the ones you'd actually reference in a draft. Type 'show me the list' to see what I found, or paste nicknames now."*

## Why we ask

> *"Drafts get faster when assets have names you can type. `--asset headshot` is way better than `--asset C:\Users\you\Pictures\social-content\headshot-2024-cropped.jpg`.*
>
> *Plus, Instagram and TikTok drafts won't schedule through Blotato without a real image attached. The asset index is where the draft skills look up that image. Even if you skipped Blotato, this still matters — you'll copy the asset name into Buffer / Later / whatever you use.*
>
> *Skip-friendly: type `skip` and I'll create an empty asset-index.md with instructions. You can fill it later by hand."*

## Logic

### Step 1 — Resolve the asset root path

Accept the user's input. Normalize:
- `~/Pictures/social-content/` → resolve `~` to user's home dir (Windows: `C:\Users\<user>\Pictures\social-content\`; Mac: `/Users/<user>/Pictures/social-content/`)
- Path with mixed slashes → normalize to the OS's native form
- Trailing slash → preserve (cosmetic)

If the folder doesn't exist:
> *"That folder doesn't exist yet. Want me to create it (`mkdir -p`), pick a different path, or skip the scan?"*

If the user says "create", create the folder. If "skip", proceed with empty asset-index.md.

### Step 2 — Scan the folder

List image + video files (extensions: `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`, `.mp4`, `.mov`, `.webm`). Skip:
- Hidden files (`.DS_Store`, `Thumbs.db`)
- Files starting with `~` or `_temp`
- Files > 100MB (warn user — likely a non-social asset)

Show the user the first 20 results in a table:

```
Found [N] assets in [path]:

  Filename                          Size      Modified
  ---                               ---       ---
  headshot-2024.jpg                 240 KB    2024-08-12
  logo-square-transparent.png       45 KB     2024-06-03
  team-offsite-2024.jpg             1.2 MB    2024-09-21
  ...

[N - 20 more files not shown — type 'show all' to see the rest]
```

### Step 3 — Walk the user through nicknames

Ask: *"For each file you'd actually reference in a draft, type a short nickname. Format: `filename → nickname`. Skip the ones you'd never reuse. Examples:*
- *`headshot-2024.jpg → headshot`*
- *`logo-square-transparent.png → logo`*
- *`team-offsite-2024.jpg → team-photo`*

*Type `done` when you've named the ones that matter. Most people stop at 5-10."*

Collect mappings. For each, also detect:
- Type: image or video (from extension)
- Description: ask once *"One-line description for `[nickname]`? (skip = use the filename)"*

### Step 4 — Build the asset-index.md

Read [`../templates/asset-index.md.template`](../templates/asset-index.md.template). Interpolate:

- `{{asset_root_path}}` → resolved folder path
- `{{nickname_1}}` ... `{{nickname_N}}` → user's nicknames
- `{{path_1}}` ... `{{path_N}}` → full local paths
- `{{type_1}}` ... `{{type_N}}` → `image` or `video`
- `{{description_1}}` ... `{{description_N}}` → user's one-liners (or filename fallback)
- `{{last_used_1}}` ... `{{last_used_N}}` → blank (gets updated by draft skills on first use)
- `{{date}}` → today's ISO date

If the user nicknamed fewer than 3 assets, the file still gets written — they can add rows later. If the user said `skip` entirely, write the file with zero rows but keep the header + "How to add an asset by hand" section intact.

### Step 5 — Plan-then-approve the write

Show the user the rendered file as a preview, then ask:

```
Plan: write projects/social-media-content/asset-index.md.

[show rendered file]

Approve? (yes / edit a row / start over)
```

On approval, write the file. On edit, let the user fix one row at a time.

### Step 6 — Update state

Append to `_aibos/state-social.md`:

```yaml
asset_root_path: <resolved path>
asset_index_path: projects/social-media-content/asset-index.md
asset_nickname_count: <N>
phase_5_assets: completed_at <ISO timestamp>
```

## Write

Files touched in this phase:

- `<workspace>/projects/social-media-content/asset-index.md` — created from template, rendered with the user's nicknames
- `_aibos/state-social.md` — append the asset block

If the user picked "create the folder", the `mkdir -p` is the only filesystem write outside the workspace.

## Resume

After this phase completes, mark `phase_5_assets` as `completed_at <ISO timestamp>` in state-social.md and set `next_pending_phase: 6`. `asset_index_path` MUST be set; `asset_nickname_count` can be 0 (skip case).

If the user pauses mid-naming, save partial mappings to state under `asset_nicknames_partial:` and resume by re-listing them: *"You had 3 named so far: [list]. Keep going, or wrap it up here?"*

## Verification before advancing

Phase 5 is complete when ALL of these are true:

- `projects/social-media-content/asset-index.md` exists (even if empty rows)
- `asset_root_path` in state is a real, accessible folder OR explicitly marked `skipped: true`
- The user explicitly approved the rendered file (no implicit "I guess that's fine")

If `asset_nickname_count` is 0, Phase 8's wrap-up surfaces a soft hint: *"Your asset index is empty — add 5 nicknames before your first Instagram or TikTok draft, since those platforms need media to schedule."*
