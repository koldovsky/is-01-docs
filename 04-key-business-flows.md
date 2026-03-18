# Key Business Flows

This document covers the most important data flows in Excalidraw. Understanding these will help you reason about how user interactions translate to state changes, persistence, and collaboration.

---

## 1. Drawing an Element

When a user clicks and drags on the canvas to draw a shape:

```mermaid
sequenceDiagram
    participant User
    participant App as App.tsx
    participant NE as newElement.ts
    participant NC as NewElementCanvas
    participant Scene as Scene
    participant SC as StaticCanvas

    User->>App: pointerDown (canvas)
    App->>App: Determine active tool (e.g., "rectangle")
    App->>NE: newElement({type, x, y, ...styleProps})
    NE-->>App: ExcalidrawElement (initial)
    App->>NC: Render in-progress element

    loop While dragging
        User->>App: pointerMove
        App->>App: Update element dimensions (width, height)
        App->>NC: Re-render preview
    end

    User->>App: pointerUp
    App->>Scene: Commit element to elements array
    Scene->>SC: Re-render static canvas
    App->>App: captureUpdate → push to History
```

**Key files:**
- `packages/excalidraw/components/App.tsx` — pointer event handlers
- `packages/element/src/newElement.ts` — element factory functions
- `packages/excalidraw/components/canvases/NewElementCanvas.tsx` — preview rendering
- `packages/element/src/Scene.ts` — scene state management

---

## 2. App Bootstrap & Scene Loading

When the app first loads:

```mermaid
sequenceDiagram
    participant Browser
    participant Index as index.tsx
    participant App as ExcalidrawWrapper
    participant Init as initializeScene()
    participant LS as localStorage
    participant Backend as Backend API
    participant Collab as Collab

    Browser->>Index: Load page
    Index->>App: Render ExcalidrawApp
    App->>Init: useEffect → initializeScene()

    alt URL has #room=...
        Init->>Collab: startCollaboration(roomLinkData)
    else URL has #json=...
        Init->>Backend: importFromBackend(id, key)
        Backend-->>Init: Decrypted scene data
    else URL has #url=...
        Init->>Backend: fetch(url) → loadFromBlob()
    else No URL params
        Init->>LS: importFromLocalStorage()
        LS-->>Init: Saved scene + appState
    end

    Init-->>App: initialData
    App->>App: Render Excalidraw with initialData
```

**Key files:**
- `excalidraw-app/App.tsx` — `initializeScene()` function (lines ~215–370)
- `excalidraw-app/data/index.ts` — `importFromBackend()`
- `excalidraw-app/data/localStorage.ts` — `importFromLocalStorage()`
- `packages/excalidraw/data/blob.ts` — `loadFromBlob()`

---

## 3. Real-Time Collaboration

When a user starts or joins a collaborative session:

```mermaid
sequenceDiagram
    participant User as User A
    participant App as App.tsx
    participant Collab as Collab.tsx
    participant Portal as Portal (Socket.io)
    participant WS as WebSocket Server
    participant Firebase as Firebase
    participant UserB as User B

    User->>App: Click "Live Collaboration"
    App->>Collab: startCollaboration()
    Collab->>Collab: generateRoomId() + encryptionKey
    Collab->>Portal: open(socket, roomId)
    Portal->>WS: Connect + join room
    Collab->>Firebase: loadFromFirebase(roomId)
    Firebase-->>Collab: Existing scene (if any)

    User->>App: Draw / modify elements
    App->>Collab: onChange → syncElements()
    Collab->>Collab: reconcileElements(local, remote)
    Collab->>Portal: broadcastScene(elements)
    Portal->>WS: Emit encrypted scene data
    WS->>UserB: Forward to other clients

    UserB->>WS: Send their changes
    WS->>Portal: Receive remote update
    Portal->>Collab: handleRemoteSceneUpdate()
    Collab->>Collab: reconcileElements()
    Collab->>App: updateScene(reconciledElements)

    par Background save
        Collab->>Firebase: queueSaveToFirebase()
    end
```

**Key concepts:**
- **Room ID + encryption key** are encoded in the URL hash (never sent to server)
- **Reconciliation** merges local and remote elements by comparing `version` numbers — the higher version wins
- **Cursor sync** is a separate channel: `broadcastMouseLocation()` sends pointer position + username
- **Firebase** acts as durable storage; Socket.io is for real-time relay only

**Key files:**
- `excalidraw-app/collab/Collab.tsx` — orchestrates the collaboration lifecycle
- `excalidraw-app/collab/Portal.tsx` — Socket.io wrapper
- `packages/excalidraw/data/reconcile.ts` — `reconcileElements()` merge algorithm
- `packages/excalidraw/data/encryption.ts` — end-to-end encryption

---

## 4. Save & Load

### Local Persistence (Auto-save)

Every change triggers a debounced save to `localStorage` + `IndexedDB`:

```mermaid
sequenceDiagram
    participant App as App.tsx
    participant LD as LocalData
    participant LS as localStorage
    participant IDB as IndexedDB

    App->>LD: onChange → LocalData.save(elements, appState)
    LD->>LS: Save appState (JSON)
    LD->>IDB: Save elements + files (binary data)
```

### Export to File

```mermaid
sequenceDiagram
    participant User
    participant Action as actionSaveToActiveFile
    participant JSON as json.ts
    participant FS as filesystem.ts

    User->>Action: Ctrl+S
    Action->>JSON: serializeAsJSON(elements, appState, files)
    JSON-->>Action: JSON string
    Action->>FS: fileSave(blob, filename)
    FS->>FS: File System Access API or download fallback
```

