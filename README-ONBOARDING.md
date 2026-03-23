========================================

Repository: ueberdosis/tiptap.
Files analyzed: 1917.
Estimated tokens: 1.6M.
Agent: Claude Sonnet 4.6 (Github Copilot).

========================================

# Welcome to the Tiptap Monorepo 👋

We're glad you're here. This guide will get you from zero to productive as quickly as possible. It covers architecture, key concepts, and day-to-day workflows. Read it end-to-end once, then keep it handy as a reference.

---

## Table of Contents

1. [What is Tiptap?](#what-is-tiptap)
2. [High-Level Architecture](#high-level-architecture)
3. [Project Structure](#project-structure)
4. [Core Concepts](#core-concepts)
5. [Development Workflow](#development-workflow)
6. [Contributor Guide](#contributor-guide)
7. [Useful Commands Cheatsheet](#useful-commands-cheatsheet)

---

## What is Tiptap?

Tiptap is a **headless, framework-agnostic rich text editor toolkit** built on top of [ProseMirror](https://prosemirror.net). "Headless" means Tiptap provides all the editor logic, document model, and behavior — but ships **no default styling**. You bring your own UI.

It is distributed as a set of focused npm packages under the `@tiptap/*` scope, designed to be composed together. You only pull in what you need.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      Your Application                   │
│        (React / Vue / Vanilla JS / any framework)       │
└────────────────────┬────────────────────────────────────┘
                     │  uses
┌────────────────────▼────────────────────────────────────┐
│           Framework Adapters (@tiptap/react,            │
│            @tiptap/vue-3, @tiptap/vue-2)                │
│   EditorContent, useEditor, ReactNodeViewRenderer, …    │
└────────────────────┬────────────────────────────────────┘
                     │  wraps
┌────────────────────▼────────────────────────────────────┐
│                   @tiptap/core                          │
│   Editor · ExtensionManager · CommandManager           │
│   Node · Mark · Extension base classes                 │
│   InputRules · PasteRules · Helpers · Utilities        │
└──────┬─────────────────────────────────┬───────────────┘
       │  extends                        │  extends
┌──────▼──────────────┐   ┌─────────────▼──────────────────┐
│  extension-bold     │   │  extension-heading              │
│  extension-italic   │   │  extension-image                │
│  extension-link     │   │  extension-table                │
│  extension-…  (50+) │   │  extension-…  (50+)            │
└─────────────────────┘   └────────────────────────────────┘
                     │  built on
┌────────────────────▼────────────────────────────────────┐
│                    @tiptap/pm                           │
│  Thin re-exports of ProseMirror packages                │
│  (model, state, view, transform, keymap, …)             │
└─────────────────────────────────────────────────────────┘
```

### Key relationships

| Layer | Package(s) | Role |
|---|---|---|
| **ProseMirror bridge** | `@tiptap/pm` | Exposes ProseMirror sub-packages under a single, versioned entry point. Prevents version skew between ProseMirror packages. |
| **Core** | `@tiptap/core` | The engine. Houses `Editor`, `CommandManager`, `ExtensionManager`, and the base `Node`, `Mark`, and `Extension` classes. Framework-agnostic. |
| **Extensions** | `@tiptap/extension-*` | Each extension is a self-contained package that registers Nodes, Marks, or plain Extensions into the core via `ExtensionManager`. They extend the core's `Node`, `Mark`, or `Extension` class. |
| **Framework adapters** | `@tiptap/react`, `@tiptap/vue-3`, `@tiptap/vue-2` | Wrap `@tiptap/core` with framework-specific components, hooks, and node view renderers. They do **not** contain editor logic — they are thin integration layers. |
| **Starter Kit** | `@tiptap/starter-kit` | A convenience package that bundles the most common extensions. Great for prototyping. |

> **Mental model:** `@tiptap/core` is the brain. Extensions add vocabulary (what content types exist). Framework adapters add a mouth (how the editor is rendered and interacted with in a given UI framework).

---

## Project Structure

```
tiptap/
├── packages/                  # All published packages
│   ├── core/                  # @tiptap/core — the editor engine
│   ├── pm/                    # @tiptap/pm — ProseMirror re-exports
│   ├── react/                 # @tiptap/react — React adapter
│   ├── vue-3/                 # @tiptap/vue-3 — Vue 3 adapter
│   ├── vue-2/                 # @tiptap/vue-2 — Vue 2 adapter
│   ├── starter-kit/           # @tiptap/starter-kit — common extension bundle
│   ├── extensions/            # Shared/composite extensions
│   ├── suggestion/            # Autocomplete suggestion helper
│   ├── static-renderer/       # Server-side / static rendering helper
│   ├── html/                  # HTML parsing/serialization utilities
│   ├── markdown/              # Markdown serialization utilities
│   └── extension-*/           # Individual first-party extensions (50+)
│
├── packages-deprecated/       # Packages moved to extension-list umbrella, kept for semver
│
├── demos/                     # Vite dev app — live examples for every extension
│   ├── src/
│   │   ├── Demos/             # Auto-discovered demo pages
│   │   ├── Examples/          # Polished example editors
│   │   └── Extensions/        # Per-extension demo files
│   └── vite.config.ts
│
├── tests/                     # Cypress end-to-end tests (run against demos on :3000)
│
├── .changeset/                # Changesets for versioning and changelog generation
│
├── .github/
│   ├── instructions/          # AI/contributor guidelines (PR, changeset, repo conventions)
│   └── workflows/             # CI pipelines
│
├── turbo.json                 # Turborepo pipeline config (build, lint, test)
├── pnpm-workspace.yaml        # Monorepo workspace definition
└── vitest.config.ts           # Unit test configuration
```

### Key directories at a glance

| Directory | Description |
|---|---|
| `packages/core/src/` | Editor engine: `Editor.ts`, `CommandManager.ts`, `ExtensionManager.ts`, base `Node.ts`, `Mark.ts`, `Extension.ts` |
| `packages/core/src/commands/` | Built-in command implementations (`toggleBold`, `setNode`, `insertContent`, …) |
| `packages/core/src/helpers/` | Pure utility functions used internally by commands and extensions |
| `packages/core/src/extensions/` | Built-in core plugins (keyboard handling, paste, focus, editable state, …) |
| `packages/pm/` | Each sub-folder re-exports one ProseMirror package (`model/`, `state/`, `view/`, `transform/`, …) |
| `packages/react/src/` | `useEditor`, `EditorContent`, `ReactNodeViewRenderer`, `ReactMarkViewRenderer` |
| `packages/vue-3/src/` | Vue 3 composables and components (mirrors the React adapter API) |
| `packages/extension-bold/src/` | A representative, simple Mark extension — good first read |
| `packages/extension-heading/src/` | A representative Node extension |
| `demos/src/` | Self-contained demo files; auto-discovered by the Vite app, no manual wiring needed |
| `tests/` | Cypress specs; each spec navigates to a demo URL and interacts with the editor |

---

## Core Concepts

### 1. Nodes

A **Node** represents a block or inline element in the document tree — think `<p>`, `<h1>`, `<img>`, `<table>`, or a custom widget. Nodes define:

- Their **content model** (what child nodes they allow, expressed as a ProseMirror content expression).
- How they **parse from HTML** (`parseHTML`).
- How they **serialize to HTML** (`renderHTML`).
- Optional **keyboard shortcuts**, **input rules**, **paste rules**, and a custom **NodeView** (for complex rendering).

```ts
// packages/extension-heading/src/heading.ts (simplified)
import { Node } from '@tiptap/core'

export const Heading = Node.create({
  name: 'heading',
  content: 'inline*',        // can contain inline content
  group: 'block',            // belongs to the 'block' group
  defining: true,

  addAttributes() {
    return { level: { default: 1 } }
  },

  parseHTML() {
    return [1, 2, 3, 4, 5, 6].map(level => ({ tag: `h${level}` }))
  },

  renderHTML({ node, HTMLAttributes }) {
    return [`h${node.attrs.level}`, HTMLAttributes, 0]
  },
})
```

### 2. Marks

A **Mark** is inline formatting applied *on top of* text — think `<strong>`, `<em>`, `<a href>`, a color, or a comment annotation. Marks layer on text without changing the document tree structure.

```ts
// packages/extension-bold/src/bold.tsx (simplified)
import { Mark } from '@tiptap/core'

export const Bold = Mark.create({
  name: 'bold',

  parseHTML() {
    return [{ tag: 'strong' }, { tag: 'b' }]
  },

  renderHTML({ HTMLAttributes }) {
    return ['strong', HTMLAttributes, 0]
  },

  addCommands() {
    return {
      setBold:    () => ({ commands }) => commands.setMark(this.name),
      toggleBold: () => ({ commands }) => commands.toggleMark(this.name),
      unsetBold:  () => ({ commands }) => commands.unsetMark(this.name),
    }
  },

  addKeyboardShortcuts() {
    return { 'Mod-b': () => this.editor.commands.toggleBold() }
  },
})
```

### 3. Extensions

A plain **Extension** (not a Node or Mark) registers behavior without introducing a new content type. Examples: keyboard shortcuts, focus handling, history, drag handles, custom paste logic.

```ts
import { Extension } from '@tiptap/core'

export const MyPlugin = Extension.create({
  name: 'myPlugin',
  addProseMirrorPlugins() {
    return [/* ProseMirror Plugin */]
  },
})
```

### 4. The Command Chain Pattern

Tiptap's command API is designed to be **composable and readable**. Every action on the editor is a *command* — a function that receives the current editor state and either applies a transaction or returns `false`.

Commands are accessed in three ways:

| API | Use case | Dispatches immediately? |
|---|---|---|
| `editor.commands.toggleBold()` | Single, fire-and-forget command | ✅ Yes |
| `editor.chain().focus().toggleBold().run()` | Multiple commands as one atomic transaction | ✅ On `.run()` |
| `editor.can().chain().toggleBold().run()` | Dry-run: check if a command *would* succeed | ❌ Never |

The **chain** is the idiomatic way to write multi-step operations:

```ts
editor
  .chain()          // open a chainable context
  .focus()          // move focus to the editor
  .toggleBold()     // toggle bold on selection
  .setTextSelection({ from: 0, to: 5 })
  .run()            // dispatch the single resulting transaction
```

Under the hood, `chain()` uses a *chainable state* — a ProseMirror `EditorState` whose transactions are accumulated rather than immediately dispatched. This ensures all chained commands act on the same document snapshot and produce a single undo history entry.

### 5. NodeViews and MarkViews

When default HTML rendering isn't enough, you can provide a **NodeView** (or **MarkView**) — a custom renderer for a Node or Mark. This is how you embed React or Vue components inside the editor:

```ts
// In a Node extension
addNodeView() {
  return ReactNodeViewRenderer(MyReactComponent)  // from @tiptap/react
}
```

The adapter packages (`@tiptap/react`, `@tiptap/vue-3`) expose `ReactNodeViewRenderer` / `VueNodeViewRenderer` for this purpose.

---

## Development Workflow

### Prerequisites

| Tool | Minimum version |
|---|---|
| Node.js | `>=20` |
| pnpm | `9.x` (version pinned in `package.json` → `packageManager`) |

It is strongly recommended to use a Node version manager (`nvm`, `fnm`) and to let **Corepack** manage pnpm:

```bash
corepack enable
```

### 1. Install dependencies

```bash
# from the repo root
pnpm install
```

This installs dependencies for all packages in the monorepo workspace in a single pass.

### 2. Build all packages

```bash
pnpm build
```

Turborepo builds packages in topological order (dependencies before dependants). Build artifacts land in `packages/*/dist`.

To build a single package while iterating:

```bash
pnpm -w -F @tiptap/core build
```

### 3. Run the demos app

```bash
pnpm dev
```

This starts the Vite dev server at **http://localhost:3000**. The app auto-discovers all demo files under `demos/src/` — no manual registration needed.

### 4. Run the tests

**Unit tests** (Vitest, runs in Node):

```bash
pnpm test:unit
```

**End-to-end tests** (Cypress, requires the demos to be running):

```bash
# Terminal A
pnpm dev

# Terminal B
pnpm test:e2e:open   # interactive UI
# or
pnpm test:e2e        # headless CI mode
```

### 5. Lint and format

```bash
pnpm lint          # report issues
pnpm lint:fix      # auto-fix with Prettier + ESLint
```

---

## Contributor Guide

### Where to find things

| What you're looking for | Where to look |
|---|---|
| Editor bootstrapping / lifecycle | `packages/core/src/Editor.ts` |
| How extensions are registered | `packages/core/src/ExtensionManager.ts` |
| Command accumulation / dispatch | `packages/core/src/CommandManager.ts` |
| All built-in commands | `packages/core/src/commands/` |
| ProseMirror integration points | `packages/pm/` — each subfolder re-exports a ProseMirror package |
| React integration | `packages/react/src/` — `useEditor`, `EditorContent`, `ReactNodeViewRenderer` |
| Vue 3 integration | `packages/vue-3/src/` — composables and component adapters |
| A simple Mark extension (good starting point) | `packages/extension-bold/src/bold.tsx` |
| A simple Node extension (good starting point) | `packages/extension-heading/src/heading.ts` |
| A complex extension with subextensions | `packages/extension-table/src/` |
| Demo for a feature | `demos/src/Extensions/<ExtensionName>/` |
| Cypress test for a feature | `tests/` |

### ProseMirror vs. Tiptap abstraction

Tiptap is deliberately layered. Keep this boundary in mind:

- **ProseMirror logic** lives in `@tiptap/pm` (re-exports) and in `ExtensionManager` / `CommandManager` inside core. If you need raw `EditorState`, `Transaction`, or ProseMirror `Plugin` APIs, import from `@tiptap/pm/*`.
- **Tiptap abstractions** (`Node.create`, `Mark.create`, `addCommands`, `addInputRules`, …) live in `@tiptap/core`. Prefer these when writing extensions — they handle wiring into ProseMirror for you.
- **Framework-specific code** (React hooks, Vue composables, component props) belongs **only** in the adapter packages (`@tiptap/react`, `@tiptap/vue-3`, `@tiptap/vue-2`). Never import from a framework adapter inside `@tiptap/core` or an extension.

### Adding a new extension

1. Copy an existing simple extension (e.g. `packages/extension-bold`) as a template.
2. Update `package.json` (name, description, peer deps).
3. Implement your `Node.create` / `Mark.create` / `Extension.create` in `src/`.
4. Add a demo under `demos/src/Extensions/<YourExtension>/`.
5. Add a Cypress test under `tests/`.
6. Run `pnpm changeset` to create a changeset for your new package.

### Creating a changeset

After making user-facing changes, run:

```bash
pnpm changeset
```

Select the affected packages, choose a bump type (`patch` / `minor` / `major`), and write a short, user-facing description. Changesets are reviewed as part of the PR.

### PR checklist

- [ ] `pnpm lint` passes with no new errors.
- [ ] `pnpm build` succeeds for all affected packages.
- [ ] A demo is added or updated for any visible behavior change.
- [ ] A Cypress test covers the new or changed behavior.
- [ ] A changeset has been created for any user-facing API change.
- [ ] JSDoc comments are added/updated for all new public APIs.

---

## Useful Commands Cheatsheet

| Command | Description |
|---|---|
| `pnpm install` | Install all dependencies |
| `pnpm build` | Build all packages (Turborepo) |
| `pnpm dev` | Start the demos on http://localhost:3000 |
| `pnpm lint` | Run ESLint across the repo |
| `pnpm lint:fix` | Auto-fix with Prettier + ESLint |
| `pnpm test:unit` | Run Vitest unit tests |
| `pnpm test:e2e` | Run Cypress headless |
| `pnpm test:e2e:open` | Open Cypress UI |
| `pnpm changeset` | Create a new changeset |
| `pnpm reset` | Full reset (node_modules, caches, build artifacts) |
| `pnpm -w -F @tiptap/<pkg> build` | Build a single package |

---

> **Questions?** Start with the [Tiptap documentation](https://tiptap.dev), search existing GitHub issues, or ask in the project's discussion channels. Welcome aboard! 🚀

