# Lint and format

One root config: shared presets plus framework rules only for selected apps. It covers JS, TS, JSON, YAML, and Markdown; an app on another language brings its own linter as its package `check` ([swift](swift.md): swift-format), and the root `check` runs the root linter plus every package `check` through the orchestrator. The root linter ignores those apps' generated and build output. `check` runs lint and format in check mode, in the pre-commit hook and CI; formatting is an explicit edit beforehand. Enable type-aware linting when the project has TS sources and the resolved TypeScript and lint tools support it. `check-types` (the compiler) stays a separate gate under any lint setup.

## Ultracite (oxlint + oxfmt, or Biome)

Ultracite 8 with the chosen provider; use Ultracite's tested tool ranges when pinning. With oxlint, type-aware linting uses `oxlint-tsgolint` and `typeAware`. When a type-aware rule set or the oxc provider is incompatible with the selection, the user decides the fallback: oxlint without type-aware rules, or Biome.

## ESLint + Prettier

ESLint flat config with typescript-eslint (type-checked configs when supported), the framework's plugin for each selected app, and `eslint-config-prettier` last. `check` runs `eslint .` and `prettier --check .`.

Acceptance: a planted lint violation (type-aware when enabled) and a planted format violation each fail `<pm> run check`; remove both afterwards. Generated sources are excluded from lint and format. Types generated from sources (ORM clients, Next.js `next typegen`) are generated at the start of `check` and `check-types`, because type-aware rules read them. Generator output the project owns afterwards (shadcn components) is fixed to the rules, not ignored.

Pitfall (oxfmt, 2026-11-02): oxfmt is pre-1.0; pin exactly, never float across minors.
Pitfall (oxfmt 0.75, 2026-11-02): oxfmt ignored `.oxfmtrc.ts` ("No config found") and formatted with defaults. Keep the config file names Ultracite writes (8.1.1: `oxlint.config.mts`, `oxfmt.config.mts`) and confirm both load (`oxlint --print-config`, a planted format violation).
Pitfall (oxlint, 2026-11-02): `ignorePatterns` from extended configs do not apply; the root config re-declares the preset's patterns (`[...core.ignorePatterns, …]`) next to its own.
Pitfall (typescript-eslint 8.72, 2026-11-02): it supports `typescript <6.1.0`; with type-aware ESLint, pin TypeScript 6.0.x.
Pitfall (eslint-config-next 16.4, 2026-11-02): on ESLint 10 its wrapped plugins report unmet peers (their ranges stop at ESLint 9, which is end-of-life); accept them.
Pitfall (Ultracite, 2026-11-02): generated sources (Start's `routeTree.gen.ts`, the better-auth Drizzle schema) fail rules such as `sort-keys`; list them in `ignorePatterns` of both configs.

Sources: [Ultracite](https://www.ultracite.ai/), [oxc type-aware linting](https://oxc.rs/blog/2026-07-22-type-aware-linting-stable), [typescript-eslint](https://typescript-eslint.io/getting-started/), [Prettier with linters](https://prettier.io/docs/integrating-with-linters).
