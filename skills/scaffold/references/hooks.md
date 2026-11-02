# Git hooks, code graph, secret scan

lefthook (exact-version root devDependency) owns all Git hooks. graphify and gitleaks are global tools ([tools](tools.md)).

## Per-clone registration

Root `package.json` gets one `prepare` line, which `bun install` runs in every clone and worktree:

```json
"prepare": "lefthook install && git config merge.graphify.name 'graphify graph.json union merge' && git config merge.graphify.driver 'graphify merge-driver %O %A %B'"
```

Confirm the `merge-driver` syntax in the current `graphify --help`. Builds without a `.git` directory (container images) install with `--ignore-scripts`.

## Code graph

- `.gitignore`: `graphify-out/*` then `!graphify-out/graph.json`; only the graph is committed.
- `.gitattributes`: `graphify-out/graph.json merge=graphify linguist-generated=true -diff`.
- Bootstrap creates the first graph with `graphify update .`.
- AGENTS.md tells agents to query the graph before broad searches (`graphify query "<question>"`, `graphify explain "<node>"`, `graphify path "<a>" "<b>"`), that every commit refreshes it, and that a merge conflict on it resolves with `graphify update .` plus `git add graphify-out/graph.json`.
- Graph guidance lives in AGENTS.md only. `graphify claude install` writes CLAUDE.md, which suppresses AGENTS.md in Claude Code; graphify's own `hook install` would compete with lefthook. Use neither.

## Pre-commit

```yaml
pre-commit:
  piped: true
  jobs:
    - name: check
      run: bun run check
    - name: graph
      run: PYTHONHASHSEED=0 graphify update . && git add graphify-out/graph.json
    - name: secrets
      run: gitleaks git --staged --redact
```

The order is the contract: `piped` runs jobs in sequence and stops at the first failure, and gitleaks runs last so it also scans the staged graph. `PYTHONHASHSEED=0` keeps graph output identical across machines. Confirm the keys against the installed lefthook's docs and the flags against `gitleaks git --help`.

Acceptance:
- a commit runs check, graph, and secrets in that order, and the commit includes the refreshed graph;
- a staged file holding a fake credential (for example a generated GitHub token pattern) fails the commit, with the value redacted in the output;
- after commit: a fresh clone plus `bun install` installs the hook and prints the driver for `git config merge.graphify.driver`.

Sources: [graphify team recipe](https://docs.graphify.com/cookbooks/shared-team-graph), [lefthook](https://lefthook.dev/configuration/), [gitleaks](https://github.com/gitleaks/gitleaks).
