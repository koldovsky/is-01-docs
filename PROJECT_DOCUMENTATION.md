# Excalidraw

<div align="center">
  <p>
    <strong>Virtual whiteboard for sketching hand-drawn like diagrams.</strong><br/>
    Collaborative and end-to-end encrypted.
  </p>
</div>

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![npm](https://img.shields.io/npm/dm/@excalidraw/excalidraw)](https://www.npmjs.com/package/@excalidraw/excalidraw)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://docs.excalidraw.com/docs/introduction/contributing)
[![Discord](https://img.shields.io/discord/723672430744174682?color=5865F2&label=discord)](https://discord.gg/UexuTaE)

## Features

- 💯 Free & open-source
- 🎨 Infinite, canvas-based whiteboard
- ✍️ Hand-drawn style rendering
- 🌓 Dark mode
- 🏗️ Highly customizable
- 📷 Image support
- 😀 Shape libraries
- 🌐 Localization (i18n)
- 🖼️ Export to PNG, SVG & clipboard
- 💾 Open format (.excalidraw JSON)
- ⚒️ Tools: rectangle, circle, diamond, arrow, line, free-draw, eraser, text
- ➡️ Arrow-binding & labeled arrows
- 🔙 Undo/Redo
- 🔍 Zoom and panning

### [excalidraw.com](https://excalidraw.com) extras

- 📡 PWA support (offline)
- 🤼 Real-time collaboration
- 🔒 End-to-end encryption
- 💾 Local-first (browser autosave)
- 🔗 Shareable links

## Project Structure

```
packages/
├── excalidraw/     # @excalidraw/excalidraw - Main React component (npm package)
├── common/         # @excalidraw/common - Shared utilities and constants
├── element/        # @excalidraw/element - Element manipulation logic
├── math/           # @excalidraw/math - Math utilities and types
└── utils/          # @excalidraw/utils - Export utilities

excalidraw-app/     # Production app (excalidraw.com)
examples/
├── with-nextjs/    # Next.js integration example
└── with-script-in-browser/  # Browser script example
dev-docs/           # Documentation site (Docusaurus)
scripts/            # Build and release scripts
```

## Quick Start

### Using the npm Package

```bash
npm install react react-dom @excalidraw/excalidraw
# or
yarn add react react-dom @excalidraw/excalidraw
```

```tsx
import { Excalidraw } from "@excalidraw/excalidraw";

function App() {
  return (
    <div style={{ height: "500px" }}>
      <Excalidraw />
    </div>
  );
}
```

See the [installation docs](https://docs.excalidraw.com/docs/@excalidraw/excalidraw/installation) for more details.

## Development

### Prerequisites

- [Node.js](https://nodejs.org/)
- [Yarn](https://yarnpkg.com/) (v1 or v2.4.2+)
- [Git](https://git-scm.com/)

### Setup

```bash
git clone https://github.com/excalidraw/excalidraw.git
cd excalidraw
yarn
yarn start
```

Open [http://localhost:3000](http://localhost:3000) to start developing.

### Commands

| Command | Description |
|---------|-------------|
| `yarn start` | Start development server |
| `yarn test:typecheck` | TypeScript type checking |
| `yarn test:update` | Run all tests (with snapshot updates) |
| `yarn fix` | Auto-fix formatting and linting |
| `yarn start:example` | Start example app on port 3001 |

### Architecture

- **Yarn workspaces** for monorepo management
- **Vite** for app bundling, **esbuild** for packages
- **Jotai** for state management
- **Vitest** for testing
- **TypeScript** with strict configuration

### Package Development

Work in `packages/*` for editor features, `excalidraw-app/` for app-specific features.

## Dependencies

### Core

| Package | Purpose |
|---------|---------|
| `react` | UI framework |
| `jotai` | State management |
| `roughjs` | Hand-drawn rendering |
| `perfect-freehand` | Freehand drawing |
| `nanoid` | ID generation |
| `pako` | Compression |

### Build & Dev

| Package | Purpose |
|---------|---------|
| `vite` | App bundler |
| `esbuild` | Package bundler |
| `typescript` | Type checking |
| `vitest` | Testing |
| `eslint`, `prettier` | Code quality |

## Contributing

1. Fork and clone the repo
2. Run `yarn` to install dependencies
3. Create a branch: `git checkout -b your-branch-name`
4. Make changes and run `yarn test:update`
5. Submit a PR with semantic prefix: `feat:`, `fix:`, `docs:`, `refactor:`, etc.

See the [contribution guide](https://docs.excalidraw.com/docs/introduction/contributing) for details.

## Documentation

- [Full Documentation](https://docs.excalidraw.com)
- [API Reference](https://docs.excalidraw.com/docs/@excalidraw/excalidraw/api)
- [Integration Guide](https://docs.excalidraw.com/docs/@excalidraw/excalidraw/integration)

## Integrations

- [VSCode Extension](https://marketplace.visualstudio.com/items?itemName=pomdtr.excalidraw-editor)
- [npm Package](https://www.npmjs.com/package/@excalidraw/excalidraw)

## Who's Using Excalidraw

Google Cloud • Meta • CodeSandbox • Obsidian • Replit • Slite • Notion • HackerRank

## Support

- [Report Issues](https://github.com/excalidraw/excalidraw/issues)
- [Discord](https://discord.gg/UexuTaE)
- [Open Collective](https://opencollective.com/excalidraw)

## License

[MIT](LICENSE)