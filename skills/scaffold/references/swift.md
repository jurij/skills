# Swift apps (macOS, iOS)

`apps/<name>` (for example `apps/macos`, `apps/ios`): SwiftUI with XcodeGen. `project.yml` is the source of truth; the generated `*.xcodeproj` and the build directory are ignored. Info.plist values come from `GENERATE_INFOPLIST_FILE: YES` plus `INFOPLIST_KEY_*` build settings, so no Info.plist is committed. Tests use Swift Testing. Alternatives (Tuist, a plain SwiftPM executable) are unvalidated; SwiftPM alone cannot bundle a `.app` with Info.plist keys.

The app is a normal workspace package: a private `package.json` without dependencies whose scripts wrap the toolchain, so the orchestrator runs the standard tasks:
- `build`: `xcodegen generate`, then `xcodebuild build` for the app scheme with a pinned destination and `-derivedDataPath build`.
- `test`: the same with `xcodebuild test`.
- `check`: `swift format lint --strict --recursive` over the app sources (swift-format ships with the Swift toolchain; confirm `swift format --help`).
- `check-types`: none; `build` compiles the types. Record this in `docs/scaffold.md`.

Orchestrator config: the package's outputs (`build/**`) and inputs (`project.yml`, `**/*.swift`, assets). The JS stack still applies at the root (package manager, orchestrator, lint for JS/JSON/YAML/Markdown, hooks, gitleaks, CI) even when no JS app exists; `config` and `env` are skipped without a JS/TS app.

Signing: ad-hoc (`CODE_SIGN_IDENTITY: "-"`, `CODE_SIGN_STYLE: Manual`) builds locally and on CI runners without a team. Team signing, hardened runtime, notarization, App Store and TestFlight are out of scope: label artifacts unsigned. GitHub Releases (macOS only) uploads the reviewed zipped `.app` after the second confirmation.

Acceptance: `<pm> run test` runs `xcodebuild test` green; every required Info.plist key is present in the built app (`/usr/libexec/PlistBuddy -c "Print :<Key>" <App>.app/Contents/Info.plist`); macOS: the built app launches and keeps running (`open`, then `pgrep`); iOS: it installs and launches on the selected simulator. Xcode is a manual prerequisite; without it every Swift gate is blocked.

Pitfall (XcodeGen 2.46, 2026-11-03): `INFOPLIST_KEY_LSUIElement: YES` makes a menu bar app without a Dock icon.
Pitfall (xcodebuild, Xcode 27.1, 2026-11-03): `-destination 'platform=macOS'` matches arm64 and x86_64 and warns; pin `platform=macOS,arch=<arch>` locally. CI on `macos-latest` passed with the plain destination.
Pitfall (SourceKit, 2026-11-03): before `xcodegen generate`, editor diagnostics report false errors ("Cannot find … in scope", "No such module"); the build is the source of truth.
Pitfall (CoreWLAN, macOS 14+, 2026-11-03): APIs behind privacy permissions (for example `CWInterface.ssid()`) return nil without the permission; request it at runtime and set the matching `INFOPLIST_KEY_NS*UsageDescription` keys.

Sources: [XcodeGen](https://github.com/yonaskolb/XcodeGen/blob/master/Docs/ProjectSpec.md), [swift-format](https://github.com/swiftlang/swift-format), [Swift Testing](https://developer.apple.com/documentation/testing), [xcodebuild](https://developer.apple.com/documentation/xcode/building-and-running-an-app).
