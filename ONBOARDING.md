# Excalidraw — New Employee Onboarding Guide

Welcome to the Excalidraw project! This guide walks you through everything you need to get productive: what the project is, how it's structured, how to run it locally, and how development workflows operate day-to-day.

---

## Table of Contents

1. [What Is Excalidraw?](#1-what-is-excalidraw)
2. [Repository Overview](#2-repository-overview)
3. [Architecture Diagram](#3-architecture-diagram)
4. [Prerequisites](#4-prerequisites)
5. [Getting Started](#5-getting-started)
6. [Project Structure Deep Dive](#6-project-structure-deep-dive)
7. [Package Dependency Graph](#7-package-dependency-graph)
8. [Development Workflows](#8-development-workflows)
9. [Testing Strategy](#9-testing-strategy)
10. [Code Quality & Linting](#10-code-quality--linting)
11. [Build System](#11-build-system)
12. [Key Concepts](#12-key-concepts)
13. [Where to Find Things](#13-where-to-find-things)
14. [Contributing](#14-contributing)

---

## 1. What Is Excalidraw?

Excalidraw is an **open-source virtual whiteboard** for sketching hand-drawn-style diagrams. It is:

- A **React component library** (`@excalidraw/excalidraw`) published to npm — so other applications can embed the editor
- A **full-featured web application** hosted at [excalidraw.com](https://excalidraw.com) — the official editor
- **Real-time collaborative**, **end-to-end encrypted**, and works **offline as a PWA**
- **MIT licensed**

```
┌─────────────────────────────────────────────────────┐
│                excalidraw.com (web app)              │
│                  excalidraw-app/                     │
│                                                     │
│  Uses ──────────────────────────────────────────►  │
│                @excalidraw/excalidraw               │
│                packages/excalidraw/                 │
└─────────────────────────────────────────────────────┘

The library can also be embedded in any React application:
npm install @excalidraw/excalidraw
```

---

## 2. Repository Overview

The repository is a **Yarn monorepo** managed with Yarn workspaces. It contains the library, the app, supporting packages, and integration examples — all in one repository.

```
excalidraw/                    ← repo root
├── excalidraw-app/            ← The web application (excalidraw.com)
├── packages/
│   ├── excalidraw/            ← Main npm library (@excalidraw/excalidraw)
│   ├── common/                ← Shared utilities (@excalidraw/common)
│   ├── element/               ← Element logic (@excalidraw/element)
│   ├── math/                  ← Geometry math (@excalidraw/math)
│   └── utils/                 ← Public utility helpers (@excalidraw/utils)
├── examples/
│   ├── with-nextjs/           ← Next.js integration example
│   └── with-script-in-browser/← Plain browser script example
├── scripts/                   ← Build and release automation
├── public/                    ← Static assets for the app
├── package.json               ← Root: workspace config + all dev scripts
├── tsconfig.json              ← Root TypeScript config (shared base)
├── vitest.config.mts          ← Test runner configuration
└── .eslintrc.json             ← ESLint configuration
```

---

## 3. Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MONOREPO (Yarn Workspaces)                   │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                      excalidraw-app/                         │   │
│  │   Vite + React app · collaboration · PWA · Sentry · i18n    │   │
│  │                                                              │   │
│  │   components/   collab/   data/   share/   app-language/    │   │
│  └──────────────────────┬───────────────────────────────────────┘   │
│                         │ imports                                   │
│  ┌──────────────────────▼───────────────────────────────────────┐   │
│  │              packages/excalidraw  (@excalidraw/excalidraw)   │   │
│  │                                                              │   │
│  │   Canvas renderer · Actions · Scene · History · Fonts       │   │
│  │   Components · Hooks · Gestures · Mermaid · WYSIWYG         │   │
│  └───────┬────────────────────────────────────────────────────--┘   │
│          │ imports                                                   │
│  ┌───────▼──────────┐  ┌─────────────────┐  ┌────────────────────┐ │
│  │ packages/element │  │  packages/math  │  │  packages/common   │ │
│  │                  │  │                 │  │                    │ │
│  │ Element creation │  │ Points, vectors │  │ Constants, colors  │ │
│  │ Binding, arrows  │  │ Curves, angles  │  │ Utilities, events  │ │
│  │ Groups, frames   │  │ Polygons, lines │  │ Keys, bounds       │ │
│  └──────────────────┘  └─────────────────┘  └────────────────────┘ │
│                                                                     │
│  ┌──────────────────┐  ┌─────────────────────────────────────────┐  │
│  │  packages/utils  │  │              examples/                  │  │
│  │                  │  │  with-nextjs · with-script-in-browser   │  │
│  │ Export, BBox,    │  │  (demonstrate library integration)      │  │
│  │ shape utilities  │  └─────────────────────────────────────────┘  │
│  └──────────────────┘                                               │
└─────────────────────────────────────────────────────────────────────┘
```

### Tech Stack at a Glance

| Layer | Technology |
|-------|-----------|
| UI Framework | React 18 + TypeScript |
| Build (app) | Vite |
| Build (packages) | esbuild (custom script) |
| Testing | Vitest + jsdom + Testing Library |
| Linting | ESLint + Prettier |
| State management | Jotai (atoms) |
| Monorepo | Yarn 1.x Workspaces |
| Git hooks | Husky + lint-staged |

---

## 4. Prerequisites

Before you begin, install:

| Tool | Version | Install |
|------|---------|---------|
| **Node.js** | >= 18.0.0 | [nodejs.org](https://nodejs.org) |
| **Yarn** | 1.22.x | `npm install -g yarn` |
| **Git** | any recent | [git-scm.com](https://git-scm.com) |

Verify your setup:

```bash
node --version   # should print v18.x.x or higher
yarn --version   # should print 1.22.x
git --version
```

---

## 5. Getting Started

### Clone and install

```bash
git clone https://github.com/excalidraw/excalidraw.git
cd excalidraw
yarn install
```

`yarn install` at the root installs dependencies for **all** workspaces at once.

### Run the development app

```bash
yarn start
```

This starts the Vite dev server for the web app. Open [http://localhost:5173](http://localhost:5173) in your browser.

### Run the browser-script example

```bash
yarn start:example
```

This builds all packages first, then starts the plain-HTML integration example.

---

## 6. Project Structure Deep Dive

### `excalidraw-app/` — The Web Application

The user-facing app at excalidraw.com. Built with Vite.

```
excalidraw-app/
├── App.tsx                   ← Root app component
├── index.tsx                 ← Entry point
├── index.html                ← HTML shell
├── vite.config.mts           ← Vite configuration
├── app_constants.ts          ← App-level constants
├── app-jotai.ts              ← Jotai atom definitions for the app
├── sentry.ts                 ← Error tracking setup
├── app-language/             ← Language/locale switching
├── collab/                   ← Real-time collaboration logic
├── components/               ← App-specific UI components
├── data/                     ← Data persistence, import/export
├── share/                    ← Link sharing
└── tests/                    ← App-level tests
```

> **Key distinction**: `excalidraw-app/` contains code that only makes sense for excalidraw.com (collab server, Sentry, link sharing). Generic editor code belongs in `packages/excalidraw/`.

### `packages/excalidraw/` — The Library

The publishable npm package. Everything here is part of the public (or internal) API.

```
packages/excalidraw/
├── index.tsx                 ← Library entry point / public exports
├── types.ts                  ← Core TypeScript types
├── appState.ts               ← Application state shape and defaults
├── history.ts                ← Undo/redo history
├── actions/                  ← Editor actions (commands pattern)
├── components/               ← Reusable UI components (toolbar, dialogs…)
├── renderer/                 ← Canvas rendering pipeline
├── scene/                    ← Scene graph management
├── hooks/                    ← React hooks
├── data/                     ← Serialization, clipboard, export
├── fonts/                    ← Embedded font handling
├── locales/                  ← i18n translation strings
├── mermaid.ts                ← Mermaid diagram integration
├── wysiwyg/                  ← In-canvas text editing
├── eraser/                   ← Eraser tool logic
├── lasso/                    ← Lasso selection tool
└── tests/                    ← Library unit & integration tests
```

### `packages/common/` — Shared Utilities

Low-level utilities shared by all other packages. No dependencies on other `@excalidraw/*` packages.

```
src/
├── constants.ts              ← App-wide constants
├── colors.ts                 ← Color palette
├── utils.ts                  ← General utility functions
├── bounds.ts                 ← Bounding box helpers
├── keys.ts                   ← Keyboard key constants
├── emitter.ts                ← Event emitter
├── appEventBus.ts            ← Global event bus
└── index.ts                  ← Public exports
```

### `packages/element/` — Element Logic

All logic related to Excalidraw elements (shapes, arrows, text, frames, etc.).

```
src/
├── newElement.ts             ← Element factory functions
├── mutateElement.ts          ← Immutable element updates
├── binding.ts                ← Arrow-to-element binding
├── bounds.ts                 ← Element bounding box
├── groups.ts                 ← Element grouping
├── frame.ts                  ← Frame element logic
├── linearElementEditor.ts    ← Line/arrow editing
├── arrows/                   ← Arrow routing logic
├── elbowArrow.ts             ← Elbow (right-angle) arrow
└── Scene.ts                  ← Scene class
```

### `packages/math/` — Geometry

Pure math utilities — no React, no DOM. Safe to use anywhere.

```
src/
├── point.ts                  ← Point operations
├── vector.ts                 ← 2D vectors
├── line.ts                   ← Line math
├── segment.ts                ← Line segments
├── curve.ts                  ← Bezier curves
├── angle.ts                  ← Angle utilities
├── ellipse.ts                ← Ellipse math
├── polygon.ts                ← Polygon operations
└── rectangle.ts              ← Rectangle helpers
```

### `packages/utils/` — Public Utility API

Utilities intended for consumers of `@excalidraw/excalidraw` (external developers).

---

## 7. Package Dependency Graph

```
         @excalidraw/common
               ▲   ▲   ▲
               │   │   │
  @excalidraw/ │   │   │ @excalidraw/
      math ────┘   │   └─── element
                   │             ▲
                   │             │
         @excalidraw/excalidraw ─┘
                   ▲
                   │
            excalidraw-app
                   │
             excalidraw.com
```

**Rule of thumb**: code flows upward. Lower packages must never import from higher ones. `@excalidraw/common` has no internal dependencies.

---

## 8. Development Workflows

### Daily development cycle

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐     ┌─────────────┐
│  yarn start │────►│  Edit code   │────►│ yarn test:   │────►│  yarn fix   │
│ (dev server)│     │ in your IDE  │     │    update    │     │ (lint+fmt)  │
└─────────────┘     └──────────────┘     └──────────────┘     └─────────────┘
```

### All Scripts Reference

#### Running the App

| Command | What it does |
|---------|-------------|
| `yarn start` | Start Vite dev server for the web app |
| `yarn start:production` | Build then serve the app locally on port 5001 |
| `yarn start:example` | Build packages + run the browser-script example |

#### Building

| Command | What it does |
|---------|-------------|
| `yarn build` | Build the web app (app + version stamp) |
| `yarn build:packages` | Build all 4 internal packages in order |
| `yarn build:common` | Build `@excalidraw/common` only |
| `yarn build:math` | Build `@excalidraw/math` only |
| `yarn build:element` | Build `@excalidraw/element` only |
| `yarn build:excalidraw` | Build `@excalidraw/excalidraw` only |
| `yarn build:app` | Build the web app with tracking enabled |
| `yarn build:app:docker` | Build the web app for Docker (Sentry disabled) |
| `yarn build:preview` | Build the app and start Vite preview server |

> **Note**: `yarn build:packages` builds in dependency order: `common → math → element → excalidraw`. Always build them in this order if doing it manually.

#### Testing

| Command | What it does |
|---------|-------------|
| `yarn test` | Run app tests (watch mode) |
| `yarn test:update` | Run all tests and update snapshots — **run before committing** |
| `yarn test:all` | Full CI test suite: typecheck + lint + format + tests |
| `yarn test:app` | Run Vitest tests (watch mode by default) |
| `yarn test:typecheck` | TypeScript type checking (`tsc`) |
| `yarn test:code` | ESLint check (zero warnings allowed) |
| `yarn test:other` | Prettier format check |
| `yarn test:coverage` | Run tests with code coverage report |
| `yarn test:coverage:watch` | Coverage in watch mode |
| `yarn test:ui` | Vitest UI browser interface with coverage |

#### Fixing & Formatting

| Command | What it does |
|---------|-------------|
| `yarn fix` | Auto-fix Prettier + ESLint issues |
| `yarn fix:code` | Auto-fix ESLint issues only |
| `yarn fix:other` | Auto-fix Prettier issues only |

#### Cleanup

| Command | What it does |
|---------|-------------|
| `yarn rm:build` | Delete all build output directories |
| `yarn rm:node_modules` | Delete all node_modules directories |
| `yarn clean-install` | Remove node_modules then fresh `yarn install` |

### Pre-commit Hooks

Husky runs **lint-staged** automatically before each commit. You don't need to do anything extra — it will run the linter and formatter on changed files.

---

## 9. Testing Strategy

Tests live in two places:

```
packages/excalidraw/tests/    ← Library unit & integration tests
excalidraw-app/tests/         ← App-level tests
packages/*/src/__tests__/     ← Package-level tests (element, math, common)
```

The test runner is **Vitest** with a **jsdom** environment (simulates a browser).

### Test coverage thresholds (enforced in CI)

| Metric | Minimum |
|--------|---------|
| Lines | 60% |
| Branches | 70% |
| Functions | 63% |
| Statements | 60% |

### Writing a test

```ts
import { describe, it, expect } from "vitest";
import { someFunction } from "../myModule";

describe("someFunction", () => {
  it("should do the thing", () => {
    expect(someFunction(input)).toEqual(expected);
  });
});
```

Test files use the `.test.ts` or `.test.tsx` convention.

---

## 10. Code Quality & Linting

### Rules enforced automatically

- **ESLint** — zero warnings allowed (`--max-warnings=0`)
- **Prettier** — formatting for CSS, SCSS, JSON, Markdown, HTML, YAML
- **TypeScript strict mode** — no implicit `any`, strict null checks

### Notable custom lint rules

1. **Import ordering**: Imports must follow a defined group order (builtins → externals → internal packages → relative). Enforced by `eslint-plugin-import`.

2. **Type imports**: Always use `import type { Foo }` for type-only imports.

3. **Jotai**: App code must import atoms from the app-specific modules (`app-jotai`), not directly from `jotai`.

4. **No barrel imports from packages**: In `packages/excalidraw`, you must import from specific files, not the package index.

### Workflow

```bash
yarn fix          # auto-fix all fixable issues (run this first)
yarn test:code    # check for remaining lint issues
yarn test:other   # check formatting
```

---

## 11. Build System

### App Build (Vite)

The `excalidraw-app/` is built with **Vite**. Config is at `excalidraw-app/vite.config.mts`.

```
yarn build
  └── yarn build:app          (vite build → excalidraw-app/dist/)
  └── yarn build:version      (stamps git SHA and version)
```

### Package Build (esbuild)

Each package is compiled with a custom `scripts/buildPackage.js` script that uses **esbuild**. TypeScript types are generated separately via `tsc`.

```
yarn build:excalidraw
  └── rimraf dist
  └── node ../../scripts/buildPackage.js   (esbuild → dist/)
  └── yarn gen:types                       (tsc → dist/types/)
```

Output for each package ends up in `packages/<name>/dist/`.

### Build Order

When building packages, dependencies must be built first:

```
1. @excalidraw/common    (yarn build:common)
2. @excalidraw/math      (yarn build:math)
3. @excalidraw/element   (yarn build:element)
4. @excalidraw/excalidraw (yarn build:excalidraw)
```

`yarn build:packages` does all four in the correct order.

---

## 12. Key Concepts

### Elements

Everything drawn on the canvas is an **element** — a plain JavaScript object describing a shape. Elements are immutable; changes produce new objects via `mutateElement()`.

```ts
// Element types include:
// "rectangle" | "ellipse" | "diamond" | "arrow" | "line" | "text"
// "image" | "frame" | "freedraw" | "embeddable" | "iframe"
```

### Scene

The **Scene** (in `packages/element/src/Scene.ts`) is the central store for all elements. It tracks which elements exist and notifies subscribers of changes.

### Actions

Editor operations are implemented as **Actions** (`packages/excalidraw/actions/`). Each action has a `perform` function, optional keyboard shortcut, and toolbar integration. This is the correct place to add new editor commands.

### AppState

`appState` (`packages/excalidraw/appState.ts`) holds UI state that is not part of the document — things like selected tool, zoom level, and open dialogs. It is separate from the element scene.

### Jotai Atoms

Global reactive state is managed with **Jotai**. App-level atoms are defined in `excalidraw-app/app-jotai.ts`; library atoms are in `packages/excalidraw/editor-jotai.ts`.

### History (Undo/Redo)

The undo/redo system (`packages/excalidraw/history.ts`) tracks deltas — the difference between element states — rather than full snapshots.

### Renderer

The canvas rendering pipeline lives in `packages/excalidraw/renderer/`. It renders elements to an HTML5 Canvas using a combination of direct 2D context drawing and rough.js for the hand-drawn style.

---

## 13. Where to Find Things

| I want to… | Look here |
|-----------|-----------|
| Add a new element type | `packages/element/src/newElement.ts` |
| Change how an element renders | `packages/excalidraw/renderer/` |
| Add a toolbar action | `packages/excalidraw/actions/` |
| Add a UI component | `packages/excalidraw/components/` |
| Change app-level behavior (collab, sharing) | `excalidraw-app/` |
| Add i18n translation string | `packages/excalidraw/locales/` |
| Change canvas math / geometry | `packages/math/src/` |
| Debug state / atoms | `packages/excalidraw/editor-jotai.ts` or `excalidraw-app/app-jotai.ts` |
| Read the public API surface | `packages/excalidraw/index.tsx` |
| Read the changelog | `packages/excalidraw/CHANGELOG.md` |
| Read contribution guidelines | [docs.excalidraw.com/docs/introduction/contributing](https://docs.excalidraw.com/docs/introduction/contributing) |

---

## 14. Contributing

### Checklist before opening a PR

```
[ ] yarn test:update      ← tests pass and snapshots are up-to-date
[ ] yarn test:typecheck   ← no TypeScript errors
[ ] yarn fix              ← code is formatted and lint-clean
[ ] Changes are in the right package (library vs. app)
[ ] New public APIs are exported from packages/excalidraw/index.tsx
[ ] New translation strings are added to locales/en.json
```

### Branch Strategy

- Work on feature branches off `master`
- PRs target `master`
- Keep PRs focused — one concern per PR

### Useful Links

| Resource | URL |
|----------|-----|
| Official docs | https://docs.excalidraw.com |
| Contributing guide | https://docs.excalidraw.com/docs/introduction/contributing |
| npm package | https://www.npmjs.com/package/@excalidraw/excalidraw |
| Issue tracker | https://github.com/excalidraw/excalidraw/issues |
| Discord community | https://discord.gg/UexuTaE |

---

*Last updated: 2026-03-26*
