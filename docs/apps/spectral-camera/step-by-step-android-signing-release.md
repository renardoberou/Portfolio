# Spectral Camera — step-by-step Android signing release

Generated: 2026-07-08T20:49:45Z

## Current evidence snapshot

| Field | Value |
|---|---|
| App / store name | Spectral Camera |
| Launcher/home-screen label | Spectral Camera |
| Repo | renardoberou/spectral-camera |
| Package / application ID | `com.renardoberou.spectralcamera` |
| Current repo status | Public repo active; Kotlin Android app; README says manually verified on Motorola Edge 60 Fusion / Android 16; installed app visible on phone home screen. |
| Current release workflow state | CI builds debug and release APK artifacts, but `release` is currently signed with the debug keystore and must be changed before public/store release. |
| Release priority | Play Store primary; GitHub Release APK/AAB as portfolio/tester channel |

| SDK / version field | Observed value |
|---|---|
| `compileSdk` | 35 |
| `targetSdk` | 35 |
| `minSdk` | 26 |
| `versionCode` | 23 |
| `versionName` | `1.8.1` |

## Non-negotiable release rules

1. **Do not change the package name** after the first public release.
2. **Do not commit** `.jks`, `.keystore`, `.aab`, `.apk`, `keystore.properties`, `.env`, or account metadata.
3. A debug-installed app and a release-signed app usually cannot update each other in place. If Android reports a signature conflict, uninstall the debug build first after exporting anything worth keeping.
4. For Play, upload `.aab`; for GitHub/direct installs, provide `.apk` plus SHA-256 checksum.
5. Keep the keystore outside every repo and back it up offline.

## Step 0 — Confirm the release branch and clean source

```bash
gh repo view renardoberou/spectral-camera --json nameWithOwner,defaultBranchRef,visibility,pushedAt
mkdir -p "$HOME/release-work"
# Clone fresh if the local checkout is stale or dirty:
gh repo clone renardoberou/spectral-camera "$HOME/release-work/spectral-camera" -- --depth 1
cd "$HOME/release-work/spectral-camera"
git status --short
git log -1 --oneline
```

Safety check before any release tag:

```bash
git grep -n -i -E 'api[_-]?key|secret|token|password|authorization|ghp_|sk-' HEAD || true
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
SIGNING_DIR="$HOME/.local/share/android-signing/spectral-camera"
mkdir -p "$SIGNING_DIR"
chmod 700 "$SIGNING_DIR"
cd "$SIGNING_DIR"

keytool -genkeypair -v   -keystore spectral-camera-release.jks   -alias spectralcamera-release   -keyalg RSA -keysize 4096 -validity 10000   -dname "CN=Resonant Systems, O=Resonant Systems"

keytool -list -v   -keystore spectral-camera-release.jks   -alias spectralcamera-release   > spectral-camera-certificate-fingerprint.txt
```

After generation:

- store the keystore password in a password manager;
- copy the `.jks` to an offline backup;
- keep `spectral-camera-certificate-fingerprint.txt` private or semi-private as release evidence;
- never paste the password, base64 keystore, or fingerprint into a public repo issue/log.

## Step 3 — Add GitHub repository secrets

From the private signing directory:

```bash
cd "$HOME/.local/share/android-signing/spectral-camera"
base64 -w0 spectral-camera-release.jks > spectral-camera-keystore.b64

gh secret set KEYSTORE_B64 --repo renardoberou/spectral-camera < spectral-camera-keystore.b64
gh secret set KEYSTORE_PASS --repo renardoberou/spectral-camera
gh secret set KEY_ALIAS --repo renardoberou/spectral-camera --body "spectralcamera-release"
gh secret set KEY_PASS --repo renardoberou/spectral-camera

rm spectral-camera-keystore.b64
```

Verify only the secret names, not values:

```bash
gh secret list --repo renardoberou/spectral-camera
```

Required names:

```text
KEYSTORE_B64
KEYSTORE_PASS
KEY_ALIAS
KEY_PASS
```

## Step 4 — Make sure the repo signs release builds

Current `.github/workflows/android.yml` builds `assembleDebug assembleRelease` on pushes/PRs and uploads `spectral-camera-release`, but `app/build.gradle.kts` signs release with the debug keystore. Replace that before any public release.

Working directory for the release build:

```text
.
```

Expected release artifacts:

```text
app/build/outputs/apk/release/app-release.apk
app/build/outputs/bundle/release/app-release.aab
```

Patch `app/build.gradle.kts` so release signing is env-driven (`KEYSTORE_FILE`, `KEYSTORE_PASS`, `KEY_ALIAS`, `KEY_PASS`) and add/adjust a tag-triggered release workflow. Do not publish the current debug-signed release APK as a real release.

## Step 5 — Build the release artifacts

If using GitHub Actions tag release:

```bash
cd "$HOME/release-work/spectral-camera"
git fetch origin
git switch main
git pull --ff-only

# Replace the version with the chosen public version.
git tag v1.8.1
git push origin v1.8.1

gh run list --repo renardoberou/spectral-camera --limit 5
gh release view v1.8.1 --repo renardoberou/spectral-camera --web
```

If building locally or reproducing the CI build:

```bash
cd "$HOME/release-work/spectral-camera/."
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

- Current release build uses debug signing; must be replaced.
- Camera permission requires clear privacy policy and Play disclosure.
- README honesty note about simulated IR should be preserved in store copy.
- No automated test suite yet; rely on documented device smoke test before release.

## Step 7 — Create the GitHub Release notes

Use this template:

```markdown
# Spectral Camera v1.8.1

## What changed
- First signed portfolio/tester release.

## Install
- Android APK: attached below.
- SHA-256: `PASTE_CHECKSUM_HERE`.

## Verification
- Package: `com.renardoberou.spectralcamera`
- APK signature verified with `apksigner verify --verbose`.
- Smoke-tested on Moto Edge 60 Fusion / Android 16.

## Known notes
- Launch from home screen.
- Grant camera permission.
- Open live preview.
- Test B&W Infrared and Aerochrome first.
- Capture image; verify saved JPEG orientation and gallery visibility.
- Confirm the app clearly presents simulated spectral/IR looks, not true infrared hardware capture.
```

## Step 8 — Prepare Google Play internal test

Before upload:

| Play field | Value / action |
|---|---|
| App name | Spectral Camera |
| Package | `com.renardoberou.spectralcamera` |
| Artifact | Signed `.aab` required for Play; signed APK okay for GitHub/direct |
| Privacy policy | Required because camera/media permissions are sensitive |
| Data safety | Camera/photos local-only only if verified; disclose no upload/analytics only if true |
| Permissions | Camera, media/image storage compatibility permissions |
| Screenshots | Capture camera UI and sample simulated effects |
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

- Launch from home screen.
- Grant camera permission.
- Open live preview.
- Test B&W Infrared and Aerochrome first.
- Capture image; verify saved JPEG orientation and gallery visibility.
- Confirm the app clearly presents simulated spectral/IR looks, not true infrared hardware capture.

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

