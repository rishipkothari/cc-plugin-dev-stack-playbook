# Auth

Authentication (who is this?) and the session that remembers them, plus the start of authorization (what may they do?). Getting it wrong is the most common way a small app leaks data, so this file leans toward proven libraries.

## Decide

### Questions to ask

- Who uses it: just you, a few trusted people, or the public?
- Is there per-user data, or does everyone see the same thing?
- Sign-in methods: password, Google/GitHub OAuth, magic link, passkeys?
- Do you need organizations/teams, roles, or admin users?
- Are preview deployments protected too?
- Do you want to own the user data in your database, or rent a hosted user system?
- What do you want to learn: how sessions and OAuth actually work, or how to integrate a provider fast?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Better Auth | ★ default | Self-hosted TS auth with users in your own database | You run it: schema, email, security settings |
| Clerk | ◆ alternative | Hosted auth with prebuilt UI, orgs, MFA | Paid beyond the free tier; user data lives with Clerk |
| Supabase Auth | ◆ alternative | Auth bundled with a Supabase database and RLS | Tied to Supabase |
| Shared password | ◆ alternative | A few trusted users, no per-user data | No individual accounts or audit trail |
| Hand-rolled sessions | ✦ distinctive | Learning exactly how auth works | Easy to get subtly wrong; not for sensitive data |
| Passkeys (WebAuthn) | ✦ distinctive | Phishing-resistant passwordless sign-in | Device/browser support flows to design |
| Magic links / email OTP | ✦ distinctive | Passwordless sign-in by email | Depends on email deliverability |

Auth.js (formerly NextAuth) is still widely used; it's now maintained alongside Better Auth, which is the better pick for new projects.

### Option notes

**Better Auth** ★
- Pros: framework-agnostic TypeScript library; email/password, OAuth providers, magic links, passkeys, 2FA, organizations via plugins; users and sessions live in your database (works with Drizzle/Prisma adapters).
- Cons: you manage the schema, the email sending and the security settings.
- Pick when: real user accounts and you want to own the data.
- What you'll learn: session tables, OAuth callback flow, cookie security, plugin-based auth.

**Clerk** ◆
- Pros: drop-in sign-in UI components, user management dashboard, organizations, MFA, bot protection.
- Cons: per-user pricing past the free tier; users live in Clerk (sync via webhooks if you need them locally).
- Pick when: you want polished auth in an afternoon, or B2B organizations.
- What you'll learn: hosted identity providers, JWT-based sessions, webhook-driven user sync.

**Supabase Auth** ◆
- Pros: integrated with Postgres Row Level Security (`auth.uid()` in policies); many providers.
- Cons: Supabase-specific.
- Pick when: the database is Supabase.
- What you'll learn: tying identity to database-level authorization.

**Shared password** ◆
- Pros: smallest possible auth: one secret in an env var, a signed cookie after login.
- Cons: everyone shares one identity; rotating means telling everyone.
- Pick when: family or small-team tools with no per-user data (nanny-comp-log).
- What you'll learn: signed cookies, middleware gating, constant-time comparison.

**Hand-rolled sessions** ✦
- Pros: you understand every piece: password hashing (Argon2id), session tokens, cookies, CSRF.
- Cons: lots of subtle security details; use only for learning or low-stakes data. The Lucia project's guides (now a learning resource rather than a library) are a good map.
- Pick when: the learning goal is "how does auth really work".
- What you'll learn: everything libraries hide.

**Passkeys** ✦
- Pros: no passwords, phishing-resistant, built into modern devices.
- Cons: fallback and recovery flows need design.
- Pick when: modern UX, or adding as a second option via Better Auth's passkey plugin.
- What you'll learn: WebAuthn, public-key credentials.

**Magic links / email OTP** ✦
- Pros: no passwords to store.
- Cons: sign-in depends on email arriving quickly (see [email](email.md)).
- Pick when: low-friction sign-in for occasional users.

### How to choose

1. Just a few trusted people, shared data → shared password.
2. Supabase database → Supabase Auth.
3. Real accounts, own the data → Better Auth.
4. Fastest polished UI or B2B orgs → Clerk.
5. Learning project with throwaway data → hand-rolled sessions.
6. Add passkeys or magic links on top of any of the above as a learning extra.

## Configure

### Common to every option

