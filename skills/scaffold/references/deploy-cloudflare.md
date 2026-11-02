# Cloudflare

Per deployable app, `wrangler.jsonc` with an explicit compatibility date, generated binding types (`wrangler types`), and only the Node compatibility flags its dependencies need. Hono uses a Worker entry, Start the Cloudflare Vite integration, Storybook static assets. Wrangler is an exact-version devDependency.

Postgres on Workers needs a Worker-compatible Drizzle driver and connection strategy (for example Hyperdrive), tested locally and in the approved environment; a Node-only driver is no fallback.

Account IDs and private URLs stay in ignored config; secrets go through `wrangler secret`, and `.dev.vars` stays ignored. Previews stay off until they have isolated bindings, a disposable database, and explicit auth origins.

Acceptance: types regenerate; local Worker dev and production build pass; after approval, deployed health, migrations, and selected auth flows pass.

Pitfall (2026-11-02): passing on Node proves nothing about Workers.

Sources: [Wrangler config](https://developers.cloudflare.com/workers/wrangler/configuration/), [Start on Workers](https://developers.cloudflare.com/workers/framework-guides/web-apps/tanstack-start/), [Hyperdrive](https://developers.cloudflare.com/hyperdrive/).
