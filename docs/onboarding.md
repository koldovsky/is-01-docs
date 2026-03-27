# Onboarding Documentation for Excalidraw Project

## Overview

The `excalidraw` project is a collaborative whiteboard tool designed for teams and individuals to create diagrams, sketches, and visualizations. It supports real-time collaboration, making it ideal for brainstorming sessions, design discussions, and educational purposes. The project emphasizes simplicity, performance, and security, with features like end-to-end encryption.

Key features:

- Infinite canvas
- Drawing tools (rectangle, arrow, free draw, etc.)
- Export to PNG/SVG
- Real-time collaboration
- End-to-end encryption

## Tech Stack

- **React**: Frontend framework for building the user interface.
- **TypeScript**: Ensures type safety and better developer experience.
- **Canvas API**: Used for rendering drawings on the canvas.
- **WebSockets**: Enables real-time collaboration.
- **Node.js**: Provides the development environment.
- **Jotai**: Manages application state.

## Key Directories and Their Roles

1. **`excalidraw-app/`**
   - Contains the main application logic.
   - Includes components, themes, and app-specific configurations.
   - Key files:
     - `App.tsx`: Entry point for the application.
     - `index.tsx`: Main rendering logic.
     - `components/`: Houses reusable UI components like `AppSidebar.tsx`, `AppFooter.tsx`.

2. **`packages/`**
   - Modularized packages for shared utilities and features.
   - Subdirectories:
     - `common/`: Shared utilities.
     - `excalidraw/`: Core drawing logic and components.
     - `utils/`: Helper functions and utilities.

3. **`dev-docs/`**
   - Documentation for developers.
   - Includes configuration files like `docusaurus.config.js` for generating documentation.

4. **`examples/`**
   - Demonstrates integration examples.
   - Subdirectories:
     - `with-nextjs/`: Example with Next.js integration.
     - `with-script-in-browser/`: Example for browser-based usage.

5. **`public/`**
   - Static assets like images, icons, and service workers.

6. **`scripts/`**
   - Automation scripts for building, testing, and deploying the project.

These directories work together to ensure modularity, reusability, and maintainability of the codebase.

## Setup Instructions

### Prerequisites

- Node.js (>=18)
- Yarn or npm

### Install

1. Clone the repository:
   ```bash
   git clone https://github.com/YOUR_USERNAME/excalidraw.git
   ```
2. Navigate to the project directory:
   ```bash
   cd excalidraw
   ```
3. Install dependencies:
   ```bash
   npm install
   ```

### Run Locally

1. Start the development server:
   ```bash
   npm start
   ```
2. Open your browser and navigate to:
   ```
   http://localhost:3000
   ```

### Verify Installation

- Run tests to ensure everything is set up correctly:
  ```bash
  npm test
  ```

## Token Evaluation

Tokens are a measure of complexity and effort required for understanding and contributing to each part of the project. Here's an evaluation:

### Why Token Evaluation Matters

Token evaluation helps contributors identify the most complex parts of the project, prioritize their focus, and estimate the effort required for modifications.

📈 Top 5 Files by Token Count:
──────────────────────────────

1.  `excalidraw/packages/excalidraw/subset/woff2/woff2-wasm.ts` (585,930 tokens, 972,016 chars, 17.2%)
2.  `excalidraw/packages/excalidraw/subset/harfbuzz/harfbuzz-wasm.ts` (462,298 tokens, 789,956 chars, 13.5%)
3.  `excalidraw/packages/excalidraw/fonts/ComicShanns/ComicShanns-Regular.sfd` (318,656 tokens, 669,376 chars, 9.3%)
4.  `excalidraw/packages/excalidraw/components/App.tsx` (85,427 tokens, 407,630 chars, 2.5%)
5.  `excalidraw/packages/excalidraw/fonts/Xiaolai/index.ts` (70,664 tokens, 121,858 chars, 2.1%)

📊 Pack Summary:
────────────────

- **Total Files**: 936 files
- **Total Tokens**: 3,414,543 tokens
- **Total Chars**: 9,784,287 chars
