# Web: TanStack Start

`apps/web`: TanStack Start with its Vite plugin, file routes, and the generated route tree. Web is a backend too; a separate Server is optional. Server functions and server-only imports stay out of client exports. Uses `ui` and `env` from [workspace](workspace.md).

Production runtime follows the deploy target: Coolify → a Start server build for the chosen Bun or Node runtime; Cloudflare → the Cloudflare Vite integration with Worker bindings; none → a small Bun entry (`apps/web/server.ts`) that serves `dist/client` and calls the built server handler. A development server never ships.

Acceptance against the production build: a meaningful route returns the expected status and content, direct reload of a nested route works, client navigation works, and hydration logs no errors. Public bundles import nothing from `db` or server env.

Commit `routeTree.gen.ts`: `check-types` can run before `build` regenerates it.

Pitfall (2026-11-02): Start's generated file syntax changes between releases; follow the docs for the frozen version.

Sources: [Start overview](https://tanstack.com/start/latest/docs/framework/react/overview), [hosting](https://tanstack.com/start/latest/docs/framework/react/guide/hosting).
