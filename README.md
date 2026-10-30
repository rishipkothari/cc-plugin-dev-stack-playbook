# Dev Stack Playbook

A personal, learning-oriented playbook for choosing and setting up the tech stack of a new project, packaged as a Claude plugin (`stack-playbook`).

It has three layers:

1. **Decide.** Big-picture decisions (below), then every stack component with its options, pros and cons, "pick when", and what you'll learn by using it.
2. **Select.** Each project gets a `docs/stack.md` in its own repo that records the decisions, the reasoning and the final choices.
3. **Configure.** Each component file has a Configure half: the checklist you work from while actually implementing.

Plus a central history, [`projects.md`](projects.md), of every project and what it used, with a coverage summary of what you've tried and what you haven't.

The goal is breadth of skill, not just shipping. Every component lists a **default** (most recommended), **alternatives** (popular, with a different trade-off) and **distinctive** picks (less common, but each teaches something the defaults don't).

## Contents

- [How to use it](#how-to-use-it)
- [Install and update](#install-and-update)
- [Big-picture decisions](#big-picture-decisions)
- [Component map](#component-map)
- [Repository layout](#repository-layout)

## How to use it

With the plugin installed, just talk to Claude. The skills trigger on their own when a request matches, or you can call them by name:

| Skill | Say something like | What it does |
| :- | :- | :- |
| `/stack-playbook:plan-stack` | "Let's plan a new project" | Interviews you through the big-picture decisions and each relevant component, then writes `docs/stack.md` in the project |
| `/stack-playbook:setup-component` | "Set up auth for this project" | Works through one component's Configure checklist, explaining each step and offering to let you do it yourself |
| `/stack-playbook:log-project` | "Log this project" | Adds or updates the project's entry in `projects.md` and regenerates the coverage summary |
| `/stack-playbook:add-option` | "Add Convex to the playbook" | Adds a new option to a component file (Decide and Configure) when a project uses something not yet covered |

Without Claude, read this README top to bottom for the overview, then open the component files you need.

### The "one new thing" rule

Every project picks at least one option that's new to you in at least one layer. `plan-stack` reads the coverage summary in `projects.md` to suggest candidates. New options are marked ★ in `docs/stack.md` and in `projects.md`.

## Install and update

The plugin is listed in the marketplace repo [`rishipkothari/cc-marketplace-rishipkothari`](https://github.com/rishipkothari/cc-marketplace-rishipkothari).

**claude.ai / Claude Desktop (recommended: follows your account everywhere).**

1. Open **Customize**, then the **Plugins** tab.
2. **Add**, then **Add marketplace**, then **Add from a repository**: `rishipkothari/cc-marketplace-rishipkothari`.
3. Install **stack-playbook** from that marketplace.

Plugins added this way are saved to your account, so they also appear in Cowork and in Claude Code when signed in with the same Claude account (Claude Code v2.1.273+, synced at each start).

**Claude Code directly (per device).**

```
/plugin marketplace add rishipkothari/cc-marketplace-rishipkothari
/plugin install stack-playbook@rishipkothari-plugins
```

Then turn on auto-update: `/plugin` → **Marketplaces** → `rishipkothari-plugins` → **Enable auto-update**. Without it, update with `/plugin marketplace update rishipkothari-plugins`.

**Updates.** `plugin.json` deliberately has no `version`, so every commit to `main` counts as a new version. Do not add a `version` field unless you also bump it on every change, or installs will stop receiving updates.

**History is not in the installed copy.** The installed plugin is a read-only cache. `log-project` writes `projects.md` to this GitHub repo, so every device sees the same history.

## Big-picture decisions

Make these first. They shape almost every component choice.

### 1. Application shape

| Shape | What it is | Pros | Cons | Pick when |
| :- | :- | :- | :- | :- |
| **Static site** | Pre-built HTML/JS files, no server code | Free to host anywhere, fast, almost nothing to break | No server-side secrets or database without a separate service | Content sites, docs, portfolios, client-only tools |
| **Full-stack framework** (default) | One app that renders pages and runs server code (Next.js, SvelteKit, Nuxt, React Router) | One repo, one deploy, server code next to UI, secrets stay on the server | Couples frontend and backend; framework conventions to learn | Most web apps with a database and a UI |
| **SPA + separate API** | A client-side app talking to its own backend service | Clean boundary; API reusable by mobile apps or others; backend in any language | Two deploys, CORS, auth across origins, more moving parts | A real API consumer exists, a Python/Go backend is wanted, or to learn the split |
| **Backend-as-a-service** | Client app plus a hosted backend (Supabase, Firebase, Convex, PocketBase) | Database, auth, storage and realtime in one place, very fast start | Vendor lock-in; harder custom logic; you learn less about how the backend works | Prototypes, realtime apps, or to learn that model |
| **Server-rendered with htmx** | Server returns HTML fragments, little or no client JS | Simple mental model, tiny client, any backend language | Less suited to rich interactive UIs; smaller ecosystem | Forms-and-tables apps; learning the hypermedia approach |

### 2. Runtime model

| Model | Pros | Cons | Pick when |
| :- | :- | :- | :- |
| **Serverless functions** (default on Vercel/Netlify) | Scale to zero, no servers to manage, pay per use | Cold starts, execution time limits, no persistent connections or in-memory state | Request/response apps with spiky or low traffic |
| **Edge functions** (Cloudflare Workers, Vercel Edge) | Run close to users, near-zero cold start | Restricted runtime (not full Node), limited libraries, database is usually far away | Latency-sensitive, light logic, or learning the edge model |
| **Long-running server** (Render, Railway, Fly.io, a VPS) | WebSockets, background loops, in-memory caches, any language | Always-on cost, you handle scaling and restarts | Realtime, queues and workers, heavy jobs, non-JS backends |

### 3. Language

- **TypeScript everywhere** (default): one language front to back, shared types, the largest web ecosystem.
- **TypeScript frontend + Python backend**: best for data, ML or AI-heavy work; teaches API boundaries. Costs two toolchains.
- **Other backend languages** (Go, Elixir, Ruby, Rust): worth one project each for learning; smaller hosting presets and fewer examples.

### 4. All-in-one platform vs. best-of-breed

- **All-in-one** (Supabase, Firebase, Convex): one dashboard, one bill, integrated auth and data. Lock-in, and when you outgrow one piece you migrate everything.
- **Best-of-breed** (Vercel + Neon + Better Auth + Resend + …): pick the best tool per layer, swap pieces independently. More accounts, more env vars, more integration work. Teaches more.

### 5. Users and accounts

- **No accounts**: public or single-user tool. Simplest.
- **Shared password**: a few trusted people, no per-user data (nanny-comp-log does this).
- **Real user accounts**: per-user data, sign-up, password reset, OAuth. Pulls in auth, email, and usually authorization rules in the database.
- **Multi-tenant** (organizations or teams): every query is scoped to a tenant. Decide this up front; retrofitting is painful.

### 6. Data shape

- **Relational (Postgres, SQLite)** (default): structured data with relationships, transactions, SQL. Fits most apps.
- **Document (MongoDB, Firestore)**: flexible nested records, schema-less. Weaker joins and constraints.
- **Key-value (Redis, Upstash, Workers KV)**: caches, sessions, rate limits, counters. Rarely the main database.
- **Local-first / sync engines** (Replicache, Zero, ElectricSQL, PowerSync): data lives on the device and syncs. Offline support and instant UI; a newer, very different model worth learning once.

### 7. Budget, scale and lock-in

- Write down the monthly budget (often $0) and check free-tier limits for every service chosen. See [costs](components/costs.md).
- Hobby tiers often forbid commercial use (Vercel Hobby does). Decide whether this project could ever make money.
- Prefer portable layers where it's cheap to do so: plain Postgres over a proprietary database, standard env vars, an ORM that supports several databases.

### 8. Learning goal

Name what this project should teach you. It can override "most practical": choosing a distinctive option on a low-stakes project is how breadth gets built.

## Component map

Each component file has a **Decide** half (options, trade-offs, questions) and a **Configure** half (checklists). ★ default · ◆ alternative · ✦ distinctive.

| Component | ★ Default | ◆ Alternatives | ✦ Distinctive |
| :- | :- | :- | :- |
| [Hosting](components/hosting.md) | Vercel | Netlify, Cloudflare Pages/Workers, Render, Railway | Fly.io, VPS with Coolify, GitHub Pages |
| [Frontend framework](components/framework.md) | Next.js (App Router) | Vite + React, React Router v7, SvelteKit, Astro | htmx, SolidStart, Expo (mobile) |
| [Backend / API](components/backend.md) | Framework server code (route handlers, server actions) | Hono, Express/Fastify, FastAPI | tRPC, PocketBase, Convex |
| [Database](components/database.md) | Neon Postgres | Supabase, MongoDB Atlas, Upstash Redis | Turso / SQLite, Cloudflare D1, Convex |
| [Schema and migrations](components/migrations.md) | Drizzle Kit | Prisma Migrate, plain SQL scripts | Atlas, Supabase CLI |
| [Auth](components/auth.md) | Better Auth | Clerk, Supabase Auth, shared password | Hand-rolled sessions, passkeys, magic links |
| [Secrets and env vars](components/secrets.md) | `.env.local` + host env vars | Vercel CLI `env pull` | 1Password CLI, Doppler, Infisical, dotenvx |
| [Payments](components/payments.md) | Stripe Checkout | Stripe Billing, Lemon Squeezy, Paddle | Polar, Stripe Connect |
| [Third-party APIs](components/apis.md) | Official SDK + server-only key | OAuth (Google APIs), webhooks | MCP servers, Vercel AI SDK |
| [File storage](components/storage.md) | Vercel Blob | Cloudflare R2, AWS S3, Supabase Storage | UploadThing |
| [Email](components/email.md) | Resend (+ React Email) | Postmark, AWS SES | Inbound email parsing, Mailpit for local dev |
| [Scheduled and background jobs](components/jobs.md) | Vercel Cron | GitHub Actions schedule, Upstash QStash | Inngest, Trigger.dev, Cloudflare Queues |
| [Domains and DNS](components/domains.md) | Cloudflare Registrar + host's DNS records | Porkbun, Namecheap, buy through the host | Cloudflare as DNS proxy |
| [Environments](components/environments.md) | Local / Preview / Production | Staging branch | Database branching per preview (Neon) |
| [Testing](components/testing.md) | Vitest + Playwright | Testing Library, Jest, pytest | MSW, property-based testing, visual regression |
| [CI/CD](components/ci-cd.md) | GitHub Actions + host git integration | Dependabot / Renovate | Branch protection + required checks, release automation |
| [Monitoring and logging](components/monitoring.md) | Host logs + Sentry | Axiom, Better Stack | PostHog, OpenTelemetry |
| [Costs](components/costs.md) | Free tiers + spend limits | Usage alerts | Per-project cost log |

## Repository layout

```
.claude-plugin/plugin.json      plugin manifest (no version: every commit is an update)
skills/
  plan-stack/                   interview → docs/stack.md (template included)
  setup-component/              implement one component from its Configure half
  log-project/                  update projects.md and its coverage summary
  add-option/                   add a new option to a component file
components/                     one file per component: Decide + Configure
projects.md                     central project history + coverage
README.md                       this overview
```

Every component file uses the same headings so Claude (and you) can jump to a section: `## Decide` → `### Questions to ask`, `### Options`, `### Option notes`, `### How to choose`; `## Configure` → one `### <Option>` per option with `Setup`, `Env vars`, `Verify`, `Gotchas`; `## Related` (other components and skills).

Facts in these files (free tiers, product names, dashboard paths) drift. When a setup step doesn't match what you see, trust the provider's current docs and fix the file with `add-option` or a direct edit.
