# Excalidraw Developer Onboarding (Confluence-Ready Markdown)

> **Info**  
> This page is a technical onboarding reference for engineers joining the Excalidraw codebase.  
> It focuses on architecture, runtime, module boundaries, and the practical path to first contribution.

---

## Page Metadata

- **Audience:** Engineers onboarding to Excalidraw
- **Primary goal:** Build accurate mental model of system architecture
- **Secondary goal:** Enable safe and fast first PR
- **Repository type:** Yarn workspaces monorepo

### Scope Clarification

This document onboards developers to the external repository `excalidraw/excalidraw`.
All file paths and architecture references are relative to that target repository, not this `is-01-docs` repository.

---

## Quick Navigation

- [System Overview](#system-overview)
- [Monorepo and Package Boundaries](#monorepo-and-package-boundaries)
- [Runtime Architecture](#runtime-architecture)
- [Critical Data Flows](#critical-data-flows)
- [Build and Test Infrastructure](#build-and-test-infrastructure)
- [Local Development Setup](#local-development-setup)
- [Module Decision Guide](#module-decision-guide)
- [First PR Technical Path](#first-pr-technical-path)
- [Appendix: File Map](#appendix-file-map)

---

## System Overview

Excalidraw is implemented as a layered monorepo:

1. **Core primitives** (`common`, `math`)
2. **Element domain logic** (`element`)
3. **Editor library** (`excalidraw`)
4. **Host web app** (`excalidraw-app`)

This structure allows:

- reusable editor package for integrators (`@excalidraw/excalidraw`)
- separation of app-specific concerns (collab backends, app shell, deployment config)
- clear ownership of geometry/model/UI runtime responsibilities

## Monorepo and Package Boundaries

### Workspace Structure

- `packages/common` — shared constants/utilities/event infrastructure
- `packages/math` — geometry primitives and 2D math utilities
- `packages/element` — element model and scene operations
- `packages/utils` — export and utility helpers
- `packages/excalidraw` — React editor package and public API surface
- `excalidraw-app` — full host application
- `examples/*` — integration references

### Dependency Direction

> **Important**  
> Preserve layering. Avoid introducing app-level dependencies into core packages.

Typical dependency graph:

- `@excalidraw/common` <- base shared layer
- `@excalidraw/math` depends on `common`
- `@excalidraw/element` depends on `common`, `math`
- `@excalidraw/excalidraw` depends on `common`, `math`, `element`, `utils`
- `excalidraw-app` depends on `@excalidraw/excalidraw` + app services

```mermaid
flowchart LR
  C["@excalidraw/common"]
  M["@excalidraw/math"]
  E["@excalidraw/element"]
  U["@excalidraw/utils"]
  X["@excalidraw/excalidraw"]
  A["excalidraw-app"]

  C --> M
  C --> E
  M --> E
  C --> X
  M --> X
  E --> X
  U --> X
  X --> A
```

---

## Runtime Architecture

### 1. Library Entrypoint and Public Surface

**File:** `packages/excalidraw/index.tsx`

Responsibilities:

- exports `Excalidraw` component
- provides `ExcalidrawAPIProvider`
- re-exports core APIs/hooks/types and utility helpers
- bridges lower-layer exports (`element`, `common`, `utils`) to consumer-facing API

Why this matters:
- this file defines what external integrators and the app can consume
- API changes here have broad impact

### 2. Editor Runtime Core

**File:** `packages/excalidraw/components/App.tsx`

Responsibilities:

- orchestrates interaction runtime (pointer, keyboard, gestures)
- coordinates app state transitions
- connects tool behavior with element operations
- triggers rendering and UI behavior
- consumes low-level functions from `@excalidraw/common`, `@excalidraw/math`, `@excalidraw/element`

Why this matters:
- most behavior changes in editing experience eventually route here

### 3. Host App Bootstrap and Composition

**Files:** `excalidraw-app/index.tsx`, `excalidraw-app/App.tsx`

Responsibilities:

- app mount + service worker registration
- composition of library editor with app shell components
- app-specific integrations:
  - collaboration wiring
  - backend share/import/export
  - local storage/session behavior
  - app-level menus, sidebars, and welcome surfaces

```mermaid
flowchart TD
  Start["Browser loads app"] --> Entry["excalidraw-app/index.tsx"]
  Entry --> SW["registerSW()"]
  Entry --> App["Render excalidraw-app/App.tsx"]
  App --> Init["initializeScene(...)"]

  Init --> LS["Import local storage scene"]
  Init --> URL["Read URL/hash scene params"]
  Init --> Collab["Read collaboration link data"]
  Init --> Backend["Import scene from backend if needed"]

  LS --> Merge["Restore + reconcile scene/appState"]
  URL --> Merge
  Collab --> Merge
  Backend --> Merge

  Merge --> Mount["Mount <Excalidraw />"]
```

---

## Critical Data Flows

### A. Scene Initialization Flow

Entry point behavior in app startup combines scene candidates from:

- browser local storage
- URL-encoded external scene identifiers
- collaboration room link data
- backend-fetched serialized data

Technical implications:

- app decides conflict/overwrite behavior when external scene exists
- restoration pipeline uses `restoreElements`/`restoreAppState` utilities

### B. Share Link Export/Import Pipeline

**File:** `excalidraw-app/data/index.ts`

Export path:

1. serialize scene
2. compress payload
3. encrypt payload
4. POST to backend
5. construct URL with hash payload metadata (`#json=...`)

Import path:

1. fetch payload
2. decompress/decrypt (supports legacy fallback)
3. parse scene JSON
4. restore scene state into editor

Security-relevant detail:

- key material is placed in URL hash instead of query to avoid server-side propagation through standard request logs.

```mermaid
sequenceDiagram
  participant User
  participant App as excalidraw-app
  participant Data as data/index.ts
  participant JSON as JSON Backend
  participant FB as Firebase Files

  User->>App: Export shareable link
  App->>Data: serialize scene + files
  Data->>Data: compress + encrypt payload
  Data->>JSON: POST encrypted scene
  JSON-->>Data: share id
  Data->>FB: upload image binaries
  Data-->>App: URL with hash key (#json=id,key)
  App-->>User: Shareable URL

  User->>App: Open share URL
  App->>Data: importFromBackend(id, key)
  Data->>JSON: GET encrypted payload
  JSON-->>Data: encrypted bytes
  Data->>Data: decompress + decrypt + parse
  Data-->>App: restored elements/appState
```

### C. Collaboration Sync Pipeline

**Primary file:** `excalidraw-app/collab/Collab.tsx`

Core elements:

- websocket room connection lifecycle
- sync of scene elements (including version-based updates)
- cursor/pointer/user status propagation
- idle/active user signal handling
- coordination with file manager for image assets

Supporting pieces:

- `Portal` for socket transport semantics
- data helpers for room link generation and syncable element filtering
- file status tracking via `fileStatusStore`

```mermaid
sequenceDiagram
  participant ClientA as Client A
  participant WS as Collab WS Server
  participant ClientB as Client B
  participant FB as Firebase Files

  ClientA->>WS: Join room (roomId, roomKey context)
  ClientB->>WS: Join room
  WS-->>ClientA: INIT/room state
  WS-->>ClientB: INIT/room state

  ClientA->>WS: SCENE_UPDATE (syncable elements)
  WS-->>ClientB: SCENE_UPDATE
  ClientA->>WS: MOUSE_LOCATION / IDLE_STATUS / BOUNDS
  WS-->>ClientB: presence updates

  ClientA->>FB: upload added image files
  ClientB->>FB: fetch missing image files
```

### D. File/Binary Asset Pipeline

Image elements reference file IDs; binaries are managed separately:

- file IDs embedded in scene elements
- binary fetch/save through Firebase data helpers
- upload/download guarded by room/link state and encryption context where needed

---

## Build and Test Infrastructure

### Toolchain Summary

- **Package manager:** Yarn v1 workspaces
- **Node version:** `>=18`
- **App bundler:** Vite (`excalidraw-app/vite.config.mts`)
- **Package builds:** custom scripts in `scripts/*`
- **Tests:** Vitest
- **Lint/format:** ESLint + Prettier

### Source Aliasing in Dev/Test

Both app Vite and root Vitest configs alias `@excalidraw/*` imports to local package source files.  
This enables in-place package development without publishing interim package versions.

### Key Root Commands

```bash
yarn start
yarn test:typecheck
yarn test:app
yarn test:update
yarn test:code
yarn test:other
yarn fix
```

> **Warning**  
> `test:code` is configured with `--max-warnings=0`; warnings fail CI-style checks.

---

## Local Development Setup

### 1) Install and Run

```bash
yarn
yarn start
```

### 2) Environment Model

Root env files:

- `.env.development`
- `.env.production`

Contain runtime config for:

- JSON backend endpoints
- websocket collaboration server
- Firebase configuration
- AI backend endpoint
- dev toggles (PWA, ESLint overlay, reload behavior)
- app port configuration

Vite app config uses root env directory (`envDir: "../"`), so environment values are sourced from repository root.

### 3) Build-Time Behavior to Know

In `excalidraw-app/vite.config.mts`:

- package source aliasing
- manual chunking (locales, codemirror, mermaid-related bundles)
- PWA plugin + runtime caching strategy
- custom asset naming and font handling

---

## Module Decision Guide

Use this when choosing edit location.

- **Geometry/algebra primitives** -> `packages/math`
- **Element model/selection/bounds/transforms** -> `packages/element`
- **Shared constants/events/utilities** -> `packages/common`
- **Editor behavior/public API/UI orchestration** -> `packages/excalidraw`
- **Collab/session/share/backend/app shell** -> `excalidraw-app`

### Practical Mapping Examples

- Arrow binding logic issue -> `packages/element`
- Tool interaction bug in editor canvas -> `packages/excalidraw/components/App.tsx`
- Public API/export shape change -> `packages/excalidraw/index.tsx`
- Collab room sync behavior regression -> `excalidraw-app/collab/Collab.tsx`
- Share link decode issue -> `excalidraw-app/data/index.ts`

---

## First PR Technical Path

### Recommended sequence

1. Identify failing behavior and map to correct layer
2. Reproduce with local app run (`yarn start`)
3. Implement minimal fix in target package/app module
4. Validate static and runtime quality gates:
   - `yarn test:typecheck`
   - `yarn test:code`
   - `yarn test:update` (when snapshot-affecting)
5. Verify behavior manually in app scenario
6. Submit focused PR with technical test notes

### Minimal Quality Gate Checklist

- [ ] Correct layer and module chosen
- [ ] Type checks pass
- [ ] Lint passes with zero warnings
- [ ] Tests/snapshots updated as needed
- [ ] Manual runtime verification completed

---

## Appendix: File Map

### High-Value Entry Points

- Root workspace config: `package.json`
- Root test/alias config: `vitest.config.mts`
- App build/runtime config: `excalidraw-app/vite.config.mts`
- Library public API entry: `packages/excalidraw/index.tsx`
- Core editor runtime: `packages/excalidraw/components/App.tsx`
- App composition root: `excalidraw-app/App.tsx`
- App bootstrap: `excalidraw-app/index.tsx`
- Collaboration runtime: `excalidraw-app/collab/Collab.tsx`
- Share/import/export logic: `excalidraw-app/data/index.ts`
- Element domain index: `packages/element/src/index.ts`
- Common/math indexes: `packages/common/src/index.ts`, `packages/math/src/index.ts`

---

### Expand: Suggested Deep-Dive Order (Day 1-2)

1. `packages/excalidraw/index.tsx` (public surface)
2. `packages/excalidraw/components/App.tsx` (runtime core)
3. `packages/element/src/index.ts` (domain operations)
4. `excalidraw-app/App.tsx` (host composition)
5. `excalidraw-app/collab/Collab.tsx` and `excalidraw-app/data/index.ts` (network/data flows)
6. `excalidraw-app/vite.config.mts` and `vitest.config.mts` (tooling/alias architecture)
