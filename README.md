# Portfolio

Play-first portfolio launch workspace for Bernado's Android apps.

This repository is a coordination hub: research, checklists, release plans, app inventory, and task handoff documents for multiple agents. It intentionally does **not** contain private Play Console account IDs, keystores, passwords, `.env` files, APKs/AABs, screenshots with account metadata, or unpublished private app data.

Generated/initialized: 2026-07-08T13:33:25-03:00

## Current priority

**Make a credible portfolio today.**

Google Play remains the preferred distribution channel when feasible, but today's operating priority is to make the portfolio work visible and organized so other agents can help in parallel.

## Contents

| Path | Purpose |
|---|---|
| `PLAN.md` | Agent-ready implementation plan for the portfolio launch |
| `docs/research/android-signing-release-research.md` | Play-first Android signing/release research |
| `docs/checklists/publish-safe-checklist.md` | Reusable safe publishing checklist |
| `docs/apps/app-inventory.md` | Fill-in matrix for the first four apps |
| `docs/store-listings/` | Draft Play/GitHub/Gumroad listing copy per app |
| `docs/privacy-policies/` | Draft privacy policies per app |

## Operating rules for agents

1. Treat Google Play as the canonical long-term release path.
2. Treat GitHub portfolio visibility as today's priority.
3. Never commit account IDs, keystores, signing passwords, `.env`, private console screenshots, or live personal data.
4. Inspect each app repo before writing claims about it.
5. Record evidence: build command, test result, artifact path, checksum, and known limitations.
6. Keep outputs phone/Termux-friendly.

## Immediate coordination workflow

1. Fill `docs/apps/app-inventory.md` with the four apps.
2. Choose the least risky first app.
3. Create a public-facing README/release page for that app.
4. Prepare screenshots and privacy policy.
5. Prepare Play Console assets and closed-testing plan if required.
6. Repeat with reusable templates.

## Source research

See `docs/research/android-signing-release-research.md` for the researched Android/Google Play constraints and source links.
