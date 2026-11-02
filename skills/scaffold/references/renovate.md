# Renovate

`renovate.json`: weekly schedule; groups for coupled toolchains (React/Expo, Start/Vite, lint/TypeScript); patch automerge only once the required check and branch protection are verified. Local-only projects keep the config but claim no bot activity.

Renovate's bun manager does not update `workspaces.catalog` entries. README and AGENTS.md document the manual weekly step: bump catalog pins, regenerate `bun.lock`, run all gates. Add no custom regex manager.

Acceptance: config validates with `npx --package renovate -- renovate-config-validator`.

Pitfall (2026-11-02): catalog support is pending in [renovatebot/renovate#42909](https://github.com/renovatebot/renovate/pull/42909) (open, approved). Once released, verify it on this repo's catalog before dropping the manual step.

Source: [configuration options](https://docs.renovatebot.com/configuration-options/).
