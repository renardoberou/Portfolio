# Android signing and official release research — portfolio apps

Generated: 2026-07-08T11:08:06-03:00

## Scope

Research basis:

- Bernado's Ping Thing release runbook: `https://github.com/renardoberou/Ping-thing-android/blob/main/docs/RELEASE.md`
- Android Developers: Sign your app, Publish your app, Version your app, Android App Bundles, command-line builds, apksigner
- Google Play Console Help: Play App Signing, create/set up app, target API requirements, testing requirements for new personal accounts, store listing assets, app review declarations, publishing status, payments policy, content rating

This note is written for a phone/Termux-first workflow and for preparing four Android apps as portfolio/public releases.

---

## Executive position

**Decision recorded 2026-07-08:** Google Play Store is the primary target if possible. Direct APK/GitHub/Gumroad distribution is secondary: useful for portfolio proof, tester delivery, or fallback, but it must not damage the Play Store path.

**Account status recorded 2026-07-08:** Google Play Developer account access has been confirmed. Account identifiers and private Play Console metadata must not be committed to public repos, screenshots, store assets, or issue trackers.

Bernado is at a valid transition point: four working apps are enough to start a portfolio, but official release work should now be Play-first:

1. **Google Play official distribution** — AAB upload, Play App Signing, policy declarations, store assets, closed testing if the Play account is a new personal account, and review.
2. **Portfolio/direct distribution** — GitHub Releases / Gumroad / website with signed APKs, screenshots, changelogs, privacy notes, and clear installation instructions, used only in a way that preserves the Play release path.

The most important decision before publishing anything is **signature strategy**. Android updates are controlled by package name + signing certificate. If the first public build is signed one way and a later official-store build is signed another way, users may have to uninstall/reinstall and lose local data. Because Play is primary, every signing and packaging decision must be checked against future Play compatibility first.

---

## What Ping Thing's RELEASE.md already gets right

The current `RELEASE.md` is strong and security-aware.

### Correct patterns

- Treats the keystore as the **crown jewel**.
- Uses Termux-first `keytool` generation.
- Uses a 4096-bit RSA key with long validity.
- Rejects public CI-generated keystores as unsafe.
- Uses GitHub Actions secrets rather than committing keys.
- Produces both APK and AAB artifacts.
- Uses GitHub Releases and tags as the release trigger.
- Calls out debug-to-release signature incompatibility and the need to export local data before switching.
- Notes that `versionCode` can be a monotonically increasing CI run number.
- Mentions Play Store essentials: privacy policy, content rating, screenshots, feature graphic, icon, target API.

### Main correction / nuance

For Google Play, there are two different key concepts:

| Key | Who holds it | Purpose | If lost |
|---|---|---|---|
| **Upload key** | Developer | Signs the `.aab` uploaded to Play | Can usually be reset in Play Console |
| **App signing key** | Google Play, or developer-provided then held by Google | Signs APKs delivered to users | Critical identity key for Play-delivered installs |

The runbook currently uses one `pingthing-release.jks` for release signing. That is fine for GitHub/Gumroad APKs and can also serve as the Play upload key. But before first Play release, decide whether we want:

1. **Google-generated Play app signing key** — easiest and recommended by Google for most apps. Downside: sideloaded APKs signed locally with our upload key will not have the same signature as APKs installed from Play. For outside-Play distribution, use Play's downloaded signed universal APK if available.
2. **Developer-provided app signing key** — better if we want the same app identity across Play, Gumroad, GitHub, F-Droid-like channels, or direct APK distribution. More responsibility: we must protect that key permanently.

For Bernado's apps, if Gumroad/direct APK distribution is part of the portfolio plan, the safest long-term strategy is usually:

- generate and guard one release/app-signing key per app;
- sign direct APKs with it;
- when enrolling in Play App Signing, provide that same app signing key to Google if cross-store update compatibility matters;
- use a separate upload key for future Play uploads if possible.

If speed matters more than cross-store signature continuity, use Google-generated Play signing and accept that Gumroad APK and Play install may be separate upgrade paths.

---

## Core signing model

Android requires every APK to be digitally signed before install/update. A debug build is signed with an insecure debug key and cannot be published to Play. Release builds must be signed with a private key.

### Rules that matter operationally

- Same package name + same signing certificate = update can install over previous version.
- Same package name + different signing certificate = Android treats it as incompatible; usually requires uninstall/reinstall.
- Debug builds and release builds have different signatures.
- A keystore should never be committed to Git.
- Passwords and base64 keystore blobs should not be pasted into public logs, public workflows, release artifacts, or issue trackers.
- `zipalign` must happen before `apksigner`; modifying an APK after signing invalidates the signature.
- Always verify signed APKs with `apksigner verify`.

### Practical Termux commands

Generate a key locally:

```bash
pkg update
pkg install openjdk-17
keytool -genkeypair -v \
  -keystore APP-release.jks \
  -alias APP \
  -keyalg RSA -keysize 4096 -validity 10000 \
  -dname "CN=Resonant Systems, O=Resonant Systems"
```

