# Onboarding (Local Development)

This guide gets you from a fresh clone to running the Excalidraw editor locally, with notes on the repository structure, the main technical stack, and the most common “new contributor” setup issues.

## Project Overview

Excalidraw is an open source, hand-drawn style whiteboard with collaborative editing and export options. This repository is a monorepo:

- `packages/*`: the core component/library code published as npm packages (for example `@excalidraw/common`, `@excalidraw/element`, `@excalidraw/excalidraw`).
- `excalidraw-app/`: the actual web app (the editor experience you run in the browser).
- `examples/*`: smaller integration examples (for example, embedding in a browser script, or a Next.js app).
- `dev-docs/`: Docusaurus site for contributor/developer documentation.

### How the app uses the packages

For local development, `excalidraw-app` consumes the workspace packages via Vite path aliases, so editing code inside `packages/*` can be reflected immediately in the running app (with HMR).

```mermaid
flowchart TD
  User[Developer] --> Repo[excalidraw monorepo]
  Repo --> App[excalidraw-app (Vite dev server)]
  Repo --> Packages[packages/*]
  App --> Aliases[Vite aliases (@excalidraw/* -> packages/*)]
  Aliases --> Packages
```

## Technical Stack

At a high level:

- Runtime: React 19 (`excalidraw-app`) and the Excalidraw component library (`packages/excalidraw`).
- Tooling/build: Vite (with React plugin) for the web app, and TypeScript across the repo.
- Testing: Vitest for app tests (`yarn test` runs `vitest` at the root).
- Code quality: ESLint + Prettier (with helper scripts at the repo root).
- Documentation: Docusaurus 2 (in `dev-docs/`).
- Collaboration (optional for local dev):
  - WebSocket client: `socket.io-client` (used by the app to connect to a collab server).
  - Local collab server: `excalidraw-room` (run separately).
  - Persistence: Firebase configuration is provided via environment variables for some collaboration-related functionality.
- PWA support: `vite-plugin-pwa` (enabled in certain modes/flags).

## Prerequisites

You need:

- Node.js `>= 18` (the repo `package.json` declares `>=18.0.0`).
- Yarn classic (this repo uses Yarn 1.22.22): `yarn`.
- Git.

## Setup Steps (Run the editor locally)

### 1. Clone the repo

```bash
git clone https://github.com/excalidraw/excalidraw.git
cd excalidraw
```

### 2. Install dependencies

From the repo root:

```bash
yarn
```

This installs workspace dependencies for `excalidraw-app`, `packages/*`, and `examples/*`.

### 3. Configure environment variables

The app is driven by `VITE_*` environment variables loaded by Vite.

1. The repo contains an example development env file at:
   - `.env.development`
2. Create overrides locally in:
   - `.env.local` (this file is ignored by git)

Common things you may want to override:

- `VITE_APP_PORT` (default in this repo is set in `.env.development`, currently `3001`).
- `VITE_APP_WS_SERVER_URL` (WebSocket collab server URL; default points to `http://localhost:3002` for dev).
- `VITE_APP_AI_BACKEND` (only needed if you use the AI feature in the UI).

Tip: if you change env vars and the dev server doesn't pick them up, restart `yarn start`.

### 4. Start the dev server

From the repo root:

```bash
yarn start
```

This runs the dev server inside `excalidraw-app` and opens a browser.

Then open the URL shown by the dev server output (the port is determined by `VITE_APP_PORT`).

### Expected local ports (defaults)

- Editor dev server: `http://localhost:3001` (default via `VITE_APP_PORT`).
- Collaboration WebSocket server: `http://localhost:3002` (default via `VITE_APP_WS_SERVER_URL`).

If you see an “address already in use” error or the page doesn't load, check those values.

## Collaboration (Optional but common)

If you want real-time collaboration locally, you need to run the collab backend separately.

1. Set up the collab server: `excalidraw-room` (run it locally as described in its repository).
2. Ensure the editor points to it using `VITE_APP_WS_SERVER_URL`.
3. Start the editor (`yarn start`) and verify the collaboration indicator in the UI.

If the collab server is not running, you will typically be able to load and edit locally, but collaboration features will fail to connect.

## Development Workflow (edit + test)

### Run the main developer checks

At the repo root:

```bash
yarn test:typecheck   # TypeScript typechecking
yarn test:code       # ESLint (format/lint rules)
yarn test:update     # Update test snapshots (when needed)
yarn test            # Vitest test run
yarn fix             # Prettier + ESLint auto-fix where possible
```

### Common “new contributor” rule of thumb before PRs

If your changes affect UI output or snapshots, run:

```bash
yarn test:update
```

Also run:

```bash
yarn test:typecheck
```

to catch TypeScript issues early.

## Docs development (optional)

To run the Docusaurus docs site locally:

```bash
yarn --cwd dev-docs start
```

This lets you preview changes to documentation under `dev-docs/docs/*`.

## Common onboarding pitfalls (read this first)

1. Port mismatch (`3000` vs `3001`)
   - Some docs/screenshots mention `3000`, but this repo's dev server is driven by `VITE_APP_PORT`.
   - If the browser doesn't load, open the correct port from your terminal output and/or check `VITE_APP_PORT`.

2. Collab WebSocket server not running
   - Collaboration requires `excalidraw-room` and a correct `VITE_APP_WS_SERVER_URL`.
   - If collaboration fails, verify the backend is running and reachable from your machine.

3. Updating env vars but not restarting
   - Vite only reliably applies env changes on server restart.

4. Modifying `.env.development` or committing local secrets
   - Prefer `.env.local` for local overrides (and keep secrets out of git).

5. Formatting/snapshots causing “it works on my machine” failures
   - Run `yarn fix` and/or `yarn test:update` when UI output changes.

6. Running commands from the wrong directory
   - `yarn start` is designed to start the app from the repo root (it forwards into `excalidraw-app`).
   - For docs, use `yarn --cwd dev-docs start`.

## Suggested first tasks

Start with “easy” items in the project roadmap, or pick small, well-scoped bugs/features. The repo's contributing docs encourage starting with beginner-friendly tasks and discussing bigger changes via an issue first.

