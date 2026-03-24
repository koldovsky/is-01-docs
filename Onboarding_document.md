# Onboarding Document

Welcome to the Excalidraw project! This document is designed to help new team members get up to speed quickly. It covers the general overview, key features, technology stack, architecture, and instructions for running the application locally.

## 1. General Description

**What is it?**
Excalidraw is an open-source, virtual, hand-drawn style whiteboard application. It is highly collaborative and end-to-end encrypted.

**What is it for?**
It is built for creating beautiful hand-drawn like diagrams, wireframes, architectural visualizations, brainstorming sessions, or whatever visual representations you need.

**What tasks does it solve?**
Excalidraw provides a frictionless, intuitive drawing interface that focuses on speed and simplicity over pixel-perfect alignment. It allows individuals and teams to quickly externalize their thoughts, collaborate in real-time securely, and easily export or embed their diagrams into other tools and platforms.

## 2. Key Features

- **Infinite Canvas:** Unrestricted drawing space.
- **Hand-drawn Aesthetic:** Elements look sketched by hand, focusing attention on the concepts rather than the exact styling.
- **Rich Toolset:** Rectangle, circle, diamond, arrow, line, free-draw, eraser, text, and more.
- **Smart Connections:** Arrow-binding and labeled arrows for diagramming.
- **Customization & Libraries:** Dark mode, customizable styles, and support for extensive custom shape libraries.
- **Export & Integration:** Export to PNG, SVG, and clipboard. Drawings can be saved in an open `.excalidraw` JSON format.
- **Collaboration & Security:** Real-time collaboration via shareable links with robust end-to-end encryption.
- **Offline First:** PWA support allows the app to work offline, with autosaving directly to the browser.

## 3. Technology Stack

The project relies on a modern frontend stack:
- **Language:** TypeScript and JavaScript
- **UI Framework:** React
- **Build & Development Tool:** Vite
- **Package Manager:** Yarn (v1.22.x) leveraging Yarn Workspaces for a monorepo setup
- **Testing:** Vitest (for unit/component tests)
- **Linting & Formatting:** ESLint, Prettier

## 4. Architecture and Key Components

The codebase is organized as a monorepo containing multiple packages and applications. The primary directory of interest is `excalidraw/`, which houses the core project:

- **`excalidraw-app/` (The Application):** This is the web application hosted at excalidraw.com. It acts as a showcase, integrating the core editor with added features like real-time collaboration, PWA support, local-first autosaving, and shareable links.
- **`packages/excalidraw/` (The Core Package):** This is the core Excalidraw editor logic packaged for npm (`@excalidraw/excalidraw`). It is designed as a drop-in React component so that other applications can embed Excalidraw's functionality.
- **`packages/math/`, `packages/element/`, `packages/common/`, `packages/utils/`:** These are internal supporting libraries and modular utilities used by the core editor and the main application.
- **`examples/`:** Contains reference implementations on how to integrate the Excalidraw npm package in various environments (e.g., Next.js, standalone scripts).

## 5. Technical Prerequisites

Before contributing or running the project locally, ensure your development environment meets the following requirements:
- **Node.js:** Version 18.0.0 or higher.
- **Yarn:** The classic version (1.22.x).
- **Git:** For version control.
- **Editor:** VSCode is recommended with the ESLint and Prettier extensions installed for a seamless development experience.

## 6. How to Run the Application Locally

Follow these steps to set up and start the application on your local machine:

1. **Install Dependencies**
   Navigate to the `excalidraw` folder (or the root if the monorepo handles it top-level) and run:
   ```bash
   yarn install
   ```
   *(Note: If you encounter dependency issues, you can run `yarn clean-install` to wipe `node_modules` and start fresh).*

2. **Start the Development Server**
   To start the main `excalidraw-app` with live reloading:
   ```bash
   yarn start
   ```
   This will run the Vite development server. Check your terminal output for the local URL (usually `http://localhost:3000` or similar).

3. **Running Tests & Quality Checks**
   Ensure your changes adhere to project standards:
   ```bash
   # Run Vitest test suites
   yarn test
   
   # Run formatting and linting
   yarn test:code
   yarn test:other
   
   # Run all checks (typecheck, linting, tests)
   yarn test:all
   ```

4. **Building for Production**
   To verify the production build completes successfully:
   ```bash
   yarn build
   ```

Welcome to the team, and happy drawing!
