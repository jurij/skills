# Docs: Fumadocs

`apps/docs`: a separate app with Fumadocs core/UI on the Web framework when it is Start or Next.js, otherwise on TanStack Start; extend mode uses the detected framework the same way. Fumadocs MDX for local content and local search (no hosted service); if the chosen MDX API generates a content collection, ignore its output and generate it before typecheck and build. The UI package matches `ui`: `@fumadocs/base-ui` with Base UI, otherwise `fumadocs-ui`. Resolve Fumadocs, MDX, the framework, React, and Tailwind peers together. The filesystem loader stays server-side; only page data reaches the client. Shared styles come from `ui` when it exists, imported before the Fumadocs theme; when `ui` defines light-only page colours, disable the Fumadocs theme switch. The runtime follows the selected deploy target, like [web](web.md).

Test: the loader resolves the intro page and returns nothing for an unknown slug.

Acceptance against the production build: an intro page renders by direct URL, navigation works, search finds the page, an unknown slug returns 404, and no filesystem module ships to the browser (search the client output for `node:fs`, `fs/promises`, `readFileSync`).

Pitfall (Vitest 5, 2026-11-02): its file-system module cache kept a removed MDX page in the loader test; run the docs tests with `--no-fsModuleCache`.

Sources: [Fumadocs](https://www.fumadocs.dev/docs), [browser MDX entry](https://www.fumadocs.dev/docs/mdx/entry/browser).
