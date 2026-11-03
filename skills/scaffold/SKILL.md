---
name: scaffold
description: Set up a new project or add building blocks to an existing repository, with a recommended stack that every axis can override, incremental decisions, and verified commits.
disable-model-invocation: true
---

# Scaffold

Build only what the user selects, one green commit per coherent unit: a new project from an empty folder, or new building blocks in an existing repository. The skill is opinionated: it recommends one stack and lets the user customize each axis. Set each tool up from its own current docs and `--help`; this skill supplies the decisions, gates, and dated pitfalls, not Better-T-Stack or other skills.

`<pm>` stands for the chosen package manager (`bun`, `pnpm`, or `npm`), as in `<pm> run check`.

Generated projects contain no copied helper scripts: tooling lives in `package.json` scripts, the Git hook config, and CI YAML as ordinary one-liners.

Everything committed must be publishable: use logical names, and keep private identifiers, endpoints, credentials, and absolute local paths in ignored `.env` files or provider secret stores. Treat the installed skill files as read-only.

## 1. Inspect the target

Inspect the directory, any enclosing Git root, branch, remotes, and staged and unstaged changes before writing. Then fix the mode:
- **new**: the target is empty and outside any Git repository.
- **resume**: an earlier scaffold whose `docs/scaffold.md` matches this invocation's decisions and has units without a commit; continue at the first of them.
- **extend**: the target is an existing repository with a `package.json`. Detect its stack from `package.json`, the lockfile, and config files: package manager, monorepo tool, apps and their frameworks, ORM, auth, lint, hook tool, CI. Work on its current branch. A dirty tree needs the user's decision before any write: they commit or stash, or explicitly accept working on top.
- Any other non-empty target: the user explicitly adopts it as extend, or names another directory.

Done when the mode is fixed; in extend mode, also when every stack axis is listed as detected (with the file it rests on) or absent.

## 2. Decide

Keep decisions given in the invocation. Ask only open, relevant questions, one round at a time, because later rounds depend on earlier answers; put independent questions into the same round, so the run needs as few rounds as possible. In Claude Code use AskUserQuestion (1–4 questions, 2–4 options each; paginate longer lists; optional multi-select groups end with *None*). In Codex or without a question tool, ask numbered plain-text questions. Silence or cancellation is never approval.

### Stack

The recommended stack, shown as one line:

> bun · Turborepo · TanStack Start · Postgres + Drizzle · better-auth · Ultracite (oxlint + oxfmt) · lefthook · GitHub Actions

New mode: the first round includes one stack question with *Use recommended* (first) and *Customize*. Only *Customize* opens axis rounds, each with the recommendation as the first option. Ask an axis only when the selection makes it relevant:

| Axis | Options, recommendation first | Relevant when |
| --- | --- | --- |
| package manager | bun, pnpm, npm | always |
| layout | Turborepo, Nx, workspaces only, single package | always; single package only with exactly one app and no companions |
| web | TanStack Start, Next.js, Vite + React SPA | Web selected |
| db | Postgres + Drizzle, Postgres + Prisma, SQLite + Drizzle | `db` selected |
| auth | better-auth, external OIDC provider | `auth` selected |
| lint | Ultracite (oxlint + oxfmt), Ultracite (Biome), ESLint + Prettier | always |
| git hooks | lefthook, husky | always |

The backend is the Server app (Hono) when selected in the Product round, otherwise the Web app's server routes. CI is GitHub Actions. Native and Desktop pick their framework in the Platforms group; Docs and Storybook have one framework each. These are the only axes.

Extend mode asks no stack question: the detected stack is the stack. New units match it, and existing tooling is never migrated.

### Rounds

Each row is a question group, not a round: a round holds every group whose dependencies are answered. New mode typically needs (1) Product with the stack question, Companions, Ownership; (2) Platforms for selected Native or Desktop apps; with *Customize*, the package manager, layout, web, lint, and git hooks axes; then Packages, Targets, Integrations; (3) the db and auth axes for the selected packages.

