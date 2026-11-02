---
name: scaffold
description: Set up a new project with selected apps, shared packages, tooling, CI, and deployment through incremental decisions and verified commits.
disable-model-invocation: true
---

# Scaffold

Build a new Bun + Turborepo project from an empty folder: only the apps and packages the user selects, one green commit per coherent unit. Set each tool up from its own current docs and `--help`; this skill supplies the decisions, gates, and dated pitfalls, not Better-T-Stack or other skills.

Generated projects contain no copied helper scripts: tooling lives in `package.json` scripts, `lefthook.yml`, and CI YAML as ordinary one-liners.

Everything committed must be publishable: use logical names, and keep private identifiers, endpoints, credentials, and absolute local paths in ignored `.env` files or provider secret stores. Treat the installed skill files as read-only.

## 1. Inspect the target

Inspect the directory, any enclosing Git root, branch, remotes, and staged changes before writing.

Done when one of these holds:
- the target is empty and outside any Git repository;
- it holds an earlier scaffold whose `docs/scaffold.md` matches this invocation's decisions: resume at the first unit without a commit;
- the user explicitly adopted the non-empty target or named another directory. Existing work stays untouched.

## 2. Decide

Keep decisions given in the invocation. Ask only open, relevant questions, one round at a time, because later rounds depend on earlier answers. In Claude Code use AskUserQuestion (1–4 questions, 2–4 options each; paginate longer lists). In Codex or without a question tool, ask numbered plain-text questions. Silence or cancellation is never approval.

| Round | Decisions |
| --- | --- |
| Product | At least one: Web (TanStack Start), Desktop (Electron), Native (Expo + uniwind), Server (Hono). |
| Companions | Docs (Fumadocs on Start) and Storybook, in their own round. |
| Packages | `db` (Drizzle + Postgres) with Web or Server. `auth` (better-auth, requires `db`) with a client plus a Web or Server backend. `qa` (Playwright + Midscene) with Web only. |
| Targets | Per app: Web/Docs/Storybook/Server → Coolify, Cloudflare, or none; Native → EAS or none; Desktop → GitHub Releases or none. Native/Desktop also pick target OS and architecture. |
| Ownership | GitHub personal account, an organization (list with paginated read-only `gh api`), or local-only. Defaults: private repository, project name from the folder (validate npm and GitHub naming), email/password auth. |
| Integrations | Project-local skills, MCP services, and target hosts (Claude Code, Codex). Coolify endpoint only if Coolify is selected; PR previews on by default. |

The auth owner is Server when selected, otherwise Web. Desktop + auth also needs the session-storage decision in [auth](references/auth.md).

For credentials, use existing provider sessions (`gh auth status`, `wrangler whoami`, …) or the host's secret input; otherwise the user configures them directly in the provider or a local ignored file. Question text never asks for a credential.

Done when every round is answered or defaulted and the auth owner is fixed.

## 3. Preflight and first confirmation

Read [tools](references/tools.md) and the reference of every selected unit; each one carries its version constraints, prerequisites, and acceptance checks:

| Unit | References |
| --- | --- |
| bootstrap | [workspace](references/workspace.md), [lint](references/lint.md), [hooks](references/hooks.md), [agents](references/agents.md) |
| Web | [web](references/web.md) |
| Server | [server](references/server.md) |
| Native, EAS | [native](references/native.md) |
| Desktop, GitHub Releases | [desktop](references/desktop.md) |
| Docs | [docs](references/docs.md) |
| Storybook | [storybook](references/storybook.md) |
| db | [db](references/db.md) |
| auth | [auth](references/auth.md) |
| qa | [qa](references/qa.md) |
| CI | [ci](references/ci.md), [renovate](references/renovate.md) |
| Coolify | [deploy-coolify](references/deploy-coolify.md) |
| Cloudflare | [deploy-cloudflare](references/deploy-cloudflare.md) |

