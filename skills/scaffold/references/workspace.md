# Workspace and packages

## Versions (preflight)

Resolve the newest mutually compatible stable set once and freeze it: exact versions for direct dependencies and `workspaces.catalog` entries, `packageManager` pinned to the frozen Bun version (`bun --version` after step 4 matches it), `bun.lock` committed. Keep version constraints that a generator writes when they are its supported range. Upgrades are separate later changes.

Resolution order:
1. Native selected: the Expo SDK dictates React and React Native; take them from its compatibility set.
2. Intersect the peer ranges of Start, Vite, the Desktop renderer, Storybook, Docs, and `ui`. An empty intersection is a blocker to show in the summary.
3. Pin generators and CLIs you invoke (for example `bunx <tool>@<version>`).

## Layout (bootstrap)

- Turborepo + Bun workspaces: `workspaces.packages: ["apps/*", "packages/*"]`, shared versions via `catalog:`, internal packages via `workspace:*`. Root is private. No remote cache.
- Linker: Bun's default, except `linker = "hoisted"` in `bunfig.toml` when Native or Desktop is selected. Each package still declares every dependency it imports.
- Turbo tasks `build`, `check`, `check-types`, `test`, `dev` (persistent, uncached). Declare `outputs` and environment inputs; integration tests and runtime checks bypass the cache.
- Root scripts call Turbo through `bun run` (`"test": "turbo run test"`), so `bun run test` never means Bun's own test runner. Root also holds `prepare` from [hooks](hooks.md) and, with `db`, the scripts from [db](db.md).
- `.gitignore` before anything generates state: `node_modules`, build outputs, `.turbo`, `.env*` with `!.env.example`, `.dev.vars*`, certificates and signing keys, and the graphify lines from [hooks](hooks.md).

## Packages

- `packages/config` always: shared TypeScript base config.
- `packages/env` always: t3-env with separate server and public exports plus `.env.example`. Each app passes its runtime's values (`process.env`, Worker bindings, public bundler variables) at its boundary. Server variables fail at server startup; a public import validates only public values and never loads server validation.
- `packages/ui` only for browser renderers (Web, Desktop renderer, Docs, Storybook): Tailwind 4, shadcn, Base UI. Each browser app imports its stylesheet and adds the `ui` sources to Tailwind's source scanning. Expo uses React Native primitives with uniwind and never imports `ui`.

Acceptance: one React instance per app and one React Native version in the workspace; public bundles contain no server modules or secrets. After commit: `bun install --frozen-lockfile` succeeds in a fresh clone.

Pitfall (2026-11-02): hoisting hides missing dependency declarations; a successful import from the root proves nothing.
Pitfall (2026-11-02): Bun loads `.env` only from the current directory. Package scripts that need the root `.env` run `bun --env-file=../../.env <bin>`; a missing file is skipped, so CI can pass variables directly.

Sources: [Bun workspaces](https://bun.com/docs/pm/workspaces), [catalogs](https://bun.com/docs/pm/catalogs), [Turbo tasks](https://turborepo.com/docs/crafting-your-repository/configuring-tasks), [t3-env](https://env.t3.gg/docs/core), [Base UI](https://base-ui.com/react/overview/quick-start), [Tailwind sources](https://tailwindcss.com/docs/detecting-classes-in-source-files), [Expo monorepos](https://docs.expo.dev/guides/monorepos/).
