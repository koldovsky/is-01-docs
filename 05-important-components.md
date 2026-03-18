# Important Components

This document covers the most critical classes, services, and modules in the codebase. Understanding these is essential for making safe, effective changes.

---

## Core Components

### `App.tsx` — The Editor Core

**Path:** `packages/excalidraw/components/App.tsx`

This is the largest and most critical file in the project. It is the main React component for the Excalidraw editor.

**Responsibilities:**
- Canvas rendering orchestration (static + interactive + new element canvases)
- All pointer event handling (click, drag, resize, rotate, pan, zoom)
- All keyboard event handling (shortcuts, text input)
- Tool state machine (selection, drawing, eraser, laser pointer, etc.)
- Drag-and-drop handling (files, library items)
- Gesture handling (pinch-to-zoom on mobile)
- Collaboration state (remote cursors, selections)

**Side effects:**
- Registers global event listeners (`window.addEventListener`)
- Interacts with clipboard API
- Triggers history recording via `captureUpdate`

**When you'll touch it:** Adding new tools, changing pointer behavior, modifying canvas interactions.

> **Warning:** Changes to `App.tsx` can have wide-reaching effects. Test thoroughly with different tools and input devices.

---

### `ActionManager` — Action Dispatch System

**Path:** `packages/excalidraw/actions/manager.tsx`

Manages the registry of all user actions and dispatches them.

**Responsibilities:**
- Registers action objects at startup
- Routes keyboard events to matching actions via `keyTest`
- Executes `action.perform()` and feeds `ActionResult` to the state updater
- Provides `renderAction()` for toolbar/panel rendering

**How it works:**
```
User presses Ctrl+D
  → ActionManager.handleKeyDown(event)
    → Finds actionDuplicateSelection (keyTest matches)
      → action.perform(elements, appState, formData, app)
        → Returns ActionResult {elements, appState, captureUpdate}
          → updater(result) → state update → re-render
```

**When you'll touch it:** Adding new keyboard shortcuts or action infrastructure.

---

### `Scene` — Element State Container

**Path:** `packages/element/src/Scene.ts`

Holds the array of all elements and manages their ordering.

**Responsibilities:**
- Stores elements with `getNonDeletedElements()` for rendering
- Manages **fractional indexing** for element ordering (`syncMovedIndices`, `syncInvalidIndices`)
- Provides element lookup by ID
- Throttled validation of element indices

**Key concept — Fractional Indexing:**
Elements have an `index` field (a fractional string like `"a0"`, `"a1"`, `"a0V"`) that determines their z-order. This allows inserting elements between others without re-indexing the entire array — critical for collaboration where multiple users may reorder simultaneously.

---

### `History` — Undo/Redo Engine

**Path:** `packages/excalidraw/history.ts`

Manages undo and redo stacks using delta-based state tracking.

**Responsibilities:**
- Records state deltas when actions are performed
- Applies inverse deltas for undo, forward deltas for redo
- Manages stack size and memory

**Key detail:** Uses `HistoryDelta` (extends `StoreDelta` from `@excalidraw/element`) — these are diffs, not full snapshots. Each delta knows how to apply itself to current state and produce an inverse.

**Risk:** If a delta is applied to state it wasn't computed against (e.g., after a collaboration merge), results may be unexpected. The reconciliation logic handles this, but be aware when modifying either system.

---

## Data Layer Components

### `reconcileElements()` — Collaboration Merge

**Path:** `packages/excalidraw/data/reconcile.ts`

The critical function that merges local and remote element arrays during collaboration.

**Algorithm:**
1. Iterate through both local and remote element arrays
2. For each element ID, compare `version` numbers
3. Higher version wins; ties go to remote (server authority)
4. Deleted elements (`isDeleted: true`) are preserved for conflict resolution
5. Returns merged array maintaining correct ordering

**Risk:** Bugs here cause data loss or duplication in collaborative sessions. Any changes must be tested with concurrent editing scenarios.

---

### `restore.ts` — Data Migration

**Path:** `packages/excalidraw/data/restore.ts`

Restores and migrates saved data to the current format.

**Responsibilities:**
- `restoreElements()` — normalizes element data, applies defaults, migrates old formats
- `restoreAppState()` — normalizes app state, applies defaults
- `restoreLibraryItems()` — normalizes library data
- `bumpElementVersions()` — increments versions after restoration

**When it matters:** When the element schema changes (new properties, renamed fields), migration logic goes here. This ensures old saved files continue to work.

---

### Encryption Module

**Path:** `packages/excalidraw/data/encryption.ts`

