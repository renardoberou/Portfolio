# App Inventory

Filled from Bernado's repo list on 2026-07-08. Do not guess missing technical fields; use `unknown` until an agent inspects the repo.

| Priority | App | Repo | Local path | Package name | Build status | targetSdk | Release risk | First channel | Next action |
|---:|---|---|---|---|---|---:|---|---|---|
| 1 | Bighart Beat (`Bighart` on home screen) | [Bighart-Beat](https://github.com/renardoberou/Bighart-Beat) | `/data/data/com.termux/files/home/bighart-beat`; `/storage/emulated/0/Documents/bighart-beat` | unknown — audit required | unknown — audit required | unknown | unknown | Play-first / portfolio now | audit repo and identify web/PWA/Android release path |
| 2 | Bighart Synth Standalone | [Bighart-synth-standalone](https://github.com/renardoberou/Bighart-synth-standalone) | `not found locally; clone if needed` | unknown — audit required | unknown — audit required | unknown | unknown | Play-first / portfolio now | clone/audit repo and classify release artifact |
| 3 | Ping Thing Android (`Ping Thing` on home screen) | [Ping-thing-android](https://github.com/renardoberou/Ping-thing-android) | `/data/data/com.termux/files/home/ping-thing-android` | unknown — audit required | unknown — audit required | unknown | unknown | Play-first / portfolio now | pull/update local clone, audit Android config, and reuse existing RELEASE.md |
| 4 | Spectral Camera | [spectral-camera](https://github.com/renardoberou/spectral-camera) | `/storage/emulated/0/Documents/spectral-camera` | unknown — audit required | unknown — audit required | unknown | unknown | Play-first / portfolio now | audit Android config, camera permissions, privacy, and release blockers |

## Verified GitHub repo status

Checked with `gh repo view` on 2026-07-08:

| Repo | Visibility | Default branch | Note |
|---|---|---|---|
| `renardoberou/Bighart-Beat` | public | `main` | exists |
| `renardoberou/Bighart-synth-standalone` | public | `main` | exists |
| `renardoberou/Ping-thing-android` | public | `main` | exists |
| `renardoberou/spectral-camera` | public | `main` | exists; GitHub canonical owner/name is lowercase `spectral-camera` |

## Local path status

Checked on the phone/Termux host:

| App | Local path status |
|---|---|
| Bighart Beat | prefer `/data/data/com.termux/files/home/bighart-beat`: same HEAD as Documents clone, newer directory mtime, clean working tree; `/storage/emulated/0/Documents/bighart-beat` has untracked `.test.*` files |
| Bighart Synth Standalone | no local directory found under home or `/storage/emulated/0/Documents` |
| Ping Thing Android | `/data/data/com.termux/files/home/ping-thing-android` exists; local clone is behind `origin/main` by 7 and has untracked `headless*.png` files |
| Spectral Camera | prefer `/storage/emulated/0/Documents/spectral-camera`: valid git worktree, clean, same mtime evidence as uppercase duplicate; uppercase `/storage/emulated/0/Documents/Spectral-camera` hit Git dubious-ownership warning, so avoid unless freshness evidence changes |

## Release-risk scale

- **Low:** builds, no sensitive permissions, clear local-only privacy story, screenshots easy.
- **Medium:** build works but needs policy/privacy clarification, screenshot polish, or release workflow.
- **High:** build broken, sensitive permissions, network/account data, unclear signing, or Play policy risk.

## Audit assignment template

For each app, create `docs/apps/APP_SLUG-audit.md` with:

- repo URL and inspected commit;
- local path or clone command;
- package name / app ID;
- minSdk / targetSdk / versionName / versionCode;
- build command and result;
- artifact paths if any;
- permissions and privacy implications;
- screenshots needed;
- Play readiness blockers;
- GitHub/Gumroad/portfolio readiness blockers;
- recommended next action.

## Duplicate-resolution policy

When duplicate local paths exist, prefer the most recent valid version. Evidence to check: directory mtime, git HEAD/commit date, remote URL, branch freshness, and working-tree cleanliness. If freshness ties, prefer the clean valid git worktree over a dubious-ownership or dirty duplicate.
