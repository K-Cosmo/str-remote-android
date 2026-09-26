# STR Remote for Android

A small Android companion app for the **phone remote built into [SoundTouch Reborn (STR)](https://github.com/JRpersonal/streborn)**.

STR Remote discovers STR-enabled Bose SoundTouch speakers on your local network, selects a reachable local STR web endpoint and opens the existing STR phone interface inside a constrained Android WebView. It does **not** reimplement playback controls or replace STR itself.

> **Independent community project.** STR Remote is not affiliated with Bose Corporation or with the SoundTouch Reborn project. Bose and SoundTouch are trademarks of their respective owner(s).

> **AI-assisted open-source project.** The project is intentionally developed with AI assistance for implementation, debugging, review and documentation. Product decisions, acceptance criteria and release decisions remain human-controlled and are checked against real builds, logs and real-device tests. The Android app itself contains no AI/LLM runtime and does not send user data to an AI service.

## Why this app exists

SoundTouch Reborn already serves a useful phone remote directly from each STR-enabled speaker. That page can be opened in a normal browser, but a dedicated Android wrapper makes day-to-day use more convenient:

- discover STR speakers automatically instead of remembering IP addresses;
- cope with DHCP address changes;
- remember the last selected speaker;
- open the STR remote directly from an app icon;
- keep the remote in a dedicated, full-screen Android shell;
- handle STR's model-dependent local web ports automatically.

The goal is deliberately narrow: **Android integration around the upstream STR web remote, not a second playback implementation.**

## Prerequisite: SoundTouch Reborn

STR Remote requires **SoundTouch Reborn to already be installed and running on the speaker**.

1. Install STR using the official upstream project:
   - GitHub: <https://github.com/JRpersonal/streborn>
   - Project website: <https://st-reborn.de>
2. Make sure the Android device and the STR speaker are reachable on the same local network.
3. Start STR Remote and grant the local-network permission when Android requests it.

STR Remote does **not** install STR, modify speaker firmware or bundle the STR agent. Model support and speaker-side functionality are provided by the upstream STR project.

## Features

- automatic DNS-SD/mDNS discovery of STR speakers;
- support for the current `_streborn._tcp.` service and the legacy `_soundtouchstick._tcp.` service;
- automatic endpoint probing on ports **8888** and **17008**;
- speaker picker for multiple discovered devices;
- remembers the last selected speaker locally;
- manual host/IP entry as a fallback;
- embedded STR phone UI using the full available Android content area;
- Android system-bar/inset handling;
- modern Android back navigation with a legacy fallback for older supported Android versions;
- constrained WebView navigation and no JavaScript bridge into native Android code;
- German and English native shell strings;
- no account, analytics, telemetry or STR Remote backend.

## How it works

```text
Android device
    |
    |  mDNS / DNS-SD
    v
STR-enabled SoundTouch speaker
    |
    |-- _streborn._tcp. / legacy _soundtouchstick._tcp.
    |
    |-- probe http://speaker:8888/api/status
    |        fallback: http://speaker:17008/api/status
    |
    `-- open http://speaker:<resolved-port>/
             inside the STR Remote WebView
```

STR currently announces its local service over mDNS. Some SoundTouch chassis expose the STR web/API endpoint through port `17008` rather than a LAN-reachable `8888`, so STR Remote probes both instead of blindly trusting the announced port.

## Current compatibility evidence

The first release candidate is verified on real hardware with:

- **SoundTouch 10**
- STR **v0.9.86**
- STR web remote reachable on **port 8888**
- Android app targeting **API 37**

Other SoundTouch models supported by STR are expected to use the same upstream web interface, but they are **not yet independently verified by this project**. Reports from additional models are welcome.

### Android baseline

| Item | Baseline |
|---|---:|
| Minimum Android API | 26 (Android 8.0) |
| Compile SDK | 37 |
| Target SDK | 37 |
| Android Gradle Plugin | 9.4.1 |
| Gradle | 9.7.1 |
| Java source/target | 17 |

The first development workstation is verified with **JetBrains Runtime 25.0.3**; Gradle 9.7.1 reports the daemon using the Android Studio JBR. Gradle Daemon JVM criteria are pinned to **Java 25**. The first verified workstation resolves that criterion to Android Studio's JetBrains Runtime 25.0.3; CI provisions Temurin 25. Java source/target for the app remains 17.

## Build from source

Prerequisites:

- Android SDK with API 37 available;
- a Gradle-compatible JDK (the first verified workstation uses Android Studio JBR 25.0.3);
- the committed Gradle Wrapper 9.7.1.

Build and lint on Windows:

```powershell
.\gradlew.bat clean :app:assembleDebug :app:lintDebug --stacktrace
```

Build on macOS/Linux:

```bash
./gradlew clean :app:assembleDebug :app:lintDebug --stacktrace
```

Debug APK:

```text
app/build/outputs/apk/debug/app-debug.apk
```

The Gradle 9.7.1 Wrapper has been generated and verified on the development workstation. `gradle/gradle-daemon-jvm.properties` pins the Gradle daemon runtime to Java 25. The wrapper bootstrap still needs a Java executable available through `JAVA_HOME` or `PATH`; the daemon criteria take effect after the wrapper has started. Release signing remains a final hardening item before 0.1.0.

## Security model

STR Remote talks to the speaker over the local network. The current upstream STR web UI is served over plain HTTP and, according to the upstream project, currently has no client authentication. **The local network is therefore the trust boundary.** A device that can reach the STR speaker can potentially reach its local control surface as well.

The Android wrapper deliberately keeps its own attack surface small:

- no `addJavascriptInterface()` bridge;
- WebView file access disabled;
- WebView content access disabled;
- external navigation handed to the system browser;
- top-level local STR navigation restricted to the expected endpoint ports;
- no analytics/tracking SDK;
- no Bluetooth permission merely to silence Chromium/WebView log messages.

See [`doc/SECURITY.md`](doc/SECURITY.md) for the normative project rules.

## Privacy

STR Remote does not operate a server, create accounts, collect analytics or transmit telemetry.

The app stores only the last selected local speaker endpoint metadata in Android app-private preferences. The embedded STR page may perform network requests required by STR features; STR Remote does not collect or proxy those requests.

See [`doc/PRIVACY.md`](doc/PRIVACY.md).

## AI-assisted development

STR Remote is also an experiment in **disciplined AI-assisted software development**.

AI is used to help with:

- implementation;
- static analysis and code review;
- debugging from real build output and Logcat;
- documentation;
- test planning and release preparation.

AI output is **not treated as proof of correctness**. The project follows a simple rule: reality beats explanation. Builds, runtime behavior, screenshots, logs and reproducible tests are the evidence used for acceptance.

Development follows:

```text
Specification -> Plan -> Tasks -> Analyze -> Implement
              -> Converge -> Tests -> Real run -> Evidence -> Acceptance
```

The key project rules live in:

- [`doc/GOVERNANCE.md`](doc/GOVERNANCE.md)
- [`doc/AI_CODING_GUARDRAILS.md`](doc/AI_CODING_GUARDRAILS.md)
- [`doc/CLEAN_CODE.md`](doc/CLEAN_CODE.md)
- [`doc/TESTS.md`](doc/TESTS.md)
- [`doc/RELEASE_GATES.md`](doc/RELEASE_GATES.md)

The app itself contains **no generative-AI feature** and has no runtime dependency on OpenAI, ChatGPT or another AI provider.

The current STR Remote app icon is also AI-generated project artwork, then adapted into Android adaptive and themed/monochrome launcher resources. It is independent branding and does not copy the upstream STR logo.

## Project documentation

All normative project documentation lives in **[`/doc`](doc/README.md)**. `/doc` is the project's single source of truth for product scope, architecture, security, compatibility, quality rules, status and acceptance criteria.

The Spec Kit-style files under `specs/` and `.specify/` are workflow artifacts only; they are not a second source of product truth.

Useful starting points:

- [`doc/SOURCE_OF_TRUTH.md`](doc/SOURCE_OF_TRUTH.md)
- [`doc/STATUS.md`](doc/STATUS.md)
- [`doc/ARCHITECTURE.md`](doc/ARCHITECTURE.md)
- [`doc/API_CONTRACTS.md`](doc/API_CONTRACTS.md)
- [`doc/ANDROID_COMPATIBILITY.md`](doc/ANDROID_COMPATIBILITY.md)
- [`doc/CHANGELOG.md`](doc/CHANGELOG.md)

## Relationship to upstream STR

STR Remote intentionally depends on the local interface exposed by SoundTouch Reborn. Upstream STR owns the speaker-side agent, playback behavior, presets, radio, Spotify, library, multiroom and the web remote itself. STR Remote owns only the Android integration around that local interface.

If upstream changes discovery, ports, authentication or the local web/API contract, STR Remote may need a compatibility update. Those assumptions are tracked explicitly in [`doc/API_CONTRACTS.md`](doc/API_CONTRACTS.md).

Issues concerning the STR agent or its speaker functionality should be reproduced against STR itself before being attributed to this wrapper.

## Contributing

Bug reports are most useful when they include:

- STR version;
- SoundTouch model;
- Android version/device;
- expected behavior;
- observed behavior;
- relevant focused Logcat/build output;
- whether the upstream STR page works directly in the browser.

See [`doc/CONTRIBUTING.md`](doc/CONTRIBUTING.md).

## Status

**0.1.0-rc3** — release-packaging convergence candidate.

The core path is verified on real hardware: build, install, STR discovery, endpoint resolution and embedded web UI. RC2 passed clean debug/lint and release builds with **0 lint errors**. RC3 only cleans the adaptive/themed launcher resources, pins the Gradle daemon JVM criterion to Java 25 and aligns CI with that runtime. Final public release still requires a green rc3 rerun, the remaining interaction/reconnect/permission checks and release signing.

## License

STR Remote is licensed under the **MIT License**. See [LICENSE](LICENSE).

SoundTouch Reborn is a separate project with its own source, releases, documentation and licensing. No STR source code or Bose firmware is bundled with STR Remote.
