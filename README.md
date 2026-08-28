# OSS Report

Monorepo for generating and browsing weekly OSS health reports for `mastra-ai/mastra`.

## Structure

```
apps/
├── mastra/   Mastra app: agents + workflows that produce and deliver reports
└── web/      Vite + React UI for generating and browsing reports
```

## Requirements

- Node `>=22.13.0`
- pnpm `10.x`

## Setup

```bash
pnpm install
cp apps/mastra/.env.example apps/mastra/.env
cp apps/web/.env.example apps/web/.env  # optional; development defaults to :4115
```

Fill in `apps/mastra/.env`:

- `OPENROUTER_API_KEY` — used by the agents and signal embeddings
- `LOCAL_DATABASE_URL` — absolute LibSQL `file:` URL for local development
- `GITHUB_PERSONAL_ACCESS_TOKEN`
- `DISCORD_BOT_TOKEN`
- `DISCORD_GENERAL_CHANNEL_ID`
- `OSS_REPORT_REPO_OWNER` / `OSS_REPORT_REPO_NAME` — repository to report on (defaults to `mastra-ai/mastra`)

For a deployed instance, set `DATABASE_URL` to use PostgreSQL instead of local LibSQL storage.

Optional Slack delivery and Q&A:

- `SLACK_BOT_TOKEN`
- `SLACK_SIGNING_SECRET`
- `SLACK_REPORT_CHANNEL_ID`
- `SLACK_REPORT_APP_URL`

Optional workflow tuning:

- `OSS_REPORT_MAX_GENERAL_MESSAGES` (default 200)
- `OSS_REPORT_MAX_THREAD_MESSAGES` (default 50)
- `OSS_REPORT_RECURRING_THRESHOLD` — cosine similarity threshold for recurring detection (default 0.82)

## Develop

Run both apps at once:

```bash
pnpm dev
```

- Mastra Studio and API: http://localhost:4115
- Web app: http://localhost:5173

Or run them individually:

```bash
pnpm dev:mastra
pnpm dev:web
```

The web app uses `VITE_MASTRA_API_URL` when set. In development it defaults to `http://localhost:4115`; in production it uses the current origin.

## How reports are stored

Each run of `oss-report-workflow` is persisted through Mastra storage. The app uses PostgreSQL when `DATABASE_URL` is set and otherwise uses the LibSQL database specified by `LOCAL_DATABASE_URL`.

The web app does not read report files. It uses [`@mastra/client-js`](https://mastra.ai/docs/server/mastra-client) to query workflow runs from the Mastra server, so that server must be running for reports to load.

## Generate a report

Use the **Generate report** form on the web app home page, or run `oss-report-workflow` from Mastra Studio with input such as:

```json
{ "start": "2026-04-20T00:00:00.000Z", "end": "2026-04-22T23:59:59.999Z" }
```

All fields are optional:

- `start` / `end` — ISO timestamps (defaults to the preceding 30 days)
- `window: "week-to-date"` — most recent Monday at 00:00 UTC through the run time
- `maxIssueAnalyses` — cap on issues analyzed per run (default 500)

The workflow is scheduled for Fridays at 19:00 UTC with `window: "week-to-date"`. A successful run appears immediately in the web app and posts a Slack digest when Slack is configured.

## Browse reports

In development, open http://localhost:5173:

- `/#/` — successful runs, newest first
- `/#/reports/:runId` — full report detail

In a combined production build, the same hash routes are served below `/app/` on the Mastra server.

## Build

```bash
pnpm build          # build the web app, then Mastra
pnpm build:web      # emit the SPA into apps/mastra/src/mastra/public/app
pnpm build:mastra   # copy public assets and build the Mastra server
pnpm typecheck
```

The build order matters: the web app must exist under `apps/mastra/src/mastra/public/app` before `mastra build` packages the server.

## Deploy

For a CLI deployment, configure `apps/mastra/.env.production` and run:

```bash
pnpm deploy
```

This builds the web app before invoking `mastra deploy`.

Mastra Platform's GitHub push-to-deploy flow runs the Mastra build from the checked-out repository; it does not build the separate Vite workspace first. For that flow, commit the generated files under `apps/mastra/src/mastra/public/app/` whenever the frontend changes. Otherwise `/app/` falls through to Mastra's default page because the deployment artifact contains no SPA.