1. Check every CLI the selection needs, read-only: always git, bun, uv, graphify, gitleaks; gh for GitHub; Docker for `db`; platform SDKs and simulators for Native/Desktop; provider CLIs for selected deploy targets. Note each one missing or older than its newest release.
2. Resolve the newest mutually compatible stable versions once, following the resolution order in workspace and each reference's constraints; freeze them for the run. Bun's frozen version is its newest release, installed in step 4.
3. Show the summary: apps, packages, auth owner, frozen versions, targets, gates per unit, tools to install or upgrade, manual instructions for Docker and SDKs, and unresolved blockers.

Done when the user confirmed this summary. The confirmation authorizes local implementation and the listed tool installs, nothing remote.

## 4. Install tools

Install or upgrade every listed tool to its newest release by the method in [tools](references/tools.md). Docker Desktop and platform SDKs are never installed automatically: give the instructions and mark their gates blocked until they are available.

Done when each listed tool's version command reports the newest release, or the gap is recorded as a blocker.

## 5. Build and commit units

A unit is one coherent change that is green on its own: bootstrap, `ui`, an app, `db`, `auth`, `qa`, CI, a deploy target. Order follows dependencies, not a fixed list: bootstrap first; `env` before its consumers; `ui` before browser apps; `db` and the auth owner app before `auth`; CI and deploy config after the app they cover builds. Each unit follows its reference from the step 3 table.

For each unit:
1. Implement only the selected apps and their prerequisites.
2. Run the gates `bun run check`, `bun run check-types`, `bun run build`, `bun run test` across the workspace. Tests run once (`vitest run --passWithNoTests`). Once an app exists it has a meaningful test.
3. Run the unit's local acceptance checks from its reference against fresh processes, never cached success; checks a reference marks "after approval" wait for step 6. HTTP apps (Web, Server, Docs, Storybook): expected status and content from the production build, not only the dev server. Electron and Expo: build or export, then launch. With `db`, run `db:up` and migrations before any auth check.
4. Add the unit's real commands and conventions to AGENTS.md and its decisions to `docs/scaffold.md`.
5. Commit with an English conventional-commit message through the pre-commit hook.
6. Run checks a reference marks "after commit" (they need a clone) in a fresh clone; a failure gets a fix commit before the next unit starts.

A unit is done when its commit exists and every gate, local acceptance check, and after-commit check passed or is not applicable. A failing check blocks the commit: fix it within the confirmed design, and ask only for decisions, credentials, external prerequisites, or scope changes. When a check needs a missing prerequisite (Docker, a simulator), ask: the user provides it, or explicitly accepts committing with that check recorded as blocked in `docs/scaffold.md`; a resumed run reruns recorded blocked checks first.

`docs/scaffold.md` is a short decisions record: selection, frozen versions, deviations, blocked checks. Create it in the bootstrap unit.

## 6. External actions

Local-only runs skip this step and report every external action as not performed.

Otherwise prepare all remote configuration locally. Before pushing, review every tracked file for personal data, client identifiers, private endpoints, and credentials; the pre-commit gitleaks scan is the floor, not the review. Then show: owner/repository, visibility, destination accounts and resources, migration target, actions, costs, secret names and how each is delivered, the required CI check name, and preview isolation.

Act only after the user gave this second confirmation; a different destination or a destructive operation needs a new one. Then create or reuse only approved resources, inspecting remote state before any retry. Push, watch CI until green, enable branch protection with the required check only when the repository's GitHub plan supports it, and deploy. Then run every check the references mark "after approval", including deployed health and the selected auth flows.

Done when every approved action has an outcome and each deployed check passed or is recorded as blocked.

## 7. Report

Report passed, failed, blocked, and not-applicable checks separately, plus tools installed or upgraded and the remaining manual steps.

Done when every selected unit and every external action appears in the report.

Out of scope: Python/.NET/Swift projects, CLI tools, libraries, Tauri, alchemy, trigger.dev, Typesense, ElectricSQL, i18n, CodeQL, release-please, Scorecard, Turbo remote cache, personal-default config, re-audit mode.
