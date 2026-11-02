# Native: Expo + uniwind

`apps/native`: Expo with uniwind's Metro/CSS integration and React Native primitives, never browser `ui`. The Expo SDK dictates React and React Native, and the hoisted layout applies ([workspace](workspace.md)). `eas.json` only when EAS is selected.

Acceptance, three separate results: `expo install --check` and `expo-doctor` pass; the JS bundle exports; the app builds and launches on the selected simulator or device and shows a real screen. A missing simulator or build toolchain blocks the launch check. A Metro HTTP response proves nothing native.

EAS builds and submissions cost money and run only after the second confirmation.

Pitfall (Expo, 2026-11-02): Metro readiness and web export do not validate iOS or Android behavior.

Sources: [Expo monorepos](https://docs.expo.dev/guides/monorepos/), [uniwind](https://docs.uniwind.dev/), [EAS](https://docs.expo.dev/build/introduction/).
