# skills

My agent skills for Claude Code, Codex, and other coding agents.

They are opinionated: built for how I work, and I am their first user. Fork and adapt freely.

## Install

With the [`skills`](https://github.com/vercel-labs/skills) CLI (Node.js 22.20+), any agent:

```bash
# All skills, globally, for Claude Code and Codex
npx skills@latest add jurij/skills -g -a claude-code -a codex --skill '*' -y

# Only scaffold
npx skills@latest add jurij/skills -g -a claude-code --skill scaffold -y

# See what is available
npx skills@latest add jurij/skills --list
```

Drop `-g` to install into the current project instead. Update with `npx skills@latest update`.

Or as a Claude Code plugin (updates itself, skills are namespaced as `/jurij-skills:<name>`):

```bash
claude plugin marketplace add jurij/skills
claude plugin install jurij-skills@jurij
```

Pick one method per agent; using both installs every skill twice.

## Skills

User-invoked only (the agent never triggers them on its own):

- [`scaffold`](./skills/scaffold/SKILL.md): set up a new project (monorepo, apps, packages, tooling, CI, deploy) through incremental question rounds, then build it commit by commit. **Under design, not usable yet.**

## License

[MIT](./LICENSE)
