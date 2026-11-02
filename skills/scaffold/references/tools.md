# Tools

Every run needs git, the chosen package manager, and gitleaks (the mandatory secret scan). The selection adds Node (pnpm, npm, and the CI acceptance checks), uv and graphify (code graph), gh (GitHub), Docker (Postgres), provider CLIs, and platform SDKs. List only the tools that apply. Always use the newest release; Node uses the newest LTS major that every selected framework and ORM supports (their `engines`, docs, and install warnings). Upgrade with the manager that installed the tool (`command -v <tool>` shows which).

| Tool | Install or upgrade |
| --- | --- |
| git | macOS `brew install git` / `brew upgrade git` (or Apple's Command Line Tools); elsewhere the system package manager |
| bun | `curl -fsSL https://bun.sh/install \| bash`; upgrade with `bun upgrade` |
| Node | the version manager already on the machine (`fnm`, `nvm`, `mise`, …), otherwise the official installer |
| pnpm | `curl -fsSL https://get.pnpm.io/install.sh \| sh -`; upgrade with `pnpm self-update` |
| npm | ships with Node; upgrade with `npm install -g npm@latest` |
| uv | `curl -LsSf https://astral.sh/uv/install.sh \| sh`; upgrade with `uv self update`, or the package manager that installed it |
| graphify | `uv tool install graphifyy@latest` (install and upgrade) |
| gitleaks | macOS `brew install gitleaks` / `brew upgrade gitleaks`; elsewhere the official release binary |
| gh | macOS `brew install gh` / `brew upgrade gh`; elsewhere the official package |
| provider CLIs | project devDependencies where the provider ships an npm package (for example `wrangler`), otherwise the provider's official installer |

The hook tool (lefthook, husky) is a project devDependency, never a global tool. CI acceptance uses actionlint (a local binary or the `rhysd/actionlint` image) and `npx --package renovate@<version> -- renovate-config-validator`.

Manual only, with instructions instead of an install: Docker Desktop (https://docs.docker.com/desktop/), Xcode, Android Studio, and other platform SDKs.

graphify stays one global install at the newest version. Its built-in auto-refresh of installed agent skills stays on.

Pitfall (pnpm, 2026-11-02): `pnpm` on PATH may be a Corepack shim; Node 25+ ships no Corepack. The official installer moves pnpm to `$PNPM_HOME/bin` and rewrites the shell profile's PATH block, so globals linked by older versions in `$PNPM_HOME` leave PATH: list the profile change in the summary and keep `$PNPM_HOME` on PATH.
Pitfall (Prisma 7.10, 2026-11-02): it supports Node 20, 22, and 24, not 26.
Pitfall (uv + graphify, 2026-11-02): `uv tool upgrade graphifyy` keeps an exact version pin from an earlier `graphifyy==<version>` install and upgrades nothing; `uv tool install graphifyy@latest` replaces the pin.
Pitfall (graphify, 2026-11-02): a second, project-local graphify version overwrote the global skill copies. Use one global install.

Sources: [bun](https://bun.com/docs/installation), [Node](https://nodejs.org/en/download), [pnpm](https://pnpm.io/installation), [uv](https://docs.astral.sh/uv/getting-started/installation/), [graphify](https://github.com/safishamsi/graphify), [gitleaks](https://github.com/gitleaks/gitleaks), [gh](https://cli.github.com/).
