# Scheduled and background jobs

Work that shouldn't happen inside a user's request: things on a schedule (nightly sync, reminders), slow tasks (AI generation, exports, email batches), and retries of failed work.

## Decide

### Questions to ask

- Scheduled (cron), triggered by an event, or both?
- How long does one job run? Longer than the host's function limit?
- Must it retry on failure? Does it have multiple steps that should resume where they failed?
- How many per day? Does ordering matter?
- What do you want to learn: cron basics, message queues, or durable workflows?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Vercel Cron | ★ default | Simple schedules calling your own routes | Subject to function time limits; limited frequency on Hobby |
| GitHub Actions schedule | ◆ alternative | Scheduled scripts independent of the host | Imprecise timing; not for user-triggered work |
| Upstash QStash | ◆ alternative | HTTP message queue with retries, delays and schedules | Calls your endpoints, so they must be public and verified |
| Inngest | ✦ distinctive | Durable multi-step workflows triggered by events | Another platform in the loop |
| Trigger.dev | ✦ distinctive | Long-running background tasks written in TS | Another platform; tasks run on their infrastructure |
| Cloudflare Queues / Cron Triggers | ✦ distinctive | Workers apps | Cloudflare-only |
| Worker process on a long-running host | ◆ alternative | Heavy or continuous work (Render/Railway/Fly worker) | Always-on cost |

### Option notes

**Vercel Cron** ★
- Pros: a few lines in `vercel.json`; calls a route on a schedule.
- Cons: the route is still a function with time limits; Hobby has limits on frequency and timing precision.
- What you'll learn: cron syntax, securing internal endpoints.

**GitHub Actions schedule** ◆
- Pros: free minutes; run any script with repo secrets.
- Cons: schedules can run late during busy periods; minimum interval is 5 minutes.
- What you'll learn: Actions workflows, cron in CI.

**Upstash QStash** ◆
- Pros: publish a message, QStash calls your endpoint with retries; delays, schedules, deduplication; serverless-friendly.
- What you'll learn: queues, at-least-once delivery, signature verification.

**Inngest** ✦
- Pros: write functions as steps (`step.run`, `step.sleep`, `step.waitForEvent`); each step retries and resumes; great local dev server.
- What you'll learn: durable execution, event-driven architecture.

**Trigger.dev** ✦
- Pros: long-running tasks (no serverless timeout) with retries, schedules, realtime status; good for AI pipelines.
- What you'll learn: task queues, running work off your host.

**Cloudflare Queues / Cron Triggers** ✦
- Pros: native to Workers; consumers process batches.
- What you'll learn: queue consumers and batching.

**Worker process** ◆
- Pros: unlimited run time; any language; can use Redis-backed queues (BullMQ) or Postgres-backed queues (pg-boss).
- What you'll learn: queue workers, concurrency, graceful shutdown.

### How to choose

1. A simple daily/hourly task on Vercel → Vercel Cron.
2. A script that isn't really part of the app → GitHub Actions schedule.
3. Fire-and-forget work with retries from serverless → QStash.
4. Multi-step workflows → Inngest; long-running tasks → Trigger.dev.
5. Heavy continuous processing → a worker process.

## Configure

### Common to every option

- [ ] Make jobs idempotent (safe to run twice).
- [ ] Protect job endpoints: verify a secret header or the provider's signature.
- [ ] Log start/end/failure; alert on repeated failures (see [monitoring](monitoring.md)).
- [ ] Remember cron times are usually UTC.

### Vercel Cron

**Setup**
- [ ] Route, e.g. `app/api/cron/<job>/route.ts`.
- [ ] `vercel.json`: `{ "crons": [{ "path": "/api/cron/<job>", "schedule": "0 6 * * *" }] }`.
- [ ] Set `CRON_SECRET`; Vercel sends `Authorization: Bearer <CRON_SECRET>`; reject requests without it.
- [ ] Crons run only on production deployments.

**Verify**
- [ ] Trigger manually from the dashboard (**Settings → Cron Jobs**); check logs.

### GitHub Actions schedule

**Setup**
- [ ] `.github/workflows/<job>.yml` with `on: schedule: - cron: "0 6 * * *"` and `workflow_dispatch` for manual runs.
- [ ] Store needed secrets in repo **Settings → Secrets and variables → Actions**.

**Verify**
- [ ] Manual run succeeds.

**Gotchas**
- Scheduled workflows in inactive repos get disabled after 60 days without activity.

### Upstash QStash

**Setup**
- [ ] `npm i @upstash/qstash`; publish with `client.publishJSON({ url, body, delay })`.
- [ ] Verify incoming requests with the `Receiver` and signing keys.
- [ ] Schedules: `client.schedules.create({ destination, cron })`.

**Env vars**
- `QSTASH_TOKEN`, `QSTASH_CURRENT_SIGNING_KEY`, `QSTASH_NEXT_SIGNING_KEY`.

**Verify**
- [ ] A failing endpoint is retried; the QStash dashboard shows delivery logs.

### Inngest

**Setup**
- [ ] `npm i inngest`; create a client and functions; serve them at `/api/inngest`.
- [ ] Local: `npx inngest-cli@latest dev`.
- [ ] Production: install the Inngest Vercel integration (or set keys) so it syncs your functions.

**Env vars**
- `INNGEST_EVENT_KEY`, `INNGEST_SIGNING_KEY`.

**Verify**
- [ ] Send an event; the dev UI shows each step; a failed step retries without rerunning earlier steps.

### Trigger.dev

**Setup**
- [ ] `npx trigger.dev@latest init`; write tasks in `src/trigger/`.
- [ ] `npx trigger.dev@latest dev` locally; deploy with `npx trigger.dev@latest deploy`.
- [ ] Trigger from the app with `tasks.trigger("task-id", payload)`.

**Env vars**
- `TRIGGER_SECRET_KEY`.

**Verify**
- [ ] A run appears in the dashboard with logs.

### Cloudflare Queues / Cron Triggers

**Setup**
- [ ] Cron: `[triggers] crons = ["0 6 * * *"]` in wrangler config; implement `scheduled()`.
- [ ] Queues: create a queue, add producer and consumer bindings; implement `queue()`.

**Verify**
- [ ] `wrangler dev --test-scheduled` triggers the cron handler.

### Worker process

**Setup**
- [ ] Separate worker service on the host (Render Background Worker, Railway service, Fly process group).
- [ ] Queue: BullMQ (Redis) or pg-boss (Postgres).
- [ ] Handle `SIGTERM` to finish the current job before exiting.

**Verify**
- [ ] Jobs enqueued by the web app are processed; killing the worker mid-job leads to a retry.

## Related

- [`apis.md`](apis.md), [`monitoring.md`](monitoring.md), [`hosting.md`](hosting.md).
