<!-- Copilot instructions tailored for the Ferdium codebase -->
# Ferdium — Copilot / AI-assistant instructions

Purpose: give an AI coding agent the specific, actionable context needed to be productive in this Electron + React + TypeScript monorepo.

- **Big picture (what this repo is):** Electron desktop app. Main process lives under `src` (entry: [src/index.ts](src/index.ts)), renderer is a React/MobX app (entry: [src/app.tsx](src/app.tsx)). Recipes and server-like API code live in `recipes/` and `src/internal-server/`.

- **Key processes & flows:**
  - Main process starts in [src/index.ts](src/index.ts): window lifecycle, tray, DBus, `ipcApi` (see `src/electron/ipc-api`).
  - Renderer mounts in [src/app.tsx](src/app.tsx): creates `ServerApi`/`LocalApi`, `stores`, and renders `FerdiumRoutes` ([src/routes.tsx](src/routes.tsx)).
  - IPC is implemented via the `src/electron/*` helpers; cross-process contracts are defined via handlers in `features/*` and `electron/ipc-api`.
  - Recipes are packaged under `recipes/recipes/*` and treated as external webviews loaded by the app.

- **How to run, build, test (exact commands):**
  - Install: use pnpm (project pins `pnpm@10.14.0`).
    - `pnpm install`
  - Development (watch build + run):
    - Start watcher: `pnpm dev` (runs `node esbuild.mjs --watch`).
    - Run electron against build output: `pnpm start` (runs `electron ./build`).
    - Combined: `pnpm start:all-dev` or `pnpm debug` for debug flags.
  - Build: `pnpm build` (runs `preval-build-info-cli && node esbuild.mjs && electron-builder`).
  - Tests: `pnpm test` (Jest). Typecheck: `pnpm typecheck`. Lint: `pnpm lint`.
  - i18n: `pnpm manage-translations` runs `formatjs extract` && `compile`.

- **Project conventions & patterns to follow:**
  - TypeScript + some `// @ts-expect-error` usage is accepted where upstream typings are lacking; prefer adding accurate types when practical.
  - State management: `mobx` stores created in `src/stores/*` and provided via `Provider` in `app.tsx`. New feature state should follow existing store patterns (see `src/stores/*`).
  - API: There are two API layers used by renderer: `ServerApi` (remote) and `LocalApi` (local stub). See `src/api/server/ServerApi` and `src/api/server/LocalApi` for examples.
  - UI routing: React Router v6 with a `HistoryRouter` (see [src/routes.tsx](src/routes.tsx)). Follow existing route/container pattern for screens.
  - Electron main vs renderer boundary: use `src/electron/ipc-api` for exposing main APIs. Avoid direct Node usage in renderer unless file explicitly allows it (renderer uses `nodeIntegration: true` historically but be conservative).
  - Packaging: assets and asar helpers are used in `src/helpers/asar-helpers` — reference these when touching packaged paths.

- **Integration points / external deps to be aware of:**
  - DBus integration: `src/lib/DBus` — Linux-specific system integration.
  - Tray: `src/lib/Tray` handles tray interactions/state.
  - Electron-builder config appears in root/build-related files and `electron-builder.*` files — packaging tweaks happen outside `src`.
  - Native modules and node versions: project pins Node `22.18.0` in `package.json` and uses `pnpm` + `volta` hints. Use matching Node/pnpm when reproducing CI locally.

- **Files to open first (quick tour):**
  - Main process: [src/index.ts](src/index.ts)
  - Renderer entry: [src/app.tsx](src/app.tsx)
  - Routes & screens: [src/routes.tsx](src/routes.tsx)
  - IPC surface: [src/electron/ipc-api](src/electron/ipc-api)
  - Stores: `src/stores/` (look for `storeFactory`)
  - Build: `esbuild.mjs` and `electron-builder` config at project root
  - Recipes: `recipes/recipes/*`

- **Short examples (follow these patterns):**
  - Add a new feature store: mirror `storeFactory` usage in `src/stores/`, expose it via `window.ferdium.stores` in `app.tsx`.
  - Add IPC method: add handler in `src/electron/ipc-api` and a corresponding handler in the renderer `features/*` that calls `ipcRenderer.invoke`.
  - Add a route/screen: add a container in `src/containers/*`, then register it inside `src/routes.tsx` following existing nested route patterns.

- **Checks the agent should run before creating PRs:**
  - `pnpm typecheck` (TS errors)
  - `pnpm lint` and `pnpm reformat-files` (style)
  - `pnpm biome:check` (biome lints)
  - `pnpm test` (quick unit tests)
  - If modifying translations: `pnpm manage-translations`.

If anything here is unclear or you want more examples (e.g., how recipes are loaded or how `ipc-api` routes are named), tell me which area to expand and I'll iterate.
