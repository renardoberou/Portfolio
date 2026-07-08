# Safe App Publishing Checklist

Use this before publishing an app to Gumroad, GitHub, itch.io, or a portfolio page.

## 1. Pick the app and publish target

- App name:
- Repo/path:
- APK/build artifact path:
- Target: Gumroad / GitHub Release / GitHub Pages / Portfolio page / Other
- Price: free / pay-what-you-want / paid

## 2. Verify the app works

- [ ] Fresh build succeeds
- [ ] Basic smoke test completed on phone
- [ ] Screenshots captured
- [ ] Short demo video or GIF captured if possible
- [ ] Known limitations written down honestly

## 3. Sanitize before public release

- [ ] No API keys, tokens, passwords, `.env`, keystores, private config
- [ ] No personal paths, private chat logs, generated local dashboards, cron output, or wiki snapshots
- [ ] No vendor/leak/provenance-sensitive wording that should not be public
- [ ] Generated artifacts excluded unless intentionally part of release
- [ ] `.gitignore` covers build output and private files

Suggested checks:

```bash
git status --short
git diff --cached --name-only
git grep -n -i -E 'api_key|token|password|secret|Authorization: Bearer|ghp_|sk-' HEAD || true
```

## 4. Package cleanly

- [ ] Public README explains: what it does, who it is for, how to install/use, screenshots, limitations
- [ ] License selected
- [ ] Version number set
- [ ] APK or release zip has a clear name
- [ ] SHA-256 recorded for release files

## 5. Gumroad page

- [ ] Clear title
- [ ] 1-line value proposition
- [ ] 3–5 bullets of features
- [ ] Screenshots/demo media
- [ ] Install instructions
- [ ] Version and changelog
- [ ] Refund/support note
- [ ] Link to public repo or portfolio page if appropriate

## 6. Portfolio entry

Use this format:

```text
Project: <name>
Problem: <what it solves>
Built with: <stack>
What I implemented: <3 bullets>
Demo: <link>
Source/release: <link>
Status: MVP / beta / stable
```

## 7. Release gate

Do not publish until these are true:

- [ ] Clean tree or intentional release commit
- [ ] Build/test result recorded
- [ ] Secret scan reviewed
- [ ] Release files verified
- [ ] Gumroad draft preview checked
- [ ] Public links opened from a private/incognito browser or another device

## Fast path tomorrow

Send Hermes the app names or repo paths and say:

```text
Inspect these apps for safe publishing and prepare Gumroad/portfolio copy.
```

Hermes should inspect files first, sanitize, verify build artifacts, then help draft the Gumroad and portfolio pages.
