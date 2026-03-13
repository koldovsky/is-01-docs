# Repository Structure

```plaintext
excalidraw/
├─ excalidraw-app/           # Production SPA (Vite) hosting the editor
│  ├─ App.tsx                # App shell orchestrating collaboration, theming, storage
│  ├─ components/            # App-specific UI (menus, sidebar, dialogs)
│  ├─ collab/                # Collaboration service and atoms
│  ├─ data/                  # Persistence helpers (LocalData, FileManager, Firebase)
│  └─ package.json           # Vite build/start scripts
├─ packages/
│  ├─ excalidraw/            # Core editor React package exported to npm
│  │  ├─ components/         # Canvas, panels, modals, registries
│  │  ├─ actions/            # Command/action definitions and manager
│  │  ├─ data/               # Element schemas, serialization, io helpers
│  │  ├─ editor-jotai.ts     # Jotai store powering hooks/contexts
│  │  ├─ appState.ts         # Default AppState definition
│  │  ├─ index.tsx           # Excalidraw component entrypoint
│  │  └─ types.ts            # Public API types
│  ├─ common/                # Shared constants, analytics, utilities
│  ├─ element/               # Element factories, mutators, geometry math
│  ├─ math/                  # Geometry helpers, coordinate transforms
│  └─ utils/                 # Cross-cutting helpers (promise, browser, clipboard)
├─ examples/                 # Integration samples (Next.js, script-in-browser, etc.)
├─ dev-docs/                 # Project documentation for contributors
├─ public/                   # Static assets for excalidraw-app
├─ scripts/                  # Build, release, and tooling scripts
├─ package.json              # Monorepo configuration (Yarn workspaces)
└─ tsconfig.json             # Shared TypeScript base config
```

## Workspace Boundaries

- **Monorepo** uses Yarn workspaces; each package has its own `package.json` but shares dependencies from the root.
- **packages/excalidraw** is published; others are internal libraries consumed by both the editor and app shell.
- **excalidraw-app** composes the editor and adds premium integrations (Firebase, Excalidraw+ entry points).
- **examples** demonstrate embedding scenarios and are valuable for onboarding.
