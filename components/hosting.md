# Hosting

Where the app's code runs and how it gets deployed. The hosting choice decides the runtime model (serverless, edge, long-running), the deploy workflow, and often the free tier you live inside.

## Decide

### Questions to ask

- Is it static, a full-stack framework app, or a long-running server (WebSockets, workers, non-JS backend)?
- Does it need to run continuously, or only respond to requests?
- Could it ever be commercial? (Some free tiers forbid it.)
- Do you want preview deploys per branch or pull request?
- What do you want to learn: zero-config platforms, edge runtimes, containers, or running a server yourself?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Vercel | ★ default | Next.js and other framework apps | Serverless limits; Hobby tier is non-commercial |
| Netlify | ◆ alternative | Static and framework sites, forms, simple functions | Next.js support lags Vercel's slightly |
| Cloudflare Pages / Workers | ◆ alternative | Edge apps, generous free tier, global latency | Workers runtime isn't full Node |
| Render | ◆ alternative | Long-running web services, background workers, cron | Free web services sleep when idle |
| Railway | ◆ alternative | Containers and databases with almost no config | Usage-based billing, small free allowance |
| Fly.io | ✦ distinctive | Containers close to users, multi-region, WebSockets | You manage more (Dockerfile, machines, regions) |
| VPS + Coolify (e.g. Hetzner) | ✦ distinctive | Learning servers; cheapest always-on hosting | You own security updates, backups, uptime |
| GitHub Pages | ✦ distinctive | Pure static sites from a repo | No server code at all |

### Option notes

**Vercel** ★
- Pros: made by the Next.js team, zero-config framework presets, preview deploy for every branch, built-in env var management, logs, cron, Blob storage, marketplace integrations (Neon, Upstash and others) that inject env vars.
- Cons: functions have execution time limits; Hobby plan is for personal, non-commercial use only; costs can climb with heavy bandwidth or function use.
- Pick when: Next.js, or any supported framework, and you want the smoothest path.
- What you'll learn: git-driven deploys, preview environments, serverless functions, framework presets.

**Netlify** ◆
- Pros: similar git-driven workflow, strong static and Jamstack support, built-in forms and identity, good free tier.
- Cons: some Next.js features land later than on Vercel.
- Pick when: Astro, SvelteKit or static-heavy sites, or to compare platforms.
- What you'll learn: the same deploy model on a different platform, which makes the portable parts obvious.

**Cloudflare Pages / Workers** ◆
- Pros: runs on Cloudflare's global edge network, very generous free tier, integrates with R2, D1, KV, Queues and Durable Objects.
- Cons: the Workers runtime is not full Node.js (compatibility flags help); some npm packages don't work; long CPU work is limited.
- Pick when: latency matters, you want Cloudflare's data products, or to learn the edge model.
- What you'll learn: edge runtimes, bindings instead of connection strings, `wrangler`.

**Render** ◆
- Pros: real always-on services, background workers, cron jobs and managed Postgres on one platform; deploy from a repo or Dockerfile.
- Cons: free web services spin down after inactivity, so the first request is slow.
- Pick when: an Express, FastAPI or Django backend, or anything needing a persistent process.
- What you'll learn: the long-running server model, health checks, worker processes.

**Railway** ◆
- Pros: deploy almost anything (detected automatically or from a Dockerfile), databases in a click, nice project canvas.
- Cons: usage-based billing; free allowance is small.
- Pick when: a quick multi-service setup (API + worker + database).
- What you'll learn: service-to-service networking, container deploys without writing Kubernetes.

**Fly.io** ✦
- Pros: runs your container as lightweight VMs in regions you choose; good for WebSockets, multi-region and anything stateful.
- Cons: more to manage (Dockerfile, `fly.toml`, machine sizing); no meaningful free tier for new accounts; pricing needs attention.
- Pick when: you want to learn containers and regions, or the app needs persistent connections.
- What you'll learn: Docker, VMs, regions, volumes, the `flyctl` workflow.

