# Ping Thing Android — step-by-step Android signing release

Generated: 2026-07-08T20:49:45Z

## Current evidence snapshot

| Field | Value |
|---|---|
| App / store name | Ping Thing |
| Launcher/home-screen label | Ping Thing |
| Repo | renardoberou/Ping-thing-android |
| Package / application ID | `com.resonantsystems.pingthing` |
| Current repo status | Public repo active; installed app visible on phone home screen; repo says Phases 0–2 device-confirmed and Phase 3 infra shipped. |
| Current release workflow state | `.github/workflows/release.yml` already exists and triggers on `v*` tags. |
| Release priority | Play Store primary; GitHub Release APK/AAB as portfolio/tester channel |

| SDK / version field | Observed value |
|---|---|
| `compileSdk` | 35 |
| `targetSdk` | 35 |
| `minSdk` | 26 |
| `versionCode` | `GITHUB_RUN_NUMBER` fallback `1` |
| `versionName` | tag-derived via `VERSION_NAME`, fallback `9.3.0` |

## Non-negotiable release rules

1. **Do not change the package name** after the first public release.
2. **Do not commit** `.jks`, `.keystore`, `.aab`, `.apk`, `keystore.properties`, `.env`, or account metadata.
3. A debug-installed app and a release-signed app usually cannot update each other in place. If Android reports a signature conflict, uninstall the debug build first after exporting anything worth keeping.
4. For Play, upload `.aab`; for GitHub/direct installs, provide `.apk` plus SHA-256 checksum.
5. Keep the keystore outside every repo and back it up offline.

## Step 0 — Confirm the release branch and clean source

```bash
gh repo view renardoberou/Ping-thing-android --json nameWithOwner,defaultBranchRef,visibility,pushedAt
mkdir -p "$HOME/release-work"
# Clone fresh if the local checkout is stale or dirty:
gh repo clone renardoberou/Ping-thing-android "$HOME/release-work/ping-thing-android" -- --depth 1
cd "$HOME/release-work/ping-thing-android"
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
SIGNING_DIR="$HOME/.local/share/android-signing/ping-thing-android"
mkdir -p "$SIGNING_DIR"
chmod 700 "$SIGNING_DIR"
cd "$SIGNING_DIR"

keytool -genkeypair -v   -keystore ping-thing-android-release.jks   -alias pingthing-release   -keyalg RSA -keysize 4096 -validity 10000   -dname "CN=Resonant Systems, O=Resonant Systems"

keytool -list -v   -keystore ping-thing-android-release.jks   -alias pingthing-release   > ping-thing-android-certificate-fingerprint.txt
```

After generation:

- store the keystore password in a password manager;
- copy the `.jks` to an offline backup;
- keep `ping-thing-android-certificate-fingerprint.txt` private or semi-private as release evidence;
- never paste the password, base64 keystore, or fingerprint into a public repo issue/log.

## Step 3 — Add GitHub repository secrets

From the private signing directory:

```bash
cd "$HOME/.local/share/android-signing/ping-thing-android"
base64 -w0 ping-thing-android-release.jks > ping-thing-android-keystore.b64

gh secret set KEYSTORE_B64 --repo renardoberou/Ping-thing-android < ping-thing-android-keystore.b64
gh secret set KEYSTORE_PASS --repo renardoberou/Ping-thing-android
gh secret set KEY_ALIAS --repo renardoberou/Ping-thing-android --body "pingthing-release"
gh secret set KEY_PASS --repo renardoberou/Ping-thing-android

rm ping-thing-android-keystore.b64
```

Verify only the secret names, not values:

```bash
gh secret list --repo renardoberou/Ping-thing-android
```

Required names:

```text
KEYSTORE_B64
KEYSTORE_PASS
KEY_ALIAS
KEY_PASS
```

## Step 4 — Make sure the repo signs release builds

Existing workflow: `.github/workflows/release.yml`. It decodes `KEYSTORE_B64`, derives `VERSION_NAME` from `v*`, and runs `gradle assembleRelease bundleRelease` with signing env vars.

Working directory for the release build:

```text
android
```

Expected release artifacts:

```text
android/app/build/outputs/apk/release/app-release.apk
android/app/build/outputs/bundle/release/app-release.aab
```

This is the best first signed-release candidate because the repo already has a strong `docs/RELEASE.md`, tag-triggered release workflow, and green CI. Use the existing runbook as the operational source for this repo.

## Step 5 — Build the release artifacts

If using GitHub Actions tag release:

```bash
cd "$HOME/release-work/ping-thing-android"
git fetch origin
git switch main
git pull --ff-only

# Replace the version with the chosen public version.
git tag v9.3.0
git push origin v9.3.0

gh run list --repo renardoberou/Ping-thing-android --limit 5
gh release view v9.3.0 --repo renardoberou/Ping-thing-android --web
```

If building locally or reproducing the CI build:

```bash
cd "$HOME/release-work/ping-thing-android/android"
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

- Uses foreground service/media playback, notification, vibration, and internet permissions; Play listing must explain why.
- First signed release not yet cut.
- Direct APK users may need to uninstall debug builds because signatures differ.

## Step 7 — Create the GitHub Release notes

Use this template:

```markdown
# Ping Thing v9.3.0

## What changed
- First signed portfolio/tester release.

## Install
- Android APK: attached below.
- SHA-256: `PASTE_CHECKSUM_HERE`.

## Verification
- Package: `com.resonantsystems.pingthing`
- APK signature verified with `apksigner verify --verbose`.
- Smoke-tested on Moto Edge 60 Fusion / Android 16.

## Known notes
- Launch from the Android home screen.
- Start/stop the ping instrument.
- Confirm audio plays.
- Confirm background/foreground behavior if used.
- Confirm notification permission request behavior on Android 13+.
- Confirm no crash after screen lock/unlock.
```

## Step 8 — Prepare Google Play internal test

Before upload:

| Play field | Value / action |
|---|---|
| App name | Ping Thing |
| Package | `com.resonantsystems.pingthing` |
| Artifact | `.aab` from release workflow |
| Privacy policy | Needed before Play listing |
| Data safety | Explain WebView/internet behavior and no collection only if verified |
| Permissions | Foreground service/media playback, notifications, vibration, internet |
| Screenshots | Capture installed Android app |
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

- Launch from the Android home screen.
- Start/stop the ping instrument.
- Confirm audio plays.
- Confirm background/foreground behavior if used.
- Confirm notification permission request behavior on Android 13+.
- Confirm no crash after screen lock/unlock.

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