Verify a signed APK:

```bash
apksigner verify --verbose path/to/app-release.apk
```

Inspect signing certificate fingerprints:

```bash
keytool -list -v -keystore APP-release.jks -alias APP
```

Base64 for CI secret entry:

```bash
base64 -w0 APP-release.jks > APP-keystore.b64
# paste into GitHub secret, then delete the clear base64 file
rm APP-keystore.b64
```

---

## Google Play requirements that affect the four-app plan

### Artifact format

- New Google Play apps must publish with **Android App Bundle (`.aab`)**, not only APK.
- Google Play uses the AAB to generate optimized APKs per device.
- APK remains useful for GitHub/Gumroad/direct distribution and local testing.

### Target API

As of the official Play target API page checked on 2026-07-08:

- Starting August 31, 2025, new apps and app updates must target **Android 15 / API 35** or higher for standard phone/tablet apps.
- Existing apps must target at least Android 14 / API 34 to remain broadly available to new users on newer Android versions.

For Bernado's new portfolio apps: treat **targetSdk 35** as the minimum release target unless a later requirement supersedes it.

### Versioning

- `versionCode` must increase on every Play upload.
- Play Console's maximum `versionCode` is `2100000000`.
- `versionName` is user-visible and can follow semantic versioning.
- CI `github.run_number` can work as versionCode if it is monotonic and remains under the max.
- For multiple apps, keep a per-app version policy documented in each repo.

Recommended:

```text
versionName = product version, e.g. 1.0.0
versionCode = monotonically increasing integer, preferably CI run number or a stored release counter
```

### New personal developer account testing gate

If Bernado's Google Play developer account is a **personal account created after 2023-11-13**, production access requires:

- closed test;
- at least **12 testers**;
- testers opted in continuously for **14 days**;
- production-access application answering questions about testing, app value, tester engagement, and readiness.

This is a real schedule constraint. If it applies, start closed tests early while portfolio/Gumroad/GitHub materials are prepared.

### Store listing essentials

For each app:

- App name: 30 chars max.
- Short description: 80 chars max.
- Full description: 4000 chars max.
- Contact email required.
- Privacy policy URL required in many cases; Google states even apps with no personal/sensitive data may still need to submit a privacy policy.
- 512 × 512 app icon, PNG with alpha, max 1024 KB.
- Feature graphic: 1024 × 500 JPG or 24-bit PNG, no alpha.
- Screenshots: at least 2 across supported device types; recommended at least 4 phone screenshots at 1080p-class resolution for promotional eligibility.
- Content rating questionnaire required.
- Target audience declaration required.
- Ads declaration required.
- Data safety / privacy/security practices required.
- Sensitive permission declarations may be required if the manifest asks for high-risk permissions.

### Monetization constraint

If an app is distributed through Google Play and charges for downloads, in-app digital features, digital content, subscriptions, ad-free unlocks, etc., Google Play billing rules usually apply. Gumroad can sell APKs outside Play, but Play-distributed apps must not route users to outside payment flows for in-app digital purchases unless a permitted alternative billing program applies.

For a portfolio launch, simplest options:

- free on Play, paid/supporter download on Gumroad outside Play; or
- paid app on Play using Play's payment system; or
- no in-app monetization at first.

Avoid putting Gumroad payment CTAs inside a Play-distributed app.

---

## Release-path decision tree

### Path 1 — Fast portfolio release outside Play

Use for immediate public proof of work.

1. Clean repo and remove secrets/generated junk.
2. Build release APK.
3. Sign with app release key.
4. Verify signature.
5. Install on owner device and run sanity test.
6. Create GitHub Release with APK, changelog, screenshots, and SHA-256 checksum.
7. Create Gumroad product/page with the APK or link to release.
8. Publish portfolio page with app summary, screenshots, GitHub link, and install warning/instructions.

Pros:

- fastest;
- phone-first friendly;
- no Play review delay;
- good for portfolio.

Cons:

- users must allow unknown-app install;
- lower trust than Play;
- no Play discovery;
- need to manage payments/downloads manually.

### Path 2 — Official Google Play release

Use for public store legitimacy.

1. Confirm Play developer account status and whether closed-test gate applies.
2. Create app in Play Console.
3. Finalize package name forever; package names cannot be reused or deleted.
4. Generate signing/upload key strategy.
5. Build signed `.aab`.
6. Upload to internal testing first.
7. Complete store listing, privacy policy, content rating, data safety, target audience, ads declaration, app access instructions.
8. Run closed testing if required.
9. Apply for production access if required.
10. Use managed publishing if timing matters.
11. Roll out production release.

Pros:

- official Android marketplace;
- easier installs/updates;
- user trust;
- Play App Signing safety net for upload-key loss.

Cons:

- review/policy burden;
- possible 14-day testing gate for new personal accounts;
- store assets and declarations required;
- payments restrictions.

### Path 3 — Hybrid recommended for Bernado

For the four-app portfolio:

