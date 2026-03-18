# Testing Guide

## Overview

Excalidraw uses **Vitest** with **jsdom** as the test environment and **@testing-library/react** for component testing. Tests run in a simulated browser environment with mocked Canvas API, IndexedDB, clipboard, and fonts.

---

## Running Tests

| Command | Purpose |
|---------|---------|
| `yarn test:app` | Run tests in watch mode |
| `yarn test:update` | Run all tests + update snapshots (use before committing) |
| `yarn test:all` | Full suite: typecheck + lint + prettier + tests |
| `yarn test:typecheck` | TypeScript type checking only |
| `yarn test:code` | ESLint only |
| `yarn test:other` | Prettier check only |
| `yarn test:coverage` | Run tests with coverage report |
| `yarn test:ui` | Vitest UI with coverage visualization |

### Running a specific test file

```bash
yarn test:app -- packages/excalidraw/tests/excalidraw.test.tsx
```

### Running tests matching a pattern

```bash
yarn test:app -- -t "should render toolbar"
```

---

## Test Structure

### Test file locations

Tests are found in two patterns:

1. **Dedicated `tests/` directories** — most common
   - `packages/excalidraw/tests/`
   - `packages/element/tests/`
   - `packages/math/tests/`
   - `excalidraw-app/tests/`

2. **Colocated with source** — for small utility modules
   - `packages/common/src/utils.test.ts`

### Test file naming

- `*.test.ts` — pure logic tests
- `*.test.tsx` — component/integration tests that render React

---

## Test Setup

### Global setup file: `setupTests.ts`

Located at the project root, this file runs before all tests and sets up:

| Mock | Purpose |
|------|---------|
| `vitest-canvas-mock` | Mocks HTML Canvas 2D API |
| `fake-indexeddb/auto` | Mocks IndexedDB |
| `throttleRAF` mock | Replaces `requestAnimationFrame` throttle with immediate execution |
| `setPointerCapture` | Mocked to no-op |
| `window.matchMedia` | Returns mock MediaQueryList |
| `FontFace` | Mocked constructor |
| `document.fonts` | Mocked FontFaceSet |
| Font fetch | Intercepts font file requests and returns empty responses |
| `#root` div | Creates root element for React rendering |

### Vitest configuration: `vitest.config.mts`

Key settings:
- **Path aliases**: Maps `@excalidraw/*` to source directories (not build output)
- **Environment**: jsdom
- **Globals**: `describe`, `it`, `expect`, `vi` available without imports
- **Coverage thresholds**: lines 60%, branches 70%, functions 63%, statements 60%

---

## Test Helpers

The project has a rich set of test utilities. Learn these before writing tests.

### `test-utils.ts`

**Path:** `packages/excalidraw/tests/test-utils.ts`

| Export | Purpose |
|--------|---------|
| `render(<Excalidraw />)` | Renders the editor in a test container, waits for initialization |
| `unmountComponent()` | Cleanly unmounts the rendered component |
| `GlobalTestState` | Shared state accessible across test helpers |
| `toggleMenu(name)` | Opens/closes menus by test ID |

### `helpers/api.ts`

**Path:** `packages/excalidraw/tests/helpers/api.ts`

The `API` object provides programmatic control of the editor in tests:

| Method | Purpose |
|--------|---------|
| `API.updateScene({elements, appState})` | Directly set scene state |
| `API.setAppState(patch)` | Update app state |
| `API.setElements(elements)` | Replace all elements |
| `API.createElement({type, ...})` | Create a test element |
| `API.getAppState()` | Read current app state |
| `API.getElements()` | Read current elements |

### `helpers/ui.ts`

**Path:** `packages/excalidraw/tests/helpers/ui.ts`

Simulates user interactions:

| Helper | Purpose |
|--------|---------|
| `UI.clickTool(toolName)` | Click a tool button in the toolbar |
| `Keyboard.keyDown(key)` | Simulate keyboard events |
| `Keyboard.withModifierKeys({ctrl: true}, () => {...})` | Simulate modifier combinations |
| `Pointer` / mouse helpers | Simulate pointer events at specific coordinates |

