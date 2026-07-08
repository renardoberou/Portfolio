# Four-app Android signing release guides

Generated: 2026-07-08T20:49:45Z

These app-specific guides convert the portfolio release research into concrete release steps for each Android app. Google Play is the primary target; GitHub Releases are the direct APK/AAB portfolio/tester channel.

| App | Guide | Current first release posture |
|---|---|---|
| Bighart Beat | [step-by-step-android-signing-release.md](Bighart-Beat/step-by-step-android-signing-release.md) | Release workflow exists; use `app-v*` tag after secrets |
| Ping Thing Android | [step-by-step-android-signing-release.md](Ping-thing-android/step-by-step-android-signing-release.md) | Best first signed release candidate; use `v*` tag after secrets |
| Bighart Synth Standalone | [step-by-step-android-signing-release.md](Bighart-synth-standalone/step-by-step-android-signing-release.md) | Needs release signing workflow and smoke-test doc update |
| Spectral Camera | [step-by-step-android-signing-release.md](spectral-camera/step-by-step-android-signing-release.md) | Must replace debug release signing before public/store release |

## Shared release order recommendation

1. **Ping Thing Android** — strongest release runbook and tag workflow.
2. **Spectral Camera** — mature app, but must replace debug signing.
3. **Bighart Beat** — web/browser version live plus Android shell; release workflow exists.
4. **Bighart Synth Standalone** — installed on phone, but repo still needs signed release workflow and documented smoke test.

## Shared safety rule

Do not commit keystores, APKs, AABs, password files, Play account metadata, or private screenshots. Use GitHub secrets and local/offline keystore backup.
