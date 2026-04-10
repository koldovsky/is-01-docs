# Excalidraw onboarding

This guide helps you **join the project**: community, how we work, and your machine setup. The same doc also serves as the **hands-on development reference** for this repo (workspaces, commands, layout).

It is for **working in this Git repository**: Yarn workspaces, `yarn install`, `yarn start` (the **`excalidraw-app`** dev server), and where code lives under `packages/` and `examples/`.

That is separate from **shipping Excalidraw inside your own app**. For that you add the published npm package [`@excalidraw/excalidraw`](https://www.npmjs.com/package/@excalidraw/excalidraw) and follow the **[package installation docs](https://docs.excalidraw.com/docs/@excalidraw/excalidraw/installation)**—you usually **do not** need to clone this monorepo.

For **how to contribute** (issues, PRs, translations, review expectations), use the **[contribution guide](https://docs.excalidraw.com/docs/introduction/contributing)**. The docs site’s **[development](https://docs.excalidraw.com/docs/introduction/development)** page also describes local setup and a **CodeSandbox** flow. **This file** is meant to match **this tree**: scripts in the root [`package.json`](package.json), Vite config under [`excalidraw-app/`](excalidraw-app), and env files at the repo root (for example, the dev server **port** comes from **`VITE_APP_PORT`** in [`.env.development`](.env.development), not a single hard-coded value in the online tutorial).

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://excalidraw.nyc3.cdn.digitaloceanspaces.com/github/excalidraw_github_cover_2_dark.png" />
  <img src="https://excalidraw.nyc3.cdn.digitaloceanspaces.com/github/excalidraw_github_cover_2.png" alt="Excalidraw logo and cover art" />
</picture>

![Excalidraw editor: canvas, shape tools, and hand-drawn style elements](https://excalidraw.nyc3.cdn.digitaloceanspaces.com/github%2Fproduct_showcase.png)

*After `yarn start`, this is the editor you should get locally: infinite canvas, tools, and normal drawing (same core experience as [excalidraw.com](https://excalidraw.com); dev-only settings, collab backends, or telemetry can differ).*

---

## Onboarding

### How to use this doc

1. Skim **Onboarding** (this section) for culture, links, and PR habits.
2. Follow **Local setup** to get `yarn start` working.
3. Keep **Development reference** (commands, folders, stack) open while you code.

If you use **Cursor** or another AI assistant in this repo, **[CLAUDE.md](CLAUDE.md)** summarizes structure, common commands, and where features usually live—point tools at it for quicker context.

### Community and where to ask

| Channel | Use it for |
|---------|------------|
| **[Discord](https://discord.gg/UexuTaE)** | Quick questions, hanging out with contributors |
| **[GitHub Issues](https://github.com/excalidraw/excalidraw/issues)** | Bug reports, feature discussion, tracking work |
| **[Contribution guide](https://docs.excalidraw.com/docs/introduction/contributing)** | Rules of the road for PRs, translations, and reviews |

For **translations**, see the [translation section](https://docs.excalidraw.com/docs/introduction/contributing#translating) in the contribution guide.

### Suggested first steps

1. **Run the app** — Finish **Local setup** below so `yarn start` works; click around and reproduce a small behavior you care about.
2. **Read the contribution guide** — Especially PR expectations and code style.
3. **Pick a starting point** — Use **[Where to make changes](#where-to-make-changes)** to see whether your idea touches `packages/excalidraw/` (editor) or `excalidraw-app/` (product features).
4. **Optional:** Browse issues labeled [**good first issue**](https://github.com/excalidraw/excalidraw/issues?q=is%3Aopen+is%3Aissue+label%3A%22good+first+issue%22) (when available).

### Before you open a PR

- **`yarn test:typecheck`** — No TypeScript errors.
- **`yarn test:update`** — Tests and snapshots updated when your change affects them (see [contribution guide](https://docs.excalidraw.com/docs/introduction/contributing)).
- **`yarn fix`** — Prettier + ESLint auto-fixes; reduces CI noise.

**Git hooks:** After `yarn install`, **Husky** installs a **`prepare`** hook. **lint-staged** runs on commit for staged files (ESLint on `.{js,ts,tsx}`, Prettier on several other extensions—see [`.lintstagedrc.js`](.lintstagedrc.js)). If a commit fails, fix or stage the files the hook rewrote, then commit again.

### No-code ways to help

Reporting bugs with clear repro steps, improving docs, answering questions on Discord, and helping with **translations** are all valuable. The [contribution guide](https://docs.excalidraw.com/docs/introduction/contributing) covers how to get started for each.

---

## Local setup

### What's in this repo?

This is a **monorepo**: one Git checkout, several packages, all tied together with **Yarn workspaces**.

- **`excalidraw-app/`** — The full app (the kind of experience you get on [excalidraw.com](https://excalidraw.com)): sharing, collaboration hooks, and app-only UI. **`yarn start` from the repo root** runs this.
- **`packages/excalidraw/`** — The editor that ships as `@excalidraw/excalidraw` on npm. Most tools, canvas logic, and editor chrome live here.
- **`packages/common`**, **`packages/element`**, **`packages/math`**, **`packages/utils`** — Shared building blocks used by the app and the library.
- **`examples/`** — Tiny apps that show how people import the package (e.g. Vite + script).

You don’t need the full map in your head to start hacking: **install once at the repo root**, then **`yarn start`** from there.

### What you’ll need

| Tool | How to check |
|---|---|
| **Git** | `git --version` |
| **Node.js 18+** (see root `package.json` `engines`) | `node -v` — 18, 20, 22, … |
| **Yarn Classic 1.x** (this repo pins `yarn@1.22.22`) | `yarn -v` should show **1.x**, not 2/3 (Berry) |
| **A normal desktop browser** | Chrome, Firefox, or Safari is fine |

**Don’t have Yarn 1 yet?** Node ships with Corepack; in a terminal:

```bash
corepack enable
corepack prepare yarn@1.22.22 --activate
yarn -v   # should be 1.22.x
```

This project really expects **Yarn** plus the checked-in **`yarn.lock`** at the root. Using **`npm install`** there can leave workspaces and versions out of step with everyone else, so it’s best to stick with Yarn for day-to-day work.

### Clone and install

From a folder where you keep code:

```bash
git clone https://github.com/excalidraw/excalidraw.git
cd excalidraw
yarn install
```

The first `yarn install` might take a few minutes. When it finishes, you should have `node_modules` at the root and no scary errors.

If things feel broken (weird resolution errors, half-installed deps), a nuked-and-paved reinstall usually helps:

```bash
yarn clean-install
```

That wipes workspace `node_modules` and runs `yarn install` again.

### Run the app

Still at the **repo root**:

```bash
yarn start
```

That boots **Vite** for `excalidraw-app`. Your terminal will show a **local URL**—often something like `http://localhost:3001/`.

Ports come from the **repo root** env files (Vite’s `envDir` points at the parent of `excalidraw-app`). In **`.env.development`**, `VITE_APP_PORT=3001` is the usual value; if that line weren’t there, the default would be **3000**. **Trust the URL Vite prints**—especially if you or someone else tweaks `.env`.

Vite is set to **open a browser tab** for you when it can.

Need a tweak that shouldn’t be committed? Put it in **`.env.local`** or **`.env.development.local`** at the repo root (both are gitignored)—e.g. if 3001 is busy, set `VITE_APP_PORT=3003` in `.env.local`.

### You’re up and running when…

- The page loads (no blank screen, no red Vite overlay stuck on top).
- You see the **big canvas** and a **row of tools** (shapes, arrow, line, draw, text, …).
- **Zoom** and **undo/redo** feel normal.
- You can actually **draw or drop shapes** on the canvas.

Edits under `excalidraw-app/` or `packages/` usually **hot-reload** in the browser without a full refresh.

**About collaboration:** `.env.development` points `VITE_APP_WS_SERVER_URL` at something like `http://localhost:3002`. Real-time collab wants a matching server (see [excalidraw-room](https://github.com/excalidraw/excalidraw-room)). **Drawing and most of the editor work fine without it**—only live multi-user sync needs that extra piece.

### Quick checks before you ship a change

| Command | What it does | You want |
|---------|----------------|----------|
| `yarn test:typecheck` | TypeScript over the whole repo | Clean exit, no errors |
| `yarn test` | Vitest in watch mode | Tests green (fix failures before commit) |
| `yarn test:update` | One-shot Vitest, refreshes snapshots | Use when snapshots need updating; exit 0 |
| `yarn test:all` | Typecheck + lint + format + tests (no watch) | Everything passes |

For PRs, the team expects **`yarn test:update`** and **`yarn test:typecheck`** (and fixes)—details in the [contribution guide](https://docs.excalidraw.com/docs/introduction/contributing).

### Other handy commands

| Command | Notes |
|---------|------|
| `yarn build` | Production build of the web app |
| `yarn build:packages` | Build packages in order: `common` → `math` → `element` → `excalidraw` |
| `yarn start:example` | Builds packages, then runs the **with-script-in-browser** example |
| `yarn start:production` | Build + serve a static build (see `excalidraw-app/package.json`) |

### When something breaks

| Situation | Try this |
|---|----------|
| Node below 18 | Install an LTS (18+), then `yarn install` again |
| `yarn -v` is 2.x or 3.x | `corepack prepare yarn@1.22.22 --activate` (see above) |
| Port in use | Change `VITE_APP_PORT` in `.env.local`, or free the port |
| `yarn install` won’t cooperate | Don’t toss **`yarn.lock`** unless a maintainer says so. Try `yarn clean-install`, Node 18+, and a stable network |
| ESLint screaming in the browser | `.env.development` can turn on ESLint in dev; fix issues or override locally in a gitignored env file |
| Collab never connects | Normal if `excalidraw-room` (or whatever `VITE_APP_WS_SERVER_URL` points at) isn’t running. Editor work doesn’t depend on it |

---

## Development reference

### Quick start

```bash
# Install dependencies
yarn install

# Start the development server
yarn start

# Run tests
yarn test

# Fix lint/formatting issues
yarn fix
```

### Monorepo structure

```
excalidraw/
├── excalidraw-app/          # Full web app (excalidraw.com)
│   ├── App.tsx              # Root app component
│   ├── collab/              # Collaboration features
│   ├── components/          # App-specific components
│   └── data/                # App data layer
├── packages/
│   ├── excalidraw/          # Core library (@excalidraw/excalidraw, published to npm)
│   │   ├── actions/         # Editor actions (undo, redo, etc.)
│   │   ├── components/      # UI components (toolbar, dialogs, etc.)
│   │   ├── scene/           # Canvas scene management
│   │   ├── renderer/        # Canvas rendering
│   │   ├── data/            # Data serialization/import/export
│   │   ├── hooks/           # React hooks
│   │   └── index.tsx        # Public API entry point
│   ├── common/              # @excalidraw/common — shared utilities
│   ├── element/             # @excalidraw/element — element types/helpers
│   ├── math/                # @excalidraw/math — geometry utilities
│   └── utils/               # @excalidraw/utils — general utilities
├── examples/                # Integration examples (Next.js, browser script)
└── scripts/                 # Build and release scripts
```

### Development commands

| Command | Description |
|---|---|
| `yarn start` | Start dev server for excalidraw-app |
| `yarn build` | Production build of excalidraw-app |
| `yarn build:packages` | Build all packages (common → math → element → excalidraw) |
| `yarn test` | Run vitest (watch mode) |
| `yarn test:update` | Run all tests with snapshot updates (use before committing) |
| `yarn test:typecheck` | TypeScript type-check the entire monorepo |
| `yarn test:code` | ESLint across all files |
| `yarn test:all` | Full CI suite (typecheck + lint + format + tests) |
| `yarn fix` | Auto-fix formatting and lint issues |
| `yarn fix:code` | ESLint auto-fix only |
| `yarn fix:other` | Prettier auto-fix only |
| `yarn rm:build` | Remove all build artifacts |
| `yarn clean-install` | Remove node_modules and reinstall |

### Where to make changes

| Goal | Directory |
|---|---|
| Editor features (tools, canvas, elements) | `packages/excalidraw/` |
| App features (collab, share, auth) | `excalidraw-app/` |
| Shared types or utilities | `packages/common/` or `packages/utils/` |
| Element geometry/math | `packages/math/` |
| Element definitions and transforms | `packages/element/` |

### Build order

When building packages, they must be built in dependency order:

```
common → math → element → excalidraw
```

Use `yarn build:packages` to build all in the correct order.

### Testing

- Test files live alongside source files (e.g., `clipboard.test.ts` next to `clipboard.ts`)
- Integration tests for the app are in `excalidraw-app/tests/`
- Run `yarn test:update` before committing to update snapshots
- Run `yarn test:typecheck` to catch TypeScript errors

### Tech stack

- **React 19** with TypeScript
- **Vite** for the app dev server and bundler
- **esbuild** for package builds
- **Vitest** for testing
- **Yarn workspaces** for monorepo management
- **jotai** for state management (`editor-jotai.ts`, `app-jotai.ts`)

### Key files

| File | Purpose |
|---|---|
| `packages/excalidraw/index.tsx` | Public API — what gets exported to npm consumers |
| `packages/excalidraw/types.ts` | Core TypeScript types |
| `packages/excalidraw/appState.ts` | App state shape and defaults |
| `packages/excalidraw/scene/` | Canvas element management |
| `excalidraw-app/App.tsx` | Top-level app component |
| `vitest.config.mts` | Test configuration and path aliases |
| `tsconfig.json` | Root TypeScript config |

### Adding a new package

1. Create directory under `packages/your-package/`
2. Add `package.json` following the pattern of existing packages
3. Add it to the root `package.json` workspaces (already covered by `packages/*`)
4. Update build order in `scripts/` if needed

### Releasing

```bash
yarn release:next    # Publish a next/preview release
yarn release:latest  # Publish a stable release
```

Releases are managed via `scripts/release.js`.

---

## Token count by folder

**Total: 3,199,338** tokens 

### Full breakdown

| Folder | Tokens | Files | Share |
|--------|-------:|------:|------:|
| `packages/excalidraw` | 2,623,796 | 554 | 82.0% |
| `packages/element` | 313,910 | 77 | 9.8% |
| `dev-docs` | 72,799 | 56 | 2.3% |
| `excalidraw-app` | 52,154 | 49 | 1.6% |
| `packages/common` | 34,087 | 30 | 1.1% |
| `public` | 21,156 | 5 | 0.7% |
| `examples` | 21,147 | 26 | 0.7% |
| `packages/math` | 19,517 | 26 | 0.6% |
| `packages/utils` | 14,546 | 15 | 0.5% |
| `scripts` | 11,200 | 13 | 0.4% |
| `.github` | 6,774 | 16 | 0.2% |
| `README.md` | 2,349 | 1 | 0.1% |
| `package.json` | 1,322 | 1 | 0.0% |
| `setupTests.ts` | 868 | 1 | 0.0% |
| `vitest.config.mts` | 565 | 1 | 0.0% |
| `.eslintrc.json` | 486 | 1 | 0.0% |
| `vercel.json` | 408 | 1 | 0.0% |
| `tsconfig.json` | 387 | 1 | 0.0% |
| `CLAUDE.md` | 364 | 1 | 0.0% |
| `.codesandbox` | 356 | 2 | 0.0% |
| `packages/tsconfig.base.json` | 259 | 1 | 0.0% |
| `LICENSE` | 221 | 1 | 0.0% |
| `Dockerfile` | 158 | 1 | 0.0% |
| `docker-compose.yml` | 139 | 1 | 0.0% |
| `packages/eslintrc.base.json` | 133 | 1 | 0.0% |
| `.lintstagedrc.js` | 120 | 1 | 0.0% |
| `firebase-project` | 58 | 2 | 0.0% |
| `crowdin.yml` | 33 | 1 | 0.0% |
| `CONTRIBUTING.md` | 26 | 1 | 0.0% |
