# Swift apps (macOS, iOS)

`apps/<name>` (for example `apps/macos`, `apps/ios`): SwiftUI with XcodeGen. `project.yml` is the source of truth; the generated `*.xcodeproj` and the build directory are ignored. Info.plist values come from `GENERATE_INFOPLIST_FILE: YES` plus `INFOPLIST_KEY_*` build settings, so no Info.plist is committed. Tests use Swift Testing. Alternatives (Tuist, a plain SwiftPM executable) are unvalidated; SwiftPM alone cannot bundle a `.app` with Info.plist keys.

The app is a normal workspace package: a private `package.json` without dependencies whose scripts wrap the toolchain, so the orchestrator runs the standard tasks:
- `build`: `xcodegen generate`, then `xcodebuild build` for the app scheme with `-derivedDataPath build` and a destination pinned to the target architecture (`platform=macOS,arch=<arch>`, default the host's); local runs and CI use the same script.
- `test`: the same with `xcodebuild test`.
- `check`: `swift format lint --strict --recursive` over the app sources (swift-format ships with the Swift toolchain; confirm `swift format --help`).
- `check-types`: none; `build` compiles the types. Record this in `docs/scaffold.md`.

Orchestrator config: the package's outputs (`build/**`) and inputs (`project.yml`, `**/*.swift`, assets). The JS stack still applies at the root (package manager, orchestrator, lint for JS/JSON/YAML/Markdown, hooks, gitleaks, CI) even when no JS app exists; `config` and `env` are skipped without a JS/TS app.

Signing: ad-hoc (`CODE_SIGN_IDENTITY: "-"`, `CODE_SIGN_STYLE: Manual`) builds locally and on CI runners without a team. Team signing, hardened runtime, notarization, App Store and TestFlight are out of scope: label artifacts unsigned. GitHub Releases (macOS only) uploads the reviewed zipped `.app` after the second confirmation.

Acceptance:
- `<pm> run test` is green and its result bundle reports more than zero passed and zero failed tests (`-quiet` prints no counts); a planted wrong expectation fails it.
- The built app contains every `INFOPLIST_KEY_*` from `project.yml`, the bundle identifier, both version keys, and the minimum system version (`/usr/libexec/PlistBuddy -c "Print :<Key>" <App>.app/Contents/Info.plist`).
- macOS: `open` the built app, after about 5 s `pgrep -x <executable>` finds it, quit it (`osascript -e 'quit app "<App>"'`), then `pgrep` finds nothing. A menu bar item can sit behind the notch; read it through System Events, not a screenshot.
- iOS: it installs and launches on the selected simulator.

Xcode is a manual prerequisite; without it every Swift gate is blocked.

Pitfall (Xcode 27.1, 2026-11-03): `swift format --version` prints `main`; record the Xcode version instead.
Pitfall (XcodeGen 2.46, 2026-11-03): `INFOPLIST_KEY_LSUIElement: YES` makes a menu bar app without a Dock icon.
Pitfall (xcodebuild, Xcode 27.1, 2026-11-03): `-destination 'platform=macOS'` matches arm64 and x86_64 and warns; pin the architecture.
Pitfall (SourceKit, 2026-11-03): before `xcodegen generate`, editor diagnostics report false errors ("Cannot find … in scope", "No such module"); the build is the source of truth.
Pitfall (CoreWLAN, macOS 14+, 2026-11-03): APIs behind privacy permissions (for example `CWInterface.ssid()`) return nil without the permission; request it at runtime and set the matching `INFOPLIST_KEY_NS*UsageDescription` keys.

Sources: [XcodeGen](https://github.com/yonaskolb/XcodeGen/blob/master/Docs/ProjectSpec.md), [swift-format](https://github.com/swiftlang/swift-format), [Swift Testing](https://developer.apple.com/documentation/testing), [xcodebuild](https://developer.apple.com/documentation/xcode/building-and-running-an-app).
