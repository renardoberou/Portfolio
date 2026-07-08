# Portfolio Launch Implementation Plan

> **For agents:** This repo is the coordination hub. Work from this plan, commit small changes, and keep all private account/signing material out of Git.

**Goal:** Make Bernado's Android app portfolio credible and visible today, while preserving the path to Google Play release.

**Architecture:** This is a documentation/control repository, not an app monorepo. App source remains in each app's own repository. This repo holds release research, app inventory, portfolio copy, Play/Gumroad/GitHub checklists, privacy-policy drafts, and task handoffs so parallel agents can work without losing context.

**Tech stack:** Markdown, GitHub repo/project coordination, Android/Gradle release artifacts in app repos, Google Play Console for official distribution, Termux-first commands.

Generated: 2026-07-08T13:33:25-03:00

---

## Acceptance criteria for today

- [ ] Public GitHub repo exists and is safe to share.
- [ ] Research and release constraints are committed.
- [ ] Four-app inventory is filled or clearly marked unknown.
- [ ] One app is selected as the first portfolio/release candidate.
- [ ] Each app has an owner-ready task list.
- [ ] No private Play Console account ID, keystore, token, password, `.env`, APK/AAB, or personal screenshot is committed.
- [ ] Other agents can pick up a task from this repo without asking for the whole backstory.

---

## Task 1: Fill the four-app inventory

**Objective:** Establish exactly which apps are in scope.

**Files:**
- Modify: `docs/apps/app-inventory.md`

**Steps:**
1. Add the four app names and repo URLs/paths.
2. For each app, record package name, current build status, target SDK, and release risk.
3. Mark unknowns explicitly as `unknown`, not guessed.
4. Commit:

```bash
git add docs/apps/app-inventory.md
git commit -m "docs: add initial app inventory"
```

**Verification:** The inventory table has four app rows and no blank app names.

---

## Task 2: Pick first release candidate

**Objective:** Choose the app most likely to produce a credible portfolio result today.

**Files:**
- Modify: `docs/apps/app-inventory.md`
- Create: `docs/apps/first-release-candidate.md`

**Selection criteria:**
1. Builds reliably.
2. Has clear user value in one sentence.
3. Needs minimal private data or sensitive permissions.
4. Can be screenshot/demoed from the phone.
5. Can ship as a portfolio MVP even before Play production access.

**Verification:** `first-release-candidate.md` states the chosen app, evidence, risks, and next three commands/steps.

---

## Task 3: Build public-facing copy for each app

**Objective:** Make the portfolio readable by humans and evaluators.

**Files:**
- Create one file per app under `docs/store-listings/APP_SLUG.md`

**Template:**

```markdown
# APP_NAME listing draft

## One-line pitch

## Problem

## What it does

## What Bernado implemented
- 
- 
- 

## Tech stack

## Screenshots needed
- [ ] Home/main screen
- [ ] Core feature
- [ ] Settings/results
- [ ] About/privacy/support

## Play Store fields
- App name:
- Short description <= 80 chars:
- Full description:
- Category:
- Contact email:

## GitHub/Gumroad fields
- Release title:
- Feature bullets:
- Install instructions:
- Known limitations:
```

**Verification:** Each draft has at least a one-line pitch, problem, implementation bullets, and screenshot list.

---

## Task 4: Prepare privacy policy drafts

**Objective:** Remove Play/Gumroad blocker around privacy disclosure.

**Files:**
- Create one file per app under `docs/privacy-policies/APP_SLUG.md`

**Rules:**
- Do not claim “no data transmitted” until the app source is inspected.
- State local storage, permissions, analytics, ads, network calls, and contact path.
- Keep policies simple and honest.

**Verification:** Each policy has effective date, app name, data collected, data sharing, permissions, contact.

---

## Task 5: App-by-app technical audit

**Objective:** For each app, produce verified release facts.

**Files:**
- Create/update: `docs/apps/APP_SLUG-audit.md`

**Required checks per app repo:**

```bash
git status --short
git log --oneline -3
# Inspect Android config, package name, minSdk, targetSdk/versioning.
# Run the app's actual build/test command where feasible.
```

**Record:**
- repo URL/path;
- package name;
- target SDK;
- build command/result;
- artifact path if built;
- permissions;
- privacy implications;
- screenshots needed;
- Play readiness blockers;
- direct portfolio readiness blockers.

**Verification:** Audit claims are backed by command output or file paths.

---

## Task 6: Play Console readiness path

**Objective:** Keep Play as the official target without blocking today's portfolio.

**Files:**
- Create/update: `docs/play-console-readiness.md`

**Steps:**
1. Record account type/status without committing account ID.
2. Record whether closed testing is required.
3. If required, create tester recruitment plan for 12 testers / 14 continuous days.
4. List per-app Play setup tasks: app creation, AAB upload, signing, content rating, data safety, target audience, screenshots.

**Verification:** The doc says whether production can be attempted now or whether closed testing gates it.

---

## Task 7: Publish portfolio-facing artifacts

**Objective:** Make public links visible even before Play approvals finish.

**Files:**
- Update: `README.md`
- Optional: create `docs/portfolio-page-copy.md`

**Steps:**
1. Add table of apps with status and links.
2. Link each source repo and release page when ready.
3. Link Play Store pages only after they exist.
4. If direct APKs are used, include checksum and unknown-sources warning.

**Verification:** Open the GitHub repo in a browser/private view and confirm the README is useful without private context.

---


## Duplicate local repo policy

When two local directories appear to represent the same app, prefer the most recent version **after verifying evidence**:

1. Compare directory/file mtimes with `stat`.
2. Compare git HEAD commit, commit date, remote URL, branch freshness, and working-tree cleanliness.
3. Prefer the newest valid working copy unless it is dirty, stale, or unsafe.
4. Record the selected path and any ignored duplicate in `docs/apps/app-inventory.md` and the relevant audit file.

## Parallel-agent task queue

Agents can safely pick these in parallel after Task 1:

| Task | Agent input needed | Output |
|---|---|---|
| App audit A | app repo/path | `docs/apps/APP-audit.md` |
| App audit B | app repo/path | `docs/apps/APP-audit.md` |
| App listing copy | app audit | `docs/store-listings/APP.md` |
| Privacy policy | app audit | `docs/privacy-policies/APP.md` |
| Screenshot checklist | app audit | section in app audit/listing |
| Play readiness | account status + app audits | `docs/play-console-readiness.md` |

---

## Non-negotiable safety checks before every commit

```bash
git diff --cached --name-only
git grep --cached -n -i -E 'account id|ID da conta|[0-9]{12,}|ghp_|sk-[A-Za-z0-9]|api[_-]?key|token|password|secret|keystore|\.jks|\.env' || true
```

Any real secret/private account metadata hit must be removed before push.
