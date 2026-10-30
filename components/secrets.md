# Secrets and environment variables

How configuration and secrets (API keys, database URLs, passwords) get into the app in each environment without ending up in git or in the browser.

## Decide

### Questions to ask

- How many secrets, and how many places need them (local, host, CI, teammates' machines, other devices of yours)?
- Do you work on several devices? (Copying `.env.local` around gets old.)
- Which values are public (fine in the browser) and which are secret?
- What do you want to learn: the platform basics, or a dedicated secrets manager?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| `.env.local` + host env vars | ★ default | Solo projects | Manual sync between places |
| Host CLI pull (`vercel env pull`) | ◆ alternative | Using the host as the source of truth | Host-specific |
| 1Password CLI (`op`) | ✦ distinctive | Secrets in your password manager, injected at runtime | Needs 1Password |
| Doppler / Infisical | ✦ distinctive | Central secrets manager syncing to hosts and CI | Another service |
| dotenvx | ✦ distinctive | Encrypted `.env` files committed to git | Key management for the decryption key |

### Option notes

**`.env.local` + host env vars** ★
- Pros: no extra tools; every framework supports it.
- Cons: you copy values by hand; easy for local and production to drift.
- What you'll learn: env loading order, build-time vs. runtime variables, public prefixes.

**Host CLI pull** ◆
- Pros: set once in the host dashboard; `vercel env pull .env.local` fetches development values on any device.
- Cons: tied to the host.
- What you'll learn: environment scopes (Development / Preview / Production).

**1Password CLI** ✦
- Pros: secrets stay in your vault; `.env` holds `op://vault/item/field` references; `op run -- npm run dev` injects them. Works on every device you're signed into.
- Cons: requires 1Password.
- What you'll learn: secret references, runtime injection.

**Doppler / Infisical** ✦
- Pros: one source of truth with environments, audit logs and integrations that sync to Vercel, GitHub Actions and others. Infisical is open source and self-hostable.
- Cons: another account and dependency.
- What you'll learn: centralized secret management, rotation workflows.

**dotenvx** ✦
- Pros: encrypt `.env` files and commit them; decrypt with a private key at runtime.
- Cons: the private key itself must be distributed securely.
- What you'll learn: encrypted config, public-key encryption basics.

### How to choose

1. Solo, one host → `.env.local` + host env vars, using the host CLI to pull on new devices.
2. Many devices or projects and you use 1Password → 1Password CLI.
3. Several services/CI pipelines sharing secrets → Doppler or Infisical.

## Configure

### Rules for every option

- [ ] `.gitignore` contains `.env*` except an `.env.example` (names only, no values) that documents every variable.
- [ ] Classify each variable: public (may reach the browser, uses the framework's public prefix) or secret (server only).
- [ ] Never prefix a secret with `NEXT_PUBLIC_`, `VITE_`, `PUBLIC_` or `EXPO_PUBLIC_`; those are inlined into client bundles.
- [ ] Know which variables are needed at build time vs. runtime. Prefer runtime; build-time values are baked into the deployment.
- [ ] Validate env at startup (e.g. a Zod schema in `src/env.ts`, or `@t3-oss/env-nextjs`) so missing values fail loudly.
- [ ] Generate secrets randomly: `openssl rand -base64 32`.
- [ ] Keep an inventory in the project's `docs/stack.md`: name, purpose, where it comes from, environments.
- [ ] If a secret is ever committed or pasted publicly, rotate it immediately; deleting the commit is not enough.

### `.env.local` + host env vars

**Setup**
- [ ] Create `.env.local` with local values; create `.env.example` with names.
- [ ] Add each variable in the host dashboard with the right environment scopes.
- [ ] Scripts that run outside the framework load env explicitly (`tsx --env-file=.env.local …` or `node --env-file`).

**Verify**
- [ ] A fresh clone + `.env.local` from `.env.example` runs; the deployed app has every runtime variable.

**Gotchas**
- Hosts apply env changes only to new deployments: redeploy after changing values.

### Host CLI pull (Vercel)

**Setup**
- [ ] `npm i -g vercel`, `vercel link` in the project.
- [ ] `vercel env add NAME` (or the dashboard) per environment.
- [ ] On any device: `vercel env pull .env.local`.

**Verify**
- [ ] Pulled file contains the Development values.

### 1Password CLI

**Setup**
- [ ] Install `op`, enable the desktop app integration, sign in.
- [ ] Store secrets as items; write `.env` with `NAME=op://Vault/Item/field`.
- [ ] Run with `op run --env-file=.env -- npm run dev`.
- [ ] For CI, use a 1Password service account token or sync the values to the host.

**Verify**
- [ ] App runs locally with no plaintext secrets on disk.

### Doppler / Infisical

**Setup**
- [ ] Create a project with dev/preview/prod configs.
- [ ] Install the CLI; `doppler run -- npm run dev` (or `infisical run -- …`).
- [ ] Enable the integration that syncs to the host and to GitHub Actions.

**Verify**
- [ ] Changing a value in the manager reaches the host after a sync and redeploy.

### dotenvx

**Setup**
- [ ] `npx @dotenvx/dotenvx encrypt` to encrypt `.env.production`; commit it; keep `.env.keys` out of git.
- [ ] Set `DOTENV_PRIVATE_KEY_PRODUCTION` on the host; run with `dotenvx run -- …`.

**Verify**
- [ ] Deploy decrypts values; the repo shows only ciphertext.

## Related

- [`environments.md`](environments.md), [`ci-cd.md`](ci-cd.md), [`hosting.md`](hosting.md).