1. Publish cleaned GitHub repos and signed GitHub/Gumroad APK releases first.
2. In parallel, prepare Play listings and closed tests.
3. Promote the portfolio immediately, then add Play badges/links as each app clears review.
4. Preserve signature strategy so users do not get trapped on incompatible builds.

---

## Four-app release readiness checklist

For each app, create this table and fill it before release:

| Field | Value |
|---|---|
| App name |  |
| Repo URL |  |
| Package name |  |
| Current versionName |  |
| Current versionCode |  |
| targetSdk |  |
| minSdk |  |
| Release key alias |  |
| Signing strategy | Play-generated / developer-provided / direct-only |
| Release artifacts | APK / AAB / both |
| Privacy policy URL |  |
| Data collection | none / local only / network / account / analytics |
| Permissions |  |
| Store screenshots ready | yes/no |
| Feature graphic ready | yes/no |
| 512 icon ready | yes/no |
| Sanity test passed | yes/no |
| GitHub release ready | yes/no |
| Gumroad page ready | yes/no |
| Play internal test uploaded | yes/no |
| Closed-test requirement applies | yes/no/unknown |

---

## Pre-release technical gate

Run per repo before tagging:

```bash
# Source hygiene
git status --short
git diff --stat
git grep -n -i -E 'api[_-]?key|secret|token|password|Authorization: Bearer|ghp_|sk-' HEAD || true

# Android build sanity
./gradlew clean
./gradlew test
./gradlew lintRelease
./gradlew assembleRelease bundleRelease

# Artifact verification
apksigner verify --verbose app/build/outputs/apk/release/*.apk
sha256sum app/build/outputs/apk/release/*.apk app/build/outputs/bundle/release/*.aab
```

If Gradle is not usable directly on Termux for a given app, use GitHub Actions as the builder but keep keystore generation local and secrets private.

---

## Recommended GitHub Actions secret set

Per app repository:

| Secret | Meaning |
|---|---|
| `KEYSTORE_B64` | Base64 of release/upload keystore |
| `KEYSTORE_PASS` | Keystore password |
| `KEY_ALIAS` | Alias inside keystore |
| `KEY_PASS` | Key password |

Safety notes:

- Do not use workflow inputs for passwords.
- Do not upload keystore as artifact.
- Do not print secret-derived paths/content.
- Delete temporary `.b64` files after secret entry.
- Keep `.jks` and any `keystore.properties` out of Git.

---

## Privacy policy baseline for local-only apps

For simple local-only utilities, a concise policy can say:

- what the app does;
- what data is stored locally;
- that no data is transmitted if true;
- whether analytics/crash reporting/ads are absent;
- permissions and why they are used;
- contact email;
- effective date.

Do not claim "transmits nothing" unless the app truly performs no network calls, analytics, crash reporting, ads, remote config, or embedded web content.

---

## Portfolio launch packaging

Each app should have:

1. **README**: what it is, screenshots, install/build instructions, privacy note.
2. **Release notes**: version, date, changes, known issues.
3. **Signed artifact**: APK for direct download; AAB for Play.
4. **Checksum**: SHA-256 for APK/AAB.
5. **Screenshots**: 4 phone screenshots if possible.
6. **Short demo**: optional 15–30 second video/GIF.
7. **Privacy policy**: GitHub Pages or repo-hosted page.
8. **Support/contact**: email or form.
9. **License/provenance**: source license and asset credits.
10. **No secrets**: verified by git grep and release artifact inspection.

---

## Immediate next operating plan

1. Inventory the four apps and classify them by release difficulty.
2. For each app, decide distribution: GitHub/Gumroad now, Play later, or Play now.
3. Generate/confirm one protected release key per app before first public release.
4. Add or verify release workflow producing signed APK + AAB.
5. Produce privacy policy and screenshots for each app.
6. Release the least risky app first as the pattern.
7. Reuse the proven workflow for the other three.

---

## Key source links

- Ping Thing release runbook: https://github.com/renardoberou/Ping-thing-android/blob/main/docs/RELEASE.md
- Sign your app: https://developer.android.com/studio/publish/app-signing
- Publish your app: https://developer.android.com/studio/publish
- Build from command line: https://developer.android.com/build/building-cmdline
- apksigner: https://developer.android.com/studio/command-line/apksigner
- Version your app: https://developer.android.com/studio/publish/versioning
- Android App Bundles: https://developer.android.com/guide/app-bundle
- Play App Signing: https://support.google.com/googleplay/android-developer/answer/9842756
- Create and set up your app: https://support.google.com/googleplay/android-developer/answer/9859152
- Target API requirements: https://support.google.com/googleplay/android-developer/answer/11926878
- New personal account testing requirements: https://support.google.com/googleplay/android-developer/answer/14151465
- Preview/store assets: https://support.google.com/googleplay/android-developer/answer/1078870
- Prepare app for review: https://support.google.com/googleplay/android-developer/answer/9859455
- Publish status/review: https://support.google.com/googleplay/android-developer/answer/6334282
- Payments policy: https://support.google.com/googleplay/android-developer/answer/9858738
