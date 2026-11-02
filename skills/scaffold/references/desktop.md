# Desktop: Electron

`apps/desktop`: Electron + electron-vite + electron-builder for the selected OS and architecture. The renderer uses browser `ui`; main and preload are the privileged side: context isolation on, renderer Node integration off, sandbox where supported, and a narrow preload API that validates every message.

Packaging under the hoisted layout: make the desktop workspace self-contained for electron-builder and confirm packaged runtime dependencies sit inside it (bun: `installConfig.hoistingLimits: "workspaces"` if the frozen Bun supports it; pnpm and npm: electron-builder's monorepo docs). Allow only required install scripts (bun `trustedDependencies`, pnpm `onlyBuiltDependencies`), confirm the Electron binary downloaded, and rebuild native modules for Electron.

Code signing, notarization, and auto-update are out of scope: label artifacts unsigned. GitHub Releases is an explicit upload of the reviewed artifact after the second confirmation, never a build side effect.

Acceptance: dev launch shows main, preload, and renderer working; the packaged app launches on the selected OS; the renderer has no direct Node access; preload rejects an invalid message.

Pitfall (electron-builder, 2026-11-02): a cross-compiled artifact proves nothing until it launches on the target OS.

Sources: [electron-vite](https://electron-vite.org/guide/), [Electron security](https://www.electronjs.org/docs/latest/tutorial/security), [electron-builder](https://www.electron.build/), [Bun workspaces](https://bun.com/docs/pm/workspaces).
