# Renovate

`renovate.json`: weekly schedule; groups for coupled toolchains (React/Expo, web framework/Vite, lint/TypeScript, ORM CLI/client/driver adapter); patch automerge only once the required check and branch protection are verified. Local-only projects keep the config but claim no bot activity.

Check in Renovate's docs whether its manager for the chosen package manager updates catalog entries. Where it does not, README and AGENTS.md document the manual weekly step: bump catalog pins, regenerate the lockfile, run all gates. Add no custom regex manager.

Acceptance: config validates with `npx --package renovate -- renovate-config-validator`.

Pitfall (bun, 2026-11-02): Renovate's bun manager does not update `workspaces.catalog`; support is pending in [renovatebot/renovate#42909](https://github.com/renovatebot/renovate/pull/42909) (open, approved). Once released, verify it on the project's catalog before dropping the manual step.

Source: [configuration options](https://docs.renovatebot.com/configuration-options/).
