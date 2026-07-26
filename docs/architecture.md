# cowork-social — architecture (v0.1.0)

Internal contributor doc. For the user-facing pitch see the [root README](../../README.md).

## v0.1.0 — Social Content Engine for the Cowork AI OS

`cowork-social` is the fourth plugin in the Cowork AI OS family. It assumes identity already exists (from `cowork-ai-os`) and assumes the user wants a weekly social-content cadence operationalized through Cowork. Its job is narrow: turn the assistant into a content engine that knows the user's voice, has the right platform skills wired, and produces drafts + a calendar that actually ships on time.

The v0.1.0 surface is **13 skills** (1 wizard + 2 foundation skills + 6 platform draft skills + 1 grader + 3 orchestrators) and **7 onboarding templates**. No background agents, no Stop-hooks, no custom MCP server in this release — those are deferred. See [Deferred to later versions](#deferred-to-later-versions).

---

## The 13-skill architecture

```
                         /onboard-social (8-phase wizard)
                                    ↓
                  ┌─────────────────┴─────────────────┐
                  ↓                                   ↓
            /brand-brief ←──────────────────→ /content-coach
            (delta-only)                      (5 ranked ideas)
                  ↓                                   ↓
                  └───────────────┬───────────────────┘
                                  ↓
        ┌─────────────┬──────────────┬──────────────┬─────────────┐
        ↓             ↓              ↓              ↓             ↓
  /draft-linkedin /draft-twitter /draft-instagram /draft-facebook /draft-tiktok /draft-threads
        ↓             ↓              ↓              ↓             ↓             ↓
        └─────────────┴──────────────┼──────────────┴─────────────┴─────────────┘
                                     ↓
                              /grade-post (hook = 50%)
                                     ↓
                  ┌──────────────────┴──────────────────┐
                  ↓                                     ↓
        Blotato MCP queue                 outputs/social-media-content/
        (auto-schedule)                   (manual scheduling fallback)

  Headless / scheduled orchestrators:
    Saturday 8am  →  /generate-weekly-ideas      (10 ranked ideas, NEXT MOVE in file footer)
    Monday 10am   →  /weekly-content-session     (ideas → draft → grade → schedule → log)
    Monthly 1st   →  /content-engine-review      (platform-mix retro + tuning proposals)
```

By category:

| Category | Skills | Count |
|---|---|---|
| Wizard | `/onboard-social` | 1 |
| Foundation | `/brand-brief`, `/content-coach` | 2 |
| Platform drafts | `/draft-linkedin`, `/draft-twitter`, `/draft-instagram`, `/draft-facebook`, `/draft-tiktok`, `/draft-threads` | 6 |
| Quality | `/grade-post` | 1 |
| Orchestrators | `/weekly-content-session`, `/generate-weekly-ideas`, `/content-engine-review` | 3 |
| **Total** | | **13** |

---

## The 3 architectural foundations

Every cowork-* plugin must apply these 3 foundations in every skill. The full canonical spec lives in [`skills/_shared/foundations.md`](../skills/_shared/foundations.md).

### Foundation A — Lazy-load context

No skill auto-loads identity at startup. Skills read only what they need, only when they need it, and only from the smallest necessary file. This keeps individual skill runs in the 2k–6k-token range instead of the 30k+ that "always-load business-brain" creates.

Per-skill load matrix:

| Skill | Reads | Doesn't read |
|---|---|---|
| `/brand-brief` | `business-brain.md` (one-shot, on first run) | `_research-hot.md`, asset-index |
| `/content-coach` | `brand-brief.md`, `platform-voice.md` | `business-brain.md` |
| `/draft-*` (6) | `brand-brief.md`, `platform-voice.md`, asset-index for that platform | `business-brain.md`, other platform voices |
| `/grade-post` | `brand-brief.md`, hook-patterns.md | platform voices, asset index |
| `/weekly-content-session` | brand-brief + this week's idea list + scheduling-defaults | full business-brain.md |
| `/generate-weekly-ideas` | trend-sources.md + brand-brief one-shot | full business-brain.md, asset-index |
| `/content-engine-review` | calendar-log + memory + skill-improvements | brand-brief.md |

### Foundation B — Self-improving skills

Every skill closes with a 10%-better self-improve prompt:

> "Anything in this run we should bake into the skill so it goes better next time? Reply with a tweak and I'll fold it in."

If the user replies with a tweak, the skill appends it to its own SKILL.md or to `projects/social-media-content/skill-improvements.md` (depending on scope). Append-only, dated, never overwrites. This is how the cohort's collective tuning compounds without breaking everyone's customizations.

### Foundation C — ⚡ NEXT MOVE close

Every skill ends with a specific next action + timing. Never "let me know if you need anything else." Examples:

| Skill | NEXT MOVE example |
|---|---|
| `/content-coach` | `⚡ NEXT MOVE: pick the 3 strongest ideas and run /draft-linkedin against them within the next 25 min while the angle is fresh.` |
| `/draft-linkedin` | `⚡ NEXT MOVE: run /grade-post on this draft, or schedule it for Tuesday 9am if you're happy with it.` |
| `/grade-post` | `⚡ NEXT MOVE: apply fixes #1 and #2 (estimated post-fix score: 82). Re-grade in 5 min.` |
| `/weekly-content-session` | `⚡ NEXT MOVE: review the 5 scheduled posts in Blotato's calendar. Adjust line breaks on the LinkedIn one before it goes live Tuesday 9am.` |

#### Headless variant for scheduled contexts

`/generate-weekly-ideas` runs on a Saturday 8am cron. There's no live user to read a NEXT MOVE block in chat. **Special case (Foundation C-headless):** the NEXT MOVE goes in the **footer of the output file** instead, where the user finds it when they open the file Monday morning.

`/content-engine-review` runs monthly but is invokable on-demand — it uses the standard in-chat NEXT MOVE close.

**Recurrence detection.** When a headless skill detects it's been run 3+ times without the user acting on prior NEXT MOVEs (e.g., the previous 3 ideas files have unchecked checkboxes), it escalates the NEXT MOVE wording — "you've ignored the last 3 weeks of ideas; want me to deactivate this cron or reset the brand-brief?"

---

## Shared spec architecture

cowork-social is built around 3 shared spec files that are the **single source of truth** for all 13 skills. Skills reference these files rather than duplicating their logic.

| Shared file | What it owns | Referenced by |
|---|---|---|
| [`_shared/foundations.md`](../skills/_shared/foundations.md) | The canonical 3-foundations spec (A/B/C with headless variant) | All 13 skills |
| [`_shared/draft-skill-spec.md`](../skills/_shared/draft-skill-spec.md) | The 12-step platform-draft pattern | All 6 `/draft-*` skills |
| [`_shared/hook-patterns.md`](../skills/_shared/hook-patterns.md) | 11 proven hook patterns (Reframe, Parallel Contrast, Specific Number, Question, Vulnerable Story, Contrarian Take, Pain Point List, Behind-the-Scenes, Receipts, Most People Reverse, Stolen Lesson) | `/content-coach`, all 6 `/draft-*`, `/grade-post` |

Rationale: the 6 platform-draft skills differ only in character limits, hashtag rules, image-mandatory rules, and platform-specific tone notes. Their 12-step process is identical. Putting that in `draft-skill-spec.md` means a bug fix or improvement in step 7 (hook selection) gets fixed in 1 place, not 6. The same logic applies to the hook library — 11 patterns referenced by 8 skills, owned by 1 file.

---

## The Cowork-vs-Claude-Code rewrite rationale

Blotato ships 5 free Claude Skills:

- `content-coach`
- `brand-brief`
- `post-writer`
- `post-grader`
- `post-scheduler`

These target **Claude Code** (the developer-facing CLI), not **Claude Cowork** (the desktop-app for non-developers). Key runtime differences that forced a rewrite:

| Concern | Claude Code (Blotato's target) | Claude Cowork (our target) |
|---|---|---|
| **Skill packaging** | Files placed in `~/.claude/skills/` folder by the user | ZIP upload via Customize → Browse Plugins |
| **Filesystem model** | Full local FS access from CWD | Sandboxed; specific safe-zone paths only |
| **Skill discovery** | `claude-mem`-style folder scan | Plugin manifest (`plugin.json`) declares skills |
| **Identity layer** | Each skill owns its own `brand-brief.md` write | cowork-ai-os owns `about-me/business-brain.md`; cowork-social derives a content-specific brief |
| **Path conventions** | User-arbitrary (`./brand-brief.md`, etc.) | `projects/social-media-content/brand-brief.md` per safe-zone |
| **3 foundations** | Not enforced in originals | Mandatory in every cowork-* skill |

We did not wrap or recommend Blotato's versions. We used their skills as **inspiration + reference** and rewrote 5 native Cowork-runtime versions, then added 2 platforms Blotato doesn't cover (`/draft-twitter`, `/draft-threads`) and 3 native orchestrators (`/weekly-content-session`, `/generate-weekly-ideas`, `/content-engine-review`).

Hook patterns, grader rubric weights, and the content-coach pattern are spiritually descended from Blotato's originals. The runtime is entirely ours.

---

## Blotato MCP integration

### Namespace + tool surface

Blotato's MCP is installed in Cowork as a Custom Connector (Settings → Connectors → Add Custom Connector → paste URL). After OAuth, the tool namespace is:

```
mcp__claude_ai_Blotato__*
```

**Single-prefix, no doubled-prefix pattern** (unlike Context7's `mcp__plugin_context7_context7__` or Playwright's `mcp__plugin_playwright_playwright__`). Live-verified 2026-05-15 — `mcp__claude_ai_Blotato__blotato_get_user` returned `{subscriptionStatus: "active"}`.

The 16 Blotato MCP tools cowork-social touches in v0.1:

| Tool | Used by | Purpose |
|---|---|---|
| `blotato_list_accounts` | `/onboard-social` Phase 4 | Verify accounts are connected for the user's chosen platforms |
| `blotato_create_post` | `/weekly-content-session`, each `/draft-*` (optional auto-schedule) | Push a draft to Blotato's queue |
| `blotato_create_presigned_upload_url` | `/weekly-content-session` (when posting with images) | Direct local-file upload — no Drive/Dropbox/S3 middleman |
| `blotato_list_posts`, `blotato_get_post_status` | `/content-engine-review` | Pull published-engagement context for monthly retro |
| `blotato_get_schedule`, `blotato_list_schedules` | `/weekly-content-session` | Surface the current week's queue before adding new posts |
| `blotato_update_schedule`, `blotato_delete_schedule` | (deferred — direct natural-language in Cowork chat for now) | Reschedule / delete operations |

`blotato_create_visual`, `blotato_create_source`, and `blotato_list_visual_templates` are NOT used in v0.1 — those land with `/create-visual` and `/remix-source` in v0.2.

### Phase 4 decision tree (Blotato setup)

```
User runs /onboard-social → wizard reaches Phase 4
   ↓
Is mcp__claude_ai_Blotato__* available in the env?
   ├── Yes → call blotato_get_user → has subscriptionStatus: "active"?
   │          ├── Yes → "You're already connected. Confirm your social accounts in blotato_list_accounts."
   │          └── No  → "Account exists but inactive — log into Blotato, then re-run /onboard-social Phase 4."
   │
   └── No → "Do you want to install Blotato?"
              ├── New user → surface affiliate link (blotato.com/?ref=nuno) + NUNO30 code (30% off 3 months)
              │              + 3-click Custom Connector install instructions
              ├── Existing user → "Just OAuth-connect via Settings → Connectors → Blotato."
              └── Skip → Mark phase complete with blotato_skipped: true.
                          Drafts go to outputs/social-media-content/. ~80% value retained.
```

The affiliate framing is gated to the "new user" branch only. We do NOT push the affiliate to users who already have Blotato.

---

## Soft-dependency on `cowork-ai-os`

`cowork-social` does not bundle identity. The wizard halts cleanly if `about-me/business-brain.md` is missing, with a paste-ready prompt directing the user to `cowork-ai-os`'s `/onboard`.

**What the wizard reads (read-only, never overwrites):**

- `about-me/about-me.md` — who the user is
- `about-me/business-brain.md` — ICP, offer, voice samples, current focus
- `about-me/connections.md` — what's already wired (so we don't re-recommend Blotato if it's installed)
- `safe-zones.md` — current scoped paths

**What the wizard appends (append-only, never rewrites):**

- `about-me/connections.md` — Blotato entry if newly wired in Phase 4
- `about-me/memory.md` — `YYYY-MM-DD: cowork-social v0.1.0 onboarded for <business-type>, <platforms>`
- `safe-zones.md` — adds `projects/social-media-content/` and `outputs/social-media-content/` to the skill scope

**Prereq check (`checks/prereq-cowork-ai-os.md`):**

A markdown checklist Claude follows in Phase 0. Verifies:

1. `about-me/` directory exists
2. `business-brain.md` is non-empty
3. `safe-zones.md` is writable
4. `cowork-ai-os` plugin version is `>= 0.10`

Failure → wizard halts with paste-ready instructions to install/onboard `cowork-ai-os` first.

---

## State.md resumability

The wizard writes `<workspace>/_aibos/state-social.md` on every phase transition. The state file tracks:

- `install_complete: <bool>`
- `current_phase: <int>` (0–8)
- `next_pending_phase: <int>`
- `business_type: <one of 6>`
- `platforms_chosen: <list of 6 candidates: linkedin, twitter, instagram, facebook, tiktok, threads>`
- `posting_cadence: <object — per-platform default times>`
- `blotato_connected: <bool>`
- `blotato_skipped: <bool>`
- `voice_calibration: <object — sample-post anchors, banned phrases, signature hooks>`
- `cadence_set: <bool>`
- `calibration_check_date: <ISO date — 14 days from install>`

Every phase reads state on entry and writes on exit. If interrupted, `/onboard-social` re-invocation reads `next_pending_phase` and resumes there.

If `install_complete: true`, re-invocation asks the user if they want to redo a specific phase. Each phase is idempotent — redoing Phase 5 (voice calibration) doesn't reset platform picks; redoing Phase 4 (Blotato setup) doesn't re-collect voice samples.

State template lives at `skills/onboard-social/templates/state-social.md.template`. Bumped to `v1` for the v0.1.0 release; future schema additions bump the template version and add migration logic in `/onboard-social` Phase 0.

---

## Workspace files written by the wizard

| File | Purpose | First written | Updated by |
|---|---|---|---|
| `projects/social-media-content/brand-brief.md` | Content-specific delta on top of business-brain | `/onboard-social` Phase 3 | `/brand-brief` |
| `projects/social-media-content/platform-voice.md` | Per-platform voice notes, banned phrases, tone anchors | `/onboard-social` Phase 5 | `/onboard-social` (re-run Phase 5) |
| `projects/social-media-content/asset-index.md` | Index of available images/videos by platform + theme | `/onboard-social` Phase 6 | `/weekly-content-session` (auto-append on new uploads) |
| `projects/social-media-content/scheduling-defaults.md` | Per-platform default posting times | `/onboard-social` Phase 7 | `/onboard-social` (re-run Phase 7) |
| `projects/social-media-content/trend-sources.md` | Source list for `/generate-weekly-ideas` to pull from | `/onboard-social` Phase 7 | `/content-engine-review` (monthly tuning) |
| `projects/social-media-content/calendar-log.md` | Append-only log of every scheduled/posted item | `/onboard-social` Phase 8 (empty stub) | `/weekly-content-session`, `/content-engine-review` |
| `projects/social-media-content/skill-improvements.md` | Append-only log of self-improvement deltas | `/onboard-social` Phase 8 (empty stub) | All 13 skills (Foundation B) |
| `_aibos/state-social.md` | Wizard state for resumability | `/onboard-social` Phase 0 | Every phase |
| `outputs/social-media-content/<YYYY-MM-DD>/<platform>-<slug>.md` | Drafts that didn't auto-schedule | First draft session | `/draft-*` per run |

---

## Connector catalog accuracy

The bundled catalog (`skills/onboard-social/catalogs/connectors-social.md`) uses the cowork-ai-os v0.10 connector schema verbatim. Live-verified entries as of 2026-05-15:

- ✅ **Blotato MCP** — `mcp__claude_ai_Blotato__blotato_get_user` returned `{subscriptionStatus: "active"}`
- ✅ **Firecrawl, Perplexity, Context7, Playwright** — already live-verified in cowork-research v0.2.0 on 2026-05-15; shared install state across plugins

Skip-list (social-relevant MCPs we deliberately don't recommend in v0.1):

- Hootsuite / Buffer official APIs — Blotato already covers multi-platform; adding more multiplies auth surface
- Native LinkedIn / X / Meta / TikTok APIs — too much auth complexity for non-developer users; Blotato is the abstraction
- Threads native API — limited public API; Blotato handles it via Meta
- Zapier MCP — possible but adds a third-party billing layer; defer until users ask

---

## Manifest version policy

- **Single source of truth:** `cowork-social/.claude-plugin/plugin.json` `version` field.
- **Mirrors:** `marketplace.json`, `CHANGELOG.md`, `RELEASE-vX.Y.Z.md`, all 13 SKILL.md frontmatter `version:` fields.
- **CI check (Phase G.1 in the implementation plan):** `grep -rn "0\.0\.\|0\.2\.\|1\.0\."` across `*.json *.md *.sh` excluding `.git/` — must return zero matches in initial release. Future releases extend the grep pattern to include the previous version.
- **Semver:** patch for bug fixes, minor for new skills/phases, major for breaking schema changes (state.md, plugin.json structure, or removed skills).

Lesson carried over from `cowork-obsidian` 0.3↔0.4 mismatch and `cowork-research` 0.1↔0.2 mirror chase: bump every mirror in the same commit as `plugin.json`. Don't trust manual chases.

---

## Folder structure

```
cowork-social/                            # repo root
├── README.md                             # user-facing
├── CHANGELOG.md                          # Keep-a-Changelog format
├── RELEASE-v0.1.0.md                     # marketing-voice release notes
├── LICENSE                               # MIT
├── .claude-plugin/
│   └── marketplace.json                  # plugin marketplace entry
├── build-release-zip.sh                  # excludes docs/sops/* defensively
├── build-release-zip.py                  # cross-platform alt
├── release/
│   └── cowork-social-v0.1.0.zip          # produced by build-release-zip.sh
└── cowork-social/                        # the actual plugin
    ├── .claude-plugin/
    │   └── plugin.json                   # version: 0.1.0
    ├── docs/
    │   └── architecture.md               # this file
    └── skills/
        ├── _shared/                      # single source of truth
        │   ├── foundations.md            # the 3 foundations spec
        │   ├── draft-skill-spec.md       # 12-step platform-draft pattern
        │   └── hook-patterns.md          # 11 proven hook patterns
        ├── onboard-social/               # the wizard
        │   ├── SKILL.md
        │   ├── phases/                   # 00..08 phase files
        │   ├── checks/                   # prereq + cloud-sync checks
        │   ├── personalization/          # 6 business-type profiles
        │   ├── catalogs/                 # connectors-social.md
        │   └── templates/                # 7 templates (state, brand-brief, platform-voice, asset-index, scheduling-defaults, trend-sources, calendar-log)
        ├── brand-brief/
        ├── content-coach/
        ├── draft-linkedin/
        ├── draft-twitter/
        ├── draft-instagram/
        ├── draft-facebook/
        ├── draft-tiktok/
        ├── draft-threads/
        ├── grade-post/
        ├── weekly-content-session/
        ├── generate-weekly-ideas/
        └── content-engine-review/
```

Course SOPs live in the **private** course workspace, NOT in this repo. This is a hard rule (cowork-obsidian v0.5 cleanup, cowork-research v0.1+v0.2 same hard rule). `build-release-zip.sh` defensively excludes `docs/sops/*` even if a contributor accidentally adds them.

---

## Deferred to later versions

Tracked here so contributors know what's intentionally NOT in v0.1:

- **`/create-visual`** (v0.2+) — Blotato visual templates (carousels, infographics, quote cards, manga, whiteboard) via `blotato_create_visual` + `blotato_list_visual_templates`. Pairs with `/weekly-content-session` to auto-attach visuals.
- **`/remix-source`** (v0.2+) — YouTube / podcast / PDF / website scrape via `blotato_create_source` → platform-specific posts + auto-visual. Sabrina's "scrape Hormozi YouTube → LinkedIn post + whiteboard infographic" workflow.
- **Mid-week tune-up skill** (v0.2+) — Wednesday refresh between Monday batch and weekend ideation. Reads `/grade-post` retro + adjusts next batch's brief.
- **Custom Blotato schedules per ICP segment** (v0.3+) — different posting cadences for different audience segments.
- **Multi-account / multi-brand support** (v0.3+) — one brand per workspace in v0.1.
- **Engagement-data feedback loop** (v0.3+) — `/grade-post` learns from `blotato_list_posts` published metrics, surfaces "your hooks score higher when you start with a number" patterns.
- **Bluesky, Pinterest, YouTube publishing** (v0.3+) — Blotato supports them; we don't wire them in v0.1.
- **Custom Cowork MCP server** (v0.4+) — `social_draft`, `social_grade`, `calendar_log` as first-class MCP tools. Ships once we've watched 20+ users hit specific friction.
- **Stop-hook auto-extract** (v0.4+) — fold session insights into `projects/social-media-content/skill-improvements.md` automatically.

---

## Hard rules (contributor checklist)

- Never re-collect identity (read `about-me/`, never overwrite)
- Append-only on `about-me/connections.md`, `safe-zones.md`, `calendar-log.md`, `skill-improvements.md`
- Plan-then-approve for every write outside `safe-zones.md`-declared paths
- 3rd–4th-grade reading level for every prompt the wizard shows the user
- Resumable: every phase reads + writes `state-social.md`
- No course SOPs in this repo (private workspace only)
- Bump every version mirror in the same commit as `plugin.json`
- Match `cowork-ai-os v0.10` connector schema verbatim in `connectors-social.md`
- Every skill applies the 3 foundations (lazy-load, self-improve, ⚡ NEXT MOVE)
- Headless / scheduled skills use the Foundation C-headless variant (NEXT MOVE in file footer)
- Affiliate link to Blotato is gated to new-user branch only — never push to existing Blotato accounts

---

## References

- [Implementation plan (private workspace)](../../../Claude%20Co-Work/docs/superpowers/plans/2026-05-15-cowork-social-v0.1-implementation.md)
- `cowork-ai-os` plugin — identity + connector + safe-zones
- `cowork-obsidian` plugin — memory architecture pattern
- `cowork-research` plugin — sibling Wave 2 plugin; same shared-spec architecture pattern
- [Blotato MCP docs](https://help.blotato.com/api/mcp) — 16 tools, OAuth flow, namespace verification
- [Sabrina Ramonov — Cowork 60-min masterclass](https://youtu.be/-q_wgmmD0e0) — Use Case 5 (Blotato + content calendar) is the framing source for v0.1
