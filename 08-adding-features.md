# How to Add New Features

A practical guide to extending Excalidraw while following existing patterns.

---

## Adding a New Action

Actions are the standard way to add user-triggerable operations (keyboard shortcuts, toolbar buttons, menu items).

### Step 1: Create the action file

Create `packages/excalidraw/actions/actionMyFeature.ts`:

```typescript
import { register } from "./register";
import type { ActionResult } from "./types";

export const actionMyFeature = register({
  name: "myFeature",
  label: "My Feature",
  icon: MyFeatureIcon, // optional, for toolbar
  trackEvent: { category: "element" },

  // Keyboard shortcut (optional)
  keyTest: (event) =>
    event.key === "m" && event[KEYS.CTRL_OR_CMD],

  // Whether the action is available in current state
  predicate: (elements, appState) => {
    return appState.selectedElementIds.length > 0;
  },

  // The actual logic
  perform: (elements, appState, formData, app) => {
    // Your logic here
    const newElements = /* transform elements */;

    return {
      elements: newElements,
      appState: { ...appState /* state changes */ },
      captureUpdate: CaptureUpdateAction.IMMEDIATELY,
    };
  },

  // Panel component for the toolbar (optional)
  PanelComponent: ({ elements, appState, updateData }) => (
    <button onClick={() => updateData(null)}>My Feature</button>
  ),
});
```

### Step 2: Register the action

Add the export to `packages/excalidraw/actions/index.ts`:

```typescript
export { actionMyFeature } from "./actionMyFeature";
```

### Step 3: Write tests

Create `packages/excalidraw/actions/actionMyFeature.test.tsx`:

```typescript
import { render, unmountComponent } from "../tests/test-utils";
import { API } from "../tests/helpers/api";
import { Keyboard } from "../tests/helpers/ui";
import { Excalidraw } from "../index";

describe("actionMyFeature", () => {
  beforeEach(async () => {
    await render(<Excalidraw />);
  });
  afterEach(() => unmountComponent());

  it("should do the thing", () => {
    // Arrange
    API.setElements([API.createElement({ type: "rectangle" })]);

    // Act
    Keyboard.withModifierKeys({ ctrl: true }, () => {
      Keyboard.keyDown("m");
    });

    // Assert
    expect(API.getElements()).toSatisfyMyCondition();
  });
});
```

### Key points for actions:

