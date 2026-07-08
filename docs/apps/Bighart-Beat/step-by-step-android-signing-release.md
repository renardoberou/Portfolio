# Bighart Beat — step-by-step Android signing release

Generated: 2026-07-08T20:49:45Z

## Current evidence snapshot

| Field | Value |
|---|---|
| App / store name | Bighart Beat |
| Launcher/home-screen label | Bighart Beat |
| Repo | renardoberou/Bighart-Beat |
| Package / application ID | `com.resonantsystems.bighartbeat` |
| Current repo status | Public repo active; GitHub Pages browser version is live; Android shell exists under `android/`. |
| Current release workflow state | `.github/workflows/android-release.yml` already exists and triggers on `app-v*` tags. |
| Release priority | Play Store primary; GitHub Release APK/AAB as portfolio/tester channel |

| SDK / version field | Observed value |
|---|---|
| `compileSdk` | 35 |
| `targetSdk` | 35 |
| `minSdk` | 26 |
| `versionCode` | `GITHUB_RUN_NUMBER` fallback `1` |
| `versionName` | tag-derived via `VERSION_NAME`, fallback `1.0.0` |

## Non-negotiable release rules

1. **Do not change the package name** after the first public release.
2. **Do not commit** `.jks`, `.keystore`, `.aab`, `.apk`, `keystore.properties`, `.env`, or account metadata.
3. A debug-installed app and a release-signed app usually cannot update each other in place. If Android reports a signature conflict, uninstall the debug build first after exporting anything worth keeping.
4. For Play, upload `.aab`; for GitHub/direct installs, provide `.apk` plus SHA-256 checksum.
5. Keep the keystore outside every repo and back it up offline.

## Step 0 — Confirm the release branch and clean source

```bash
gh repo view renardoberou/Bighart-Beat --json nameWithOwner,defaultBranchRef,visibility,pushedAt
mkdir -p "$HOME/release-work"
# Clone fresh if the local checkout is stale or dirty:
gh repo clone renardoberou/Bighart-Beat "$HOME/release-work/bighart-beat" -- --depth 1
cd "$HOME/release-work/bighart-beat"
git status --short
git log -1 --oneline
```

Safety check before any release tag:

```bash
git grep -n -i -E 'api[_-]?key|secret|token|password|authorization: bearer|ghp_|sk-[A-Za-z0-9]' HEAD || true
git grep -n -E '\.(jks|keystore|apk|aab)$' HEAD || true
```

Expected result: only documentation/workflow placeholder names, not real secret values or binary release artifacts.

## Step 1 — Decide signing strategy before first public cut

Recommended for this app:

| Decision | Recommendation |
|---|---|
| Primary channel | Google Play |
| Portfolio/tester channel | GitHub Release signed APK |
| Key model | Generate one private release/upload keystore for this app now; use it for GitHub direct APK and as Play upload key unless Play strategy changes |
| Cross-store update compatibility | If direct APK users must later update to Play without reinstall, provide the same app-signing key to Play or distribute Play-signed APKs outside Play. Otherwise accept separate direct-vs-Play upgrade paths. |

## Step 2 — Generate the private release key on the phone

Run from Termux, **outside the repo**:

```bash
pkg install openjdk-17
SIGNING_DIR="$HOME/.local/share/android-signing/bighart-beat"
mkdir -p "$SIGNING_DIR"
chmod 700 "$SIGNING_DIR"
cd "$SIGNING_DIR"

keytool -genkeypair -v   -keystore bighart-beat-release.jks   -alias bighartbeat-release   -keyalg RSA -keysize 4096 -validity 10000   -dname "CN=Resonant Systems, O=Resonant Systems"

keytool -list -v   -keystore bighart-beat-release.jks   -alias bighartbeat-release   > bighart-beat-certificate-fingerprint.txt
```

After generation:

- store the keystore password in a password manager;
- copy the `.jks` to an offline backup;
- keep `bighart-beat-certificate-fingerprint.txt` private or semi-private as release evidence;
- never paste the password, base64 keystore, or fingerprint into a public repo issue/log.

## Step 3 — Add GitHub repository secrets

From the private signing directory:

```bash
cd "$HOME/.local/share/android-signing/bighart-beat"
base64 -w0 bighart-beat-release.jks > bighart-beat-keystore.b64

gh secret set KEYSTORE_B64 --repo renardoberou/Bighart-Beat < bighart-beat-keystore.b64
gh secret set KEYSTORE_PASS --repo renardoberou/Bighart-Beat
gh secret set KEY_ALIAS --repo renardoberou/Bighart-Beat --body "bighartbeat-release"
gh secret set KEY_PASS --repo renardoberou/Bighart-Beat

rm bighart-beat-keystore.b64
```

Verify only the secret names, not values:

```bash
gh secret list --repo renardoberou/Bighart-Beat
```

