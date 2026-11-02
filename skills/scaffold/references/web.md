# Web

`apps/web` (or the root in a single package). Start and Next.js are backends too, so a separate Server is optional; a Vite SPA has no server and needs Server for `db` and `auth`. Server-only code and imports stay out of client bundles. Uses `ui` and `env` from [workspace](workspace.md).

The production runtime follows the deploy target, and a development server never ships. Deploy none means the framework's own production server on the package manager's runtime (Bun or Node).

## TanStack Start

Start's Vite plugin, file routes, and the generated route tree. Server functions and server-only imports stay out of client exports. Commit `routeTree.gen.ts`: `check-types` can run before `build` regenerates it.

Runtime: Coolify, a Start server build for the chosen Bun or Node runtime; Cloudflare, the Cloudflare Vite integration with Worker bindings; none, a small server entry (`apps/web/server.ts`) that serves `dist/client` and calls the built server handler.

Pitfall (Start, 2026-11-02): Start's generated file syntax changes between releases; follow the docs for the frozen version.
Pitfall (Start, 2026-11-02): a static import of `./dist/server/server.js` in `server.ts` fails `check-types` before `build`, CI's order; load it with a non-literal `await import(entry)` and a type guard.
Pitfall (Start + Ultracite, 2026-11-02): `throw notFound()` fails `only-throw-error`; disable that rule on the line with a reason.

## Next.js

App Router. Server-only modules import `server-only`; tests alias it to an empty module. `next.config.ts` imports the env modules, so `next build` and `next start` validate them at startup; those modules skip `server-only`, which throws there. Runtime: Coolify, standalone output in the image; Cloudflare, the OpenNext Cloudflare adapter; none, `next start`.

## Vite + React SPA

Vite with the React plugin and a client router. The static build needs a fallback to `index.html` for client routes on every target; deploy none serves it with `vite preview` for checks.

Acceptance against the production build: a meaningful route returns the expected status and content, direct reload of a nested route works, client navigation works, and hydration and the console log no errors. Public bundles import nothing from `db` or server env.

Sources: [Start hosting](https://tanstack.com/start/latest/docs/framework/react/guide/hosting), [Next.js deploying](https://nextjs.org/docs/app/getting-started/deploying), [OpenNext Cloudflare](https://opennext.js.org/cloudflare), [Vite static deploy](https://vite.dev/guide/static-deploy).
