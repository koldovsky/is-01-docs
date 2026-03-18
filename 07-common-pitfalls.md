# Common Pitfalls

Real risks and gotchas that can cause bugs, data loss, or production issues.

---

## 1. Element Immutability

**Risk:** Mutating elements directly instead of creating new versions.

Elements in Excalidraw are treated as **immutable**. Every modification must create a new object with an incremented `version` and new `versionNonce`.

```typescript
// WRONG — direct mutation
element.x = 100;

// CORRECT — use mutateElement or create a new element
mutateElement(element, { x: 100 });
```

**Why it matters:**
- The rendering pipeline uses reference equality to skip unchanged elements
- The collaboration reconciliation algorithm uses `version` to determine which copy wins
- History (undo/redo) relies on deltas computed from version changes

If you mutate an element without incrementing its version, collaboration will silently drop the change, undo won't work correctly, and the canvas may not re-render.

---

## 2. Collaboration Reconciliation Edge Cases

**Risk:** Data loss when local and remote changes conflict.

The `reconcileElements()` function merges element arrays by comparing versions. The higher version wins; **ties go to remote** (server authority). This means:

- If two users modify the same element in the same frame, one change will be lost
- Deleted elements (`isDeleted: true`) must be kept in the array for a period to prevent "resurrection" by stale remote updates
- Adding new properties to elements requires careful handling in `restoreElements()` to ensure old saved data gets sensible defaults

**What to watch for:**
- Never remove deleted elements from arrays during reconciliation
- Always test concurrent editing scenarios when changing element structure
- Be aware that `version` increments happen per-mutation, not per-action

---

## 3. Fractional Indexing Corruption

**Risk:** Ordering bugs or crashes from invalid fractional indices.

Elements use **fractional indexing** (the `index` field) for z-ordering. This avoids the need to re-number all elements when one is moved, which is critical for collaboration.

**Pitfalls:**
- Inserting an element without computing a valid fractional index between its neighbors
- Bulk operations that don't maintain monotonically increasing indices
- `Scene.ts` has validation (`syncMovedIndices`, `syncInvalidIndices`) but it's throttled — bugs may not surface immediately

**Safe approach:** Use the `Scene` methods for reordering; don't manually assign `index` values.

---

## 4. `App.tsx` Side Effects

**Risk:** Breaking seemingly unrelated features by changing `App.tsx`.

`App.tsx` is the largest file in the project and handles many cross-cutting concerns. Event handler changes can have unexpected effects:

- Changing `pointerDown` handling may break drawing, selection, AND resize
- Modifying keyboard handlers may break shortcuts, text editing, AND command palette
- Gesture handling code affects both desktop and mobile

**Approach:**
- Test on both desktop and mobile (or at least with touch event simulation)
- Test with multiple tools active (selection, rectangle, arrow, text, freedraw)
- Test with collaboration active (remote cursors, selections)

---

## 5. Canvas Rendering Performance

**Risk:** Rendering lag or jank from unnecessary re-renders.

The multi-canvas architecture is specifically designed to avoid re-rendering the static canvas on every pointer move. Breaking this optimization can make the editor feel sluggish.

**Don'ts:**
- Don't force static canvas re-render on pointer move events
- Don't add expensive computations inside render loops
- Don't create new Rough.js drawables on every render (they're cached per element version)

**Check:** If your change affects rendering, test with 500+ elements on the canvas.

---

## 6. Import Path Restrictions

**Risk:** Build failures or circular dependencies from wrong import paths.

ESLint enforces strict import rules:

- **Inside `@excalidraw/excalidraw`**: Do NOT import from the barrel `index.tsx` — import from the specific module
- **Jotai imports**: Must come from `@excalidraw/excalidraw/editor-jotai` or `excalidraw-app/app-jotai`, NOT directly from `jotai`
- **Cross-package imports**: Only go through the `@excalidraw/*` aliases, never relative paths to other packages

```typescript
// WRONG (inside packages/excalidraw/)
import { something } from "./index";
import { atom } from "jotai";

// CORRECT
import { something } from "./specific-module";
import { atom } from "../editor-jotai";
```

---

## 7. Encryption Key in URL Hash

**Risk:** Accidentally exposing encryption keys.

Shared scene encryption keys are stored in the URL **hash** (fragment). Browsers don't send the hash to servers, which is the security model. But:

- Logging the full URL (e.g., in error tracking) may leak the key
- Redirects to other domains may include the hash
- Browser extensions have access to the full URL

**Rule:** Never log or transmit the full URL including hash in analytics or error reporting.

---

## 8. State Update Ordering

**Risk:** Stale state bugs from async state updates.

The action system returns `ActionResult` objects that update state synchronously. But some operations (font loading, image processing, file I/O) are async. Mixing sync and async state updates can cause:

- Reading stale `appState` after an async operation
- Lost updates when two async operations complete out of order
- React re-render timing issues

**Approach:** Use `captureUpdate` in `ActionResult` to ensure history records the correct state boundary. For async operations, consider whether the state might have changed by the time the promise resolves.

---

## 9. Font Loading and Text Measurement

**Risk:** Incorrect text bounding boxes when fonts aren't loaded.

Text elements compute their dimensions based on font metrics. If a font hasn't loaded when `measureText()` runs, the measurements will be wrong, causing:

- Overlapping text
- Incorrect auto-sizing of text containers
- Wrong bounding boxes for selection/export

**The project handles this** via the `InitializeApp` component which loads fonts at startup and the font mock in tests. But if you add a new font or change text measurement logic, verify that measurements are correct after fonts load.

---

## 10. `isDeleted` Flag vs Array Removal

**Risk:** Breaking undo, collaboration, or references by removing elements from arrays.

Elements are **soft-deleted** by setting `isDeleted: true`. They are NOT removed from the elements array. This is by design:

- Undo needs deleted elements to restore them
- Collaboration needs deleted elements to prevent "ghost" resurrections
- Bound elements (arrows to shapes) reference IDs that may point to deleted elements

**Rule:** Use `isDeleted: true` for deletion. Use `getNonDeletedElements()` when you need the visible set. Never filter deleted elements out of the source-of-truth array.

---

## 11. Testing Pitfalls

- **Forgetting `await` with `render()`**: The test `render()` function is async. Forgetting to `await` it causes timing-related flaky tests.
- **Not cleaning up**: Always call `unmountComponent()` in `afterEach`. Leaking mounted components causes state pollution between tests.
- **Snapshot drift**: If you see unexpected snapshot changes, don't blindly update. Review the diffs — they may indicate a real regression.
- **Canvas mocking limitations**: `vitest-canvas-mock` doesn't implement all Canvas 2D methods. Some rendering logic cannot be tested in unit tests.
