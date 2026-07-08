# Bighart Synth Standalone — step-by-step Android signing release

Generated: 2026-07-08T20:49:45Z

## Current evidence snapshot

| Field | Value |
|---|---|
| App / store name | Bighart |
| Launcher/home-screen label | Bighart |
| Repo | renardoberou/Bighart-synth-standalone |
| Package / application ID | `com.resonantsystems.bighart` |
| Current repo status | Public repo active; CI debug build is green; app is visible installed on phone home screen, so repo docs that say “not yet verified on-device” need a follow-up smoke-test correction after launch/audio is confirmed. |
| Current release workflow state | Only debug CI exists now; `.github/workflows/android.yml` explicitly marks signed release as Phase B not active. |
| Release priority | Play Store primary; GitHub Release APK/AAB as portfolio/tester channel |

| SDK / version field | Observed value |
|---|---|
| `compileSdk` | 35 |
| `targetSdk` | 35 |
| `minSdk` | 26 |
| `versionCode` | 1 |
| `versionName` | `1.0` |

## Non-negotiable release rules

1. **Do not change the package name** after the first public release.
2. **Do not commit** `.jks`, `.keystore`, `.aab`, `.apk`, `keystore.properties`, `.env`, or account metadata.
3. A debug-installed app and a release-signed app usually cannot update each other in place. If Android reports a signature conflict, uninstall the debug build first after exporting anything worth keeping.
4. For Play, upload `.aab`; for GitHub/direct installs, provide `.apk` plus SHA-256 checksum.
5. Keep the keystore outside every repo and back it up offline.

## Step 0 — Confirm the release branch and clean source

```bash
gh repo view renardoberou/Bighart-synth-standalone --json nameWithOwner,defaultBranchRef,visibility,pushedAt
mkdir -p "$HOME/release-work"
# Clone fresh if the local checkout is stale or dirty:
gh repo clone renardoberou/Bighart-synth-standalone "$HOME/release-work/bighart-synth-standalone" -- --depth 1
cd "$HOME/release-work/bighart-synth-standalone"
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
SIGNING_DIR="$HOME/.local/share/android-signing/bighart-synth-standalone"
mkdir -p "$SIGNING_DIR"
chmod 700 "$SIGNING_DIR"
cd "$SIGNING_DIR"

keytool -genkeypair -v   -keystore bighart-synth-standalone-release.jks   -alias bighart-release   -keyalg RSA -keysize 4096 -validity 10000   -dname "CN=Resonant Systems, O=Resonant Systems"

keytool -list -v   -keystore bighart-synth-standalone-release.jks   -alias bighart-release   > bighart-synth-standalone-certificate-fingerprint.txt
```

After generation:

- store the keystore password in a password manager;
- copy the `.jks` to an offline backup;
- keep `bighart-synth-standalone-certificate-fingerprint.txt` private or semi-private as release evidence;
- never paste the password, base64 keystore, or fingerprint into a public repo issue/log.

## Step 3 — Add GitHub repository secrets

From the private signing directory:

```bash
cd "$HOME/.local/share/android-signing/bighart-synth-standalone"
base64 -w0 bighart-synth-standalone-release.jks > bighart-synth-standalone-keystore.b64

gh secret set KEYSTORE_B64 --repo renardoberou/Bighart-synth-standalone < bighart-synth-standalone-keystore.b64
gh secret set KEYSTORE_PASS --repo renardoberou/Bighart-synth-standalone
gh secret set KEY_ALIAS --repo renardoberou/Bighart-synth-standalone --body "bighart-release"
gh secret set KEY_PASS --repo renardoberou/Bighart-synth-standalone

rm bighart-synth-standalone-keystore.b64
```

Verify only the secret names, not values:

```bash
gh secret list --repo renardoberou/Bighart-synth-standalone
```

Required names:

```text
KEYSTORE_B64
KEYSTORE_PASS
KEY_ALIAS
KEY_PASS
```

## Step 4 — Make sure the repo signs release builds

Release workflow must be added before a signed GitHub Release. Current workflow builds `:app:assembleDebug` only and uploads `bighart-debug-apk`.

Working directory for the release build:

```text
.
```

Expected release artifacts:

```text
app/build/outputs/apk/release/app-release.apk
app/build/outputs/bundle/release/app-release.aab
```

Before tagging, add env-driven release signing to `app/build.gradle.kts` like Ping Thing/Bighart Beat, then add a tag-triggered workflow that decodes `KEYSTORE_B64`, runs `./gradlew :app:assembleRelease :app:bundleRelease`, and attaches signed APK/AAB to a GitHub Release.

## Step 5 — Build the release artifacts

If using GitHub Actions tag release:

```bash
cd "$HOME/release-work/bighart-synth-standalone"
git fetch origin
git switch main
git pull --ff-only

# Replace the version with the chosen public version.
git tag v1.0.0
git push origin v1.0.0

gh run list --repo renardoberou/Bighart-synth-standalone --limit 5
gh release view v1.0.0 --repo renardoberou/Bighart-synth-standalone --web
```

If building locally or reproducing the CI build:

```bash
cd "$HOME/release-work/bighart-synth-standalone/."
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

- Signing is not configured yet.
- README/status should be updated after smoke test because the home-screen screenshot proves installation but not full audio verification.
- First Play/direct release should wait until the smoke test is documented.
- Internet permission exists; verify whether the app truly needs network or only uses WebView secure-origin behavior.

## Step 7 — Create the GitHub Release notes

Use this template:

```markdown
# Bighart v1.0.0

## What changed
- First signed portfolio/tester release.

## Install
- Android APK: attached below.
- SHA-256: `PASTE_CHECKSUM_HERE`.

## Verification
- Package: `com.resonantsystems.bighart`
- APK signature verified with `apksigner verify --verbose`.
- Smoke-tested on Moto Edge 60 Fusion / Android 16.

## Known notes
- Launch from the Android home screen.
- Confirm WebView loads the bundled synth.
- Confirm audio starts only after user gesture if Web Audio requires it.
- Confirm presets/localStorage persist if that is a feature.
- Confirm no crash after rotate/background/return.
```

## Step 8 — Prepare Google Play internal test

Before upload:

| Play field | Value / action |
|---|---|
| App name | Bighart |
| Package | `com.resonantsystems.bighart` |
| Artifact | Add signed APK + AAB release workflow first |
| Privacy policy | Needed before Play listing |
| Data safety | Likely local/bundled WebView; verify network behavior before claiming no transmission |
| Permissions | Internet |
| Screenshots | Capture installed Android app, especially synth/audio UI |
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
- Confirm WebView loads the bundled synth.
- Confirm audio starts only after user gesture if Web Audio requires it.
- Confirm presets/localStorage persist if that is a feature.
- Confirm no crash after rotate/background/return.

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

