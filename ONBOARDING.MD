# Excalidraw monorepo — onboarding

## Project overview

**Excalidraw** is an open-source, hand-drawn-style virtual whiteboard. This repository is a **Yarn workspaces monorepo** that contains:

- The **`@excalidraw/excalidraw`** React library (what integrators install from npm).
- The **excalidraw.com web app** (`excalidraw-app/`), which uses that library and adds hosting-specific features (PWA, collaboration, etc.).
- **Shared packages** (`packages/*`) with types, math, element logic, and utilities used by the library and tests.

If you only want to **run the product locally**, you mostly work from the repo root with `yarn install` and `yarn start`. If you want to **change the editor**, you will spend time in `packages/excalidraw/` and related packages.

---

## High-level architecture

- **`packages/excalidraw/`** — Main editor UI and public API surface published as `@excalidraw/excalidraw`.
- **`packages/common`**, **`packages/element`**, **`packages/math`**, **`packages/utils`** — Shared building blocks; the library and Vitest resolve them via path aliases (see `vitest.config.mts`).
- **`excalidraw-app/`** — Vite-based SPA that composes the library with app-only wiring (Firebase, sockets, etc.).
- **`examples/`** — Small sample apps (e.g. Next.js, script tag) showing how consumers embed the package.
- **Root** — Workspace scripts (`package.json`), TypeScript project references (`tsconfig.json`), tests (`vitest.config.mts`, `setupTests.ts`), and tooling config.

Data flow in development: you edit package sources → Vite (in `excalidraw-app`) loads them according to workspace linking / aliases → browser shows the app.

---

## Repository structure

### Quick project map (top level)

| Directory / file | Responsibility |
|------------------|----------------|
| `excalidraw-app/` | Full web application (Vite dev server, production build). Entry point for local “run the site.” |
| `packages/excalidraw/` | Main React component library; npm package `@excalidraw/excalidraw`. |
| `packages/common/` | Shared constants, helpers, and cross-cutting code. |
| `packages/element/` | Element model and element-related logic. |
| `packages/math/` | Geometry and math utilities for the canvas. |
| `packages/utils/` | General utilities used across packages. |
| `examples/` | Integration examples (`with-nextjs`, `with-script-in-browser`). |
| `scripts/` | Build, release, and maintenance Node scripts. |
| `dev-docs/` | Internal documentation site (separate from user-facing docs). |
| `firebase-project/` | Firebase-related project assets (when working on app backends). |
| `public/` | Static assets served or copied as needed by the app/tooling. |
| `package.json` (root) | Workspace definition and **primary commands** (`yarn start`, `yarn test`, etc.). |
| `vitest.config.mts` | Test runner config and `@excalidraw/*` path aliases. |
| `tsconfig.json` (root) | TypeScript solution / references for the monorepo. |
| `yarn.lock` | Locked dependency tree — commit changes when dependencies change. |
| `Dockerfile`, `docker-compose.yml` | Container-based run/build (optional). |
| `CLAUDE.md` | Short internal notes for AI-assisted development in this repo. |

---

## Tech stack

- **Language:** TypeScript (strict), some JavaScript in scripts.
- **UI:** React (app and library).
- **App bundler / dev server:** Vite (`excalidraw-app`).
- **Package builds:** esbuild-driven scripts per package (see root `build:*` scripts).
- **Monorepo:** Yarn 1 workspaces (`packageManager`: `yarn@1.22.22`).
- **Tests:** Vitest (+ jsdom, canvas mocks where configured).
- **Lint / format:** ESLint, Prettier (see `yarn test:code`, `yarn fix`).

---

## Prerequisites

- **Node.js** `>= 18` (see root `package.json` → `engines`).
- **Yarn Classic** `1.x` (lockfile and `packageManager` field expect **Yarn 1**; use `corepack enable` + `corepack prepare yarn@1.22.22 --activate` if you standardize versions).

Check versions:

```bash
node -v    # expect v18.x or newer
yarn -v    # expect 1.22.x
```

---

## Environment variables and secrets

