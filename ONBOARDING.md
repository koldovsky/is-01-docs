# Excalidraw — Developer Onboarding Guide

Welcome to the Excalidraw codebase. This guide will help you get up and running as a contributor.

## Table of Contents

- [Project Overview](#project-overview)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Architecture Overview](#architecture-overview)
- [Key Concepts](#key-concepts)
- [Testing](#testing)
- [Code Quality & Linting](#code-quality--linting)
- [Environment Variables](#environment-variables)
- [Common Tasks](#common-tasks)
- [Codebase Token Evaluation](#codebase-token-evaluation)
- [Troubleshooting](#troubleshooting)
- [Further Reading](#further-reading)

---

## Project Overview

Excalidraw is an open-source, MIT-licensed virtual whiteboard with a hand-drawn aesthetic. It supports infinite canvas, dark mode, real-time collaboration, export (PNG/SVG/JSON), shape libraries, i18n (61+ languages), and more.

The repository is a **Yarn workspaces monorepo** with two main concerns:

1. **`packages/`** — Reusable libraries published to npm (the `@excalidraw/*` family)
2. **`excalidraw-app/`** — The full-featured web application deployed at excalidraw.com

### High-Level Monorepo Map

```mermaid
graph TB
    subgraph Monorepo["excalidraw monorepo"]
        APP["excalidraw-app<br/>(Full web app - excalidraw.com)"]
        subgraph Packages["packages/"]
            CORE["@excalidraw/excalidraw<br/>(Core React library)"]
            ELEM["@excalidraw/element<br/>(Element manipulation)"]
            MATH["@excalidraw/math<br/>(2D math utilities)"]
            COMMON["@excalidraw/common<br/>(Shared constants and helpers)"]
            UTILS["@excalidraw/utils<br/>(Export and file I/O)"]
        end
        EXAMPLES["examples/<br/>(Next.js, browser)"]
    end

    APP --> CORE
    APP --> UTILS
    CORE --> ELEM
    CORE --> MATH
    CORE --> COMMON
    ELEM --> MATH
    ELEM --> COMMON
    UTILS --> CORE
    EXAMPLES --> CORE

    style APP fill:#4A90D9,color:#fff
    style CORE fill:#E8744F,color:#fff
    style ELEM fill:#F5A623,color:#fff
    style MATH fill:#7B68EE,color:#fff
    style COMMON fill:#50C878,color:#fff
    style UTILS fill:#DA70D6,color:#fff
```

## Tech Stack

| Category | Technology |
|---|---|
| **UI Framework** | React 19, TypeScript 5.9 |
| **Build** | Vite 5 (app), esbuild (packages) |
| **Package Manager** | Yarn 1.22 (workspaces) |
| **State Management** | Jotai (atom-based) |
| **Drawing Engine** | RoughJS, perfect-freehand, Canvas API |
| **UI Primitives** | Radix UI |
| **Testing** | Vitest, Testing Library |
| **Collaboration** | Socket.io, Firebase |
| **Error Tracking** | Sentry |
| **Linting/Formatting** | ESLint, Prettier, Husky + lint-staged |

**Node requirement:** >= 18.0.0

## Repository Structure

```text
excalidraw/
├── packages/
│   ├── excalidraw/        # Core React component library (@excalidraw/excalidraw)
│   ├── common/            # Shared utilities, constants, colors
│   ├── element/           # Element creation, mutation, geometry, binding
│   ├── math/              # 2D math (vectors, matrices, geometry)
│   └── utils/             # Export helpers, file I/O, format conversion
├── excalidraw-app/        # Full web app (excalidraw.com)
│   ├── App.tsx            # Main app component
│   ├── collab/            # Real-time collaboration
│   ├── data/              # Firebase, storage integration
│   └── share/             # Shareable links, room management
├── examples/              # Integration examples (Next.js, browser script)
├── dev-docs/              # Developer documentation
├── scripts/               # Build & utility scripts
├── firebase-project/      # Firebase configuration
└── public/                # Static assets
```

### Key Packages

**`packages/excalidraw/`** — The main library and largest package:
- `index.tsx` — Entry point; exports `Excalidraw`, `ExcalidrawAPIProvider`, `useExcalidrawAPI()`
- `components/` — ~156 React components (`App.tsx` is the core editor)
- `actions/` — 48 action modules (align, clipboard, export, undo/redo, etc.)
- `data/` — Persistence (JSON, blob, encode/decode, restore)
- `renderer/` — Canvas and SVG rendering pipeline
- `scene/` — Scene management, zoom, scroll, export
- `hooks/` — Custom React hooks
- `locales/` — 61+ i18n translation files
- `fonts/` — Font subsetting and loading

**`packages/element/`** — Element manipulation:
- `binding.ts` — Arrow-to-element binding
- `newElement.ts` / `mutateElement.ts` — Create and modify elements
- `bounds.ts` / `collision.ts` — Geometry and hit-testing
- `linearElementEditor.ts` — Path/arrow editing
- `resizeElements.ts` / `dragElements.ts` — Transform operations

**`packages/math/`** — Vector operations, matrix transforms, geometry utilities

**`packages/common/`** — Constants, colors, keyboard mappings, shared helpers

**`packages/utils/`** — PNG/SVG export, file handling, format utilities

## Getting Started

### Prerequisites

- **Node.js** >= 18.0.0
- **Yarn** 1.22.x (v1 Classic)

### Initial Setup

```bash
# Clone the repository
git clone <repo-url>
cd excalidraw

# Install all dependencies (workspaces resolved automatically)
yarn

# Start the development server (excalidraw-app)
yarn start
```

The dev server runs at `http://localhost:3001` by default.

### Clean Reinstall

If you encounter dependency issues:

```bash
yarn clean-install
```

## Development Workflow

### Developer Flow Overview

```mermaid
flowchart TD
    START["Clone repo and yarn install"] --> DEV["yarn start<br/>(Dev server on :3001)"]
    DEV --> CODE["Write code"]
    CODE --> TEST["yarn test:app<br/>(Run Vitest)"]
    TEST -->|Fail| CODE
    TEST -->|Pass| TYPES["yarn test:typecheck"]
    TYPES -->|Fail| CODE
    TYPES -->|Pass| FIX["yarn fix<br/>(Auto-format and lint)"]
    FIX --> SNAPSHOT["yarn test:update<br/>(Update snapshots)"]
    SNAPSHOT --> COMMIT["git commit<br/>(Husky runs lint-staged)"]
    COMMIT -->|Hook fails| FIX
    COMMIT -->|Pass| PUSH["git push and open PR"]

    style START fill:#50C878,color:#fff
    style PUSH fill:#4A90D9,color:#fff
    style CODE fill:#E8744F,color:#fff
```

### Day-to-Day Commands

| Command | Description |
|---|---|
| `yarn start` | Start the dev server (excalidraw-app) |
| `yarn build` | Production build of the app |
| `yarn build:packages` | Build all publishable packages |
| `yarn test:all` | Run the full test suite |
| `yarn test:app` | Run Vitest unit/integration tests |
| `yarn test:typecheck` | TypeScript type checking |
| `yarn test:code` | ESLint checks |
| `yarn test:other` | Prettier formatting checks |
| `yarn fix` | Auto-fix formatting and lint issues |
| `yarn test:update` | Update test snapshots |

### Before Committing

Always run these before pushing:

```bash
yarn test:update      # Update snapshots
yarn test:typecheck   # Ensure types are correct
yarn fix              # Auto-fix lint/format issues
```

Git hooks (Husky + lint-staged) will automatically run ESLint on staged `.js/.ts/.tsx` files and Prettier on `.css/.scss/.json/.md/.html/.yml` files during pre-commit.

## Architecture Overview

### Rendering Pipeline

Excalidraw uses a **Canvas-based rendering** system with three modes:

1. **Interactive rendering** (`interactiveScene.ts`) — Live editor canvas with user interactions
2. **Static rendering** (`staticScene.ts`) — For thumbnails and non-interactive views
3. **SVG rendering** (`staticSvgScene.ts`) — Vector export

The `Renderer` class manages context, caching, and animation frame batching.

```mermaid
flowchart LR
    subgraph Input
        POINTER["Pointer Events<br/>(mouse/touch/pen)"]
        KEYBOARD["Keyboard<br/>Shortcuts"]
    end

    subgraph Core["Core Engine"]
        ACTIONS["Action System"]
        STATE["AppState<br/>(Jotai atoms)"]
        ELEMENTS["Element Store"]
        HISTORY["History<br/>(undo/redo)"]
    end

    subgraph Rendering
        IR["Interactive<br/>Renderer<br/>(Canvas)"]
        SR["Static<br/>Renderer<br/>(Canvas)"]
        SVG["SVG<br/>Renderer"]
    end

    subgraph Output
        SCREEN["Screen"]
        EXPORT["Export<br/>(PNG/SVG/JSON)"]
    end

    POINTER --> ACTIONS
    KEYBOARD --> ACTIONS
    ACTIONS --> STATE
    ACTIONS --> ELEMENTS
    ACTIONS --> HISTORY
    STATE --> IR
    ELEMENTS --> IR
    ELEMENTS --> SR
    ELEMENTS --> SVG
    IR --> SCREEN
    SR --> EXPORT
    SVG --> EXPORT

    style Core fill:#2D3748,color:#fff
    style Rendering fill:#4A5568,color:#fff
```

### State Management

- **Jotai atoms** drive reactive state throughout the app
- `AppState` type defines the global editor state (tool selection, zoom, scroll, theme, etc.)
- Editor-specific atoms live in `editor-jotai.ts`
- State updates follow an **immutable pattern**

### Element System

Elements are the core data model. Types include: `rectangle`, `diamond`, `ellipse`, `arrow`, `line`, `freedraw`, `text`, `image`, `frame`, `embeddable`, `magicframe`, and more.

```mermaid
classDiagram
    class ExcalidrawElement {
        +id: string
        +type: string
        +x: number
        +y: number
        +width: number
        +height: number
        +angle: number
        +strokeColor: string
        +backgroundColor: string
        +groupIds: string list
        +frameId: string
    }

    class ExcalidrawLinearElement {
        +points: Point list
        +startBinding: Binding
        +endBinding: Binding
    }

    class ExcalidrawTextElement {
        +text: string
        +fontSize: number
        +fontFamily: string
        +containerId: string
    }

    class ExcalidrawImageElement {
        +fileId: string
        +status: string
        +scale: number
    }

    class ExcalidrawFrameElement {
        +name: string
    }

    ExcalidrawElement <|-- ExcalidrawLinearElement : arrow, line
    ExcalidrawElement <|-- ExcalidrawTextElement : text
    ExcalidrawElement <|-- ExcalidrawImageElement : image
    ExcalidrawElement <|-- ExcalidrawFrameElement : frame
    ExcalidrawLinearElement --> ExcalidrawElement : binds to (start/end)
    ExcalidrawTextElement --> ExcalidrawElement : contained in (boundText)
```

Key relationships:
- **`id`** — Unique element identifier (nanoid)
- **`groupIds`** — Groups multiple elements
- **`frameId`** — Associates elements with frames
- **Bindings** — Arrows connect to shapes via `startBinding` / `endBinding`
- **Bound text** — Text elements attached to shapes

### Component Architecture

```mermaid
graph TD
    EX["Excalidraw<br/>(Root component)"]
    APP["App.tsx<br/>(Core editor, ~407KB)"]

    subgraph UI["UI Layer"]
        TOOLBAR["Toolbar and<br/>Actions"]
        CANVAS["Canvas<br/>(Rendering surface)"]
        SIDEBAR["Sidebar and<br/>Panels"]
        CMD["Command<br/>Palette"]
        COLOR["Color<br/>Picker"]
        DIALOG["Dialog<br/>System"]
    end

    subgraph API["Public API"]
        PROVIDER["ExcalidrawAPIProvider"]
        HOOK["useExcalidrawAPI()"]
        CALLBACKS["onChange / onExport<br/>onPointerUpdate"]
    end

    EX --> APP
    APP --> TOOLBAR
    APP --> CANVAS
    APP --> SIDEBAR
    APP --> CMD
    APP --> COLOR
    APP --> DIALOG
    EX --> PROVIDER
    PROVIDER --> HOOK
    APP --> CALLBACKS

    style EX fill:#E8744F,color:#fff
    style APP fill:#4A90D9,color:#fff
    style API fill:#2D3748,color:#fff
```

### Collaboration (excalidraw-app only)

```mermaid
sequenceDiagram
    participant User A
    participant Browser A
    participant Socket.io Server
    participant Browser B
    participant User B

    User A->>Browser A: Draw element
    Browser A->>Browser A: Encrypt (E2EE)
    Browser A->>Socket.io Server: Send increment
    Socket.io Server->>Browser B: Broadcast increment
    Browser B->>Browser B: Decrypt (E2EE)
    Browser B->>Browser B: Reconcile state
    Browser B->>User B: Render update

    Note over Browser A, Browser B: Cursor positions sent<br/>as ephemeral increments
```

- **Socket.io** for real-time sync
- Durable and ephemeral increments for operations
- Collaborator cursor/pointer tracking
- End-to-end encryption for room data

### Data Persistence

```mermaid
flowchart TD
    subgraph Local["Local Storage"]
        IDB["IndexedDB"]
        LS["localStorage"]
    end

    subgraph Cloud["Cloud (excalidraw-app)"]
        FB["Firebase"]
        JSON_API["JSON API<br/>(scene sharing)"]
    end

    subgraph Formats["File Formats"]
        EXCALIDRAW[".excalidraw<br/>(JSON)"]
        BINARY["Compressed<br/>Binary"]
        PNG["PNG Export"]
        SVG_F["SVG Export"]
    end

    EDITOR["Editor State"] --> IDB
    EDITOR --> FB
    EDITOR --> JSON_API
    EDITOR --> EXCALIDRAW
    EDITOR --> BINARY
    EDITOR --> PNG
    EDITOR --> SVG_F
    IDB --> EDITOR
    FB --> EDITOR
    EXCALIDRAW --> EDITOR

    style EDITOR fill:#4A90D9,color:#fff
```

- **Local**: IndexedDB / localStorage
- **Cloud**: Firebase (in excalidraw-app)
- **File formats**: `.excalidraw` (JSON), compressed binary
- Schema migrations handled via `restore` / `reconcile` logic

## Key Concepts

### Path Aliases

TypeScript path aliases map `@excalidraw/*` to the corresponding packages. These are configured in `tsconfig.json` and mirrored in `vitest.config.mts` for test resolution:

- `@excalidraw/common` → `packages/common`
- `@excalidraw/element` → `packages/element`
- `@excalidraw/math` → `packages/math`
- `@excalidraw/utils` → `packages/utils`
- `@excalidraw/excalidraw` → `packages/excalidraw`

### Build System

- **App** — Vite bundles `excalidraw-app` with PWA, SVG-as-component, and sitemap plugins
- **Packages** — esbuild produces `dist/` with dev/prod variants; TypeScript declarations generated separately via `gen:types`
- Only `packages/*` are published to npm; `excalidraw-app` deploys to Vercel

### Input Handling

- Shortcut registry system with key combination mapping
- Gesture detection for touch devices
- Unified pointer events (mouse, touch, pen)

## Testing

### Setup

Tests use **Vitest** with a **jsdom** environment. The setup file (`setupTests.ts`) provides:

- Canvas mocking (`vitest-canvas-mock`)
- Testing Library Jest-DOM matchers
- Fake IndexedDB
- Window API mocks (`FontFace`, `matchMedia`, `fonts`)
- Font loading mock (reads local files)

### Running Tests

```bash
yarn test:app              # Run all Vitest tests
yarn test:app -- --watch   # Watch mode
yarn test:coverage         # Generate coverage report
yarn test:update           # Update snapshots
```

### Coverage Thresholds

| Metric | Minimum |
|---|---|
| Lines | 60% |
| Statements | 60% |
| Branches | 70% |
| Functions | 63% |

### Writing Tests

- Test files: `*.test.ts` or `*.test.tsx`, colocated with source
- Use Testing Library for component tests
- Snapshot tests are supported — run `yarn test:update` after intentional changes

## Code Quality & Linting

### ESLint

- Extends `@excalidraw/eslint-config` and `react-app`
- Enforces import ordering (`@excalidraw/*` after external packages)
- Enforces type-only imports via TypeScript plugin
- Restricts direct imports from barrel `index` files
- Custom rules for Jotai usage

### Prettier

- Configured via `@excalidraw/prettier-config`
- Covers `.css`, `.scss`, `.json`, `.md`, `.html`, `.yml`

### Git Hooks

- **Pre-commit** (Husky + lint-staged): ESLint `--max-warnings=0 --fix` on code files, Prettier on config/style files

## Environment Variables

All prefixed with `VITE_APP_` for Vite exposure to the client.

### Development Defaults (`.env.development`)

| Variable | Value |
|---|---|
| `VITE_APP_BACKEND_V2_GET_URL` | `https://json-dev.excalidraw.com/api/v2/` |
| `VITE_APP_BACKEND_V2_POST_URL` | `https://json-dev.excalidraw.com/api/v2/post/` |
| `VITE_APP_WS_SERVER_URL` | `http://localhost:3002` |
| `VITE_APP_PORT` | `3001` |
| `VITE_APP_ENABLE_TRACKING` | `true` |
| `VITE_APP_ENABLE_PWA` | `false` |

### Production Overrides (`.env.production`)

| Variable | Value |
|---|---|
| `VITE_APP_BACKEND_V2_GET_URL` | `https://json.excalidraw.com/api/v2/` |
| `VITE_APP_WS_SERVER_URL` | `https://oss-collab.excalidraw.com` |

### Other Notable Variables

- `VITE_APP_LIBRARY_URL` — Shape library CDN
- `VITE_APP_AI_BACKEND` — AI service endpoint
- `VITE_APP_FIREBASE_CONFIG` — Firebase credentials
- `VITE_APP_PLUS_LP` / `VITE_APP_PLUS_APP` — Excalidraw+ URLs
- `VITE_APP_DEBUG_*` — Debug flags

## Common Tasks

### Adding a New Element Type

1. Define the type in `packages/element/types.ts`
2. Add creation logic in `packages/element/newElement.ts`
3. Implement rendering in `packages/excalidraw/renderer/`
4. Add the toolbar action in `packages/excalidraw/actions/`
5. Update restore/migration logic in `packages/excalidraw/data/restore.ts`

### Adding a New Action

1. Create a file in `packages/excalidraw/actions/`
2. Define the action using the `Action` type (name, perform, keyTest, etc.)
3. Register it in the action manager

### Adding a Translation

1. Add the key to `packages/excalidraw/locales/en.json`
2. Use `t()` or the `useI18n()` hook in components
3. Translations for other languages are community-contributed

### Building Just the Library

```bash
yarn build:packages
```

Output goes to each package's `dist/` directory.

### Using the Library in Another Project

```bash
npm install @excalidraw/excalidraw
```

```tsx
import { Excalidraw } from "@excalidraw/excalidraw";

function App() {
  return <Excalidraw onChange={(elements, state) => console.log(elements)} />;
}
```

See `examples/` for Next.js and browser-script integrations.

## Codebase Token Evaluation

Understanding the size of each part of the codebase in tokens helps plan LLM-assisted development, code reviews, and context window budgeting.

> **Methodology:** Source files (`*.ts`, `*.tsx`, `*.js`, `*.jsx`, `*.css`, `*.scss`, `*.json`) measured by byte count. Token estimate uses the standard approximation of **~4 bytes per token**. Excludes `node_modules/`, `dist/`, and `.git/`.

### Token Size by Package

```mermaid
pie title Codebase Token Distribution (~2.4M total)
    "packages/excalidraw (core)" : 1288927
    "packages/excalidraw/locales" : 432892
    "packages/element" : 314705
    "Root config / lock files" : 188482
    "excalidraw-app" : 51227
    "packages/common" : 31606
    "packages/math" : 16276
    "packages/utils" : 12498
```

### Detailed Breakdown

| Directory | Bytes | Lines | Est. Tokens | % of Total |
| --- | ---: | ---: | ---: | ---: |
| `packages/excalidraw/` (excl. locales) | 5,155,707 | 113,613 | ~1,288,927 | 54.2% |
| `packages/excalidraw/locales/` | 1,731,569 | 40,834 | ~432,892 | 18.2% |
| `packages/element/` | 1,258,820 | 44,102 | ~314,705 | 13.2% |
| Root config files | 753,928 | 20,519 | ~188,482 | 7.9% |
| `excalidraw-app/` | 204,906 | 7,253 | ~51,227 | 2.2% |
| `packages/common/` | 126,425 | 4,660 | ~31,606 | 1.3% |
| `packages/math/` | 65,103 | 2,467 | ~16,276 | 0.7% |
| `packages/utils/` | 49,992 | 1,812 | ~12,498 | 0.5% |
| **Total** | **9,514,549** | **241,318** | **~2,378,637** | **100%** |

### Key Observations

- **Total project: ~2.4M tokens** — larger than most LLM context windows (even 1M). Full-project analysis requires chunking or selective loading.
- **Core library (`packages/excalidraw/`)** dominates at **~1.3M tokens (54%)** — this is where most development happens.
- **Locales: ~433K tokens (18%)** — translation JSONs that can safely be excluded for code analysis tasks.
- **Element package: ~315K tokens (13%)** — the second-largest code package; critical for geometry/rendering work.
- **Practical core** (excluding locales and lock files): **~1.7M tokens** — the actual codebase a developer interacts with.
- **Small packages** (`math`, `common`, `utils`): **~60K tokens combined** — easily fit in a single LLM context window.
- **App shell** (`excalidraw-app`): **~51K tokens** — lightweight wrapper around the core library.

### Context Window Planning

| LLM Context Size | What Fits |
| --- | --- |
| 8K tokens | A single file or focused diff |
| 32K tokens | `packages/math` + `packages/utils` + `packages/common` |
| 128K tokens | `packages/element/` or a large subsection of `packages/excalidraw/` |
| 200K tokens | `excalidraw-app/` + all small packages |
| 1M tokens | Most of the core source (excluding locales) |
| 2M+ tokens | Full project including locales and configs |

---

## Troubleshooting

| Issue | Solution |
|---|---|
| Dependencies won't install | Run `yarn clean-install` to wipe and reinstall |
| Type errors after pulling | Run `yarn test:typecheck` to identify issues |
| Snapshot tests failing | Run `yarn test:update` to regenerate snapshots |
| Port 3001 in use | Set `VITE_APP_PORT` in `.env.development` |
| Canvas tests failing locally | Ensure `vitest-canvas-mock` is set up — check `setupTests.ts` |
| Path alias not resolving | Verify `tsconfig.json` and `vitest.config.mts` are in sync |

---

## Further Reading

- [Excalidraw Developer Docs](https://docs.excalidraw.com)
- [Contributing Guide](https://docs.excalidraw.com/docs/introduction/contributing)
- `dev-docs/` directory in the repository
- `CLAUDE.md` at the project root for AI-assisted development notes