- [ ] Session cookie flags: `HttpOnly`, `Secure`, `SameSite=Lax` (or `Strict`), sensible expiry.
- [ ] Check auth on the server for every protected read and write, not just by hiding UI or in middleware alone.
- [ ] Rate-limit login endpoints (e.g. Upstash ratelimit).
- [ ] OAuth: register callback URLs for local (`http://localhost:3000/...`), production, and decide how previews work (preview URLs change per deploy; use a stable preview domain or a proxy, or skip OAuth on previews).
- [ ] Decide whether preview deployments are publicly reachable (Vercel Deployment Protection can gate them).

### Better Auth

**Setup**
- [ ] `npm i better-auth`.
- [ ] Create `src/lib/auth.ts` with `betterAuth({ database: <adapter>, emailAndPassword: { enabled: true }, socialProviders: {...} })`.
- [ ] Generate and apply the auth tables: `npx @better-auth/cli generate` (or `migrate`), then your migration tool.
- [ ] Mount the handler (Next.js: `app/api/auth/[...all]/route.ts` with `toNextJsHandler(auth)`).
- [ ] Create the client (`createAuthClient`) for sign-in/out in the UI.
- [ ] Wire email sending for verification and password reset (see [email](email.md)).
- [ ] Add plugins as needed (organization, passkey, magicLink, twoFactor).

**Env vars**
- `BETTER_AUTH_SECRET` (random, 32+ bytes), `BETTER_AUTH_URL` (the app's base URL).
- Per OAuth provider: e.g. `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`.

**Verify**
- [ ] Sign up, sign out, sign in, reset password; session survives reload; a protected route rejects a signed-out request.

### Clerk

**Setup**
- [ ] Create an application in the Clerk dashboard; choose sign-in methods.
- [ ] `npm i @clerk/nextjs` (or the framework SDK).
- [ ] Wrap the app in `<ClerkProvider>`; add `clerkMiddleware()` in `middleware.ts`.
- [ ] Use `<SignIn />`, `<UserButton />` components; `auth()` on the server.
- [ ] If you need users in your DB, add a webhook endpoint for `user.created`/`user.updated` and verify its signature.
- [ ] Before launch, create a production instance and set its keys and domain.

**Env vars**
- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`, optionally `CLERK_WEBHOOK_SIGNING_SECRET`.

**Verify**
- [ ] Sign in works on production with production keys (development keys show a banner and have limits).

### Supabase Auth

**Setup**
- [ ] Enable providers in **Authentication → Providers**; set the Site URL and redirect URLs.
- [ ] Use `@supabase/ssr` for cookie-based sessions in SSR frameworks.
- [ ] Write RLS policies using `auth.uid()`.

**Verify**
- [ ] A signed-in user can only see their own rows.

### Shared password

**Setup**
- [ ] Env var for the password (e.g. `APP_PASSWORD`), plus a separate secret for signing cookies (`SESSION_SECRET`).
- [ ] Login route: compare with a constant-time comparison (`crypto.timingSafeEqual` on equal-length buffers or hashes).
- [ ] On success, set a signed (HMAC) or encrypted session cookie.
- [ ] Middleware redirects requests without a valid cookie to `/login`; API routes also check.
- [ ] Optional second level (admin password) for editing.

**Verify**
- [ ] Wrong password fails; tampering with the cookie logs you out; API routes reject requests without the cookie.

**Gotchas**
- Changing the password should invalidate old sessions: sign cookies with a key derived from the password, or rotate `SESSION_SECRET`.

### Hand-rolled sessions

**Setup**
- [ ] `users` table with a password hash (Argon2id via `@node-rs/argon2`), `sessions` table with a hashed random token and expiry.
- [ ] On login: verify hash, create a session, set the token cookie.
- [ ] On each request: hash the cookie token, look up the session, extend or expire.
- [ ] CSRF: rely on `SameSite` cookies plus checking the `Origin` header on mutations.
- [ ] Logout deletes the session row.

**Verify**
- [ ] Sessions expire; logout works on other tabs; stolen-cookie scenarios are understood.

### Passkeys

**Setup**
- [ ] Easiest via Better Auth's passkey plugin (or SimpleWebAuthn directly).
- [ ] Set the relying party ID to the production domain; passkeys don't work across domains.

**Verify**
- [ ] Register and sign in with a passkey on two devices.

### Magic links / email OTP

**Setup**
- [ ] Better Auth `magicLink` or `emailOTP` plugin, or Supabase/Clerk built-ins.
- [ ] Short expiry (10–15 minutes), single use.
- [ ] Working transactional email (see [email](email.md)).

**Verify**
- [ ] Link works once, then fails; expired links fail.

## Related

- [`secrets.md`](secrets.md), [`email.md`](email.md), [`environments.md`](environments.md).
