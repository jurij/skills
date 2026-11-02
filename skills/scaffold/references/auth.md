# Auth

better-auth with the Drizzle adapter; requires [db](db.md). The single owner fixed in the Decide step mounts the GET/POST auth routes; other apps use client helpers from a package that never imports server config.

## Setup

- Generate the better-auth schema, then generate and apply Drizzle migrations.
- Explicit base URL, trusted origins, secure cookies in production, credentialed CORS only where a cross-origin client needs it. Development origins stay out of production config.
- Start server functions that set cookies use better-auth's TanStack Start cookie plugin, listed last.
- Native: `@better-auth/expo` with `expo()` on the owner; `expoClient()` with `expo-secure-store` storage, an explicit app scheme, and `expo-network` on the client.
- Every protected server operation checks the session; client route guards authorize nothing.
- Desktop: decide session storage before building, either a secure cookie session with a fixed origin or main-process protected storage behind a narrow validated preload API. The renderer never sees the database or the auth secret.

## Web + Server

- Proxy: Web exposes a fixed `/api/auth/*` reverse proxy to the configured Server owner, so browsers stay same-origin with host-only cookies. The upstream comes from config, never from the request. Forward Cookie and Origin; keep each Set-Cookie header separate. Check: a signup through the proxy sets the cookie on the Web origin.
- SSR: Start forwards the incoming cookie only to the owner's session endpoint, without caching or following redirects, and passes refreshed cookies into its response. Web never instantiates a second auth owner. Check: authenticated SSR reload.

Acceptance, per selected client platform, against disposable data:
- signup creates a user;
- the next request carries a session;
- an unauthenticated request is rejected;
- logout invalidates the session.

Web also checks an authenticated SSR reload; Native also checks the session survives an app restart.

Pitfall (2026-11-02): cross-origin browser cookies do not authenticate Start SSR; the same-origin proxy and explicit SSR cookie forwarding are needed together.

Sources: [Drizzle adapter](https://better-auth.com/docs/adapters/drizzle), [TanStack Start](https://better-auth.com/docs/integrations/tanstack), [Expo](https://better-auth.com/docs/integrations/expo).
