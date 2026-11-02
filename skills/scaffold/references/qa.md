# Web QA

`packages/qa`: Playwright + Midscene at compatible versions, browsers pinned by the Playwright version. Required offline tests start the production Web build with Playwright's `webServer` (no `reuseExistingServer` in CI) and cover a meaningful route, plus the [auth](auth.md) flow when selected. Midscene tests call a paid model: a separate opt-in script, credentials from ignored env, never part of required CI.

Acceptance: breaking the tested route fails the offline suite; selected auth flow and SSR reload pass.

Pitfall (2026-11-02): AI assertions are nondeterministic and paid; they never replace the deterministic suite.

Sources: [Playwright webServer](https://playwright.dev/docs/test-webserver), [Midscene + Playwright](https://midscenejs.com/integrate-with-playwright.html).