**VPS + Coolify** ✦
- Pros: cheapest always-on compute; Coolify gives you a self-hosted Vercel/Heroku-like dashboard; full control.
- Cons: you are the ops team: OS updates, firewall, backups, TLS, monitoring.
- Pick when: the goal is learning how servers actually work, or many small apps on one cheap box.
- What you'll learn: Linux administration, SSH, Docker, reverse proxies, TLS certificates.

**GitHub Pages** ✦
- Pros: free, lives next to the code, custom domains supported.
- Cons: static files only.
- Pick when: docs, a portfolio, or a client-only tool.
- What you'll learn: static builds, GitHub Actions deploys.

### How to choose

1. Long-running process, WebSockets or a non-JS backend → Render, Railway or Fly.io (VPS to learn ops).
2. Static only → GitHub Pages, Cloudflare Pages or Netlify.
3. Next.js → Vercel unless you specifically want to learn another platform.
4. Other frameworks → Vercel, Netlify or Cloudflare all work; Cloudflare if you want the edge.
5. Commercial on a $0 budget → check each free tier's terms; Cloudflare and Netlify allow commercial use on free plans more readily than Vercel Hobby.

## Configure

### Vercel

**Setup**
- [ ] Push the project to GitHub (Vercel deploys from the repo).
- [ ] Vercel dashboard → **Add New… → Project** → import the repo.
- [ ] **Framework Preset**: confirm it matches the framework (e.g. Next.js). It is usually detected from `package.json`; if it shows "Other", set it by hand, or the build may pass but routes 404.
- [ ] **Root Directory**: leave as the repo root unless the app is in a subfolder (monorepo).
- [ ] Build / Output / Install commands: leave the defaults unless the project needs something special.
- [ ] Add environment variables before the first deploy (see [secrets](secrets.md)). Set each for Production, Preview and Development as appropriate.
- [ ] Deploy. Production deploys come from the default branch; every other branch and PR gets a preview URL.
- [ ] Optional: connect a database or other service through **Storage** or the **Marketplace**, which adds its env vars for you.
- [ ] Optional: add a custom domain (see [domains](domains.md)).

**Env vars**
- Vercel provides `VERCEL_ENV` (`production` / `preview` / `development`) and `VERCEL_URL` automatically.
- Changing an env var does not affect existing deployments. Redeploy to pick it up.

**Verify**
- [ ] The deployment log shows the framework detected and the build succeeding.
- [ ] Load the production URL and every main route.
- [ ] Hit one route that uses each secret (database, API key) to prove runtime env vars are set. A build can pass with a missing runtime variable.
- [ ] Open a PR and confirm a preview URL appears.

**Gotchas**
- Hobby plan: personal, non-commercial use only.
- Function duration and payload limits apply to API routes and server actions; long work belongs in a [job](jobs.md).
- `vercel.json` is optional; only add it for rewrites, headers, cron or region settings.
- Install the CLI (`npm i -g vercel`) for `vercel env pull`, `vercel logs` and local linking.

### Netlify

**Setup**
- [ ] **Add new site → Import an existing project** → pick the repo.
- [ ] Confirm the detected framework, build command and publish directory.
- [ ] Add env vars under **Site configuration → Environment variables**.
- [ ] Deploy; deploy previews are on for PRs by default.

**Verify**
- [ ] Production URL and routes load; a route using each secret works; a PR gets a deploy preview.

**Gotchas**
- Functions live in `netlify/functions/` for plain sites; frameworks use Netlify's adapter automatically.
- Optional `netlify.toml` holds build settings and redirects in the repo.

### Cloudflare Pages / Workers

