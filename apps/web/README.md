# @oss-report/web

Vite + React + Tailwind UI for generating and browsing OSS reports.

The app uses `@mastra/client-js` to query `oss-report-workflow` runs from the Mastra server. Set `VITE_MASTRA_API_URL` to override the server URL; development defaults to `http://localhost:4115`, while production uses the current origin.

## Scripts

From this package:

```bash
pnpm dev        # Vite development server on :5173
pnpm build      # typecheck and emit into ../mastra/src/mastra/public/app
pnpm preview    # preview the production build
pnpm typecheck  # tsc --noEmit
```

The generated files under `apps/mastra/src/mastra/public/app/` must be present before the Mastra production build runs.

## Routes

The app uses hash routing so it works below the production `/app/` path:

- `/#/` — list successful workflow runs
- `/#/reports/:runId` — full report detail

In production these become `/app/#/` and `/app/#/reports/:runId`.
