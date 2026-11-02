# Docs: Fumadocs on TanStack Start

`apps/docs`: a separate Start app (never Next.js) with Fumadocs core/UI, its Start provider and routes, Fumadocs MDX for local content, and local search (no hosted service). The content collection generates before typecheck and build. Resolve Fumadocs, MDX, Start, React, and Tailwind peers together. The filesystem loader stays server-side; only page data reaches the client. Shared styles come from `ui`. The runtime follows the selected deploy target, like [web](web.md).

Acceptance against the production build: an intro page renders by direct URL, navigation works, search finds the page, an unknown slug returns 404, and no filesystem module ships to the browser.

Sources: [Fumadocs](https://www.fumadocs.dev/docs), [browser MDX entry](https://www.fumadocs.dev/docs/mdx/entry/browser).
