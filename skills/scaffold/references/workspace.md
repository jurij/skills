# Workspace and packages

## Versions (preflight)

Resolve the newest mutually compatible stable set once and freeze it. Stable means the highest non-prerelease, non-deprecated version that the package manager's configured `minimumReleaseAge` allows; the `latest` dist-tag can point at a prerelease. Freeze exact versions for direct dependencies and catalog entries, `packageManager` pinned to the frozen package-manager version (`<pm> --version` after step 4 matches it), the lockfile committed. pnpm and npm run on Node: pin the frozen Node major in `engines.node` and a committed `.node-version`, which CI reads. Keep version constraints that a generator writes when they are its supported range. Upgrades are separate later changes.

Resolution order:
1. Native selected: the Expo SDK dictates React and React Native; take them from its compatibility set.
2. Intersect the peer ranges of the web framework, Vite, the Desktop renderer, Storybook, Docs, and `ui`. An empty intersection is a blocker to show in the summary.
3. Pin generators and CLIs you invoke (`bunx`, `pnpm dlx`, or `npx` with `<tool>@<version>`), and re-pin exactly any range a generator saves for packages it installs.

## Layout (bootstrap)

- Monorepo: `apps/*` and `packages/*`, private root, internal packages referenced by workspace protocol (`workspace:*` with bun and pnpm; npm links workspaces by name with `*`). Declare workspaces where the package manager reads them: `workspaces` in `package.json` (bun, npm) or `pnpm-workspace.yaml` (pnpm).
- Shared versions: a catalog where the package manager has one (bun `workspaces.catalog`, pnpm `catalog` in `pnpm-workspace.yaml`); npm has none, so every package pins the same exact version.
- Orchestrator: Turborepo (`turbo.json`) or Nx (`nx.json` target defaults) runs tasks `build`, `check`, `check-types`, `test`, `dev` (persistent, uncached), with declared `outputs` and environment inputs; integration tests and runtime checks bypass the cache; no remote cache. Workspaces only: root scripts run the task in every package that has it (`bun run --filter '*' <task>`, `pnpm -r <task>`, `npm run <task> --workspaces --if-present`).
- Single package: the one app sits at the root; packages become folders (`src/env`, `src/db`, `src/auth`, `src/ui`) with the same boundaries, and the root scripts run the tools directly. `ui` before the app uses shadcn's manual installation, because `shadcn init` needs a detected framework.
- pnpm blocks dependency build scripts: decide each one with `pnpm approve-builds`, which records the decision in `pnpm-workspace.yaml` (a single package has that file too).
- Each HTTP app has its own default port in `dev` and `start`; `PORT` overrides it.
- Linker: Native or Desktop needs a hoisted layout (bun `linker = "hoisted"` in `bunfig.toml`, pnpm `nodeLinker: hoisted` in `pnpm-workspace.yaml`; npm hoists by default). Otherwise keep the package manager's default. Each package still declares every dependency it imports.
- Root scripts `check`, `check-types`, `build`, `test`, `dev` cover the whole workspace. Root also holds `prepare` from [hooks](hooks.md) and, with `db`, the scripts from [db](db.md).
- Package scripts that need the root `.env` load it explicitly, and a missing file is skipped, so CI can pass variables directly.
- `.gitignore` before anything generates state: `node_modules`, build outputs, orchestrator caches (`.turbo`, `.nx`), `.env*` with `!.env.example`, `.dev.vars*`, certificates and signing keys, and the graphify lines from [hooks](hooks.md) when the graph is selected.

## Packages

- `config` always: shared TypeScript base config (monorepo only; a single package keeps one `tsconfig.json`).
- `env` always: t3-env with separate server and public exports plus `.env.example`. Each app passes its runtime's values (`process.env`, Worker bindings, public bundler variables) at its boundary. Server variables fail at server startup; a public import validates only public values and never loads server validation.
- `ui` only for browser renderers (Web, Desktop renderer, Docs, Storybook): Tailwind 4, shadcn, Base UI. Each browser app imports its stylesheet and adds the `ui` sources to Tailwind's source scanning. Expo uses React Native primitives with uniwind and never imports `ui`.

## Extend mode

Read the layout from the repository: workspace globs, orchestrator config, catalog, internal package scope, script names. A new app or package follows them: same folder pattern, same scope, its orchestrator's task names, and the same version-pinning style (catalog entries where the repository uses a catalog; per-package exact pins stay per-package).

Acceptance: one React instance per app and one React Native version in the workspace; public bundles contain no server modules or secrets. After commit: the frozen-lockfile install (`bun install --frozen-lockfile`, `pnpm install --frozen-lockfile`, `npm ci`) succeeds in a fresh clone.

Pitfall (2026-11-02): hoisting hides missing dependency declarations; a successful import from the root proves nothing.
Pitfall (bun, 2026-11-02): `bun test` is Bun's own test runner; gates call `bun run test`, which runs the script.
Pitfall (bun, 2026-11-02): Bun loads `.env` only from the current directory. Package scripts that need the root `.env` run `bun --env-file=../../.env <bin>`.

Sources: [Bun workspaces](https://bun.com/docs/pm/workspaces), [Bun catalogs](https://bun.com/docs/pm/catalogs), [pnpm workspaces](https://pnpm.io/workspaces), [pnpm catalogs](https://pnpm.io/catalogs), [npm workspaces](https://docs.npmjs.com/cli/using-npm/workspaces), [Turbo tasks](https://turborepo.com/docs/crafting-your-repository/configuring-tasks), [Nx tasks](https://nx.dev/docs/features/run-tasks), [t3-env](https://env.t3.gg/docs/core), [Base UI](https://base-ui.com/react/overview/quick-start), [Tailwind sources](https://tailwindcss.com/docs/detecting-classes-in-source-files), [Expo monorepos](https://docs.expo.dev/guides/monorepos/).
