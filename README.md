# skills

Opinionated agent skills for Claude Code and Codex. Fork and adapt freely.

## Install

With the [skills CLI](https://github.com/vercel-labs/skills/blob/main/docs/commands.md), selecting agents interactively:

```bash
npx skills add jurij/skills --skill scaffold -g
```

Non-interactive, for both hosts (auto-detection does not select every installed agent):

```bash
npx skills add jurij/skills --skill scaffold -g -a claude-code -a codex -y
```

Omit `-g` to install into the current project.

## Scaffold

[`scaffold`](skills/scaffold/SKILL.md) sets up a repository: workspace, the selected Web, Server, Native, or Desktop apps, optional Docs and Storybook, shared packages, lint, Git hooks, CI, and deployment config, so that everything installs, builds, tests, and starts. How the apps talk to each other at runtime stays the project's decision. Invoke `/scaffold` in Claude Code or `$scaffold` in Codex; both hosts allow explicit invocation only.

**New project or existing repository.** An empty target becomes a new project. In an existing repository the skill detects the stack from `package.json`, the lockfile, and config files, offers only the missing building blocks that fit it (for example Docs, a database, or CI), and adds each as one commit on the current branch. It never migrates existing tooling, and it asks before touching a dirty tree.

**Recommended stack, or customize.** The first round shows one recommended stack:

> bun · Turborepo · TanStack Start · Postgres + Drizzle · better-auth · Ultracite (oxlint + oxfmt) · lefthook · GitHub Actions

*Use recommended* keeps it. *Customize* asks only the axes that apply to the selection, recommendation first: package manager (bun, pnpm, npm), layout (Turborepo, Nx, workspaces only, single package), web (TanStack Start, Next.js, Vite + React SPA), db (Postgres + Drizzle, Postgres + Prisma, SQLite + Drizzle), auth (better-auth, an external OIDC provider), lint (Ultracite with oxlint + oxfmt or Biome, ESLint + Prettier), and Git hooks (lefthook, husky). Native is Expo or Swift (iOS) and Desktop is Electron or Swift (macOS), asked for every run that selects them; Docs (Fumadocs), Storybook, and CI (GitHub Actions) have one option each. Decisions given in the invocation prompt are not asked again.

**Flow.** Questions come in as few rounds as possible. A summary then lists the stack (every choice outside the recommendation marked "unvalidated"), versions, gates, and the tools to install, for a first confirmation: local build and tool installs. The skill builds one green commit per unit and asks a second time before anything remote: repository, push, migrations, deployment.

**Always included:** `AGENTS.md` (never `CLAUDE.md`) and a gitleaks secret scan in the pre-commit hook and CI. A [graphify](https://github.com/safishamsi/graphify) code graph is on by default and can be deselected. Generated projects commit only ordinary config: `package.json` scripts, the hook config, and GitHub Actions YAML.

**Prerequisites:** git, the chosen package manager (plus Node for pnpm and npm), and gitleaks; uv and graphify when the code graph is selected. The skill lists missing or outdated tools before the first confirmation and installs their newest versions afterwards. Docker Desktop (for local Postgres), Xcode, and other platform SDKs stay manual.

**Validation status (2026-11-02, v0.1.0):** local runs passed every local gate and acceptance check for these combinations (deploy none, local-only):
- recommended stack: Web + db, Web + Server + db + auth (v0.0.2), Web only via *Use recommended* (v0.1.0);
- customized: pnpm + single package + Next.js + Postgres + Prisma + ESLint + Prettier + husky, code graph off;
- existing repository: Docs (Fumadocs on TanStack Start) added to a Bun + Turborepo Web + Server + db + auth repository.
- Desktop as Swift (macOS) menu bar app in a Bun + Turborepo repository with code graph (2026-11-03); its macOS CI job ran only locally.

Remote steps, Nx, workspaces only, npm, Vite SPA, SQLite, OIDC, Ultracite with Biome, Native, Desktop, Swift (iOS), Storybook, qa, and every deploy target are unvalidated. No technology is excluded: anything without a reference (Python, .NET, Tauri, App Store, notarization, …) runs as an unvalidated choice from its official docs. Renovate does not update bun `workspaces.catalog` yet ([renovatebot/renovate#42909](https://github.com/renovatebot/renovate/pull/42909)); generated bun projects document a manual weekly catalog update.

The Codex `quick_validate.py` rejects Claude's `disable-model-invocation` frontmatter field. Keep it: Claude Code needs it for explicit-only invocation.

## License

[MIT](LICENSE)
