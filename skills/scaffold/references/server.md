# Server: Hono

`apps/server`: runtime-neutral Hono routes plus one entry per selected target (Bun or Node for Coolify or none, a Worker entry for Cloudflare). Listeners, environment access, and db connections live in the entry; shared routes receive them by injection. Plain Worker by default; Durable Objects only when the product needs them.

Acceptance: a handler test with injected dependencies; a health route returns the expected status and content from the production entry. The `db` and `auth` units run their checks against this runtime.

Pitfall (Hono, 2026-11-02): Bun or Node globals in shared routes pass local tests and break on Workers.

Sources: [Hono runtimes](https://hono.dev/docs/getting-started/basic), [Workers](https://hono.dev/docs/getting-started/cloudflare-workers).