| Group | Decisions |
| --- | --- |
| Product | New mode: at least one of Web, Server (Hono), Native (mobile), Desktop; the stack question joins this round. Extend mode: one round offering only the building blocks the repository lacks that fit its stack (apps, companions, packages, CI), each option naming what it adds to existing config (for example a hoisted linker for Native or Desktop). |
| Platforms | Per selected Native or Desktop app, recommendation first: Native: Expo + uniwind, or Swift (iOS); Desktop: Electron, or Swift (macOS). Asked with *Use recommended* too, because it is a product decision. An answer that names a language ("a Swift app") maps here. |
| Companions | New mode: Docs (Fumadocs) and Storybook. |
| Packages | New mode: `db` with a backend (Server, or Web server routes; a Vite SPA has none). `auth` with a backend as its owner; better-auth requires `db`. `qa` (Playwright; optional Midscene AI tests, recommended on) with Web only. |
| Targets | Per app, none first: Web/Docs/Storybook/Server: none, Coolify, or Cloudflare; Native (Expo): none or EAS; Desktop (Electron) and Swift (macOS): none or GitHub Releases; Swift (iOS): none. Native/Desktop also pick target OS and architecture. |
| Ownership | New mode: GitHub personal account, an organization (list with paginated read-only `gh api`), or local-only. Defaults: private repository, project name from the folder (validate npm and GitHub naming), email/password with better-auth. Extend mode keeps the existing remote. |
| Integrations | Code graph with graphify (on by default, deselectable), project-local skills, MCP services; target hosts (Claude Code, Codex; default both) only when skills or MCP services are selected. Coolify endpoint only if Coolify is selected; PR previews on by default. Extend mode detects existing integrations instead of asking. |

AGENTS.md and gitleaks secret scans in the pre-commit hook and in CI are never optional: new mode builds them in bootstrap and the CI unit (local-only runs validate CI without running it); extend mode adds whichever part is missing. The auth owner is Server when selected, otherwise Web.

For credentials, use existing provider sessions (`gh auth status`, `wrangler whoami`, …) or the host's secret input; otherwise the user configures them directly in the provider or a local ignored file. Question text never asks for a credential.

Done when every round is answered or defaulted, every relevant axis has a value, and the auth owner is fixed.

## 3. Preflight and first confirmation

Read [tools](references/tools.md) and the reference of every selected unit. Each reference states its decision, version constraints, prerequisites, and acceptance checks for every option, plus dated pitfalls from real runs, labelled by tool:

| Unit | References |
| --- | --- |
| bootstrap | [workspace](references/workspace.md), [lint](references/lint.md), [hooks](references/hooks.md), [agents](references/agents.md) |
| Web | [web](references/web.md) |
| Server | [server](references/server.md) |
| Native, EAS | [native](references/native.md) |
| Desktop (Electron), its GitHub Releases | [desktop](references/desktop.md) |
| Swift (iOS, macOS), its GitHub Releases | [swift](references/swift.md) |
| Docs | [docs](references/docs.md) |
| Storybook | [storybook](references/storybook.md) |
| db | [db](references/db.md) |
| auth | [auth](references/auth.md) |
| qa | [qa](references/qa.md) |
| CI | [ci](references/ci.md), [renovate](references/renovate.md) |
| Coolify | [deploy-coolify](references/deploy-coolify.md) |
| Cloudflare | [deploy-cloudflare](references/deploy-cloudflare.md) |

Extend mode reads [workspace](references/workspace.md) for the detected layout, [lint](references/lint.md) for ignore patterns of the new unit's generated sources, [hooks](references/hooks.md) when the repository lacks a pre-commit secret scan, and [ci](references/ci.md) when it lacks a CI secret scan.

1. Check every CLI the selection needs, read-only, per [tools](references/tools.md). Note each one missing or older than its newest release.
2. Resolve the newest mutually compatible stable versions once, following the resolution order in [workspace](references/workspace.md) and each reference's constraints; freeze them for the run. Extend mode resolves new dependencies against the versions the repository already pins.
3. Show the summary: mode (extend: detected stack with evidence), the stack with every choice outside the recommended stack marked "unvalidated" (it follows official docs at runtime and has no recorded run), apps, packages, auth owner, frozen versions, targets, gates per unit, tools to install or upgrade, manual instructions for Docker and SDKs, and unresolved blockers.

Done when the user confirmed this summary. The confirmation authorizes local implementation and the listed tool installs, nothing remote.

## 4. Install tools

Install or upgrade every listed tool to its newest release by the method in [tools](references/tools.md). Docker Desktop and platform SDKs are never installed automatically: give the instructions and mark their gates blocked until they are available.

