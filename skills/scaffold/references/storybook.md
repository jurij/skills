# Storybook

`apps/storybook`: React + Vite framework, consuming `packages/ui` and its Tailwind stylesheet. Requires `ui`, so at least one browser renderer. Pin core, framework, and addons with the frozen Vite/React set. One real shared-component story with an interaction and accessibility assertion, fed by explicit props, never app env or auth config. Output `storybook-static` is static: Coolify serves it from a static image, Cloudflare as static assets.

Acceptance: the static build serves the story with the expected content, and the interaction test passes without backend secrets.

Pitfall (Storybook, 2026-11-02): its Vite framework is not the web framework's runtime; mock framework APIs (Start server functions, Next.js router) explicitly.

Sources: [React + Vite](https://storybook.js.org/docs/get-started/frameworks/react-vite), [testing](https://storybook.js.org/docs/writing-tests).