**Setup**
- [ ] `npm create cloudflare@latest` for a new app (offers framework templates), or connect an existing repo under **Workers & Pages → Create**.
- [ ] Configure `wrangler.toml` (or `wrangler.jsonc`): name, `compatibility_date`, `compatibility_flags = ["nodejs_compat"]` if you need Node APIs.
- [ ] Add bindings for any Cloudflare resources (D1, R2, KV, Queues) in the wrangler config.
- [ ] Secrets: `npx wrangler secret put NAME` (or the dashboard). Non-secret vars go in `[vars]`.
- [ ] Deploy with `npx wrangler deploy`, or let git integration deploy on push.

**Verify**
- [ ] `npx wrangler dev` runs locally; the deployed URL works; bindings resolve.

**Gotchas**
- Bindings are accessed from the request `env` object, not `process.env` (frameworks paper over this differently).
- Check every npm dependency works in the Workers runtime.

### Render

**Setup**
- [ ] **New → Web Service** → connect the repo.
- [ ] Choose runtime (Node, Python, Docker…), build command and start command.
- [ ] Set the port the app listens on from `PORT` (Render provides it).
- [ ] Add env vars; optionally group shared ones into an **Environment Group**.
- [ ] Add a **Health Check Path** (e.g. `/health`).
- [ ] Optional: a `render.yaml` blueprint describes all services in the repo.

**Verify**
- [ ] Deploy log shows the start command running and the health check passing; the URL responds.

**Gotchas**
- Free web services sleep after inactivity; the first request wakes them slowly.
- Background workers and cron jobs are separate service types.

### Railway

**Setup**
- [ ] **New Project → Deploy from GitHub repo**.
- [ ] Add a database from **New → Database** if needed; reference its variables in the app service (`${{Postgres.DATABASE_URL}}`).
- [ ] Generate a public domain under the service's **Settings → Networking**.

**Verify**
- [ ] Deploy logs are clean; the public domain responds; the app reaches its database.

**Gotchas**
- Billing is per usage; set a usage limit in account settings.

### Fly.io

**Setup**
- [ ] Install `flyctl`, then `fly auth login`.
- [ ] `fly launch` in the project: detects the framework, writes a `Dockerfile` (if missing) and `fly.toml`, picks a region.
- [ ] Secrets: `fly secrets set NAME=value`.
- [ ] `fly deploy`.
- [ ] Persistent data needs a volume (`fly volumes create`) or an external database.

**Verify**
- [ ] `fly status` shows machines running; `fly logs` is clean; the `.fly.dev` URL responds.

**Gotchas**
- Machines can auto-stop when idle (configurable in `fly.toml`); first request then cold-starts.
- Watch pricing: every machine and volume costs money.

### VPS + Coolify

**Setup**
- [ ] Create a small VPS (e.g. Hetzner, DigitalOcean) with Ubuntu LTS; add your SSH key.
- [ ] Harden: non-root user, SSH keys only, firewall (only 22, 80, 443), automatic security updates.
- [ ] Install Coolify with its one-line installer; open its dashboard and set an admin account.
- [ ] Point a domain at the server (see [domains](domains.md)); Coolify issues TLS certificates.
- [ ] Add the GitHub repo as an application; set build pack, env vars and domain.
- [ ] Set up backups for databases and volumes (Coolify can push them to S3-compatible storage).

**Verify**
- [ ] HTTPS works on the domain; deploys run on push; a backup restores.

**Gotchas**
- You are responsible for patching and backups. Put a reminder in your calendar.

### GitHub Pages

**Setup**
- [ ] Repo **Settings → Pages** → Source: **GitHub Actions**.
- [ ] Use the starter workflow for your static generator (or upload a `dist/` folder via `actions/upload-pages-artifact` and `actions/deploy-pages`).
- [ ] For a project site, set the framework's base path to `/<repo-name>/`.

**Verify**
- [ ] The workflow succeeds and `https://<user>.github.io/<repo>/` loads with working asset paths.

## Related

- [`ci-cd.md`](ci-cd.md) for checks that must pass before deploys.
- [`environments.md`](environments.md) for preview vs. production configuration.
