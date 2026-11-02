# Tools

Every run needs git, bun, uv, graphify, and gitleaks; the selection adds gh, Docker, provider CLIs, and platform SDKs. Always use the newest release. Upgrade with the manager that installed the tool (`command -v <tool>` shows which).

| Tool | Install or upgrade |
| --- | --- |
| bun | `curl -fsSL https://bun.sh/install \| bash`; upgrade with `bun upgrade` |
| uv | `curl -LsSf https://astral.sh/uv/install.sh \| sh`; upgrade with `uv self update`, or the package manager that installed it |
| graphify | `uv tool install graphifyy@latest` (install and upgrade) |
| gitleaks | macOS `brew install gitleaks` / `brew upgrade gitleaks`; elsewhere the official release binary |
| gh | macOS `brew install gh` / `brew upgrade gh`; elsewhere the official package |
| provider CLIs | project devDependencies where the provider ships an npm package (for example `wrangler`), otherwise the provider's official installer |

lefthook is a project devDependency, never a global tool.

Manual only, with instructions instead of an install: Docker Desktop (https://docs.docker.com/desktop/), Xcode, Android Studio, and other platform SDKs.

graphify stays one global install at the newest version. Its built-in auto-refresh of installed agent skills stays on.

Pitfall (2026-11-02): `uv tool upgrade graphifyy` keeps an exact version pin from an earlier `graphifyy==<version>` install and upgrades nothing; `uv tool install graphifyy@latest` replaces the pin.
Pitfall (2026-11-02): a second, project-local graphify version overwrote the global skill copies. Use one global install.

Sources: [bun](https://bun.com/docs/installation), [uv](https://docs.astral.sh/uv/getting-started/installation/), [graphify](https://github.com/safishamsi/graphify), [gitleaks](https://github.com/gitleaks/gitleaks), [gh](https://cli.github.com/).
