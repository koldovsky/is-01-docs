# Onboarding Checklist

## Week 1 – Environment & Orientation

- [ ] Install dependencies: `yarn install` at repo root.
- [ ] Run the playground: `yarn start` (launches `excalidraw-app` via Vite).
- [ ] Explore `<Excalidraw />` public API using `examples/with-script-in-browser`.
- [ ] Read `packages/excalidraw/README.md` for integration notes.
- [ ] Skim `excalidraw-app/App.tsx` to understand bootstrapping flow.

## Week 2 – Core Systems

- [ ] Trace the render loop: `packages/excalidraw/components/App.tsx` ➜ `Scene` ➜ `Renderer`.
- [ ] Step through `ActionManager` and at least 3 representative actions.
- [ ] Review persistence pipeline: `LocalData`, `FileManager`, `exportToBackend`.
- [ ] Investigate collaboration path: `excalidraw-app/collab/Collab.tsx`.
- [ ] Study Jotai integration via `editor-jotai.ts` and hooks in `hooks/useAppStateValue.ts`.

## Week 3 – Contribution Ready

- [ ] Run lint/tests: `yarn test:code`, `yarn test:app`.
- [ ] Build packages: `yarn build:packages` and ensure artifacts generate.
- [ ] Implement a small feature or bugfix touching actions or UI panel.
- [ ] Document your change in `dev-docs/` or relevant README.
- [ ] Pair with senior dev to review collaboration flows and deployment checklist.