### `helpers/mocks.ts`

**Path:** `packages/excalidraw/tests/helpers/mocks.ts`

| Mock | Purpose |
|------|---------|
| `mockThrottleRAF()` | Makes RAF-throttled functions execute synchronously |
| `mockMermaidToExcalidraw()` | Mocks the mermaid-to-excalidraw integration |
| `mockHTMLImageElement()` | Mocks Image loading |

---

## Mocking Strategy

### General approach

- **Canvas API**: Fully mocked via `vitest-canvas-mock` (no real rendering in tests)
- **IndexedDB**: Mocked via `fake-indexeddb` for persistence tests
- **Network**: Use `vi.mock()` for Firebase, fetch, and Socket.io
- **requestAnimationFrame**: Mocked to execute synchronously via `throttleRAF` mock
- **Fonts**: Mocked at the fetch level to avoid loading real font files
- **Crypto**: Override `window.crypto` in tests that need encryption

### Mocking imports

Use Vitest's `vi.mock()` for module-level mocks:

```typescript
vi.mock("../data/firebase", () => ({
  loadFromFirebase: vi.fn().mockResolvedValue(null),
  saveToFirebase: vi.fn().mockResolvedValue(undefined),
}));
```

### Snapshot testing

Some tests use snapshot assertions for rendered output. When you change UI, snapshots may break:

```bash
yarn test:update
```

Review the snapshot diffs in `__snapshots__/` directories before committing.

---

## Writing a New Test

Here's a template for a typical component/integration test:

```typescript
import { vi } from "vitest";
import { render, unmountComponent } from "../tests/test-utils";
import { API } from "../tests/helpers/api";
import { UI, Keyboard } from "../tests/helpers/ui";
import { Excalidraw } from "../index";

describe("my feature", () => {
  beforeEach(async () => {
    localStorage.clear();
    await render(<Excalidraw />);
  });

  afterEach(() => {
    unmountComponent();
  });

  it("should do something when user clicks", async () => {
    // Arrange: set up initial state
    const rect = API.createElement({ type: "rectangle", x: 100, y: 100 });
    API.setElements([rect]);

    // Act: simulate user interaction
    UI.clickTool("select");
    // ... simulate clicks, drags, etc.

    // Assert: check the result
    const elements = API.getElements();
    expect(elements).toHaveLength(1);
    expect(elements[0].type).toBe("rectangle");
  });

  it("should handle keyboard shortcuts", async () => {
    const rect = API.createElement({ type: "rectangle" });
    API.setElements([rect]);

    // Select all
    Keyboard.withModifierKeys({ ctrl: true }, () => {
      Keyboard.keyDown("a");
    });

    const appState = API.getAppState();
    expect(appState.selectedElementIds[rect.id]).toBe(true);
  });
});
```

### For pure unit tests (no React):

```typescript
import { pointFrom, pointRotateRads } from "@excalidraw/math";

describe("point rotation", () => {
  it("should rotate point around origin", () => {
    const result = pointRotateRads(
      pointFrom(10, 0),
      pointFrom(0, 0),
      Math.PI / 2,
    );
    expect(result[0]).toBeCloseTo(0);
    expect(result[1]).toBeCloseTo(10);
  });
});
```

---

## CI Integration

Tests run automatically in GitHub Actions:

| Workflow | Trigger | What it checks |
|----------|---------|----------------|
| `test.yml` | Push to `master` | `yarn test:app` |
| `lint.yml` | Pull requests | `yarn test:other` + `yarn test:code` + `yarn test:typecheck` |
| `test-coverage-pr.yml` | Pull requests | Coverage report via `davelosert/vitest-coverage-report-action` |

### Before submitting a PR, always run:

```bash
yarn test:update    # Tests + snapshot updates
yarn test:typecheck # Type checking
yarn fix            # Auto-fix lint and formatting
```
