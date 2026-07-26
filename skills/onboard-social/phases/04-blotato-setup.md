# Phase 4 — Blotato Decision + Setup

## What this phase does

THE KEY PHASE. Walks the user through the Blotato decision: connect existing account, sign up fresh (with affiliate link + 30%-off code `NUNO30`), or skip entirely. All three paths leave `state-social.md` in a clean, consistent state. Every subsequent phase tolerates a `blotato_status: skipped` value — drafts still save to `outputs/social-media-content/` and the user posts manually.

This is also the phase where the user sees Nuno's explicit framing: cowork-social is built for Blotato because Blotato works best with everything else in the stack. The affiliate framing is honest — if you already have a Blotato account, ignore the link and OAuth-connect directly.

The full Blotato catalog entry lives in [`../catalogs/connectors-social.md`](../catalogs/connectors-social.md) — pricing, alternatives skip-list, and the live-verified MCP smoke test all live there.

## Ask

The flow is **framing first**, then the **3-path branch**.

### Framing (shown before any question)

> *"Heads up — this is the one connector cowork-social was built around.*
>
> *cowork-social is built for Blotato. We chose it because of our experience — it works best with everything else in the cowork stack. Clean MCP namespace, real per-platform fidelity, one account covers 20 social accounts, and Cowork can offer 'schedule this for [time]?' right inside any /draft skill.*
>
> *If you already have a Blotato account, ignore the affiliate link below — just OAuth-connect directly when I prompt you. The affiliate link is for people new to Blotato, not for swapping an existing account.*
>
> *If you don't want to pay $29/mo for Blotato right now, that's fine too. cowork-social still works — drafts save to your workspace and you post manually until you connect Blotato later. You keep about 80% of the value."*

### The branch question

> *"Which path fits you?*
>
> *1. I have Blotato already — OAuth-connect now*
> *2. I'm new to Blotato — show me the signup link with the discount*
> *3. Skip Blotato — I'll post manually for now*
>
> *Type 1, 2, or 3."*

## Why we ask

> *"You can't get end-to-end 'draft + schedule' without a posting MCP. Blotato is the one that works. But $29/mo is real money and you might already have a different scheduler. I want to make the choice transparent: here's why Blotato, here's the discount, here's the skip path. Pick what fits."*

## Logic

### Path A — "I have Blotato"

1. Tell the user: *"Great. Two things — first, you'll OAuth-connect Blotato to Cowork. Second, I'll smoke-test it by reading your account info."*
2. Walk the user through Custom Connector setup in Claude:
   - *"Open Claude → Settings → Connectors → Add Custom Connector."*
   - *"URL: `https://mcp.blotato.com/mcp`"*
   - *"Click connect, complete the OAuth flow in your browser."*
   - *"Tell me when you see 'Connected' in the Connectors list."*
3. On user confirmation, invoke `mcp__claude_ai_Blotato__blotato_get_user` to verify the connection works.
4. If the call returns a valid user object → write `blotato_status: connected`, log the `subscriptionStatus` field from the response.
5. If the call fails → surface the error in plain English (e.g., *"Looks like the connector isn't seeing your account yet — check the Connectors list shows 'Connected', or try re-OAuthing."*) and loop.

### Path B — "I'm new to Blotato"

1. Show the affiliate framing exactly:

   > *"Here's the signup path:*
   >
   > *Step 1: Go to `https://blotato.com/?ref=nuno` and sign up.*
   > *Step 2: At checkout, apply code `NUNO30` → 30% off your first 3 months ($20.30/mo for 3 months, then $29/mo after).*
   > *Step 3: Inside Blotato, connect your social accounts (each platform = one OAuth click — LinkedIn, Twitter, Instagram, etc.).*
   > *Step 4: Tell me when you're done — I'll walk you through the Cowork side next."*

2. When user reports signup done, walk them through the Custom Connector setup (same as Path A Step 2):
   - *"Open Claude → Settings → Connectors → Add Custom Connector."*
   - *"URL: `https://mcp.blotato.com/mcp`"*
   - *"Click connect, complete OAuth."*

3. Smoke-test via `mcp__claude_ai_Blotato__blotato_get_user`. Same success/failure handling as Path A.

4. On success → write `blotato_status: connected`, also append to `about-me/connections.md`:

   ```
   <ISO date> | Blotato connected via /onboard-social Phase 4 (used affiliate ref=nuno + code NUNO30)
   ```

### Path C — "Skip Blotato"

1. Confirm the user knows what they're skipping:

   > *"Skipping Blotato is fine. Three things to know:*
   >
   > *1. Every `/draft-*` skill still works — drafts save to `outputs/social-media-content/`*
   > *2. The skills won't offer 'schedule this for [time]?' — you'll copy the draft into each platform yourself*
   > *3. Re-run `/onboard-social` phase 4 any time to add Blotato later (5 minutes)*
   >
   > *Skip Blotato? (yes / actually let me reconsider)"*

2. On confirmed skip → write `blotato_status: skipped` to `state-social.md` and append to `about-me/connections.md`:

   ```
   <ISO date> | Blotato skipped via /onboard-social Phase 4 — re-run phase 4 to install
   ```

3. Phase 6 (scheduling defaults) treats this as optional — the file still gets created with defaults, but Phase 7's "test schedule" step is skipped.

### State writes (all three paths)

Append to `_aibos/state-social.md`:

```yaml
blotato_status: <connected|skipped|awaiting-oauth>
blotato_decided_at: <ISO timestamp>
blotato_signup_via_affiliate: <true|false|n/a>   # true only on Path B success
phase_4_blotato: completed_at <ISO timestamp>
```

## Write

Files touched in this phase:

- `_aibos/state-social.md` — append the Blotato status block above
- `<workspace>/about-me/connections.md` — append (append-only) a one-line log of the decision

No writes to `brand-brief.md`, `platform-voice.md`, or other content files in this phase.

## Resume

After this phase completes, mark `phase_4_blotato` as `completed_at <ISO timestamp>` in state-social.md and set `next_pending_phase: 5`. The `blotato_status` field MUST be one of the three valid values (`connected`, `skipped`, `awaiting-oauth`) — never blank.

If the user pauses mid-OAuth (Path A or B), the wizard saves `blotato_status: awaiting-oauth` and resumes by re-prompting: *"Did the OAuth flow finish? Want me to smoke-test again, or restart Phase 4?"*

## Verification before advancing

Phase 4 is complete when ALL of these are true:

- `blotato_status` in state-social.md is `connected` OR `skipped` (not `awaiting-oauth`)
- If `connected`: the `mcp__claude_ai_Blotato__blotato_get_user` smoke test returned a valid user object
- If user picked Path B (new signup): `about-me/connections.md` reflects the affiliate ref code + `NUNO30` discount
- If `skipped`: `about-me/connections.md` reflects the skip + re-run instructions
- The user explicitly picked one of the 3 paths (no implicit defaults)

## Affiliate disclosure (transparency)

The affiliate link `https://blotato.com/?ref=nuno` and code `NUNO30` give Nuno a small revenue share when new users sign up. The 30%-off discount is real and goes to the user. **If you already have a Blotato account, do not use the affiliate link** — there's no benefit and it muddles the attribution.
