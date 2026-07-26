# connectors-social.md — Social-specific MCP connector catalog

> Used by `/onboard-social` Phase 7 to recommend the social-content connector. Schema mirrors `cowork-research` / `cowork-ai-os` v0.10 `browse-connectors` exactly. Last verified: 2026-05-15 (Blotato live smoke-tested).

> **v0.1 + cowork-ai-os v0.10.2 update:** As of cowork-ai-os v0.10.2, `/browse-connectors` live-fetches the official Cowork directory from `claude.com/connectors`. This bundled catalog is the **offline cache fallback** + the source-of-truth for the Blotato-specific install instructions (affiliate link + discount code).

> **Authentication note (read first):** MCPs need authentication on first use (API key or OAuth). Cowork prompts for credentials when you invoke a tool that needs them. **"Already installed" ≠ "ready to use"** — a connector listed as installed still needs its first-use auth handshake before any `/draft-*` skill can schedule through it. Look for the "Authenticate first if Cowork prompts (one-time)" prefix on the first-week test below.

---

## Primary recommendation: Blotato

### Blotato

- **slug:** `blotato`
- **bucket:** Posting (multi-platform scheduling + publishing)
- **what it does:** One MCP that schedules + publishes to 10+ platforms (LinkedIn, Twitter/X, Instagram, Facebook, TikTok, Threads, YouTube, Pinterest, Bluesky, more). Handles per-platform formatting, media uploads, and scheduling queues. Returns post IDs you can track from any `/draft-*` skill.
- **best for:** Every cowork-social user. This is the ONE connector this plugin actually depends on if you want end-to-end "draft + schedule" without leaving chat.
- **why we picked it:** We tested every multi-platform scheduling MCP we could find. Blotato is the one that works best with everything else in the cowork stack — clean MCP namespace, real per-platform fidelity, scheduled-vs-published states you can query later, and a single account that covers 20 social accounts. The other options either don't have a real MCP (Hootsuite/Buffer/Later — see Skip list), only cover one platform (Postiz/Typefully), or have brittle MCP coverage (Ayrshare). If you already have Blotato, just connect it. If you don't, the install link below has our community discount.
- **add it:**
  1. Go to **[blotato.com/?ref=nuno](https://blotato.com/?ref=nuno)** and sign up.
  2. At checkout, apply code **`NUNO30`** → **30% off your first 3 months**.
  3. After signup, connect your social accounts inside Blotato's dashboard (each platform = its own OAuth click).
  4. Generate your Blotato API key (Account → API).
  5. Tell Cowork the key when prompted on first use of `mcp__claude_ai_Blotato__*` — Cowork stores it once, then every cowork-social draft skill can schedule through it.
  6. **If you already have a Blotato account, ignore the affiliate link** — just plug in your existing API key when prompted. The affiliate is for people new to Blotato, not for swapping an existing account.
- **what you unlock:**
  - Every `/draft-*` skill can offer "Schedule this for [default time]?" inline — no copy-paste to other apps
  - Calendar-log automatically tracks `scheduled` / `published` / `failed` states
  - One queue across all your platforms — visible inside Blotato's dashboard if you want to override Cowork
- **first-week test:** Authenticate first if Cowork prompts (one-time API key). Then run `/draft-twitter "test post: just connected blotato"` and accept the scheduling offer. Confirm the scheduled post shows up in your Blotato dashboard within 30 seconds.
- **gotchas:**
  - Instagram + TikTok posts **require media** — drafts won't schedule without an asset reference (`asset-index.md`). The skill catches this at Step 9 and falls back to "draft only — post manually."
  - Blotato's free tier doesn't include scheduling — you need the paid plan ($29/mo, see pricing below) for `/onboard-social` Phase 7 to actually work end-to-end.
  - Platform OAuth tokens expire periodically (Twitter ~90 days, others vary). When a scheduled post fails with auth-error, reconnect that platform in the Blotato dashboard.
  - 20-account limit per Blotato plan (per Sabrina's Blotato masterclass, May 2026). Fine for solo + small agency; agencies with 20+ client accounts need to layer plans.
- **permission level:** Reads = act (list accounts, get post status). Writes = ask (every `blotato_create_post` is plan-then-approve at draft Step 5 + scheduling confirmation at draft Step 9). Destructive (delete schedule) = ask.
- **pricing:** **$29/month** at full price. With `NUNO30` at [blotato.com/?ref=nuno](https://blotato.com/?ref=nuno) → **$20.30/month for 3 months**, then $29/mo after. Includes 20 social accounts. If you skip Blotato, drafts still save to `outputs/social-media-content/` and you post manually inside each platform — about 80% of cowork-social's value still works.
- **last verified:** 2026-05-15 (live smoke test — `mcp__claude_ai_Blotato__blotato_get_user` returned `subscriptionStatus: active`)

---

## Cross-cutting baseline (pointer, not re-documented)

The 4 cross-cutting MCPs — **Perplexity**, **Firecrawl**, **Fathom**, **Context7** — are documented in `cowork-research`'s catalog at `cowork-research/cowork-research/skills/onboard-research/catalogs/connectors-research.md`. cowork-social doesn't re-document them.

| If you have cowork-research installed | You're set — same 4 MCPs cover cowork-social too |
| If you don't have cowork-research installed | Install it for the cross-cutting baseline, OR see the cowork-research catalog as a reference and install the 4 MCPs the same way |

cowork-social uses the cross-cutting baseline for:
- **Perplexity** → `/generate-weekly-ideas` (Lesson 6) trend scanning
- **Firecrawl** → `/generate-weekly-ideas` site scrapes from `trend-sources.md`
- **Fathom** → repurposing call quotes into social posts (future v0.2)
- **Context7** → tech-content drafts that need current library docs

---

## Skip list (social-relevant tools we deliberately don't recommend in v0.1)

| Tool | Why we skip in v0.1 |
|---|---|
| **Hootsuite** | Not an MCP — browser/API only. No way to integrate with Cowork's draft skills without custom plumbing. Surface only when MCP coverage exists. |
| **Buffer** | Same — no MCP. Solid scheduling tool but won't work inline from Cowork chat. |
| **Later** | Same — no MCP. Heavy on visual-first scheduling (IG-centric) which limits multi-platform use. |
| **Postiz** | Open-source self-hosted scheduler. Real product, but no published MCP server. Re-evaluate v0.2. |
| **Typefully** | Single-platform (Twitter/X + Threads). Great for Twitter-only creators but cowork-social is multi-platform by design — adding Typefully alongside Blotato is wasted complexity. |
| **Ayrshare** | Multi-platform API competitor to Blotato. MCP coverage less mature; per-platform field schemas inconsistent. Re-evaluate v0.2 when their MCP matures. |
| **Make.com / Zapier** | General-purpose automation, not a posting MCP. If a user already has Make/Zapier social workflows, those can run in parallel — Cowork doesn't need to replace them. |

---

## How the wizard uses this catalog

`/onboard-social` Phase 7:

1. Read this file.
2. Surface Blotato as the primary recommendation with the full pricing pitch + affiliate framing.
3. Offer three paths:
   - **Connect Blotato** → walk through the API-key handshake; on success, update `state-social.md` → `blotato_status: connected`
   - **Skip for now** → update `state-social.md` → `blotato_status: skipped`. Tell user: "Drafts still save to `outputs/social-media-content/` — you'll post manually until you connect Blotato later. Re-run `/onboard-social` Phase 7 to add it."
   - **Already have Blotato** → ask for API key, skip the affiliate framing entirely, run the smoke test
4. Run the first-week test from this entry (test post).
5. Move to Phase 8 (safe-zones).

---

## What changes in v0.2+

| Version | Likely addition |
|---|---|
| v0.2 | Re-evaluate Ayrshare + Postiz MCP coverage |
| v0.2 | Add Fathom-driven repurposing skill → call quote to social post (uses Fathom from baseline) |
| v0.3+ | Beehiv integration for cross-posting social → newsletter |

Until then, this catalog stays a single-primary-recommendation file by design. Less choice paralysis. One excellent tool > four mediocre ones.
