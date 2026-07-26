# Prereq check: cowork-ai-os

> Phase 0 runs this BEFORE proceeding. The wizard depends on `about-me/business-brain.md` to auto-derive the brand brief — it cannot ask the user from scratch.

## Step 1: Look for `about-me/business-brain.md`

```bash
ls "<workspace>/about-me/business-brain.md" 2>&1
```

If found → return OK, proceed to Phase 0 welcome.
If not found → halt with this message:

> "I need a file called `about-me/business-brain.md` to know about your business. That comes from cowork-ai-os. Run this first:
>
> `/plugin install cowork-ai-os@cowork-ai-os`
> `/onboard`
>
> When you're done, come back and run `/onboard-social` again."

## Step 2: Look for `about-me/connections.md`

If missing, the user partially onboarded cowork-ai-os. Halt with:

> "Your cowork-ai-os onboarding is incomplete (missing `connections.md`). Run `/onboard` to finish it, then come back."

## Step 3: Confirm `claude.md` exists and is wired in

Spot-check that `<workspace>/claude.md` exists and is referenced in Cowork's Global Instructions. If not, surface a soft warning:

> "Heads up: your handbook (`claude.md`) doesn't seem wired into Cowork's Global Instructions. cowork-ai-os usually does this in Phase 1. Continue anyway?" (Y/N — default Y, just a heads-up)

## Step 4: Check cowork-ai-os version (soft check)

If `cowork-ai-os` SKILL files report a version below `0.10`, surface a soft warning:

> "Heads up: you're on cowork-ai-os `<version>`. cowork-social v0.1 was designed against cowork-ai-os v0.10+. Most phases will still work, but `/browse-connectors` live-fetch (used in the Blotato catalog cross-reference) needs v0.10.2+. Continue anyway?" (Y/N — default Y)

Do not halt — this is a recommendation, not a hard dependency.

## All-clear

If Step 1 + 2 pass, return OK and proceed to Phase 0 welcome. Steps 3 + 4 are soft warnings only.
