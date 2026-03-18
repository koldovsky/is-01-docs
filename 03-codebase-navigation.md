# Codebase Navigation Guide

## Top-Level Structure

```
excalidraw/
├── excalidraw-app/          # Web application (excalidraw.com)
├── packages/
│   ├── excalidraw/          # Main React editor component (@excalidraw/excalidraw)
│   ├── element/             # Element types and manipulation (@excalidraw/element)
│   ├── common/              # Shared utilities (@excalidraw/common)
│   ├── math/                # Geometry primitives (@excalidraw/math)
│   └── utils/               # Export and bounds helpers (@excalidraw/utils)
├── examples/
│   ├── with-script-in-browser/  # Vite integration example
│   └── with-nextjs/             # Next.js integration example
├── dev-docs/                # Docusaurus documentation site
├── firebase-project/        # Firebase configuration
├── public/                  # Static assets (favicon, service worker)
├── scripts/                 # Build and release scripts
├── .github/                 # CI/CD workflows
├── .env.development         # Dev environment variables
├── .env.production          # Prod environment variables
├── vitest.config.mts        # Test configuration
├── tsconfig.json            # Root TypeScript config with path aliases
└── package.json             # Root workspace config
```

---

## Where to Find Things

### Business Logic

| What | Where |
|------|-------|
| Element types and definitions | `packages/element/src/types.ts` |
| Element creation | `packages/element/src/newElement.ts` |
| Element mutation | `packages/element/src/mutateElement.ts` |
| Element transforms (resize, rotate) | `packages/element/src/transform.ts` |
| Arrow binding logic | `packages/element/src/binding.ts` |
| Text wrapping and sizing | `packages/element/src/textElement.ts` |
| Grouping | `packages/element/src/groups.ts` |
| Frame containment | `packages/element/src/frame.ts` |
| Z-index ordering | `packages/element/src/zindex.ts` |
| Collision detection | `packages/element/src/collision.ts` |
| Scene state + fractional indexing | `packages/element/src/Scene.ts` |

### User Actions

| What | Where |
|------|-------|
| Action type definitions | `packages/excalidraw/actions/types.ts` |
| Action manager (registration, dispatch) | `packages/excalidraw/actions/manager.tsx` |
| All action implementations | `packages/excalidraw/actions/action*.ts(x)` |
| Keyboard shortcuts | `packages/excalidraw/actions/shortcuts.ts` |

Key actions: `actionDeleteSelected`, `actionDuplicateSelection`, `actionCopy`, `actionPaste`, `actionGroup`, `actionChangeStrokeColor`, `actionChangeFontSize`, `actionZoomIn`, `actionToggleTheme`, `actionSaveToActiveFile`, `actionLoadScene`.

### Data Layer (Serialization, Persistence)

| What | Where |
|------|-------|
| JSON serialization/deserialization | `packages/excalidraw/data/json.ts` |
| Binary file loading (PNG/SVG/JSON) | `packages/excalidraw/data/blob.ts` |
| Data compression/encoding | `packages/excalidraw/data/encode.ts` |
| Encryption for collaboration | `packages/excalidraw/data/encryption.ts` |
| Element reconciliation (collab merge) | `packages/excalidraw/data/reconcile.ts` |
| State restoration and migration | `packages/excalidraw/data/restore.ts` |
| Library persistence | `packages/excalidraw/data/library.ts` |
| PNG metadata embedding | `packages/excalidraw/data/image.ts` |
| File System Access API | `packages/excalidraw/data/filesystem.ts` |
| Exported data types | `packages/excalidraw/data/types.ts` |

### App-Level Data (excalidraw.com specific)

| What | Where |
|------|-------|
| Firebase integration | `excalidraw-app/data/firebase.ts` |
| Local storage persistence | `excalidraw-app/data/LocalData.ts` |
| File upload/download management | `excalidraw-app/data/FileManager.ts` |
| Cross-tab synchronization | `excalidraw-app/data/tabSync.ts` |
| Text-to-diagram storage | `excalidraw-app/data/TTDStorage.ts` |

### UI Components

