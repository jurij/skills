# Auth

Auth is installed and configured in its owner app, fixed in the Decide step (Server when selected, otherwise Web). How other apps consume it is the project's decision.

## better-auth

Requires [db](db.md) and uses the selected ORM's adapter (Drizzle by default).

- Generate the better-auth schema with the `auth` CLI at the better-auth version (`bunx`, `pnpm dlx`, or `npx` with `auth@<version> generate`), then generate and apply the ORM's migrations.
- The owner mounts the auth routes, with an explicit base URL, secret, and trusted origins from env; secure cookies in production.
- Owner on Start or Next.js: server code that sets cookies uses the framework's cookie plugin (`tanstackStartCookies`, `nextCookies`), listed last.
- For cross-origin clients, follow the better-auth docs on trusted origins and cookies.

## External OIDC provider

The user names a standards-compliant OIDC provider. The owner uses a maintained OIDC client library for its runtime; issuer URL, client ID, and client secret live in ignored env, never in committed config. Local checks need a disposable local OIDC provider or the user's test tenant; without one, the auth checks are blocked.

Acceptance, against the owner app directly with disposable data: signup (OIDC: first login) creates a user, a session lookup with the returned cookie returns that user, and after logout the same lookup returns no session.

Pitfall (better-auth, 2026-11-02): `@better-auth/cli` is deprecated; the CLI ships as the `auth` package with the same version as `better-auth`.
Pitfall (better-auth, 2026-11-02): the client bundle contains environment variable names such as `BETTER_AUTH_SECRET`; scan public bundles for secret values, not names.

Sources: [better-auth installation](https://better-auth.com/docs/installation), [Drizzle adapter](https://better-auth.com/docs/adapters/drizzle), [Prisma adapter](https://better-auth.com/docs/adapters/prisma), [OpenID Connect Core](https://openid.net/specs/openid-connect-core-1_0.html).
