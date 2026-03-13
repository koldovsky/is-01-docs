---
title: New Employee Onboarding
---

# New Employee Onboarding

This guide helps you become productive in the Excalidraw monorepo quickly.
It covers project layout, local setup, development workflow, and where to make changes.

## 1. What You Are Working On

Excalidraw in this repository is a monorepo with two main products and shared packages:

- `packages/excalidraw`: published editor library (`@excalidraw/excalidraw`)
- `excalidraw-app`: the full web app experience (excalidraw.com)
- `packages/common`, `packages/element`, `packages/math`, `packages/utils`: shared internals
- `examples/*`: integration examples for consumers
- `dev-docs/`: this documentation website

### System Overview (high-level)

```text
                        +-----------------------------+
                        |      Excalidraw Monorepo    |
                        +-----------------------------+
                                      |
         +----------------------------+----------------------------+
         |                                                         |
 +---------------------------+                       +---------------------------+
 |    packages/excalidraw    |<--------------------->|       excalidraw-app      |
 |  Reusable editor library  |   imports + wraps     |     Hosted web app        |
 +---------------------------+                       +---------------------------+
         ^                                                         ^
         |                                                         |
 +-------+--------+  +---------------+  +---------------+  +------+-------+
 | common package |  | element pkg   |  | math package  |  | utils package |
 +----------------+  +---------------+  +---------------+  +---------------+
```

## 2. First-Day Setup

Prerequisites:

- Node.js 18+
- Yarn 1.x (repo uses workspaces)
- Git

Bootstrap:

```bash
git clone https://github.com/excalidraw/excalidraw.git
cd excalidraw
yarn
yarn start
```

Open `http://localhost:3000`.

## 3. Essential Commands

Run from repo root:

- `yarn start`: run app locally
- `yarn test:typecheck`: TypeScript validation
- `yarn test:app`: unit/integration tests (Vitest)
- `yarn test:update`: update snapshots
- `yarn test:code`: lint checks
- `yarn fix`: auto-fix formatting/lint issues

## 4. Development Workflow

```text
Pick issue/task
   |
   v
Find owner area (app vs package)
   |
   v
Implement change + local manual verification
   |
   v
Run checks: typecheck, tests, lint
   |
   v
Open PR with semantic title (feat/fix/docs/...)
```

## 5. How To Locate Code Quickly

Use this map when deciding where to work:

- Editor behavior, canvas tools, scene logic: `packages/excalidraw/`
- Element data models and transforms: `packages/element/`
- Geometry and computational helpers: `packages/math/`
- Generic utilities reused across packages: `packages/utils/`, `packages/common/`
- App shell, menus, app-only UI: `excalidraw-app/components/`
- Collaboration and room logic: `excalidraw-app/collab/`
- Persistence/storage integrations: `excalidraw-app/data/`
- Public docs and guides: `dev-docs/docs/`

### Feature Routing Diagram

```text
        Is this needed by external npm consumers?
                       /                 \
                     yes                 no
                     /                    \
       packages/excalidraw (+shared pkg)   excalidraw-app/*
                 |
                 v
    Consider whether logic belongs in common/element/math/utils
```

## 6. Quality Gates Before PR

Minimum checklist:

- Code compiles and typechecks (`yarn test:typecheck`)
- Relevant tests pass (`yarn test:app`)
- Lint/format clean (`yarn test:code` or `yarn fix`)
- Manual smoke test in local app for touched behavior
- PR title uses semantic prefix (`feat`, `fix`, `docs`, `refactor`, ...)

## 7. Collaboration and Communication

- Use GitHub issues for scope and design discussion
- For larger changes, align early with maintainers in the issue thread
- Keep PRs focused and reviewable; split large refactors when possible
- Add or update tests for bug fixes and behavior changes

## 8. First Week Plan

Day 1-2:

- Build and run locally
- Read `README.md`, `CONTRIBUTING.md`, and docs under `docs/introduction`
- Trace one small feature end-to-end in code

Day 3-4:

- Pick a small issue and ship one PR
- Add tests for the change
- Learn the review feedback patterns from maintainers

Day 5:

- Pair with a teammate on a medium-scope issue
- Document one thing you learned in `dev-docs/`

## 9. Common Pitfalls

- Mixing app-specific behavior into reusable library code
- Forgetting to run `yarn test:typecheck` before pushing
- Making broad refactors together with feature changes in one PR
- Updating snapshots without reviewing what changed

## 10. Where To Ask For Help

- GitHub issue comments for task-specific questions
- Team channels/maintainer sync for architecture decisions
- Existing tests and recent PRs for implementation patterns

Welcome aboard.
