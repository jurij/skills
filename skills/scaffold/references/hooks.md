# Git hooks, secret scan, code graph

The chosen hook tool (lefthook or husky) is an exact-version root devDependency and owns all Git hooks. gitleaks always scans in pre-commit; graphify runs only when the code graph is selected. Both are global tools ([tools](tools.md)).

## Per-clone registration

Root `package.json` gets one `prepare` line, which the install runs in every clone and worktree: `lefthook install` or `husky`. With the code graph, append the merge-driver registration:

```json
"prepare": "lefthook install && git config merge.graphify.name 'graphify graph.json union merge' && git config merge.graphify.driver 'graphify merge-driver %O %A %B'"
```

Confirm the `merge-driver` syntax in the current `graphify --help`. After adding the hook tool, run `<pm> run prepare` once: some installs (pnpm 12 `add`, an up-to-date `install`) skip the root `prepare`. Builds without a `.git` directory (container images) install with lifecycle scripts ignored.

## Pre-commit

Jobs run in this order and stop at the first failure: `<pm> run check`, the graph refresh (only with the code graph), the secret scan last, so it also scans the staged graph.

lefthook (`lefthook.yml`):

```yaml
pre-commit:
  piped: true
  jobs:
    - name: check
      run: <pm> run check
    - name: graph
      run: PYTHONHASHSEED=0 graphify update . && git add graphify-out/graph.json
    - name: secrets
      run: gitleaks git --staged --redact --verbose
```

husky (`.husky/pre-commit`, the same commands one per line):

```sh
<pm> run check
PYTHONHASHSEED=0 graphify update . && git add graphify-out/graph.json
gitleaks git --staged --redact --verbose
```

`PYTHONHASHSEED=0` keeps graph output identical across machines. `--verbose` prints each finding with the secret redacted; without it the output shows only the count. After a rejected commit, rerun `graphify update .` and restage: the staged graph still names the rejected file. Confirm the keys or the hook's stop-on-failure behaviour against the installed tool's docs, and the flags against `gitleaks git --help`.

Extend mode: add the missing jobs to the repository's existing hook tool, in the same order; with no hook tool, add lefthook, the recommendation.

## Code graph (when selected)

- `.gitignore`: `graphify-out/*` then `!graphify-out/graph.json`; only the graph is committed.
- `.gitattributes`: `graphify-out/graph.json merge=graphify linguist-generated=true -diff`.
- Bootstrap creates the first graph with `graphify update .`.
- AGENTS.md tells agents to query the graph before broad searches (`graphify query "<question>"`, `graphify explain "<node>"`, `graphify path "<a>" "<b>"`), that every commit refreshes it, and that a merge conflict on it resolves with `graphify update .` plus `git add graphify-out/graph.json`.
- Graph guidance lives in AGENTS.md only. `graphify claude install` writes CLAUDE.md, which suppresses AGENTS.md in Claude Code; graphify's own `hook install` would compete with the hook tool. Use neither.

Acceptance:
- a commit runs the jobs in order, and with the code graph the committed graph equals the output of `graphify update .`;
- a staged file holding a fake credential (for example a generated GitHub token pattern) fails the commit, with the value redacted in the output;
- after commit: a fresh clone plus install installs the hook and, with the code graph, prints the driver for `git config merge.graphify.driver`.

Sources: [graphify team recipe](https://docs.graphify.com/cookbooks/shared-team-graph), [lefthook](https://lefthook.dev/configuration/), [husky](https://typicode.github.io/husky/), [gitleaks](https://github.com/gitleaks/gitleaks).
