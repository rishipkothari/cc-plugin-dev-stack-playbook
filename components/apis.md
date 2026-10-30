# Third-party APIs

Calling other services from the app: AI models, Google APIs, maps, weather, and receiving their webhooks. The key questions are how you authenticate (API key, OAuth on behalf of a user, service account) and where the call runs.

## Decide

### Questions to ask

- Does the call act as the app (API key) or on behalf of a user's own account (OAuth)?
- Is the call fast enough for a request, or should it run as a [job](jobs.md)?
- Does the service push events to you (webhooks)?
- Rate limits, cost per call, and what happens when the service is down?
- For AI: which model, streaming or not, structured output, tools?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Official SDK + server-only API key | ★ default | Most APIs (Anthropic, OpenAI, Stripe, Resend…) | Key must never reach the browser |
| OAuth 2.0 (user consent) | ◆ alternative | Acting on a user's Google Calendar, Drive, GitHub… | Consent screens, token storage and refresh |
| Service account (Google) | ◆ alternative | Server-to-server access to your own Google resources | Only resources shared with the service account |
| Incoming webhooks | ◆ alternative | Services notifying you of events | Signature verification, retries, idempotency |
| Vercel AI SDK | ✦ distinctive | Provider-agnostic AI calls, streaming UI helpers | Another abstraction over provider SDKs |
| MCP servers | ✦ distinctive | Exposing your app's data/tools to AI agents | Emerging standard; auth design |

### Option notes

**Official SDK + server-only key** ★
- Pros: typed clients, retries, pagination helpers.
- Cons: key handling discipline needed.
- What you'll learn: server-side API integration, error and retry handling.

**OAuth 2.0** ◆
- Pros: act with the user's permission on their data.
- Cons: consent screen setup, app verification for sensitive scopes (Google), refresh-token storage, token revocation.
- Pick when: e.g. writing to a user's Google Calendar.
- What you'll learn: authorization code flow, scopes, refresh tokens, PKCE.

**Service account** ◆
- Pros: no user consent; good for your own calendar, sheet or bucket.
- Cons: the resource must be shared with the service account's email (or domain-wide delegation in Workspace).
- What you'll learn: server-to-server auth, JSON key handling.

**Incoming webhooks** ◆
- What you'll learn: signature verification, at-least-once delivery, idempotency.

**Vercel AI SDK** ✦
- Pros: same API across Anthropic, OpenAI, Google and others; `streamText`, `generateObject`, tool calling; React hooks for chat UIs.
- Cons: abstraction lags newest provider features sometimes.
- What you'll learn: streaming responses to the UI, structured output, tool loops.

**MCP servers** ✦
- Pros: Model Context Protocol lets Claude and other agents call your app's tools and read its data.
- Cons: auth and permission design for agents.
- What you'll learn: agent tool design, the MCP spec.

### How to choose

1. Acting as the app → SDK + server-only key.
2. Acting on a user's account → OAuth.
3. Your own Google resources from a server → service account.
4. AI features with streaming UI → provider SDK, or Vercel AI SDK for multi-provider.

## Configure

### Common to every option

- [ ] Keys live in server-only env vars; calls happen in server code.
- [ ] Set timeouts and handle failures (retry with backoff for 429/5xx, user-friendly errors).
- [ ] Respect rate limits; cache responses that don't change often.
- [ ] Put a spend cap or budget alert on paid APIs (see [costs](costs.md)).
- [ ] Log request IDs from providers to debug failures.

### Official SDK + server-only key (example: Anthropic)

**Setup**
- [ ] Create an API key in the provider console; set a spend limit.
- [ ] `npm i @anthropic-ai/sdk` (or the provider's SDK).
- [ ] Instantiate the client in a server-only module; read the key from env.
- [ ] For long generations, stream the response or move the call to a background job (serverless time limits).

**Env vars**
- `ANTHROPIC_API_KEY` (or the provider's equivalent).

**Verify**
- [ ] A server route returns a model response; the key doesn't appear in any client bundle (search the build output).

**Gotchas**
- Model IDs and parameters change; check current docs (use the `claude-api` skill if available).

### OAuth 2.0 (example: Google Calendar)

**Setup**
- [ ] Google Cloud Console: create a project, enable the API (e.g. Google Calendar API).
- [ ] Configure the OAuth consent screen (app name, scopes, test users while unverified).
- [ ] Create an OAuth client ID (Web application) with authorized redirect URIs for local and production.
- [ ] Request the narrowest scopes (e.g. `calendar.events` rather than full `calendar`).
- [ ] Use `access_type=offline` and `prompt=consent` to receive a refresh token; store it encrypted in the database.
- [ ] Use the `googleapis` package (or Better Auth's Google provider with extra scopes) to call the API with refreshed tokens.

**Env vars**
- `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, plus a key for encrypting stored tokens.

**Verify**
- [ ] Connect flow completes; an event is created on the calendar; the call still works after the access token expires (refresh works).

**Gotchas**
- Unverified apps show a warning and are limited to test users; sensitive scopes need Google verification for public use.
- Refresh tokens for apps in "Testing" status expire after about a week.

### Service account (Google)

**Setup**
- [ ] Create a service account; create a JSON key.
- [ ] Share the target resource (calendar, sheet, folder) with the service account's email.
- [ ] Store the key JSON (or its fields) in env vars; authenticate with `google-auth-library`.

**Env vars**
- `GOOGLE_SERVICE_ACCOUNT_EMAIL`, `GOOGLE_PRIVATE_KEY` (watch newline escaping).

**Verify**
- [ ] Server can read/write the shared resource.

### Incoming webhooks

**Setup**
- [ ] Dedicated route per provider (`/api/webhooks/<provider>`).
- [ ] Verify the signature with the raw body and the provider's secret.
- [ ] Respond 2xx quickly; do slow work in a job.
- [ ] Store event IDs to ignore duplicates.
- [ ] Local testing via the provider's CLI forwarder, or a tunnel (`cloudflared tunnel`, ngrok).

**Verify**
- [ ] A forged request without a valid signature is rejected; a replayed event is ignored.

### Vercel AI SDK

**Setup**
- [ ] `npm i ai @ai-sdk/anthropic` (or other providers).
- [ ] Server route uses `streamText({ model, messages })` and returns its stream response.
- [ ] Client uses the `useChat` hook from `@ai-sdk/react`.

**Verify**
- [ ] Tokens stream into the UI; tool calls work if defined.

### MCP servers

**Setup**
- [ ] Use the official MCP TypeScript SDK (or a framework adapter) to define tools and resources.
- [ ] Expose a remote (HTTP) MCP endpoint with auth, or a local stdio server for personal use.
- [ ] Add it to Claude as a connector to test.

**Verify**
- [ ] Claude lists the tools and can call them with correct permissions.

## Related

- [`backend.md`](backend.md), [`jobs.md`](jobs.md), [`secrets.md`](secrets.md), [`costs.md`](costs.md).
- Skill: `claude-api` for anything calling Claude.
