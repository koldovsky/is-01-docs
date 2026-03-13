# Token Evaluation Snapshot

Token counts generated with `npx repomix` using the default `o200k_base` tokenizer (approx. GPT-4o).

## packages/excalidraw

- **Total tokens:** 2,788,876
- **Files scanned:** 557
- **Top files by tokens:**
  1. `subset/woff2/woff2-wasm.ts` – 585,930 tokens (21%)
  2. `subset/harfbuzz/harfbuzz-wasm.ts` – 462,298 tokens (16.6%)
  3. `fonts/ComicShanns/ComicShanns-Regular.sfd` – 318,656 tokens (11.4%)
  4. `components/App.tsx` – 83,646 tokens (3%)
  5. `fonts/Xiaolai/index.ts` – 70,664 tokens (2.5%)
- **Observations:** Binary-font conversion assets dominate token share; editor runtime (`components/App.tsx`) is ~3%.

## excalidraw-app

- **Total tokens:** 108,355
- **Files scanned:** 50
- **Top files by tokens:**
  1. `repomix-output.xml` (existing artifact) – 54,168 tokens (50%)
  2. `App.tsx` – 8,716 tokens (8%)
  3. `collab/Collab.tsx` – 7,066 tokens (6.5%)
  4. `components/DebugCanvas.tsx` – 3,265 tokens (3%)
  5. `vite.config.mts` – 2,193 tokens (2%)
- **Observations:** Remove `repomix-output.xml` from future runs to reduce noise; collaboration and main app shell are leading code contributors.

## Guidance for AI Context Budgets

- When sharing the **core editor** with an LLM, strip large font/wasm files or reference the npm package instead of raw sources.
- For troubleshooting the **app shell**, focus on `App.tsx`, `collab/Collab.tsx`, and targeted components (~20k tokens combined).
- Consider creating a `repomix.config.json` that excludes generated artifacts (`repomix-output.xml`, fonts) for leaner packs.
