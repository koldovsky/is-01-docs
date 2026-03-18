# TL;DR for New Developers

Your quick-start reference. Bookmark this.

---

## Day 1 Checklist

- [ ] Clone the repo: `git clone https://github.com/excalidraw/excalidraw.git`
- [ ] Ensure Node.js >= 18 and Yarn Classic are installed
- [ ] Run `yarn` to install all dependencies
- [ ] Run `yarn start` and open http://localhost:3001
- [ ] Draw a few shapes, export a PNG, try undo/redo — confirm everything works
- [ ] Run `yarn test:app --watch=false` to verify tests pass
- [ ] Run `yarn test:typecheck` to verify TypeScript compiles
- [ ] Read this TL;DR page fully

---

## Key Commands

| Command | What it does |
|---------|-------------|
| `yarn start` | Start dev server (port 3001) |
| `yarn test:update` | Run tests + update snapshots (**always run before committing**) |
| `yarn test:typecheck` | TypeScript type check |
| `yarn fix` | Auto-fix formatting and linting |
| `yarn test:all` | Full CI-equivalent check |
| `yarn build` | Build the web app |
| `yarn build:packages` | Build only the npm packages |

---

## Files to Read First

In order of priority:

1. **`packages/excalidraw/index.tsx`** — main entry point, see what's exported
2. **`packages/element/src/types.ts`** — element type definitions (the data model)
3. **`packages/excalidraw/types.ts`** — `AppState` and component props
4. **`packages/excalidraw/appState.ts`** — default app state (all the knobs)
5. **`packages/excalidraw/actions/types.ts`** — how the action system works
6. **`packages/excalidraw/actions/manager.tsx`** — action dispatch logic
7. **`excalidraw-app/App.tsx`** — app bootstrap and scene loading
8. **`packages/excalidraw/data/reconcile.ts`** — collaboration merge logic
9. **`setupTests.ts`** — test environment setup

---

## The Mental Model

```
User draws a shape
  → App.tsx handles pointer events
    → Creates a new element via newElement()
      → ActionManager processes it
        → Returns ActionResult {elements, appState, captureUpdate}
          → State updates → Canvas re-renders
            → History records the delta
              → LocalData auto-saves
                → If collaborating: sync to peers via Socket.io + Firebase
```

---

## Project Structure in 30 Seconds

```
excalidraw/
├── excalidraw-app/     ← The website (excalidraw.com)
│   ├── collab/         ← Real-time collaboration
│   └── data/           ← Firebase, localStorage, file management
├── packages/
│   ├── excalidraw/     ← The npm library (editor component)
│   │   ├── actions/    ← User actions (copy, paste, delete, etc.)
│   │   ├── components/ ← React UI components
│   │   ├── renderer/   ← Canvas rendering
│   │   ├── data/       ← Serialization, encryption, import/export
│   │   └── scene/      ← Export helpers, render configs
│   ├── element/        ← Element types, mutation, transforms
│   ├── common/         ← Shared constants and utilities
│   ├── math/           ← Geometry (points, vectors, curves)
│   └── utils/          ← Export and bounds helpers
└── examples/           ← Integration examples (Next.js, Vite)
```

---

## The 5 Most Important Concepts

1. **Elements are immutable** — use `mutateElement()`, never modify properties directly
2. **Actions are the state change API** — all user operations go through `ActionManager`
3. **Jotai for reactive state** — atoms from `editor-jotai.ts`, not raw `jotai`
4. **Multi-canvas rendering** — static (cached), interactive (selection), new element (preview)
5. **Collaboration via reconciliation** — `reconcileElements()` merges by version numbers

---

## Before Your First PR

```bash
yarn test:update     # Tests + snapshot updates
yarn test:typecheck  # TypeScript
yarn fix             # Formatting + linting
```

All three must pass. CI will check them.

---

## When You're Stuck

| Problem | Where to look |
|---------|---------------|
| "How does X tool work?" | `packages/excalidraw/components/App.tsx` — pointer handlers |
| "How is Y element created?" | `packages/element/src/newElement.ts` |
| "How does Z action work?" | `packages/excalidraw/actions/actionZ.ts` |
| "Why is my element not rendering?" | `packages/excalidraw/renderer/staticScene.ts` |
| "How does collab work?" | `excalidraw-app/collab/Collab.tsx` |
| "Where is state defined?" | `packages/excalidraw/appState.ts` + `types.ts` |
| "How do tests work?" | `setupTests.ts` + `packages/excalidraw/tests/helpers/` |
| "How is data saved?" | `packages/excalidraw/data/json.ts` + `excalidraw-app/data/LocalData.ts` |

---

## Quick Links

- [Excalidraw Docs](https://docs.excalidraw.com/) — official documentation
- [Contributing Guide](https://docs.excalidraw.com/docs/introduction/contributing) — PR guidelines
- [Development Guide](https://docs.excalidraw.com/docs/introduction/development) — official setup guide
