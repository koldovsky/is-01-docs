# Project Overview

## What Is This Project?

Excalidraw is an open source monorepo containing a browser-based whiteboard editor and the reusable editor packages that power it. The repository includes the published `@excalidraw/excalidraw` package plus the `excalidraw-app` showcase application, examples, and supporting shared packages.

This project solves the problem of creating a hand-drawn style collaborative drawing tool with strong local-first and offline capabilities. Users can create diagrams, wireframes, and notes in a familiar whiteboard experience that also supports export, localization, and package integration.

The target audience includes developer consumers of the npm package, end users of the public app at `excalidraw.com`, and contributors who extend the editor, build examples, or maintain the app experience.

## Goals & Success Metrics

| Goal | How We Measure It | Current Target |
| --- | --- | --- |
| Maintain a stable open-source editor package | Passing CI on all PRs and releases | `TBD` |
| Keep editor features reliable | PR test coverage, lint, and typecheck success | `TBD` |
| Support the public app and showcase | Successful app build and preview runs | `TBD` |
| Keep localization and translation quality high | Crowdin locale completeness and repo locale coverage | `TBD` |

## What You Will Be Working On

As a new engineer, you will typically work on editor features in `packages/excalidraw`, app improvements in `excalidraw-app`, and cross-package integration in shared packages like `@excalidraw/common`, `@excalidraw/math`, and `@excalidraw/element`. You may also help maintain examples, run the local app, and ensure CI and release scripts remain healthy.

You will often be fixing bugs, improving type safety, writing tests, and updating build or release tooling. The repo is a monorepo, so changes frequently span package boundaries and require end-to-end validation.

## Quick Links

- GitHub repository: https://github.com/excalidraw/excalidraw
- Public app: https://excalidraw.com
- Project documentation: https://docs.excalidraw.com/docs/introduction/contributing
- Discord community: https://discord.gg/UexuTaE
- Vercel: https://vercel.com
- Sentry: https://sentry.io
- Crowdin translation platform: https://crowdin.com
- GitHub Actions workflows: `.github/workflows/`

## First Week Checklist

- [ ] Clone the repository and verify branch access.
- [ ] Install Node.js and Yarn versions compatible with the repo.
- [ ] Run `yarn install` and confirm the repo builds.
- [ ] Start the local app with `yarn start` and open it in the browser.
- [ ] Run the core test commands: `yarn test:app`, `yarn test:code`, `yarn test:typecheck`.
- [ ] Read `README.md`, `CONTRIBUTING.md`, and `.github/workflows/` files.
- [ ] Review the package structure in `excalidraw-app/` and `packages/`.
- [ ] Meet your onboarding buddy or team contact and confirm communication channels.