Handles end-to-end encryption for shared scenes and collaboration.

**Functions:**
- `generateEncryptionKey()` — creates an AES-GCM key
- `encryptData(key, data)` — encrypts scene data
- `decryptData(key, iv, ciphertext)` — decrypts scene data

**Key design:** The encryption key is stored in the URL **hash** (fragment), which browsers do not send to servers. This means the server storing the encrypted data cannot read it.

---

## Collaboration Components

### `Collab.tsx` — Collaboration Orchestrator

**Path:** `excalidraw-app/collab/Collab.tsx`

Manages the full lifecycle of a collaborative session.

**Responsibilities:**
- Start/stop collaboration sessions
- Sync elements between local state and remote participants
- Manage file (image) uploads to Firebase Storage
- Handle connection/disconnection events
- Queue saves to Firebase Firestore

**Key methods:**
- `startCollaboration()` — generates room, connects socket, loads existing data
- `syncElements()` — called on every local change, broadcasts to peers
- `handleRemoteSceneUpdate()` — receives remote changes, reconciles, updates local state
- `fetchImageFilesFromFirebase()` — lazy-loads images that other participants added

---

### `Portal.tsx` — WebSocket Transport

**Path:** `excalidraw-app/collab/Portal.tsx`

Low-level Socket.io wrapper for real-time communication.

**Responsibilities:**
- Establish/close WebSocket connections
- Broadcast scene data (`broadcastScene`)
- Broadcast mouse positions (`broadcastMouseLocation`)
- Broadcast visible scene bounds (`broadcastVisibleSceneBounds`)

**Protocol subtypes:** `INIT`, `UPDATE`, `MOUSE_LOCATION`, `USER_VISIBLE_SCENE_BOUNDS`, `IDLE_STATUS`

---

## Firebase Integration

**Path:** `excalidraw-app/data/firebase.ts`

Firebase is used for durable storage in collaborative sessions and for shared links.

**Functions:**
- `loadFromFirebase(roomId)` — reads encrypted scene from Firestore `scenes` collection
- `saveToFirebase(roomId, elements)` — writes encrypted scene to Firestore
- `loadFilesFromFirebase(prefix, ids)` — reads image files from Firebase Storage
- `saveFilesToFirebase(prefix, files)` — writes image files to Firebase Storage

**Configuration:** Loaded from `VITE_APP_FIREBASE_CONFIG` env var (JSON string parsed at runtime).

---

## Rendering Components

### `renderStaticScene()`

**Path:** `packages/excalidraw/renderer/staticScene.ts`

Renders all committed elements onto the static canvas.

**How it works:**
1. Clears canvas
2. Applies zoom and scroll transforms
3. Iterates visible elements
4. For each element, generates Rough.js drawable (cached) and renders
5. Handles special cases: images, text, frames, embeds

### `renderInteractiveScene()`

**Path:** `packages/excalidraw/renderer/interactiveScene.ts`

Renders the overlay layer with selection UX.

**What it renders:**
- Selection rectangles and handles
- Rotation handles
- Binding indicators (arrow snap points)
- Remote user cursors and selections
- Snap guidelines
- Lasso selection path

---

## Background Services

### `LocalData` — Auto-Persistence

**Path:** `excalidraw-app/data/LocalData.ts`

Runs debounced saves to `localStorage` (for `appState`) and `IndexedDB` (for elements and binary files).

**Trigger:** Called from `App.tsx`'s `onChange` callback on every state change.

### `FileManager` — Binary File Management

**Path:** `excalidraw-app/data/FileManager.ts`

Manages image file uploads, downloads, and caching for collaboration.

### `tabSync` — Cross-Tab Sync

**Path:** `excalidraw-app/data/tabSync.ts`

Synchronizes state across multiple browser tabs using `BroadcastChannel` or `localStorage` events.

---

## Library System

### `library.ts`

**Path:** `packages/excalidraw/data/library.ts`

Manages the reusable element library (templates/components users can save and reuse).

**Architecture:**
- `LibraryPersistenceAdapter` — interface for storage backends (IndexedDB by default)
- `libraryItemsAtom` — Jotai atom holding the library state
- Items are arrays of elements with metadata (`id`, `name`, `status`, `created`)

### Font System

**Path:** `packages/excalidraw/fonts/`

Manages loading and subsetting of fonts (Virgil, Excalifont, Nunito, etc.).

- `ExcalidrawFontFace.ts` — font face loading abstraction
- `index.ts` — font registration and loading orchestration
- `fonts.css` — `@font-face` declarations
- Font subsetting in `subset/` for optimized export
