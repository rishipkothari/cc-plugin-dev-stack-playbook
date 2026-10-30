# Costs

Keeping track of what each project costs and making sure a free project can't surprise you with a bill. Most personal projects can run at $0; the risk is usage-based services without a cap.

## Decide

### Questions to ask

- What's the monthly budget? ($0 is a valid answer.)
- Which services are usage-billed (AI APIs, functions, bandwidth, email, storage)?
- Which free tiers forbid commercial use or pause inactive projects?
- What's the worst case if something loops or gets abused (a bot hammering an AI endpoint)?
- Do you want a running record of costs across projects?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Free tiers + hard spend limits | ★ default | Personal and learning projects | Limits can pause the app |
| Usage/budget alerts | ◆ alternative | Paid tiers where you want warnings, not hard stops | Alerts can be missed |
| Per-project cost log | ✦ distinctive | Seeing what each project really costs over time | Manual upkeep |
| Rate limiting expensive endpoints | ◆ alternative | AI and email endpoints exposed to users | Code to maintain |

### Option notes

**Free tiers + spend limits** ★
- Pros: predictable; many providers let you set a hard monthly cap (AI API consoles, Vercel spend management on Pro, Railway usage limits).
- Cons: hitting the cap can stop the service.
- What you'll learn: where each provider's limits are.

**Budget alerts** ◆
- Pros: no surprise shutdowns; AWS Budgets, Google Cloud budgets and most dashboards support them.
- What you'll learn: cloud billing consoles.

**Per-project cost log** ✦
- Pros: a table in `docs/stack.md` with each service, plan and monthly cost; reviewed occasionally.
- What you'll learn: the real cost shape of the stacks you build.

**Rate limiting** ◆
- Pros: protects expensive endpoints from abuse (Upstash ratelimit, per-user quotas).
- What you'll learn: rate-limit algorithms, abuse prevention.

### How to choose

1. Always: note each service's plan and limits in `docs/stack.md`.
2. Any usage-billed API → set a hard limit or budget alert before writing code.
3. Public endpoints that cost money per call → rate limit them.

## Configure

### Free tiers + spend limits

**Setup**
- [ ] For each service in the stack, record: plan, free-tier limits, what happens at the limit, and whether commercial use is allowed.
- [ ] Set hard limits where available:
  - AI APIs (Anthropic, OpenAI): monthly spend limit in the console.
  - Vercel: Hobby has hard usage limits; on Pro, configure Spend Management.
  - Railway / Render / Fly.io: usage limits or billing alerts.
  - AWS / Google Cloud: budgets with alerts (they can't hard-stop by default).
- [ ] Note services that pause on inactivity (e.g. free Supabase projects, free Render services).

**Verify**
- [ ] Every usage-billed service in `docs/stack.md` has a limit or alert recorded.

### Usage/budget alerts

**Setup**
- [ ] Create alerts at 50% / 90% of budget per provider.

**Verify**
- [ ] Alert emails go to an inbox you read.

### Per-project cost log

**Setup**
- [ ] In the project's `docs/stack.md`, a "Costs" table: service, plan, monthly cost, notes.
- [ ] Review when logging the project with `log-project`.

### Rate limiting

**Setup**
- [ ] `@upstash/ratelimit` (or a database counter) keyed by user or IP on expensive routes.
- [ ] Return 429 with a friendly message.

**Verify**
- [ ] Rapid repeated calls get 429 after the limit.

## Related

- [`monitoring.md`](monitoring.md), [`apis.md`](apis.md), [`hosting.md`](hosting.md).
