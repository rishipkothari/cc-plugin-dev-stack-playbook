# Schema and migrations

How the database schema is defined, changed over time and kept in sync across local, preview and production databases. Often paired with a query layer (ORM or query builder).

## Decide

### Questions to ask

- Do you want to write SQL directly, or define the schema in TypeScript/Python?
- Will the schema change often after launch? (Then real migration files matter.)
- Do multiple environments need the same schema applied reliably?
- Do you want type-safe queries generated from the schema?
- What do you want to learn: SQL itself, an ORM, or declarative schema management?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Drizzle (ORM + Drizzle Kit) | ★ default | TS schema, SQL-like typed queries, generated migration files | You still need to understand SQL |
| Prisma (ORM + Prisma Migrate) | ◆ alternative | Schema-first modeling, very approachable client | Its own schema language; heavier abstraction |
| Plain SQL scripts | ◆ alternative | Small apps; learning SQL; full control | No automatic diffing; you track what has run |
| Atlas | ✦ distinctive | Declarative schema: describe the target, it plans the migration | Another tool to learn |
| Supabase CLI migrations | ✦ distinctive | Supabase projects with local dev stack | Supabase-specific |
| Alembic (with SQLAlchemy) | ◆ alternative | Python backends | Python-only |

### Option notes

**Drizzle** ★
- Pros: schema in TypeScript; queries read like SQL with full types; `drizzle-kit generate` writes SQL migration files you can read and commit; works with Neon, Postgres, SQLite, Turso, D1, MySQL; light runtime, good on serverless and edge.
- Cons: more SQL knowledge needed than Prisma; relational query API is a second style to learn.
- Pick when: TypeScript app with a SQL database.
- What you'll learn: SQL through a typed lens, migration files, schema-as-code.

**Prisma** ◆
- Pros: readable `schema.prisma`, excellent docs, Prisma Studio for browsing data, mature migrations.
- Cons: abstraction hides SQL; its own schema language; check edge/serverless driver adapter setup.
- Pick when: you want the most approachable ORM, or to compare with Drizzle.
- What you'll learn: ORM modeling, relations, migration workflow.

**Plain SQL scripts** ◆
- Pros: nothing to install; you see exactly what runs; great for learning SQL.
- Cons: you must make scripts idempotent (`create table if not exists`, `alter table … add column if not exists`) or track applied migrations yourself.
- Pick when: small schemas (nanny-comp-log uses `scripts/setup.ts`), or learning SQL directly.
- What you'll learn: DDL, idempotent migrations, why migration tools exist.

**Atlas** ✦
- Pros: declare the desired schema (HCL, SQL or from an ORM); Atlas diffs and plans safe migrations, lints for destructive changes, integrates with CI.
- Cons: extra tool alongside the query layer.
- Pick when: learning declarative schema management or adding migration safety checks to CI.
- What you'll learn: desired-state vs. versioned migrations, migration linting.

**Supabase CLI** ✦
- Pros: local Postgres + auth + storage in Docker; `supabase db diff` generates migrations; `supabase db push` applies them.
- Cons: tied to Supabase; needs Docker.
- Pick when: the database is Supabase.
- What you'll learn: local-first database workflow, migration diffing.

**Alembic** ◆
- Pros: the standard for SQLAlchemy; autogenerates migrations from models.
- Pick when: FastAPI/Python backend with SQLAlchemy or SQLModel.

### How to choose

1. TypeScript + SQL → Drizzle (Prisma if you prefer its style).
2. Tiny schema or learning SQL → plain SQL scripts.
3. Supabase → Supabase CLI (optionally with Drizzle for queries).
4. Python → SQLAlchemy/SQLModel + Alembic.
5. Want safety checks in CI → add Atlas.

## Configure

### Drizzle

**Setup**
- [ ] `npm i drizzle-orm` and `npm i -D drizzle-kit`; plus the driver (e.g. `@neondatabase/serverless`).
- [ ] Define tables in `src/db/schema.ts`.
- [ ] `drizzle.config.ts` with `dialect`, `schema` path, `out: "./drizzle"` and `dbCredentials.url` from env.
- [ ] Scripts: `"db:generate": "drizzle-kit generate"`, `"db:migrate": "drizzle-kit migrate"`, `"db:studio": "drizzle-kit studio"`.
- [ ] Commit the generated `drizzle/` migration files.
- [ ] Decide when production migrations run: manually before deploy, or in CI on merge (see [ci-cd](ci-cd.md)).

**Env vars**
- `DATABASE_URL` (a direct, unpooled URL is safer for running migrations).

**Verify**
- [ ] `db:generate` produces a SQL file you can read; `db:migrate` applies it to a fresh database; the app's typed queries compile.

**Gotchas**
- `drizzle-kit push` syncs the schema without migration files: fine for prototyping, risky for production.
- Review generated SQL for renames (the tool may drop and re-create a column).

### Prisma

**Setup**
- [ ] `npm i -D prisma` and `npm i @prisma/client`; `npx prisma init`.
- [ ] Model in `prisma/schema.prisma`.
- [ ] `npx prisma migrate dev --name <change>` locally; `npx prisma migrate deploy` in production.
- [ ] Run `prisma generate` during build (add to `postinstall` or the build script).
- [ ] For serverless/edge, follow the current docs for driver adapters (e.g. Neon adapter).

**Verify**
- [ ] `migrate deploy` applies cleanly against a fresh database.

### Plain SQL scripts

**Setup**
- [ ] One `scripts/setup.(ts|sql)` with idempotent DDL, or numbered files `migrations/001_init.sql`, `002_…`.
- [ ] For numbered files, keep a `schema_migrations` table recording which files ran; a small runner applies the rest in order.
- [ ] Wrap each migration in a transaction.
- [ ] npm script to run it with the right env file (e.g. `tsx --env-file=.env.local scripts/setup.ts`).

**Verify**
- [ ] Running setup twice is harmless; a fresh database ends up identical to production.

### Atlas

**Setup**
- [ ] Install the Atlas CLI.
- [ ] Describe the schema (`schema.sql`, HCL, or load from Drizzle/Prisma).
- [ ] `atlas migrate diff` to generate versioned migrations; `atlas migrate lint` in CI; `atlas migrate apply` to deploy.

**Verify**
- [ ] Lint flags a deliberately destructive change.

### Supabase CLI

**Setup**
- [ ] `npx supabase init`, `npx supabase start` (Docker).
- [ ] Change schema locally, then `npx supabase db diff -f <name>`.
- [ ] `npx supabase link` and `npx supabase db push` to apply to the hosted project.

**Verify**
- [ ] `supabase db reset` rebuilds local from migrations without errors.

### Alembic

**Setup**
- [ ] `uv add alembic sqlalchemy`; `alembic init migrations`.
- [ ] Point `env.py` at the models' metadata and `DATABASE_URL`.
- [ ] `alembic revision --autogenerate -m "<change>"`, review, then `alembic upgrade head`.

**Verify**
- [ ] `alembic upgrade head` on a fresh database matches the models.

## Related

- [`database.md`](database.md), [`environments.md`](environments.md), [`ci-cd.md`](ci-cd.md).