Done when each listed tool's version command reports the newest release, or the gap is recorded as a blocker.

## 5. Build and commit units

A unit is one coherent change that is green on its own: bootstrap (new mode: workspace, lint, hooks, AGENTS.md, plus `config` and `env` when a JS/TS app is selected), `ui`, an app, `db`, `auth`, `qa`, CI, a deploy target; in extend mode also AGENTS.md, the pre-commit secret scan, or the CI secret scan when the repository lacks them. Order follows dependencies, not a fixed list: bootstrap first; `env` before its consumers; `ui` before browser apps; `db` and the auth owner app before `auth`; CI and deploy config after the app they cover builds. Each unit follows its reference from the step 3 table.

The gates are `<pm> run check`, `check-types`, `build`, and `test` across the workspace; a package without a task (a Swift app has no `check-types`) records it as not applicable. Gates bypass the orchestrator cache (`--force` with Turborepo, `--skip-nx-cache` with Nx). Extend mode uses the repository's existing scripts for them and reports a missing one instead of adding it.

For each unit:
1. Implement only the selected building blocks and their prerequisites. Extend mode changes existing files only where the unit must register itself: dependency declarations and package scripts the unit needs, workspace or catalog entries, the lockfile, CI jobs, route or plugin registration in an existing app the unit plugs into (for example mounting auth routes in its owner), `ui` source scanning, hook jobs, lint ignore patterns for the unit's generated sources, its AGENTS.md section, and its `docs/scaffold.md` entry. The graph the hook refreshes belongs to every commit.
2. Run the gates. Tests run once (`vitest run --passWithNoTests`). Once an app exists it has a meaningful test; before that, `build` is not applicable and recorded so.
3. Run the unit's local acceptance checks from its reference against fresh processes, never cached success; checks a reference marks "after approval" wait for step 6. HTTP apps (Web, Server, Docs, Storybook): expected status and content from the production build, not only the dev server. Electron, Expo, and Swift: build or export, then launch. With `db`, start the database and apply migrations before any auth check.
4. Add the unit's real commands and conventions to AGENTS.md and its decisions to `docs/scaffold.md`.
5. Commit with an English conventional-commit message through the pre-commit hook.
6. Run checks a reference marks "after commit" (they need a clone) in a fresh clone; a failure gets a fix commit before the next unit starts.

A unit is done when its commit exists, touches only that unit's files, and every gate, local acceptance check, and after-commit check passed or is not applicable. A failing check blocks the commit: fix it within the confirmed design, and ask only for decisions, credentials, external prerequisites, or scope changes. When a check needs a missing prerequisite (Docker, a simulator, an OIDC provider), ask: the user provides it, or explicitly accepts committing with that check recorded as blocked in `docs/scaffold.md`; a resumed run reruns recorded blocked checks first.

`docs/scaffold.md` is a short decisions record: mode, selection, frozen versions, unvalidated choices, deviations, blocked checks. The first unit creates it; extend mode appends to an existing one.

## 6. External actions

Local-only runs skip this step and report every external action as not performed.

Otherwise prepare all remote configuration locally. Before pushing, review every tracked file for personal data, client identifiers, private endpoints, and credentials; the pre-commit gitleaks scan is the floor, not the review. Then show: owner/repository, visibility, destination accounts and resources, migration target, actions, costs, secret names and how each is delivered, the required CI check name, and preview isolation.

Act only after the user gave this second confirmation; a different destination or a destructive operation needs a new one. Then create or reuse only approved resources, inspecting remote state before any retry. Push, watch CI until green, enable branch protection with the required check only when the repository's GitHub plan supports it, and deploy. Then run every check the references mark "after approval", including deployed health and the selected auth flows.

Done when every approved action has an outcome and each deployed check passed or is recorded as blocked.

## 7. Report

Report passed, failed, blocked, and not-applicable checks separately, plus the unvalidated choices used, tools installed or upgraded, and the remaining manual steps.

Done when every selected unit and every external action appears in the report.

A new language enters as an app under `apps/*` with its own reference, inside the same root stack. Out of scope: Python and .NET apps (no reference yet), CLI tools, libraries, Tauri, alchemy, trigger.dev, Typesense, ElectricSQL, i18n, CodeQL, release-please, Scorecard, Turbo remote cache, migrating existing tooling, personal-default config, re-audit mode.
