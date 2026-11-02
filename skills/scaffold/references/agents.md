# Agent files

- Root `AGENTS.md` exists from the bootstrap commit and grows per unit with real commands, boundaries, and the graph guidance from [hooks](hooks.md). Generate no `CLAUDE.md`: in Claude Code it suppresses AGENTS.md.
- A minimal `GLOSSARY.md`; ADRs only for decisions actually made.
- Selected project-local skills: `npx skills add <source> --skill <name> -a claude-code -a codex -y` (no `-g`), naming the selected hosts explicitly, because auto-detection skips installed agents. Commit the skills lockfile.
- Selected MCP services: `.mcp.json` for Claude Code, `.codex/config.toml` for Codex, only for selected hosts. Credentials come from environment references, never literals. Codex loads project config only in a trusted project; the user grants trust in Codex itself.

Acceptance: AGENTS.md lists commands that run as written; no CLAUDE.md exists; each selected host lists the configured MCP tools in a read-only discovery check, or that check is recorded as blocked. No selected services means MCP checks are not applicable.

Pitfall (2026-11-02): installing a skill or config file does not prove the host loads it; verify in the host.

Sources: [skills CLI](https://github.com/vercel-labs/skills/blob/main/docs/commands.md), [Claude MCP](https://code.claude.com/docs/en/mcp), [Codex MCP](https://developers.openai.com/codex/mcp/).
