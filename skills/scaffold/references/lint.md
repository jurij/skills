# Lint and format

Ultracite 8 with oxlint + oxfmt, one root config: shared presets plus framework rules only for selected apps. Use Ultracite's tested tool ranges when pinning. Enable type-aware linting (`oxlint-tsgolint`, `typeAware`) when the resolved TypeScript and framework tools support it. `check` runs in check mode in the pre-commit hook and CI; formatting is an explicit edit beforehand. `check-types` (the compiler) stays a separate gate under any lint setup.

Fallbacks need the user's decision before use: type-aware lint incompatible → oxlint without type-aware rules; Ultracite's oxc provider incompatible → Biome.

Acceptance: a planted type-aware violation (when enabled) and a planted format violation each fail `bun run check`; remove both afterwards.

Pitfall (2026-11-02): oxfmt is pre-1.0; pin exactly, never float across minors.
Pitfall (2026-11-02): oxfmt 0.75 ignored `.oxfmtrc.ts` ("No config found") and formatted with defaults; name the files `oxlint.config.ts` and `oxfmt.config.ts`, which both tools discover.
Pitfall (2026-11-02): generated sources (Start's `routeTree.gen.ts`, the better-auth Drizzle schema) fail Ultracite rules such as `sort-keys`; list them in `ignorePatterns` of both configs.

Sources: [Ultracite 8](https://www.ultracite.ai/changelog/ultracite-8-0-0), [oxc type-aware linting](https://oxc.rs/blog/2026-07-22-type-aware-linting-stable).
