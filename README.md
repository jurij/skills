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

[`scaffold`](skills/scaffold/SKILL.md) sets up a Bun + Turborepo project with selected Web, Server, Native, or Desktop apps, optional Docs and Storybook, shared packages, lint, Git hooks, CI, and deployment. Invoke `/scaffold` in Claude Code or `$scaffold` in Codex; both hosts allow explicit invocation only.

It asks in rounds, shows a summary for a first confirmation (local build and tool installs), builds one green commit per unit, and asks a second time before anything remote: repository, push, migrations, deployment.

Prerequisites: git, Bun, uv, [graphify](https://github.com/safishamsi/graphify), and gitleaks as global tools; the skill lists missing or outdated ones and installs their newest versions after the first confirmation. Docker Desktop (for local Postgres) and platform SDKs stay manual.

Generated projects commit only ordinary config: `package.json` scripts, `lefthook.yml` (pre-commit: lint check, `graphify update .`, `gitleaks git --staged --redact`), and GitHub Actions YAML.

**Validation status (2026-11-02):** one local run (Web + db, deploy none, local-only) passed all local gates; remote steps and every other combination are unvalidated. Renovate does not update bun `workspaces.catalog` yet ([renovatebot/renovate#42909](https://github.com/renovatebot/renovate/pull/42909)); generated projects document a manual weekly catalog update.

The Codex `quick_validate.py` rejects Claude's `disable-model-invocation` frontmatter field. Keep it: Claude Code needs it for explicit-only invocation.

## License

[MIT](LICENSE)
