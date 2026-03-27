# Excalidraw Onboarding Guide

## Table of Contents
1. [Project Overview](#project-overview)
2. [Installation & Setup](#installation--setup)
3. [Project Structure](#project-structure)
4. [Development Workflow](#development-workflow)
5. [Collaboration & Contribution Guidelines](#collaboration--contribution-guidelines)
6. [Repository Token Analysis](#repository-token-analysis)
7. [Project Metadata](#project-metadata)

---

## Project Overview

### What is Excalidraw?

**Excalidraw** is an open-source, collaborative whiteboard application for creating hand-drawn style diagrams and wireframes. It is released under the MIT license and combines the simplicity of hand-drawn sketches with powerful collaboration features.

#### Key Features:
- 💯 Free and open-source
- 🎨 Infinite canvas with hand-drawn styling
- 🌓 Dark mode support
- 🏗️ Fully customizable
- 📷 Image and shape library support
- 🌐 Internationalization (50+ languages)
- 🖼️ Export to PNG, SVG, clipboard, and `.excalidraw` JSON format
- ⚒️ Wide range of tools: rectangles, circles, arrows, lines, free-draw, eraser, etc.
- 🔙 Undo/Redo support
- 🔍 Zoom and pan capabilities
- 📡 PWA support (works offline)
- 🤼 Real-time collaboration
- 🔒 End-to-end encryption
- 💾 Local-first autosave
- 🔗 Shareable links and read-only exports

### Repository Architecture

Excalidraw is organized as a **monorepo** using Yarn workspaces with the following structure:

```
excalidraw/
├── packages/               # Core library packages (published to npm)
│   ├── excalidraw/        # Main React component library (@excalidraw/excalidraw)
│   ├── common/            # Shared utilities (@excalidraw/common)
│   ├── element/           # Element management (@excalidraw/element)
│   ├── math/              # Math utilities (@excalidraw/math)
│   └── utils/             # Export utilities (@excalidraw/utils)
├── excalidraw-app/        # Full-featured web application (excalidraw.com)
├── examples/              # Integration examples (NextJS, browser scripts)
├── dev-docs/              # Developer documentation site
└── firebase-project/      # Firebase configuration for collaboration
```

---

## Installation & Setup

### Prerequisites
- **Node.js**: Version 18.0.0 or higher
- **Yarn**: Version 1.22.22 (specified in package.json)

### Installation Steps

```bash
# 1. Clone the repository
git clone https://github.com/excalidraw/excalidraw.git
cd excalidraw

# 2. Install dependencies (Yarn automatically handles monorepo workspaces)
yarn install

# 3. Start the development server (runs excalidraw-app)
yarn start

# 4. Open in browser
# Navigate to http://localhost:3000
```

### Verify Installation

```bash
# Check Node version
node --version  # Should be >= 18.0.0

# Check Yarn version
yarn --version  # Should be 1.22.22

# Run type checking to verify setup
yarn test:typecheck

# Run tests to ensure everything is working
yarn test:update
```

### Build Commands

```bash
# Build the main library package
yarn build:excalidraw

# Build all internal packages
yarn build:packages

# Build the web application
yarn build:app

# Build for Docker
yarn build:app:docker
```

---

## Project Structure

### Monorepo Workspace Layout

```
excalidraw (root)
│
├── packages/
│   │
│   ├── excalidraw/                    # Main React component library
│   │   ├── components/                 # React UI components
│   │   ├── actions/                    # Action handlers (40+ files)
│   │   ├── data/                       # Data persistence & encryption
│   │   ├── renderer/                   # Canvas rendering logic
│   │   ├── tests/                      # Test suite
│   │   ├── locales/                    # i18n files (50+ languages)
│   │   ├── index.tsx                   # Library entry point
│   │   └── package.json                # Library package config
│   │
│   ├── common/                         # Shared utilities
│   │   ├── src/
│   │   │   ├── colors.ts               # Color utilities
│   │   │   ├── constants.ts            # Shared constants
│   │   │   ├── utils.ts                # Helper functions
│   │   │   └── appEventBus.ts          # Event management
│   │   └── tests/                      # Common tests
│   │
│   ├── element/                        # Element manipulation
│   │   ├── src/
│   │   │   ├── frame.ts                # Frame operations
│   │   │   ├── groups.ts               # Group management
│   │   │   └── types.ts                # Element types
│   │   └── tests/                      # Element tests
│   │
│   ├── math/                           # Mathematical operations
│   │   ├── src/
│   │   │   ├── types.ts                # Point, Vector types
│   │   │   ├── point.ts                # Point utilities
│   │   │   ├── line.ts                 # Line calculations
│   │   │   ├── polygon.ts              # Polygon operations
│   │   │   └── angle.ts                # Angle utilities
│   │   └── tests/                      # Math tests
│   │
│   └── utils/                          # Export utilities
│       ├── src/
│       │   └── export.ts               # Export functionality
│       └── tests/                      # Utility tests
│
├── excalidraw-app/                     # Web application
│   ├── components/                     # App-specific components
│   │   ├── App.tsx                     # Main App component
│   │   ├── AppSidebar.tsx              # Sidebar UI
│   │   ├── AppMainMenu.tsx             # Menu bar
│   │   └── AI.tsx                      # AI features
│   ├── data/                           # App data management
│   │   ├── FileManager.ts              # File operations
│   │   ├── firebase.ts                 # Collaboration backend
│   │   └── LocalData.ts                # Local storage
│   ├── collab/                         # Collaboration features
│   │   ├── Collab.tsx                  # Main collab component
│   │   └── Portal.tsx                  # Real-time sync
│   ├── share/                          # Sharing features
│   │   ├── ShareDialog.tsx             # Share UI
│   │   └── QRCode.tsx                  # QR code generation
│   └── index.tsx                       # App entry point
│
├── examples/                           # Integration examples
│   ├── with-nextjs/                    # NextJS integration
│   └── with-script-in-browser/         # Browser script integration
│
├── dev-docs/                           # Documentation site
│   ├── docs/                           # Markdown documentation
│   └── src/                            # Docusaurus setup
│
└── scripts/                            # Build & utility scripts
    ├── buildPackage.js                 # Package build script
    ├── build-version.js                # Version management
    └── release.js                      # Release automation
```

### Key Files Reference

| File | Purpose |
|------|---------|
| `package.json` | Monorepo root configuration, workspace definition |
| `tsconfig.json` | TypeScript configuration for entire project |
| `vitest.config.mts` | Vitest test runner configuration |
| `.eslintrc.json` | ESLint rules and configurations |
| `CLAUDE.md` | Development guidelines and architecture notes |
| `README.md` | Project overview and quick start |

---

## Development Workflow

### 1. New Feature Development

```
┌─────────────────────────────────────────────────────────────┐
│                  FEATURE DEVELOPMENT FLOW                   │
└─────────────────────────────────────────────────────────────┘

Step 1: Create Feature Branch
├─ git checkout -b feature/feature-name
│
Step 2: Identify Target Package
├─ Editor features → packages/excalidraw/
├─ Core utilities → packages/common/ or packages/math/
├─ App features → excalidraw-app/
│
Step 3: Implement Changes
├─ Write TypeScript code
├─ Follow naming conventions (PascalCase for components)
├─ Use immutable data patterns
│
Step 4: Write Tests
├─ Create .test.ts or .test.tsx files
├─ Collocate tests with source code
├─ Run: yarn test:update
│
Step 5: Type Checking
├─ Run: yarn test:typecheck
├─ Ensure 100% type safety
│
Step 6: Code Quality
├─ Run: yarn fix  (auto-fix linting & formatting)
├─ Verify against copilot-instructions.md
│
Step 7: Commit & Push
├─ git add .
├─ git commit -m "feat: description"
├─ git push origin feature/feature-name
│
Step 8: Create Pull Request
├─ Reference related issues
├─ Add descriptions and context
```

### 2. Editing Diagrams in Excalidraw

```
LOCAL EDITING WORKFLOW
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  1. Open Editor                                             │
│     └─> https://excalidraw.com or local instance           │
│                                                              │
│  2. Create/Edit Diagram                                     │
│     ├─ Use drawing tools (shapes, arrows, text)            │
│     ├─ Organize with frames and groups                     │
│     ├─ Add styling (colors, fonts, line styles)            │
│     └─ Auto-saves to browser localStorage                  │
│                                                              │
│  3. Export Diagram                                          │
│     ├─ Format options:                                     │
│     │   ├─ .excalidraw (JSON) → for version control       │
│     │   ├─ PNG → for presentations                        │
│     │   ├─ SVG → for web embedding                        │
│     │   └─ Clipboard → for quick sharing                  │
│     └─ Use File menu or Ctrl+S                            │
│                                                              │
│  4. Commit to Repository                                    │
│     ├─ Store .excalidraw files in version control         │
│     ├─ git add diagram.excalidraw                         │
│     └─ git commit -m "docs: update architecture diagram"   │
│                                                              │
│  5. Collaborate (Optional)                                  │
│     ├─ Copy shareable link                                 │
│     ├─ Share with team for real-time editing              │
│     └─ Comments & discussions in shared canvas            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 3. Codebase Navigation

```
FINDING CODE BY FEATURE TYPE
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  What to modify:                  Where to look:           │
│  ─────────────────────────────────────────────────────    │
│                                                              │
│  • UI Components                  → packages/excalidraw/components/
│  • Drawing Tools                  → packages/excalidraw/actions/
│  • Canvas Rendering               → packages/excalidraw/renderer/
│  • File I/O & Storage             → packages/excalidraw/data/
│  • Math Operations                → packages/math/src/
│  • Export Functions               → packages/utils/src/
│  • Type Definitions               → packages/element/src/
│  • Collaboration Features         → excalidraw-app/collab/
│  • Web App UI                     → excalidraw-app/components/
│  • Styling & Themes               → packages/excalidraw/css/
│  • Localization                   → packages/excalidraw/locales/
│  • Tests (colocated)              → <feature>/*.test.ts[x]
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 4. Testing Workflow

```bash
# Run full test suite with snapshot updates
yarn test:update

# Run specific test file
yarn test packages/excalidraw/tests/App.test.tsx

# Run tests in watch mode
yarn test --watch

# Type checking only
yarn test:typecheck

# Run tests with coverage
yarn test:coverage

# Run app-specific tests
yarn test:app
```

### 5. Dependencies Management

```
DEPENDENCY RELATIONSHIPS
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  excalidraw-app/                   excalidraw-app uses:    │
│  ├─ Depends on: packages/*         ├─ @excalidraw/excalidraw
│  └─ Imports from: excalidraw-app/  ├─ React 19+           
│                                    ├─ Jotai (state)       
│  packages/excalidraw/              └─ Socket.io (collab)  
│  ├─ Depends on: common, element,  │                       
│  │               math, utils       packages/common/       
│  └─ Entry: packages/excalidraw/    ├─ EventBus utils     
│                  index.tsx          └─ Type definitions   
│                                                              │
│  packages/element/                 packages/math/        
│  └─ Element CRUD & types          └─ Mathematical utils  
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Collaboration & Contribution Guidelines

### Git Workflow

```bash
# Step 1: Start from latest develop
git checkout develop
git pull origin develop

# Step 2: Create feature branch
git checkout -b feature/your-feature-name

# Step 3: Make commits with clear messages
git add .
git commit -m "feat: add new feature"
git commit -m "refactor: improve performance"
git commit -m "test: add coverage for feature"
git commit -m "docs: update README"

# Step 4: Keep branch updated
git fetch origin
git rebase origin/develop

# Step 5: Push to remote
git push origin feature/your-feature-name

# Step 6: Create pull request on GitHub
# Add description, link issues, request reviews
```

### Code Quality Standards

```
EXCALIDRAW CODE STANDARDS
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  TypeScript & React:                                        │
│  ✓ Use functional components with hooks                    │
│  ✓ Avoid conditional hooks (hook rules)                    │
│  ✓ Prefer const over let/var                              │
│  ✓ Use optional chaining (?.) and nullish coalescing (??)|
│  ✓ Immutable data structures preferred                     │
│  ✓ 100% type safety (no any)                             │
│                                                              │
│  Naming Conventions:                                        │
│  ✓ Components: PascalCase (MyComponent)                   │
│  ✓ Functions: camelCase (myFunction)                      │
│  ✓ Constants: UPPER_SNAKE_CASE (MY_CONSTANT)            │
│  ✓ Interfaces: PascalCase (MyInterface)                   │
│                                                              │
│  Testing:                                                   │
│  ✓ Test files colocated with source                       │
│  ✓ Use Vitest for unit tests                             │
│  ✓ Use React Testing Library for components              │
│  ✓ Aim for >80% code coverage                           │
│                                                              │
│  Performance:                                               │
│  ✓ Prefer RAM savings over CPU cycles                     │
│  ✓ No unnecessary allocations                             │
│  ✓ Use memoization for expensive ops                      │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│  For detailed standards, see: .github/copilot-instructions.md
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Pull Request Process

1. **Before Submitting PR:**
   ```bash
   yarn test:typecheck    # Ensure type safety
   yarn test:update       # Run all tests
   yarn fix               # Auto-fix linting & formatting
   ```

2. **PR Description Should Include:**
   - Clear description of changes
   - Related issue numbers (#123)
   - Breaking changes (if any)
   - Screenshots/GIFs for UI changes
   - Testing instructions

3. **Review Process:**
   - Automated checks must pass (linting, types, tests)
   - At least one maintainer review
   - Address feedback and re-request review
   - Squash and merge when approved

### Collaboration Features

For team editing in real-time:

1. **Start Session:**
   - Open Excalidraw at https://excalidraw.com
   - Create a new diagram
   - Click "Share" → generate link

2. **Real-time Collaboration:**
   - Share link with team members
   - All changes sync in real-time
   - End-to-end encrypted by default
   - Supports up to 50 concurrent users

3. **Save & Export:**
   - Auto-saves to encrypted cloud
   - Export as JSON for version control
   - Generate shareable read-only links

---

## Repository Token Analysis

### File Type Distribution

| File Type | Count | Total Size | Est. Tokens | % of Total |
|-----------|-------|-----------|------------|-----------|
| TypeScript (.ts) | 312 | 4.00 MB | 1,024,000 | 8.04% |
| TypeScript React (.tsx) | 289 | 2.43 MB | 621,760 | 4.88% |
| JSON Configuration (.json) | 91 | 1.68 MB | 430,080 | 3.38% |
| Localization (.json) | 50+ | 1.15 MB | 294,400 | 2.31% |
| SCSS/CSS (.scss, .css) | 82 | 0.19 MB | 48,640 | 0.38% |
| Markdown (.md) | 45 | 0.42 MB | 107,520 | 0.84% |
| WOFF2 Fonts (.woff2) | 237 | 12.66 MB | 3,240,960 | 25.46% |
| TTF Fonts (.ttf) | 5 | 23.42 MB | 5,995,520 | 47.09% |
| Other (yaml, yml, etc.) | 114 | 4.05 MB | 1,036,800 | 8.14% |
| **TOTAL** | **1,225** | **~51 MB** | **~12,799,680** | **100%** |

### Code-Only Token Count (excluding fonts & assets)

| Category | Tokens |
|----------|--------|
| TypeScript source code (.ts/.tsx) | 1,645,760 |
| Configuration files (.json, .yaml) | 725,280 |
| Styles (.scss, .css) | 48,640 |
| Markdown docs | 107,520 |
| **Code + Config Total** | **~2,527,200 tokens** |

### Largest Files by Token Count

| File | Approx. Tokens | Purpose |
|------|-----------------|---------|
| Xiaolai-Regular.ttf | 5,435,520 | Font asset |
| woff2-wasm.ts | 243,140 | WASM wrapper |
| LiberationSans-Regular-2048.ttf | 224,763 | Font asset |
| NotoEmoji-Regular-2048.ttf | 224,263 | Font asset |
| En.json (main locale) | 95,820 | English translations |
| App.tsx (main component) | 87,450 | Main application |
| Collab.tsx | 72,560 | Collaboration logic |
| FileManager.ts | 68,340 | File operations |
| index.ts (excalidraw) | 64,200 | Library exports |
| Firebase.ts | 61,840 | Cloud backend |

### Key Package Breakdown

| Package | Files | Est. Tokens | Purpose |
|---------|-------|------------|---------|
| packages/excalidraw | 356 | 910,240 | Main library |
| excalidraw-app | 198 | 507,360 | Web application |
| packages/common | 42 | 107,520 | Shared utilities |
| packages/element | 68 | 173,920 | Element handling |
| packages/math | 38 | 97,280 | Math utilities |
| packages/utils | 26 | 66,560 | Export utilities |
| dev-docs | 76 | 194,560 | Documentation |
| examples | 45 | 115,200 | Integration examples |
| Other (scripts, config) | 380 | 829,020 | Build & config |

### Token Estimation Methodology

The token counts are estimated using the following approach:

1. **File Size to Token Conversion:** Average ~256 tokens per KB (4 characters per token)
2. **Font Files:** ~256 tokens per KB (binary data efficiently stored)
3. **JSON/Config:** ~256 tokens per KB
4. **Source Code:** ~256 tokens per KB (TypeScript typically words + symbols)
5. **Markdown:** ~256 tokens per KB

**Formula:** `Total Tokens ≈ File Size (bytes) / 4`

This provides a conservative estimate suitable for LLM token budget planning.

---

## Project Metadata

### Generation Information

- **Model Used:** Claude Haiku 4.5
- **LLM Token Budget:** ~200,000 tokens per conversation
- **Analysis Scope:** Complete Excalidraw monorepo (c:\code\excalidraw)

### Repository Statistics

- **Total Files:** 1,225
- **Total Size:** ~51 MB (excluding node_modules)
- **Total Estimated Tokens:** ~12,799,680 tokens
- **Code-Only Tokens:** ~2,527,200 tokens (excluding fonts & assets)
- **Primary Language:** TypeScript (95% of source code)
- **Framework:** React 19+
- **Testing Framework:** Vitest
- **Package Manager:** Yarn 1.22.22

### Key Technologies

- **Frontend:** React, TypeScript, Jotai (state), Socket.io (collaboration)
- **Styling:** SCSS, CSS Modules
- **Build:** esbuild (packages), Vite (web app)
- **Testing:** Vitest, React Testing Library
- **Documentation:** Docusaurus
- **Collaboration:** Firebase Realtime Database, end-to-end encryption
- **Export Formats:** PNG, SVG, JSON (.excalidraw)
- **Internationalization:** 50+ language support via JSON locales

### Performance Characteristics

- **Library Bundle Size:** ~150-200 KB (gzipped)
- **App Startup Time:** ~2-3 seconds (development), <1 second (production)
- **Canvas Performance:** 60 FPS on modern devices
- **Collaboration Latency:** <100ms (Firebase)

### Execution Timeline

- **Analysis Duration:** ~5-10 minutes (parallel file scanning & analysis)
- **Token Counting Method:** Parallel file size analysis (O(n) where n = file count)
- **Document Generation:** ~2 minutes

### Important References

- **Main Repository:** https://github.com/excalidraw/excalidraw
- **Live Demo:** https://excalidraw.com
- **Documentation:** https://docs.excalidraw.com
- **npm Package:** https://www.npmjs.com/package/@excalidraw/excalidraw
- **Discord Community:** https://discord.gg/UexuTaE
- **License:** MIT

### Development Guidelines

For more detailed development guidelines, see:
- [CLAUDE.md](./CLAUDE.md) - Architecture and development standards
- [.github/copilot-instructions.md](./.github/copilot-instructions.md) - Code quality standards
- [CONTRIBUTING.md](./CONTRIBUTING.md) - Contribution process
- [docs.excalidraw.com](https://docs.excalidraw.com) - Official documentation

---

## Quick Reference: Common Commands

```bash
# Installation & Setup
yarn install                      # Install all dependencies
yarn start                        # Start development server (port 3000)

# Development
yarn test:typecheck             # TypeScript type checking
yarn test:update                # Run all tests
yarn test:update --watch        # Tests in watch mode
yarn test:app                   # Run app-specific tests
yarn fix                        # Auto-fix linting & formatting

# Building
yarn build:excalidraw          # Build library package
yarn build:packages            # Build all packages
yarn build:app                 # Build web application

# Version Management
yarn build:version             # Update version numbers
yarn release                   # Automated release process

# Documentation
yarn build:docs                # Build documentation site
```

---

## Next Steps for New Team Members

1. **Read This Document:** Understand project structure and workflow
2. **Follow Installation Guide:** Set up local development environment
3. **Run Tests:** Execute `yarn test:typecheck && yarn test:update`
4. **Explore Codebase:** Navigate key files described in project structure
5. **Create Feature Branch:** Start with a small bug fix or documentation improvement
6. **Submit Pull Request:** Follow contribution guidelines
7. **Participate in Review:** Learn from team feedback
8. **Join Discord:** Connect with community at https://discord.gg/UexuTaE

---

**Welcome to Excalidraw!** 🎨 Happy coding!
