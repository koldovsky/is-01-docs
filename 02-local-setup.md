# Local Setup Guide

## Prerequisites

| Tool | Version | How to Verify |
|------|---------|---------------|
| **Node.js** | >= 18.0.0 | `node --version` |
| **Yarn** | 1.x (Classic) | `yarn --version` |
| **Git** | Any recent | `git --version` |

> The project uses Yarn Classic (v1) workspaces. Do **not** use npm or Yarn Berry.

## Step 1: Clone the Repository

```bash
git clone https://github.com/excalidraw/excalidraw.git
cd excalidraw
```

## Step 2: Install Dependencies

```bash
yarn
```

This installs dependencies for all workspaces: `excalidraw-app/`, `packages/*`, and `examples/*`.

If you run into issues, try a clean install:

```bash
yarn clean-install
```

This removes all `node_modules` directories and reinstalls everything.

## Step 3: Start the Development Server

```bash
yarn start
```

This starts the Vite dev server for `excalidraw-app/` on **http://localhost:3001** (configured via `VITE_APP_PORT` in `.env.development`).

The app hot-reloads on changes to both `excalidraw-app/` and `packages/` source files thanks to the path aliases in the Vite config.

## Step 4: Verify It Works

1. Open **http://localhost:3001** in your browser
2. You should see the Excalidraw whiteboard editor
3. Try drawing a rectangle — if the hand-drawn style renders, fonts and canvas are working

---

## Environment Variables

Environment variables are defined in two files at the project root:

| File | Purpose |
|------|---------|
| `.env.development` | Used during `yarn start` |
| `.env.production` | Used during `yarn build` |

All variables are prefixed with `VITE_APP_` (Vite convention).

### Key Variables

| Variable | Purpose | Default (dev) |
|----------|---------|---------------|
| `VITE_APP_BACKEND_V2_GET_URL` | Backend for loading shared drawings | `https://json.excalidraw.com/api/v2/` |
| `VITE_APP_BACKEND_V2_POST_URL` | Backend for saving shared drawings | `https://json.excalidraw.com/api/v2/post/` |
| `VITE_APP_WS_SERVER_URL` | WebSocket server for collaboration | `https://oss-collab.excalidraw.com` |
| `VITE_APP_FIREBASE_CONFIG` | Firebase config (JSON string) | Pre-configured for dev |
| `VITE_APP_PORT` | Dev server port | `3001` |
| `VITE_APP_DISABLE_TRACKING` | Disable analytics | (not set in dev) |
| `VITE_APP_ENABLE_TRACKING` | Enable analytics | (not set in dev) |

> For local-only development, the defaults work out of the box. You do **not** need to configure Firebase or WebSocket for basic editor work.

---

## Building the Project

### Build the web app

```bash
yarn build
```

Output goes to `excalidraw-app/build/`. You can serve it:

```bash
yarn start:production
```

This builds and serves on **http://localhost:5001**.

### Build only the packages (for library development)

```bash
yarn build:packages
```

This builds all packages in order: `common` → `math` → `element` → `excalidraw`.

### Build individual packages

```bash
yarn build:common
yarn build:math
yarn build:element
yarn build:excalidraw
```

---

## Running with Docker

```bash
docker-compose up --build -d
```

This builds the app in a Node 18 container and serves it via nginx on **http://localhost:3000**.

---

## Running Examples

### Browser script example

```bash
yarn start:example
```

Runs the Vite-based example at `examples/with-script-in-browser/` on port 5002.

### Next.js example

```bash
cd examples/with-nextjs
yarn dev
```

Runs on port 3005.

---

## Common Issues and Fixes

### `canvas` module errors in tests

The project uses `vitest-canvas-mock` to mock the Canvas API. If you see canvas-related errors, ensure `setupTests.ts` is being picked up (configured in `vitest.config.mts`).

### Font loading failures

Fonts are loaded from local files during development. The `setupTests.ts` file mocks font fetches. If fonts don't render in the browser, check that `public/` assets are being served correctly.

### Port already in use

If port 3001 is occupied, either kill the existing process or change `VITE_APP_PORT` in `.env.development`.

### `yarn start` fails with module resolution errors

Run `yarn` again to ensure all workspace symlinks are correct. If that doesn't help:

```bash
yarn clean-install
```

### TypeScript errors after pulling new changes

```bash
yarn test:typecheck
```

This runs `tsc` across the entire project and will show you what needs fixing.

### Snapshot test failures after pulling

```bash
yarn test:update
```

This updates all Vitest snapshots.
