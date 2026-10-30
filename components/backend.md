# Backend / API

Where server-side logic lives: reading and writing data, calling third-party APIs with secret keys, enforcing permissions. In a full-stack framework this is just part of the app; otherwise it's a separate service.

## Decide

### Questions to ask

- Will anything other than this web app call the API (a mobile app, other services, the public)?
- Does the backend need a different language (Python for data/AI work)?
- Any long-running work, WebSockets or realtime updates?
- How much backend do you want to build vs. rent (backend-as-a-service)?
- What do you want to learn: REST design, typed RPC, a new language, or a hosted backend model?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Framework server code (Next.js route handlers / server actions, SvelteKit endpoints) | ★ default | One app that owns its UI and data | API isn't a clean standalone product |
| Hono | ◆ alternative | Small, fast TypeScript APIs on any runtime (Node, Workers, Bun) | You assemble validation, auth, ORM yourself |
| Express / Fastify | ◆ alternative | Classic Node servers, huge middleware ecosystem | Long-running server to host |
| FastAPI (Python) | ◆ alternative | Python backends, data/AI work, auto-generated OpenAPI docs | Second language and toolchain |
| tRPC | ✦ distinctive | End-to-end typed calls between a TS client and server | TS-only clients; not a public REST API |
| PocketBase | ✦ distinctive | A whole backend (DB, auth, files, realtime) in one Go binary | Single-server SQLite; you host it |
| Convex | ✦ distinctive | Reactive backend: queries auto-update the UI | Proprietary platform and query model |

### Option notes

**Framework server code** ★
- Pros: no separate service; secrets and database access stay server-side; server actions remove a lot of fetch boilerplate; deploys with the app.
- Cons: harder to reuse from a mobile app; serverless time limits apply.
- Pick when: the web app is the only client.
- What you'll learn: request handling, server actions, validating input at the boundary.

**Hono** ◆
- Pros: tiny, web-standard `Request`/`Response`, runs on Node, Bun, Deno, Cloudflare Workers and Vercel; typed client (`hc`) and Zod validation middleware.
- Cons: minimal by design, so you choose each piece.
- Pick when: a standalone TS API, especially on the edge.
- What you'll learn: routing and middleware fundamentals, multi-runtime code, edge APIs.

**Express / Fastify** ◆
- Pros: Express is the most documented Node server; Fastify is faster with schema-based validation.
- Cons: needs an always-on host; Express is older in style.
- Pick when: a traditional server with WebSockets or background loops.
- What you'll learn: the classic Node server model, middleware chains.

**FastAPI** ◆
- Pros: type hints drive validation (Pydantic) and generate OpenAPI docs at `/docs`; async; best fit for Python AI and data libraries.
- Cons: two languages in the project; separate deploy; CORS and auth across origins.
- Pick when: Python is the right tool for the backend work, or to learn Python web development.
- What you'll learn: Python packaging (uv), Pydantic, OpenAPI, ASGI servers.

**tRPC** ✦
- Pros: call server functions from the client with full type inference and no schema files.
- Cons: only TypeScript clients benefit; not suitable as a public API.
- Pick when: a TS monorepo where client and server ship together.
- What you'll learn: end-to-end type safety, RPC vs. REST trade-offs.

**PocketBase** ✦
- Pros: one executable gives you SQLite, auth, file storage, realtime subscriptions and an admin UI; extend with Go or JavaScript hooks.
- Cons: single-server; you host it (Fly.io, a VPS).
- Pick when: small apps, prototypes, or learning the BaaS model without a big vendor.
- What you'll learn: BaaS concepts, SQLite in production, self-hosting.

**Convex** ✦
- Pros: write queries and mutations as TS functions; clients subscribe and update live; built-in scheduling, file storage, auth integrations.
- Cons: proprietary platform and data model; harder to leave.
- Pick when: realtime, collaborative apps, or to learn the reactive backend model.
- What you'll learn: reactive queries, transactional functions, a very different data model from SQL.

### How to choose

