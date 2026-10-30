# Environments

The separate copies of the app (and its data and secrets): local development, preview deployments, maybe staging, and production. The goal is to test changes realistically without touching real data.

## Decide

### Questions to ask

- Is it fine for previews to share the production database? (Usually not, once real data exists.)
- Do previews need realistic data, or is seed data enough?
- Do you want a long-lived staging environment, or are per-branch previews enough?
- Which third-party services have test modes (Stripe test, Clerk dev instance) and which need separate accounts?
- What do you want to learn: basic env separation, or database branching per preview?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Local / Preview / Production | ★ default | Most projects on Vercel/Netlify | Previews need their own data strategy |
| Staging branch | ◆ alternative | Testing a release candidate in a stable place | One more environment to keep in sync |
| Database branching per preview (Neon) | ✦ distinctive | Each preview gets a copy of the database | Setup and cleanup automation |
| Local database in Docker | ◆ alternative | Offline dev, safe experiments | Docker on every device |
| Seed scripts | ◆ alternative | Repeatable fake data for local and previews | Must keep in sync with schema |

### Option notes

**Local / Preview / Production** ★
- Pros: built into Vercel and Netlify; env vars scoped per environment.
- What you'll learn: environment scoping, preview workflows.

**Staging branch** ◆
- Pros: a stable URL (`staging.example.com`) for a fixed environment.
- What you'll learn: release flow (feature → staging → main).

**Neon branching** ✦
- Pros: branches copy the database instantly (copy-on-write); the Vercel integration can create one per preview and inject its URL.
- What you'll learn: copy-on-write databases, ephemeral environments.

**Local Docker database** ◆
- Pros: free, offline, disposable.
- What you'll learn: Docker Compose.

**Seed scripts** ◆
- Pros: known data for demos and tests.
- What you'll learn: fixtures, data factories.

### How to choose

1. Personal app with real data → production DB for production only; previews use a branch or a separate dev database.
2. No real data yet → one dev database for local and previews is fine.
3. Learning → Neon per-preview branching.

## Configure

### Common to every option

- [ ] List every env var and its value source per environment in `docs/stack.md`.
- [ ] Use test/sandbox keys outside production (Stripe test mode, Clerk dev instance, separate OAuth clients if needed).
- [ ] Make it visible which environment you're in (a banner when not production, using `VERCEL_ENV` or similar).
- [ ] Never run destructive scripts (reset, seed) against production; check the env in the script and refuse.

### Local / Preview / Production (Vercel)

**Setup**
- [ ] For each env var, choose scopes: Production, Preview, Development.
- [ ] Optionally scope Preview values to a specific branch.
- [ ] Optionally enable Deployment Protection for previews.

**Verify**
- [ ] A preview uses preview values (e.g. the dev database), production uses production values.

### Staging branch

**Setup**
- [ ] Create a `staging` branch; assign a domain to it in **Settings → Domains** (git branch).
- [ ] Scope env vars to that branch if they differ.

**Verify**
- [ ] Pushing to `staging` updates the staging domain only.

### Neon branching per preview

**Setup**
- [ ] Install the Neon integration on Vercel (or use the Neon GitHub Action) with preview branching enabled.
- [ ] Ensure migrations run against the preview branch (e.g. in the build command for previews, or a GitHub Action).
- [ ] Configure cleanup of branches when PRs close.

**Verify**
- [ ] A PR's preview writes data that doesn't appear in production.

### Local database in Docker

**Setup**
- [ ] `docker-compose.yml` with `postgres:<version>` and a volume.
- [ ] `.env.local` points at `postgres://postgres:postgres@localhost:5432/app`.
- [ ] Run migrations and seeds against it.

**Verify**
- [ ] `docker compose down -v && docker compose up -d` plus migrate + seed gives a working app.

### Seed scripts

**Setup**
- [ ] `scripts/seed.ts` that inserts representative data; refuse to run when the env is production.
- [ ] npm script `db:seed`.

**Verify**
- [ ] Fresh database + seed = usable app.

## Related

- [`database.md`](database.md), [`secrets.md`](secrets.md), [`ci-cd.md`](ci-cd.md).
