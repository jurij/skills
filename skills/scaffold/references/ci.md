# CI (GitHub Actions)

One PR/push workflow with least-privilege `contents: read`, action revisions pinned to commit SHAs (an annotated tag resolves to its commit via `refs/tags/<tag>^{}`, not the tag object), and no deployment credentials.

Jobs:
- **checks**: set up the package manager at the `packageManager` version (and Node from `.node-version` for pnpm and npm), run the frozen-lockfile install, then `<pm> run check`, `check-types`, `build`, `test`. With Postgres: a Postgres service container, migrations applied before tests. With SQLite: migrations against a file in the workspace.
- **secrets**: checkout with full history (`fetch-depth: 0`), download the pinned gitleaks release binary from its official GitHub release, verify it against the release checksums file, run `gitleaks git --redact`.
- **platform jobs**: OS-specific runners for Native or Desktop build/launch checks when those apps exist.
- **required**: one aggregate job that depends on all of the above and fails when any of them failed or was skipped. Its name is the single required check for branch protection.

Extend mode: an existing workflow keeps its jobs; add only the missing ones, and the aggregate job when the repository has no required check yet.

Branch protection on private repositories depends on the GitHub plan; verify support before promising it. A missing entitlement stays an open item; visibility never changes to work around it.

Acceptance: workflow YAML validates (for example with `actionlint`). After approval: the aggregate check goes green on a real run.

Pitfall (gitleaks-action, 2026-11-02): it requires a license key for repositories owned by an organization. The plain binary has no such requirement, so every project uses it.

Sources: [GitHub Actions](https://docs.github.com/en/actions), [protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches), [gitleaks releases](https://github.com/gitleaks/gitleaks/releases), [gitleaks-action](https://github.com/gitleaks/gitleaks-action).