1. Only the web app calls it → framework server code.
2. Python needed → FastAPI.
3. Standalone TS API → Hono (edge or serverless) or Fastify (long-running).
4. Realtime/collaboration → Convex, or PocketBase realtime; WebSockets on a long-running host otherwise.
5. Typed TS monorepo → add tRPC on top of the framework.

## Configure

### Framework server code (Next.js)

**Setup**
- [ ] Route handlers in `app/api/<name>/route.ts` for endpoints called by fetch, webhooks or external clients.
- [ ] Server actions (`"use server"`) for form submissions and mutations from your own UI.
- [ ] Validate every input with a schema library (Zod or Valibot) at the boundary.
- [ ] Check auth and permissions inside each handler/action, not only in middleware.
- [ ] Centralize data access in server-only modules (`src/lib/db.ts`, etc.).

**Verify**
- [ ] Unauthenticated requests to protected handlers return 401/403.
- [ ] Invalid input returns 400 with a useful message.

**Gotchas**
- Server actions are public HTTP endpoints; treat them exactly like API routes.
- Webhook handlers need the raw request body for signature verification (see [apis](apis.md)).

### Hono

**Setup**
- [ ] `npm create hono@latest` → choose the runtime template.
- [ ] Add `@hono/zod-validator` for input validation.
- [ ] Add CORS middleware if a browser on another origin calls it.
- [ ] Export the typed app (`export type AppType = typeof app`) for the `hc` client.

**Verify**
- [ ] Runs locally with the template's dev command; deployed URL responds.

### Express / Fastify

**Setup**
- [ ] `npm i fastify` (or `express`), TypeScript with `tsx` for dev.
- [ ] Listen on `process.env.PORT`.
- [ ] Add a `/health` route for the host's health check.
- [ ] Structured logging (Fastify's built-in pino, or pino for Express).
- [ ] Graceful shutdown on `SIGTERM`.

**Verify**
- [ ] Health check passes on the host; logs appear in the host's log viewer.

### FastAPI

**Setup**
- [ ] `uv init`, then `uv add fastapi uvicorn` (or `fastapi[standard]`).
- [ ] Dev: `uv run fastapi dev main.py`; prod: `uvicorn main:app --host 0.0.0.0 --port $PORT`.
- [ ] Pydantic models for request and response bodies.
- [ ] `CORSMiddleware` allowing only your frontend origins.
- [ ] Settings via `pydantic-settings` reading env vars.

**Verify**
- [ ] `/docs` shows the OpenAPI UI; the frontend can call it from its deployed origin.

**Gotchas**
- Use `async` routes with async database drivers; don't block the event loop with sync I/O.

### tRPC

**Setup**
- [ ] Install `@trpc/server`, `@trpc/client` and the framework adapter (e.g. TanStack Query integration).
- [ ] Define a router with procedures; validate inputs with Zod.
- [ ] Add auth context so protected procedures check the session.

**Verify**
- [ ] Changing a procedure's return type breaks the client at compile time.

### PocketBase

**Setup**
- [ ] Download the binary (or Docker image); `./pocketbase serve`.
- [ ] Create collections and API rules in the admin UI (`/_/`).
- [ ] Client: `npm i pocketbase`.
- [ ] Production: run on a host with a persistent volume for `pb_data`; back it up.

**Verify**
- [ ] API rules block unauthorized reads/writes; backups restore.

### Convex

**Setup**
- [ ] `npm i convex`, then `npx convex dev` (creates a project and `convex/` folder).
- [ ] Define the schema in `convex/schema.ts`; write queries and mutations in `convex/`.
- [ ] Wrap the app in `ConvexProvider`; use `useQuery` / `useMutation`.
- [ ] Production: `npx convex deploy`, set `CONVEX_DEPLOY_KEY` in the host for builds.

**Verify**
- [ ] Two browser windows update live when data changes.

## Related

- [`database.md`](database.md), [`auth.md`](auth.md), [`apis.md`](apis.md).
- Skill: if the backend calls Claude, use the `claude-api` skill if available.