| What | Where |
|------|-------|
| Main editor (canvas, events, state) | `packages/excalidraw/components/App.tsx` |
| Layer controls | `packages/excalidraw/components/LayerUI.tsx` |
| Static canvas rendering | `packages/excalidraw/components/canvases/StaticCanvas.tsx` |
| Interactive canvas (selection, handles) | `packages/excalidraw/components/canvases/InteractiveCanvas.tsx` |
| Main menu | `packages/excalidraw/components/main-menu/MainMenu.tsx` |
| Sidebar | `packages/excalidraw/components/Sidebar/Sidebar.tsx` |
| Color picker | `packages/excalidraw/components/ColorPicker/` |
| Font picker | `packages/excalidraw/components/FontPicker/` |
| Library panel | `packages/excalidraw/components/LibraryMenu.tsx` |
| Command palette | `packages/excalidraw/components/CommandPalette/CommandPalette.tsx` |
| Welcome screen | `packages/excalidraw/components/welcome-screen/` |
| Text-to-diagram dialog | `packages/excalidraw/components/TTDDialog/TTDDialog.tsx` |
| Element stats inspector | `packages/excalidraw/components/Stats/` |

### Rendering

| What | Where |
|------|-------|
| Static scene rendering | `packages/excalidraw/renderer/staticScene.ts` |
| Interactive scene rendering | `packages/excalidraw/renderer/interactiveScene.ts` |
| SVG export rendering | `packages/excalidraw/renderer/staticSvgScene.ts` |
| Scene export (PNG/SVG/clipboard) | `packages/excalidraw/scene/export.ts` |
| Render config types | `packages/excalidraw/scene/types.ts` |

### Collaboration

| What | Where |
|------|-------|
| Collaboration manager | `excalidraw-app/collab/Collab.tsx` |
| WebSocket portal | `excalidraw-app/collab/Portal.tsx` |
| Collab error handling | `excalidraw-app/collab/CollabError.tsx` |
| Live collaboration trigger UI | `packages/excalidraw/components/live-collaboration/` |

### State Management

| What | Where |
|------|-------|
| Default app state | `packages/excalidraw/appState.ts` |
| Editor Jotai provider | `packages/excalidraw/editor-jotai.ts` |
| App Jotai provider | `excalidraw-app/app-jotai.ts` |
| App state hooks | `packages/excalidraw/hooks/useAppStateValue.ts` |
| Core types (AppState, Props, etc.) | `packages/excalidraw/types.ts` |

---

## Naming Conventions

### Files

- **React components**: PascalCase (e.g., `LayerUI.tsx`, `ColorPicker.tsx`)
- **Utilities and modules**: camelCase (e.g., `mutateElement.ts`, `reconcile.ts`)
- **Action files**: `action` prefix (e.g., `actionDeleteSelected.ts`, `actionZoomToFit.ts`)
- **Test files**: `.test.ts` or `.test.tsx` suffix, either colocated or in a `tests/` directory
- **Style files**: `.scss` next to their component (e.g., `App.scss`, `LayerUI.scss`)
- **Type-only files**: `types.ts` for shared type definitions

### Code

- **Element types**: string literals (`"rectangle"`, `"arrow"`, `"text"`, `"freedraw"`)
- **Actions**: objects conforming to the `Action` interface with a unique `name` string
- **Jotai atoms**: suffixed with `Atom` (e.g., `collabAPIAtom`, `libraryItemsAtom`)
- **Constants**: UPPER_SNAKE_CASE in `@excalidraw/common` constants
- **TypeScript**: Strict mode, `consistent-type-imports` enforced via ESLint

### Imports

The project uses **path aliases** for cross-package imports:

```typescript
import { something } from "@excalidraw/common";
import { ExcalidrawElement } from "@excalidraw/element";
import { pointFrom } from "@excalidraw/math";
```

These are resolved via `tsconfig.json` paths and duplicated in `vitest.config.mts` for tests.

> **Important**: ESLint enforces that you do NOT import from barrel `index.ts` files within `@excalidraw/excalidraw` when you're inside that package. Import from the specific module instead.

---

## Key Entry Points

| Entry Point | File | Description |
|-------------|------|-------------|
| npm package | `packages/excalidraw/index.tsx` | Exports `Excalidraw` component and all public APIs |
| Web app | `excalidraw-app/index.tsx` → `App.tsx` | Bootstraps the full application |
| Browser example | `examples/with-script-in-browser/index.tsx` | Minimal integration example |
| Next.js example | `examples/with-nextjs/src/app/page.tsx` | SSR integration example |
