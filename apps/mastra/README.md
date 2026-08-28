# @oss-report/mastra

Mastra app containing the OSS report workflow, Slack digest workflow, report agents, storage configuration, and custom web-app route.

## Storage and output

Each `oss-report-workflow` run is persisted through the configured Mastra storage. `DATABASE_URL` selects PostgreSQL; otherwise `LOCAL_DATABASE_URL` selects LibSQL. The run result matches `reportSchema` in `src/mastra/workflows/oss-report.ts` and is queried by the web app through `@mastra/client-js`.

The production web app is served at `/app/`. Its compiled files must exist in `src/mastra/public/app/` before `mastra build` runs so they are copied into `.mastra/output/app/`.

## Scripts

From this package:

```bash
pnpm dev        # mastra dev on :4115
pnpm build      # mastra build; does not build the Vite workspace first
pnpm deploy     # deploy using .env.production
pnpm typecheck  # tsc --noEmit
```

From the repository root, use `pnpm build` or `pnpm deploy` to build the web app before Mastra.
