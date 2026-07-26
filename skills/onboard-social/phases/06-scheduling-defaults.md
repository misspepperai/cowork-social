# Phase 6 — Scheduling Defaults

## What this phase does

For each picked platform (from Phase 2), captures the user's default day-of-week + hour-of-day + timezone for scheduling. These defaults drive the "Schedule this for [time]?" offer that every `/draft-*` skill makes after building a draft.

If Blotato was skipped in Phase 4, this phase is **optional but still useful** — the user can post manually at the same suggested times. The template's industry-baseline defaults apply if the user can't decide.

## Ask

Three questions, asked once up front, then a per-platform loop.

### Up-front

1. *"What's your timezone? (e.g., `America/New_York`, `Europe/Lisbon`, `Asia/Singapore`.) Defaults to your system timezone if you skip."*

### Per picked platform

2. *"For [PLATFORM], what day of the week + hour do you want as your default post time? Type something like `Tuesday 9am` or `Wednesday 11:30am`. Skip to use the industry baseline ([template default for this platform])."*

3. *"Why that time? One line — helps you remember later. Skip if obvious."*

## Why we ask

> *"Every `/draft-*` skill ends by asking 'Schedule this for [your default time]?' That offer is only useful if I know what your default is.*
>
> *I have industry baselines from Sabrina's masterclass — LinkedIn Tuesday 9am, Twitter Wednesday 10am, Instagram Wednesday 11am, etc. They're decent starting points. Type a custom time if you've got data on what works for your audience. Skip any platform to use the baseline.*
>
> *If you skipped Blotato in Phase 4, this still matters — it's the time you'll post manually until you connect a scheduler later."*

## Logic

### Step 1 — Detect timezone

If the user skips Q1: try to read the system timezone (`Intl.DateTimeFormat().resolvedOptions().timeZone` equivalent). If that fails, default to `America/New_York` and note it: *"Defaulted to America/New_York. Edit the file directly if that's wrong."*

### Step 2 — Read Phase 2 platforms

```
read _aibos/state-social.md → platforms_selected, blotato_status
```

Loop only over `platforms_selected` — the unpicked platforms get the template baseline (so the file is complete and re-runnable if the user adds a platform later).

### Step 3 — Per-platform loop

For each platform in `platforms_selected`:

1. Show the industry baseline: *"For [PLATFORM], the baseline is `[default day + hour from template]` — [reasoning, e.g., 'B2B morning scroll' for LinkedIn]. Keep that, or pick your own?"*
2. Accept user's answer. Parse:
   - "Tuesday 9am" → `day_of_week: Tuesday`, `hour: 09:00`
   - "wed 11:30" → `day_of_week: Wednesday`, `hour: 11:30`
   - "skip" / "default" / "baseline" → use template default
3. Ask the optional reasoning question. Accept short or skip.

### Step 4 — For unpicked platforms

Fill each unpicked platform's row with the template baseline (so the file is complete). The draft skills never read these rows (they only read the platforms the user actually drafts for), but having them prevents an "edit and add a platform" flow from breaking later.

The template's industry baselines (from `scheduling-defaults.md.template`):

| Platform | day_of_week | hour | reasoning |
|---|---|---|---|
| LinkedIn | Tuesday | 09:00 | B2B morning scroll |
| Twitter | Wednesday | 10:00 | mid-week peak |
| Instagram | Wednesday | 11:00 | lunch-break scroll |
| Facebook | Thursday | 13:00 | afternoon engagement |
| TikTok | Tuesday | 19:00 | evening watch window |
| Threads | Wednesday | 12:00 | lunch-break, US-anchored |

### Step 5 — Build + write

Read [`../templates/scheduling-defaults.md.template`](../templates/scheduling-defaults.md.template). Interpolate:

- `{{user_timezone}}` → user's timezone string
- `{{<platform>_day}}`, `{{<platform>_hour}}`, `{{<platform>_reasoning}}` → per-platform values (user's or baseline)
- `{{date}}` → today's ISO date

Show the rendered file as a preview, then ask:

```
Plan: write projects/social-media-content/scheduling-defaults.md.

[show rendered file]

Approve? (yes / edit a row)
```

On approval, write the file.

### Step 6 — Update state

Append to `_aibos/state-social.md`:

```yaml
timezone: <user timezone>
scheduling_defaults_path: projects/social-media-content/scheduling-defaults.md
scheduling_custom_count: <number of platforms where user overrode the baseline>
phase_6_scheduling: completed_at <ISO timestamp>
```

If `blotato_status` was `skipped` in Phase 4, also append:
```yaml
phase_6_scheduling_note: blotato_skipped — defaults still saved for manual posting
```

## Write

Files touched in this phase:

- `<workspace>/projects/social-media-content/scheduling-defaults.md` — created from template, all 6 rows filled (user's choices for picked platforms, baselines for the rest)
- `_aibos/state-social.md` — append scheduling block

## Resume

After this phase completes, mark `phase_6_scheduling` as `completed_at <ISO timestamp>` in state-social.md and set `next_pending_phase: 7`. `scheduling_defaults_path` MUST be set.

If the user pauses mid-loop, save partial values to state and resume by re-prompting the un-asked platforms.

## Verification before advancing

Phase 6 is complete when ALL of these are true:

- `projects/social-media-content/scheduling-defaults.md` exists with all 6 rows populated (user values or baselines)
- `timezone` in state is a valid IANA timezone string (e.g., `America/New_York`)
- The user explicitly approved the rendered file

If the user picked "skip" for every platform, the file still gets written (with all baselines) and the phase is complete. The draft skills will offer those baseline times.
