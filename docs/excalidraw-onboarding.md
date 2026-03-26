# Excalidraw Onboarding

## 1. Project Overview

Excalidraw is an open-source drawing and whiteboard project. It lets people create diagrams, wireframes, notes, and sketches with a hand-drawn look.

It solves a simple problem: making visual ideas easy to create and share without using heavy design software. The project includes both the reusable Excalidraw library and the full web app used on `excalidraw.com`.

Main features:

- Infinite canvas for free-form drawing
- Hand-drawn visual style
- Shapes, arrows, lines, text, and free-draw tools
- Image support
- Export to formats like PNG and SVG
- Collaboration support in the app
- Local-first behavior and shareable links
- Localization support

## 2. Tech Stack

Frontend:

- TypeScript
- React
- Vite
- Sass / CSS

Project and tooling:

- Yarn workspaces for the monorepo
- Vitest for tests
- ESLint for linting
- Prettier for formatting
- esbuild for package builds

Libraries and services used in the app:

- Firebase
- Socket.IO client
- Jotai
- Sentry

Backend:

- There is no main backend in this repository.
- The web app connects to external services for collaboration, storage, sharing, AI-related features, and analytics.

## 3. How to Run the Project Locally

### Prerequisites

- Node.js 18 or newer
- Yarn 1.22.x
- Git

The root `package.json` uses `yarn@1.22.22`, so it is best to match that version.

### Installation

From the project root:

```bash
corepack enable
corepack prepare yarn@1.22.22 --activate
yarn install
```

If `corepack` is not available, install Yarn 1.22.x another way and then run:

```bash
yarn install
```

### Start the app

To start the main web app:

```bash
yarn start
```

This runs the app from `excalidraw-app/`.

### Build the project

Build everything:

```bash
yarn build
```

Build only the app:

```bash
yarn build:app
```

Build the internal packages:

```bash
yarn build:packages
```

## 4. Project Structure

This repo is a monorepo. The main parts are:

- `packages/excalidraw/`  
  Main reusable React component library published as `@excalidraw/excalidraw`.

- `excalidraw-app/`  
  Full web app for `excalidraw.com`. This is where app-only behavior lives, such as collaboration-related UI and production app setup.

- `packages/common/`  
  Shared helpers and shared logic used across packages.

- `packages/element/`  
  Element-related logic for shapes and drawing data.

- `packages/math/`  
  Math helpers used by the editor.

- `packages/utils/`  
  General utility functions.

- `examples/`  
  Small example integrations, including Next.js and browser script usage.

- `scripts/`  
  Repo-level build and helper scripts.

- `public/`  
  Static public assets.

- `vitest.config.mts`  
  Test configuration and local package aliases.

- `tsconfig.json`  
  Main TypeScript configuration.

- `package.json`  
  Root scripts for running, building, linting, and testing the monorepo.

## 5. Development Workflow

### How to make changes

1. Pick the right area first.
2. Use `packages/*` for editor or library changes.
3. Use `excalidraw-app/` for app-specific changes.
4. Make your changes in small, focused commits.
5. Run checks before opening a pull request.

### Development mode

Use:

```bash
yarn start
```

Useful scripts from the repo root:

```bash
yarn test:typecheck
yarn test:update
yarn test:code
yarn test:other
yarn fix
```

What they do:

- `yarn test:typecheck` checks TypeScript types
- `yarn test:update` runs app tests and updates snapshots
- `yarn test:code` runs ESLint
- `yarn test:other` checks formatting with Prettier
- `yarn fix` auto-fixes formatting and lint issues where possible

## 6. Environment Configuration

The project already includes tracked environment files:

- `.env.development`
- `.env.production`

These define many app URLs and feature flags used by Vite.

Important notes:

- Local-only overrides should go in `.env.local` or `.env.development.local`
- Do not commit personal or temporary local values
- The development config points the collaboration server to `http://localhost:3002`
- The development config points the AI backend to `http://localhost:3016`
- If some app features do not work locally, check whether the related service is running and whether the matching `VITE_APP_*` variable is set correctly

## 7. Testing

Testing is available in this repo.

Main tools:

- Vitest
- ESLint
- Prettier
- TypeScript compiler

Useful commands:

```bash
yarn test
yarn test:update
yarn test:coverage
yarn test:typecheck
yarn test:code
yarn test:other
```

The app also has test files inside `excalidraw-app/tests/`, and the root Vitest config uses a `jsdom` test environment.

## 8. Common Issues / Tips

- If `yarn` is missing, enable it with Corepack and use Yarn 1.22.22.
- If install fails, make sure your Node.js version is 18 or newer.
- If the app starts but collaboration does not work, check the WebSocket server URL in the environment config.
- If AI or other connected features fail locally, check the related `VITE_APP_*` variables.
- If formatting or lint checks fail, run `yarn fix`.
- If TypeScript errors look unrelated to your change, run a clean install and try again.
- The app and the library live in the same repo. Make sure you edit the right place before changing files.

## 9. Contribution Basics

Create a branch:

```bash
git checkout -b your-branch-name
```

Make changes and run checks:

```bash
yarn test:typecheck
yarn test:update
```

Commit your work:

```bash
git add .
git commit -m "Describe your change"
```

Create a pull request:

1. Push your branch to GitHub.
2. Open a pull request against `master`.
3. Add a clear title and short description.
4. Mention any testing you ran.

## Quick Start Summary

If you want the shortest path to a running app:

```bash
corepack prepare yarn@1.22.22 --activate
yarn install
yarn start
```

For most editor work, start by looking in `packages/excalidraw/`. For app-only work, start in `excalidraw-app/`.
