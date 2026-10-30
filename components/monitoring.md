# Monitoring and logging

Knowing when the app breaks and why: logs, error tracking, uptime checks, and optionally product analytics and traces.

## Decide

### Questions to ask

- Would you notice if the app broke right now? How?
- Are errors happening in the browser, on the server, or in jobs?
- Do you need logs kept longer than the host keeps them?
- Do you want usage analytics (who uses what), and are there privacy constraints?
- What do you want to learn: error tracking, structured logging, analytics, or distributed tracing?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Host logs (Vercel/Netlify/Render) | ★ default | Quick debugging of recent requests | Short retention, limited search |
| Sentry | ★ default | Error tracking with stack traces, client and server | Another SDK in the bundle |
| Axiom | ◆ alternative | Long-retention, queryable logs (Vercel log drains) | Another service |
| Better Stack | ◆ alternative | Uptime monitoring, status pages, logs | Another service |
| PostHog | ✦ distinctive | Product analytics, session replay, feature flags | Privacy/consent considerations |
| OpenTelemetry | ✦ distinctive | Vendor-neutral traces across services | Setup effort; most useful with several services |
| Vercel Web Analytics / Speed Insights | ◆ alternative | Privacy-friendly page analytics and Core Web Vitals | Vercel-only |

### Option notes

**Host logs** ★
- Pros: already there.
- Cons: retention is short on free plans.
- What you'll learn: reading request logs, function logs.

**Sentry** ★
- Pros: groups errors, shows stack traces with source maps, release tracking, alerts by email; free developer tier.
- What you'll learn: source maps, error grouping, alerting.

**Axiom** ◆
- Pros: generous free tier; Vercel integration streams all logs; fast queries.
- What you'll learn: structured logging, log queries, dashboards.

**Better Stack** ◆
- Pros: uptime checks with alerts, status pages, log management.
- What you'll learn: uptime/heartbeat monitoring.

**PostHog** ✦
- Pros: analytics, session replay, feature flags and experiments in one tool; generous free tier.
- What you'll learn: event tracking, funnels, feature flags.

**OpenTelemetry** ✦
- Pros: standard for traces/metrics/logs; send to any backend (Axiom, Honeycomb, Grafana, Sentry).
- What you'll learn: spans, trace context propagation.

**Vercel Analytics / Speed Insights** ◆
- Pros: one component; cookie-less page analytics; real-user performance metrics.

### How to choose

1. Every app → host logs + Sentry + an uptime check.
2. Need logs older than a day or two → Axiom (or Better Stack logs).
3. Want to understand usage → Vercel Analytics (simple) or PostHog (deep).
4. Multiple services → OpenTelemetry.

## Configure

### Common to every option

- [ ] Log structured JSON (`{ level, msg, ...context }`), not just strings; never log secrets or full personal data.
- [ ] Alerts go somewhere you'll see them (email or phone push).
- [ ] Add a `/api/health` route that checks the database for uptime monitors.

### Host logs (Vercel)

**Setup**
- [ ] Dashboard **Logs** tab; filter by function, status code.
- [ ] CLI: `vercel logs <deployment-url>`.

**Verify**
- [ ] A `console.error` in a route appears in logs.

### Sentry

**Setup**
- [ ] Create a project; `npx @sentry/wizard@latest -i nextjs` (or the framework's wizard).
- [ ] Confirm source map upload in the build (needs `SENTRY_AUTH_TOKEN` in the host/CI).
- [ ] Set alert rules (new issue → email).
- [ ] Tag the environment (`VERCEL_ENV`) and release.

**Env vars**
- `SENTRY_DSN` / `NEXT_PUBLIC_SENTRY_DSN` (the DSN is safe to expose), `SENTRY_AUTH_TOKEN` (build only, secret), `SENTRY_ORG`, `SENTRY_PROJECT`.

**Verify**
- [ ] Trigger a test error; it appears with a readable stack trace and an alert email arrives.

### Axiom

**Setup**
- [ ] Install the Axiom integration from the Vercel Marketplace (log drain), or use `@axiomhq/js` / `next-axiom`.
- [ ] Create a dataset and a saved query for errors.

**Verify**
- [ ] Recent logs are queryable in Axiom.

### Better Stack

**Setup**
- [ ] Create an uptime monitor on the production URL or `/api/health`; set alert channels.
- [ ] Optional: heartbeat monitors for cron jobs (job pings a URL when it succeeds).
- [ ] Optional: public status page.

**Verify**
- [ ] Taking the health route down triggers an alert.

### PostHog

**Setup**
- [ ] Create a project; `npm i posthog-js` (and `posthog-node` for server events).
- [ ] Initialize in a client provider; consider a reverse proxy path to avoid ad blockers.
- [ ] Identify signed-in users; respect consent where required.

**Env vars**
- `NEXT_PUBLIC_POSTHOG_KEY`, `NEXT_PUBLIC_POSTHOG_HOST`.

**Verify**
- [ ] Page views and a custom event appear in the live events view.

### OpenTelemetry

**Setup**
- [ ] Next.js: `@vercel/otel` and `registerOTel({ serviceName })` in `instrumentation.ts`.
- [ ] Configure an OTLP exporter endpoint and headers for your backend.

**Env vars**
- `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_EXPORTER_OTLP_HEADERS`.

**Verify**
- [ ] A request shows as a trace with spans for database and fetch calls.

### Vercel Web Analytics / Speed Insights

**Setup**
- [ ] Enable in the project dashboard; add `<Analytics />` from `@vercel/analytics/react` and `<SpeedInsights />` from `@vercel/speed-insights/next`.

**Verify**
- [ ] Visits appear after deploying.

## Related

- [`jobs.md`](jobs.md), [`costs.md`](costs.md), [`environments.md`](environments.md).
