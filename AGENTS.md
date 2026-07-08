# Agent Handoff

This repository is public-facing coordination for Bernado's app portfolio.

## Rules

- Do not commit account IDs, private Play Console screenshots, keystores, passwords, tokens, `.env`, APKs, AABs, or private app data.
- Inspect source repos before claiming build status, permissions, SDK versions, or privacy behavior.
- Prefer small commits with evidence-backed docs.
- Keep Termux/mobile constraints in mind.
- Play Store is the preferred official release path; GitHub portfolio visibility is today's priority.
- When duplicate local paths exist, prefer the most recent version after checking directory mtime, git commit date, branch freshness, and working-tree cleanliness. Document the selected path and why.

## Start here

1. Read `PLAN.md`.
2. Fill or update `docs/apps/app-inventory.md`.
3. Pick one task from the Parallel-agent task queue.
4. Commit docs only after running the safety grep in `PLAN.md`.
