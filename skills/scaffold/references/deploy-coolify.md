# Coolify

Per deployable app, a multi-stage Dockerfile pinned to the frozen Bun (or explicitly chosen Node) runtime: `bun install --frozen-lockfile --ignore-scripts` (the image has no `.git` for `prepare`), build only the app and its packages, copy production output, run as non-root, expose a health route. Storybook uses a static serving image. Secrets never enter image layers or build args.

Production DB and migrations run only against the approved environment after the second confirmation. PR previews are requested by default: enable them only once Coolify's preview support is confirmed and each preview has its own disposable database, credentials, and auth origins. If isolation is impossible, present the limitation; production credentials never reach a preview.

Endpoint and resource IDs stay in ignored local config.

Acceptance: the image builds locally and its health route answers; after approval, deployed health and selected auth flows pass; a preview cannot reach production data.

Pitfall (2026-11-02): preview capability varies by Coolify instance and version.

Sources: [Coolify docs](https://coolify.io/docs/), [Dockerfile build pack](https://coolify.io/docs/applications/build-packs/dockerfile).