Required names:

```text
KEYSTORE_B64
KEYSTORE_PASS
KEY_ALIAS
KEY_PASS
```

## Step 4 — Make sure the repo signs release builds

Existing workflow: `.github/workflows/android-release.yml`. It decodes `KEYSTORE_B64`, derives `VERSION_NAME` from `app-v*`, and runs `gradle assembleRelease bundleRelease` with signing env vars.

Working directory for the release build:

```text
android
```

Expected release artifacts:

```text
android/app/build/outputs/apk/release/app-release.apk
android/app/build/outputs/bundle/release/app-release.aab
```

No release workflow patch should be needed if the four required secrets exist. First release tag format must include the `app-` prefix, for example `app-v1.0.0`.

## Step 5 — Build the release artifacts

If using GitHub Actions tag release:

```bash
cd "$HOME/release-work/bighart-beat"
git fetch origin
git switch main
git pull --ff-only

# Replace the version with the chosen public version.
git tag app-v1.0.0
git push origin app-v1.0.0

gh run list --repo renardoberou/Bighart-Beat --limit 5
gh release view app-v1.0.0 --repo renardoberou/Bighart-Beat --web
```

If building locally or reproducing the CI build:

```bash
cd "$HOME/release-work/bighart-beat/android"
gradle assembleRelease bundleRelease
```

## Step 6 — Verify artifacts before publishing links

Download the release assets, then:

```bash
apksigner verify --verbose path/to/*.apk
sha256sum path/to/*.apk path/to/*.aab
```

Install smoke test on the phone:

```bash
adb install -r path/to/*.apk
```

If `adb` is not available from Termux, install the APK with Android package installer from shared storage and document the exact APK filename and SHA-256.

Smoke test checklist:

- Has Android foreground service, media playback, notification, vibration, and internet permissions; Play listing/privacy copy must explain the app behavior clearly.
- Because a browser version exists, describe it as “also available in browser,” not as a replacement for the installed Android app.
- First signed Android release has not yet been cut.

## Step 7 — Create the GitHub Release notes

Use this template:

```markdown
# Bighart Beat app-v1.0.0

## What changed
- First signed portfolio/tester release.

## Install
- Android APK: attached below.
- SHA-256: `PASTE_CHECKSUM_HERE`.

## Verification
- Package: `com.resonantsystems.bighartbeat`
- APK signature verified with `apksigner verify --verbose`.
- Smoke-tested on Moto Edge 60 Fusion / Android 16.

## Known notes
- Confirm Android app opens from the launcher.
- Confirm the bundled instrument/audio UI loads.
- Confirm the app content matches the live GitHub Pages browser version where expected.
- Confirm audio/vibration/notification behavior if those permissions are part of the user-facing feature set.
```

## Step 8 — Prepare Google Play internal test

Before upload:

| Play field | Value / action |
|---|---|
| App name | Bighart Beat |
| Package | `com.resonantsystems.bighartbeat` |
| Artifact | `.aab` from release workflow |
| Privacy policy | Needed before Play listing |
| Data safety | Explain internet/WebView behavior and no analytics/collection only if verified |
| Permissions | Foreground service/media playback, notifications, vibration, internet |
| Screenshots | Capture installed Android app, not only web page |
| Feature graphic | 1024×500 required |

Build/upload the `.aab` first to **Internal testing**, not production.

If the Play account is treated as a new personal account, plan for the closed-test gate:

```text
12 opted-in testers for 14 continuous days before production access.
```

## Step 9 — Portfolio update after the release is real

After the GitHub Release exists and artifacts verify:

1. Add the release link to `renardoberou/portfolio-site`.
2. Add screenshots/GIFs.
3. Add privacy policy link.
4. Add Play link only after the Play listing is actually approved/live.
5. Do not call the app “Play Store available” until the public Play URL returns live.

## Release blockers to clear

- Confirm Android app opens from the launcher.
- Confirm the bundled instrument/audio UI loads.
- Confirm the app content matches the live GitHub Pages browser version where expected.
- Confirm audio/vibration/notification behavior if those permissions are part of the user-facing feature set.

## Official source links

- Android signing: https://developer.android.com/studio/publish/app-signing
- Android App Bundles: https://developer.android.com/guide/app-bundle
- Build from command line: https://developer.android.com/build/building-cmdline
- `apksigner`: https://developer.android.com/studio/command-line/apksigner
- Android versioning: https://developer.android.com/studio/publish/versioning
- Play App Signing: https://support.google.com/googleplay/android-developer/answer/9842756
- Play target API requirements: https://support.google.com/googleplay/android-developer/answer/11926878
- Play testing requirements for new personal accounts: https://support.google.com/googleplay/android-developer/answer/14151465
- Play store listing assets: https://support.google.com/googleplay/android-developer/answer/1078870