- `captureUpdate` controls undo/redo: use `CaptureUpdateAction.IMMEDIATELY` for undoable actions
- Return `false` from `perform` to no-op (don't update state)
- `formData` comes from `PanelComponent`'s `updateData()` call
- `app` provides access to `AppClassProperties` for imperative operations

---

## Adding a New Element Property

When you need to add a new visual or behavioral property to elements.

### Step 1: Update the type

In `packages/element/src/types.ts`, add the property to the relevant element type or base type:

```typescript
type _ExcalidrawElementBase = {
  // ... existing properties
  myNewProp?: string;
};
```

### Step 2: Set the default

In `packages/element/src/newElement.ts`, include the default value:

```typescript
export const newElement = (opts) => ({
  // ... existing defaults
  myNewProp: opts.myNewProp ?? "default",
});
```

### Step 3: Add migration logic

In `packages/excalidraw/data/restore.ts`, handle old elements that don't have the property:

```typescript
// Inside restoreElement()
element.myNewProp = element.myNewProp ?? "default";
```

### Step 4: Wire up the UI

If the property is user-editable, create a panel component in `packages/excalidraw/components/` and register an action that changes it.

### Step 5: Update rendering (if visual)

If the property affects how elements look, update the rendering in:
- `packages/excalidraw/renderer/staticScene.ts` — for canvas rendering
- `packages/excalidraw/renderer/staticSvgScene.ts` — for SVG export

---

## Adding a New Tool

Tools determine what happens when the user interacts with the canvas.

### Step 1: Register the tool type

Add the tool type to `packages/excalidraw/types.ts` in the `ActiveTool` definition.

### Step 2: Add the toolbar button

Update `packages/excalidraw/components/Actions.tsx` or the relevant toolbar component.

### Step 3: Handle canvas interactions

The main event handlers are in `packages/excalidraw/components/App.tsx`:
- `handleCanvasPointerDown()` — start of interaction
- `handleCanvasPointerMove()` — during interaction
- `handleCanvasPointerUp()` — end of interaction

Add your tool's behavior in the appropriate switch/if blocks.

### Step 4: Add keyboard shortcut

Add the shortcut key in `packages/excalidraw/actions/shortcuts.ts`.

---

## Adding a New UI Component

### For library-level components (part of the npm package):

Place it in `packages/excalidraw/components/`:

```
packages/excalidraw/components/
├── MyComponent.tsx
└── MyComponent.scss (if needed)
```

Follow existing patterns:
- Use Jotai atoms from `editor-jotai.ts` for state
- Use `useAppStateValue()` to read app state
- Use SCSS modules for styling (or plain SCSS with BEM-like naming)
- Use `useDevice()` from `@excalidraw/common` for responsive behavior

### For app-level components (excalidraw.com only):

Place it in `excalidraw-app/components/`:

```
excalidraw-app/components/
├── MyAppComponent.tsx
└── MyAppComponent.scss
```

---

## Adding a New Package Utility

If adding a utility function to one of the core packages:

1. **Determine the right package:**
   - Pure math/geometry → `packages/math/`
   - Element manipulation → `packages/element/`
   - Shared utility (no element/UI deps) → `packages/common/`
   - Export/bounds helpers → `packages/utils/`

2. **Add to the appropriate source file** (or create a new module)

3. **Export from the package index** (`src/index.ts`)

4. **Respect the dependency graph:** `common` and `math` cannot import from other `@excalidraw` packages

---

## Patterns to Follow

### State updates via actions

All user-initiated state changes should go through the action system. Don't directly modify state from event handlers when an action would be more appropriate.

### Element immutability

Always use `mutateElement()` or create new element objects. Never modify element properties directly.

### Consistent type imports

Use `import type { ... }` for type-only imports (enforced by ESLint):

```typescript
import type { ExcalidrawElement } from "@excalidraw/element/types";
```

### Jotai atom scoping

Editor-level atoms go through `editor-jotai.ts`. App-level atoms through `app-jotai.ts`. Never import `jotai` directly.

---

## What to Avoid

- **Don't bypass the action system** for state changes that should be undoable
- **Don't add dependencies** to `packages/common/` or `packages/math/` on other `@excalidraw` packages
- **Don't import from barrel files** (`index.ts`) when inside the same package
- **Don't add large dependencies** without discussing — the bundle size is monitored (`size-limit.yml`)
- **Don't add browser-specific APIs** without fallbacks — the library runs in SSR contexts (Next.js)
- **Don't skip snapshot updates** — run `yarn test:update` and review diffs
- **Don't hardcode strings** — use the i18n system for user-facing text (see `packages/excalidraw/locales/`)

---

## Checklist for New Features

- [ ] Types defined (in appropriate `types.ts`)
- [ ] Default values set (in `newElement.ts` or `appState.ts`)
- [ ] Migration logic added (in `restore.ts`) if changing element schema
- [ ] Action created with `keyTest` if it has a shortcut
- [ ] UI wired up (toolbar, panel, menu)
- [ ] Rendering updated if it's visual
- [ ] Tests written (unit + integration)
- [ ] Snapshots updated (`yarn test:update`)
- [ ] Type checking passes (`yarn test:typecheck`)
- [ ] Linting passes (`yarn fix` then `yarn test:code`)
- [ ] Works on both desktop and mobile
- [ ] Works with collaboration (if it changes elements)
- [ ] Export/import handles the new data (PNG, SVG, JSON)
