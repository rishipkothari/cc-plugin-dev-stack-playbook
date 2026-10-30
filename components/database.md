# Database

Where the app's data lives. The data shape (relational, document, key-value, local-first) is a big-picture decision; this file picks the product.

## Decide

### Questions to ask

- Relational data with relationships and transactions, or flexible documents?
- How much data, and how many writes? (Most personal projects are tiny.)
- Does it run from serverless or edge functions? (Connection handling matters.)
- Do previews need their own copy of the data?
- Do you need realtime updates, full-text search or vector search?
- What do you want to learn: Postgres deeply, SQLite at the edge, documents, or a reactive database?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Neon (Postgres) | ★ default | Serverless Postgres with branching and scale-to-zero | Cold start after idle |
| Supabase (Postgres) | ◆ alternative | Postgres plus auth, storage, realtime in one platform | Free projects pause after inactivity |
| MongoDB Atlas | ◆ alternative | Document data, flexible schemas | Weaker relations and constraints |
| Upstash Redis | ◆ alternative | Caching, rate limits, sessions, counters (HTTP-based) | Not a primary database for most apps |
| Turso / libSQL (SQLite) | ✦ distinctive | SQLite with replicas, database-per-user patterns | SQLite feature set; younger platform |
| Cloudflare D1 | ✦ distinctive | SQLite bound directly to Workers | Cloudflare-only |
| Convex | ✦ distinctive | Reactive document database with server functions | Proprietary query model |

### Option notes

**Neon** ★
- Pros: plain Postgres (portable); serverless HTTP/WebSocket driver works from serverless and edge; branching gives each preview its own database copy; scales to zero; Vercel integration injects `DATABASE_URL`.
- Cons: first query after idle has a short cold start; free tier has compute and storage caps.
- Pick when: any relational app, especially on Vercel.
- What you'll learn: Postgres, connection pooling, database branching.

**Supabase** ◆
- Pros: Postgres plus auth, storage, realtime, edge functions and an admin UI; Row Level Security lets the client query safely; generated client libraries.
- Cons: free projects pause after a period of inactivity; the all-in-one model encourages lock-in.
- Pick when: you want auth and storage bundled with the database, or realtime.
- What you'll learn: Row Level Security policies, Postgres roles, the BaaS model.

**MongoDB Atlas** ◆
- Pros: flexible documents, nested data, rich query language, free shared cluster.
- Cons: joins and constraints are weaker; schema discipline is on you.
- Pick when: data really is document-shaped (varied records, nested content), or to learn NoSQL.
- What you'll learn: document modeling, aggregation pipelines, indexes.

**Upstash Redis** ◆
- Pros: HTTP API works from serverless and edge; pay-per-request; libraries for rate limiting (`@upstash/ratelimit`).
- Cons: key-value, not relational.
- Pick when: alongside a main database for caching, rate limits or queues.
- What you'll learn: key-value patterns, TTLs, rate-limiting algorithms.

**Turso / libSQL** ✦
- Pros: SQLite over the network with embedded replicas (read locally, sync from primary); cheap to create many databases (one per user/tenant).
- Cons: SQLite types and concurrency model; smaller ecosystem than Postgres.
- Pick when: read-heavy apps, per-tenant databases, or learning SQLite in production.
- What you'll learn: SQLite, replication, database-per-tenant architecture.

**Cloudflare D1** ✦
- Pros: SQLite as a Workers binding, no connection strings; time-travel restore.
- Cons: Cloudflare-only; size limits per database.
- Pick when: the app is on Cloudflare Workers.
- What you'll learn: bindings, SQLite at the edge.

**Convex** ✦
- See [backend](backend.md): the database and server functions come together. Pick when realtime is central.

### How to choose

1. Relational and on Vercel/serverless → Neon.
2. Want auth/storage/realtime bundled → Supabase.
3. On Cloudflare Workers → D1 (or Turso).
4. Document-shaped data → MongoDB Atlas.
5. Realtime collaborative app → Convex or Supabase realtime.
6. Need caching or rate limits too → add Upstash Redis alongside.

## Configure

### Neon

**Setup**
- [ ] Create a project at neon.tech (or through the Vercel Marketplace, which links env vars automatically). Pick a region close to the app's functions.
- [ ] Copy the pooled connection string (host contains `-pooler`) for the app.
- [ ] Install a driver: `@neondatabase/serverless` (HTTP/WebSocket, best for serverless/edge), or `pg`/`postgres` for long-running servers.
- [ ] Create the schema (see [migrations](migrations.md)).
- [ ] Optional: enable branching for previews (see [environments](environments.md)).

**Env vars**
- `DATABASE_URL`: pooled connection string, with `sslmode=require`.
- Optionally `DATABASE_URL_UNPOOLED` for migrations that need a direct connection.

**Verify**
- [ ] A local script runs `select 1` against the database.
- [ ] A deployed route reads data.

**Gotchas**
- The app can build without a real `DATABASE_URL` but fails at runtime; check runtime, not just the build.
- Function region and database region should match, or every query crosses the world.

### Supabase

**Setup**
- [ ] Create a project; save the database password.
- [ ] Client: `npm i @supabase/supabase-js` (and `@supabase/ssr` for SSR frameworks).
- [ ] Enable Row Level Security on every table and write policies before exposing tables to the client.
- [ ] For direct SQL from servers, use the pooled connection string (Supavisor).

**Env vars**
- `SUPABASE_URL` / `NEXT_PUBLIC_SUPABASE_URL`, the publishable (anon) key for clients, and the secret (service role) key for server-only admin tasks. The service role key bypasses RLS: never ship it to the browser.

**Verify**
- [ ] With RLS on, an anonymous client cannot read protected rows.

**Gotchas**
- Free projects pause after inactivity; un-pause from the dashboard.

### MongoDB Atlas

**Setup**
- [ ] Create a free cluster; create a database user.
- [ ] Network access: allow the host's IPs, or `0.0.0.0/0` for serverless (then rely on strong credentials).
- [ ] `npm i mongodb` (or Mongoose); reuse one client across requests (cache it on `globalThis` in serverless).

**Env vars**
- `MONGODB_URI`.

**Verify**
- [ ] Insert and read a document from a deployed function.

### Upstash Redis

**Setup**
- [ ] Create a database (or via the Vercel Marketplace).
- [ ] `npm i @upstash/redis` (and `@upstash/ratelimit` if needed).

**Env vars**
- `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`.

**Verify**
- [ ] `set` then `get` a key from a deployed function.

### Turso / libSQL

**Setup**
- [ ] Install the Turso CLI; `turso db create <name>`.
- [ ] `turso db show <name> --url` and `turso db tokens create <name>`.
- [ ] `npm i @libsql/client`.

**Env vars**
- `TURSO_DATABASE_URL`, `TURSO_AUTH_TOKEN`.

**Verify**
- [ ] Query from a deployed function; for embedded replicas, confirm sync.

### Cloudflare D1

**Setup**
- [ ] `npx wrangler d1 create <name>`; add the binding to the wrangler config.
- [ ] Migrations: `npx wrangler d1 migrations create` / `apply` (local and `--remote`).

**Verify**
- [ ] `env.DB.prepare("select 1").first()` works in `wrangler dev` and deployed.

### Convex

See [backend](backend.md#convex).

## Related

- [`migrations.md`](migrations.md), [`environments.md`](environments.md), [`backend.md`](backend.md).
