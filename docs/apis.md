# Core APIs

## Imperative API

The mounted editor exposes the `ExcalidrawImperativeAPI` via `onExcalidrawAPI` or the `useExcalidrawAPI()` hook.

| Capability | Methods | Notes |
| --- | --- | --- |
| Canvas control | `updateScene`, `resetScene`, `refresh()` | Update elements/app state, force-rendering | 
| Element access | `getSceneElements()`, `getSceneElementsIncludingDeleted()` | Read current drawing state |
| Files & assets | `addFiles`, `getFiles` | Manage binary file map |
| History | `history.clear()`, undo/redo via actions | History service lives in `App` |
| Collaboration | `setCollaborators`, `setUsername` | Proxies to `CollabAPI` when enabled |

@excalidraw/excalidraw exports helpers like `exportToSvg`, `exportToClipboard`, and utilities under the same package.

## React Props (`<Excalidraw />`)

Key props defined in `packages/excalidraw/index.tsx`:

- **`initialData`**: seed elements/app state for fresh loads.
- **`onChange(elements, appState, files)`**: invoked on every mutation; used for persistence hooks.
- **`onPointerUpdate`**, **`onPointerDown`**, **`onPointerUp`**: listen to pointer events for custom tooling.
- **`viewModeEnabled`**, **`zenModeEnabled`**, **`gridModeEnabled`**: toggle major UI states.
- **`renderTopLeftUI` / `renderTopRightUI`**: extend the editor chrome.
- **`onExport`**, **`onLibraryChange`**: hook into export flows and library sync.

## App Shell Services (`excalidraw-app/App.tsx`)

- **Collaboration (`Collab`, `CollabAPI`)**: start/stop sessions, sync elements, fetch shared assets.@filepath references.
- **Persistence (`LocalData`, `FileStatusStore`)**: autosave to browser storage, track file health.
- **Theming (`useHandleAppTheme`)**: resolves system theme, persists preference.
- **Tab Sync (`tabSync.ts`)**: reconciles state across multiple browser tabs.

## Shared Packages

- **`@excalidraw/common`**: constants, localization, analytics helpers.
- **`@excalidraw/element`**: core element factories, mutators, and types.
- **`@excalidraw/math`**: geometry utilities consumed by bindings, snapping, and renderer.