### Share via Link

```mermaid
sequenceDiagram
    participant User
    participant Export as exportToBackend()
    participant Backend as Backend V2 API
    participant Firebase as Firebase Storage

    User->>Export: Click "Share" → export link
    Export->>Export: encryptData(elements + appState)
    Export->>Backend: POST encrypted data
    Backend-->>Export: {id}
    Export->>Firebase: saveFilesToFirebase(files)
    Export-->>User: URL with #json=id,encryptionKey
```

**Key files:**
- `excalidraw-app/data/LocalData.ts` — auto-save logic
- `packages/excalidraw/data/json.ts` — `serializeAsJSON()`, `saveAsJSON()`, `loadFromJSON()`
- `packages/excalidraw/data/filesystem.ts` — File System Access API wrapper
- `excalidraw-app/data/index.ts` — `exportToBackend()`, `importFromBackend()`

---

## 5. Export to Image (PNG/SVG)

```mermaid
sequenceDiagram
    participant User
    participant Action as exportCanvas()
    participant Scene as scene/export.ts
    participant Renderer as renderStaticScene()
    participant Clipboard as Clipboard API

    User->>Action: Export as PNG / Copy as PNG
    Action->>Scene: exportToCanvas(elements, appState)
    Scene->>Renderer: renderStaticScene() on offscreen canvas
    Renderer-->>Scene: Canvas with rendered elements

    alt Export to file
        Scene->>Scene: canvas.toBlob("image/png")
        Scene-->>User: Download PNG file
    else Copy to clipboard
        Scene->>Clipboard: copyBlobToClipboardAsPng(blob)
    else Export as SVG
        Scene->>Scene: renderSceneToSvg()
        Scene-->>User: SVG string / file
    end
```

**Key files:**
- `packages/excalidraw/scene/export.ts` — `exportToCanvas()`, `exportToSvg()`
- `packages/excalidraw/renderer/staticScene.ts` — `renderStaticScene()`
- `packages/excalidraw/renderer/staticSvgScene.ts` — `renderSceneToSvg()`
- `packages/excalidraw/data/image.ts` — PNG metadata embedding (scene data inside PNG)

---

## 6. Undo/Redo

```mermaid
sequenceDiagram
    participant User
    participant App as App.tsx
    participant History as History
    participant Delta as HistoryDelta

    User->>App: Make a change (draw, move, delete)
    App->>History: record(delta)
    History->>History: Push inverse delta to undoStack

    User->>App: Ctrl+Z (Undo)
    App->>History: undo()
    History->>Delta: undoStack.pop()
    Delta->>Delta: applyTo(currentState)
    Delta-->>History: New state + inverse delta
    History->>History: Push inverse to redoStack
    History-->>App: Updated elements + appState
```

**Key concepts:**
- History uses **deltas**, not full snapshots — this is memory-efficient
- Each `HistoryDelta` extends `StoreDelta` from `@excalidraw/element`
- `captureUpdate` in `ActionResult` controls whether the action is recorded in history

**Key files:**
- `packages/excalidraw/history.ts` — `History` class with `undoStack`/`redoStack`
- `packages/excalidraw/actions/actionHistory.tsx` — undo/redo action definitions

---

## 7. Clipboard (Copy/Paste)

```mermaid
sequenceDiagram
    participant User
    participant Action as actionCopy/actionPaste
    participant Clipboard as clipboard.ts
    participant System as System Clipboard

    User->>Action: Ctrl+C
    Action->>Clipboard: copyToClipboard(elements)
    Clipboard->>Clipboard: serializeAsClipboardJSON(elements)
    Clipboard->>System: navigator.clipboard.writeText(json)

    User->>Action: Ctrl+V
    Action->>Clipboard: parseClipboard(event)
    Clipboard->>System: navigator.clipboard.read()
    System-->>Clipboard: ClipboardItems

    alt Excalidraw JSON detected
        Clipboard-->>Action: Parsed elements
        Action->>Action: Insert elements at cursor
    else Plain text
        Clipboard-->>Action: Create text element
    else Image
        Clipboard-->>Action: Create image element
    end
```

**Key files:**
- `packages/excalidraw/clipboard.ts` — `copyToClipboard()`, `parseClipboard()`
- `packages/excalidraw/actions/actionClipboard.tsx` — copy/paste/cut actions

---

## 8. Library (Reusable Components)

```mermaid
sequenceDiagram
    participant User
    participant LibMenu as LibraryMenu
    participant Library as library.ts
    participant IDB as IndexedDB
    participant Scene as Scene

    User->>LibMenu: Open library panel
    LibMenu->>Library: Load libraryItemsAtom
    Library->>IDB: LibraryPersistenceAdapter.load()
    IDB-->>Library: LibraryItem[]
    Library-->>LibMenu: Render items

    User->>LibMenu: Click item to insert
    LibMenu->>Scene: onInsertLibraryItems(elements)
    Scene->>Scene: Add elements to canvas

    User->>LibMenu: Select elements → "Add to Library"
    LibMenu->>Library: Save new LibraryItem
    Library->>IDB: LibraryPersistenceAdapter.save()
```

**Key files:**
- `packages/excalidraw/components/LibraryMenu.tsx` — UI
- `packages/excalidraw/data/library.ts` — persistence adapter and Jotai atom