- The app is a **Vite** project: only variables prefixed with **`VITE_`** are exposed to client-side code.
- **`.env.*` files belong at the repository root** — `excalidraw-app/vite.config.mts` sets `envDir` to the parent folder so Vite reads `.env.development` / `.env.production` next to root `package.json`.
- Repo-root **`.env.development`** and **`.env.production`** are committed templates for local and prod-shaped builds. Copy or adjust locally as needed.
- For **machine-specific overrides** (tokens, ports, flags you do not want to commit), use **`.env.development.local`** or **`.env.local`** at that same **root** (see [Vite env files](https://vitejs.dev/guide/env-and-mode.html)); keep secrets out of git.

Common categories of variables you will see:

- **Backend URLs** — `VITE_APP_BACKEND_V2_GET_URL`, `VITE_APP_BACKEND_V2_POST_URL` (shareable link / JSON storage).
- **Collaboration** — `VITE_APP_WS_SERVER_URL` (WebSocket server; local dev often `http://localhost:3002` if you run [excalidraw-room](https://github.com/excalidraw/excalidraw-room) separately).
- **Libraries** — `VITE_APP_LIBRARY_URL`, `VITE_APP_LIBRARY_BACKEND`.
- **Firebase** — `VITE_APP_FIREBASE_CONFIG` (JSON string). Treat contents as sensitive in forks; use your own project for experiments when possible.
- **Dev UX** — `VITE_APP_PORT` (dev server port, often `3001`), `VITE_APP_ENABLE_ESLINT`, `VITE_APP_ENABLE_PWA`, etc.

If the app starts but features fail (share link, live collaboration), check the matching `VITE_*` value first.

---

## Local setup (step-by-step)

From the **repository root**:

```bash
cd /path/to/excalidraw
yarn install
```

Optional: copy env for local tweaks (only if you need overrides):

```bash
# Example: create a local override file (do not commit secrets)
cp .env.development .env.development.local
# Edit .env.development.local as needed
```

Start the web app:

```bash
yarn start
```

**Expected result:** Vite starts the `excalidraw-app` dev server; terminal shows a local URL (by default often **`http://localhost:3001`** if `VITE_APP_PORT=3001` in `.env.development`). Opening it loads the editor.

---

## Run, test, typecheck, build commands

Run all commands from the **repo root** unless noted.

| Goal | Command | Notes |
|------|---------|--------|
| Dev server (main app) | `yarn start` | Runs `vite` in `excalidraw-app/`. |
| Production build (app) | `yarn build` | App build + version script. |
| Preview production build | `yarn build:preview` | Build then `vite preview` (port **5000**). |
| Build all publishable packages | `yarn build:packages` | `common` → `math` → `element` → `excalidraw`. |
| Example: script-in-browser | `yarn start:example` | Builds packages then starts `examples/with-script-in-browser`. |
| Unit / integration tests | `yarn test` | Vitest (watch mode by default). |
| Update snapshots | `yarn test:update` | `vitest --update --watch=false`. |
| Typecheck | `yarn test:typecheck` | Runs `tsc` from root `tsconfig.json`. |
| Lint | `yarn test:code` | ESLint, zero warnings. |
| Format check | `yarn test:other` | Prettier `--list-different`. |
| Auto-fix format + lint | `yarn fix` | Prettier write + ESLint fix. |
| Full local gate (heavy) | `yarn test:all` | typecheck + lint + prettier + app tests non-watch. |

**Example — typecheck success (abbreviated):**

```text
yarn run v1.22.22
$ tsc
Done in 12.34s.
```

**Example — tests success (abbreviated):**

```text
yarn run v1.22.22
$ vitest
 ✓ packages/excalidraw/...some.test.tsx (3 tests) 45ms
 Test Files  1 passed (1)
      Tests  3 passed (3)
```

---

## Where to start reading the code

Use this order to build a mental model quickly:

- **`excalidraw-app/index.tsx`** — Vite loads this module from `excalidraw-app/index.html`; follow imports to see how the site boots and wraps the editor.
- **`packages/excalidraw/index.tsx`** — Public entry for **`@excalidraw/excalidraw`**; re-exports the component API consumers use.
- **`packages/excalidraw/`** (components, hooks, actions) — Most UI and editor behavior lives here; search by feature name (e.g. toolbar, export, selection).
- **`packages/element/src/`** — How elements are represented, updated, and validated on the canvas.
- **`packages/math/src/`** and **`packages/common/src/`** — Low-level geometry and shared types/helpers reused everywhere.
- **`vitest.config.mts`** — Shows how **`@excalidraw/*`** imports resolve during tests (mirrors Vite aliases in `excalidraw-app/vite.config.mts`).

**Tip:** After you know one flow (for example “export to PNG”), use repo search on a visible string or symbol from the UI to jump straight to the handler.

---

## Setup validation

After `yarn install`:

1. **Dependencies installed**

   ```bash
   ls node_modules | head
   ```

   **Expected:** Directory listing including `.bin`, `@types`, etc.

2. **TypeScript compiles**

   ```bash
   yarn test:typecheck
   ```

   **Expected:** Exit code **0**, no `error TS` lines, ends with `Done in …s`.

3. **Dev server**

   ```bash
   yarn start
   ```

   **Expected:** Log line with **Local:** `http://localhost:<port>/` (port from `VITE_APP_PORT`, default commonly **3001**). Browser shows Excalidraw.

4. **Tests (smoke)**

   ```bash
   yarn test:app run
   ```

   **Expected:** Vitest runs **once** (no watch), then exits with passed tests (or use `yarn test:update` when intentionally refreshing snapshots).

Stop the dev server with `Ctrl+C` in that terminal.

---

## Troubleshooting

| Problem | What to try |
|---------|-------------|
| `yarn: command not found` | Install Yarn 1 globally or via Corepack; reopen shell. |
| `tsc: command not found` when running `yarn test:typecheck` | Run `yarn install` from repo root; `typescript` is a root `devDependency`. |
| Wrong Node version / odd native errors | Use Node **18+**; switch with `nvm`, `fnm`, or `asdf`. |
| `EACCES` or permission errors in `node_modules` | Remove root `node_modules`, fix npm/yarn global prefix permissions, reinstall. |
| Port already in use (`EADDRINUSE`) | Change `VITE_APP_PORT` in `.env.development` or `.env.development.local`, or stop the other process. |
| Blank screen / env not applied | Ensure vars are prefixed with `VITE_`; restart dev server after editing `.env*`. |
| Collaboration / WS features do not work | Set `VITE_APP_WS_SERVER_URL` to a running [excalidraw-room](https://github.com/excalidraw/excalidraw-room) instance or expect those features to be disabled offline. |
| Stale package output after editing `packages/*` | Run `yarn build:packages` or use `yarn start:example` flow; for app dev, Vite usually picks up sources—if not, clear caches / restart dev server. |
| ESLint errors in editor only | Run `yarn fix`; ensure IDE uses workspace ESLint + Prettier settings. |
| Test snapshot failures | If changes are intentional, `yarn test:update`; otherwise fix code or snapshots deliberately. |

---

## Glossary

| Term | Meaning |
|------|---------|
| **Workspace** | Yarn monorepo package; each folder under `packages/*`, `excalidraw-app`, `examples/*` with its own `package.json`. |
| **`@excalidraw/excalidraw`** | Published npm name for the main React library in `packages/excalidraw`. |
| **`excalidraw-app`** | The full product shell at excalidraw.com in this repo—not the same as the npm package alone. |
| **Element** | A shape or object on the canvas (rectangle, arrow, text, etc.) and its serialized data. |
| **Scene** | The current document state: elements, appState, and related data the editor renders. |
| **Vite** | Frontend toolchain used to serve and bundle `excalidraw-app`. |
| **`VITE_*` variable** | Build-time environment variable exposed to browser code (Vite convention). |
| **Vitest** | Test runner used across the monorepo (`vitest.config.mts`). |
| **Path alias** | Import shortcut like `@excalidraw/common` mapped to `packages/common/src` in tests/build configs. |

---

## Useful links

- **User & integration docs:** [https://docs.excalidraw.com](https://docs.excalidraw.com)
- **Development guide (official):** [https://docs.excalidraw.com/docs/introduction/development](https://docs.excalidraw.com/docs/introduction/development)
- **Contributing:** [https://docs.excalidraw.com/docs/introduction/contributing](https://docs.excalidraw.com/docs/introduction/contributing)
- **Collaboration server (reference):** [https://github.com/excalidraw/excalidraw-room](https://github.com/excalidraw/excalidraw-room)
- **This repo’s quick structure note:** `CLAUDE.md` (root)

---

## Document owner + last updated

- **Owner:** Team maintaining this repository — keep this file accurate when scripts, ports, or layout change.
- **Last updated:** 2025-03-26
