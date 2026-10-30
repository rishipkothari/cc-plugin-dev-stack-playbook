# Frontend framework

The framework that renders the user interface, and often (for full-stack frameworks) runs the server code too. It's the biggest single choice in the stack because it sets the language, routing, data-loading patterns and hosting presets.

## Decide

### Questions to ask

- Does the app need server code (database, secrets), or is it purely client-side or static?
- Mostly content (marketing, docs, blog), or mostly interactive app screens?
- Is a mobile app in scope now or later?
- Which ecosystem do you want to deepen (React) or explore (Svelte, Vue, Solid, hypermedia)?
- How much do you value convention and batteries included vs. a minimal setup you assemble yourself?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Next.js (App Router) | ★ default | Full-stack React apps | Server/client component model takes time to learn |
| Vite + React (SPA) | ◆ alternative | Client-only apps with a separate API | No server rendering or server code built in |
| React Router v7 (framework mode) | ◆ alternative | Full-stack React with loaders/actions (the Remix model) | Smaller ecosystem than Next.js |
| SvelteKit | ◆ alternative | Full-stack apps with less boilerplate | Not React; smaller job market |
| Astro | ◆ alternative | Content-heavy sites with islands of interactivity | Not designed for highly interactive apps |
| Nuxt | ◆ alternative | Full-stack Vue | Vue ecosystem instead of React |
| htmx + server templates | ✦ distinctive | Server-rendered apps with minimal JavaScript | Rich client interactivity is harder |
| SolidStart | ✦ distinctive | Fine-grained reactivity, fast UIs | Young ecosystem |
| Expo (React Native) | ✦ distinctive | iOS and Android apps (plus web) | App store builds, native modules, separate deploy story |

### Option notes

**Next.js (App Router)** ★
- Pros: the most used React framework; server components, server actions, route handlers, streaming; first-class on Vercel; huge ecosystem and examples.
- Cons: the server vs. client component boundary, caching rules and `"use client"` take real learning; tied closely to Vercel's direction.
- Pick when: a full-stack React app, especially on Vercel.
- What you'll learn: server components, file-based routing, server actions, rendering strategies (static, dynamic, streaming).

**Vite + React (SPA)** ◆
- Pros: fastest dev server, minimal and unopinionated; deploys as static files anywhere.
- Cons: no server; secrets and database access need a separate backend; SEO and first-load performance are weaker.
- Pick when: a dashboard behind login, a client-only tool, or a frontend for a separate API (e.g. FastAPI).
- What you'll learn: client-side routing (React Router or TanStack Router), client data fetching (TanStack Query), API boundaries.

**React Router v7 (framework mode)** ◆
- Pros: the Remix programming model (loaders, actions, progressive enhancement) as part of React Router; web-standards first; runs on many hosts.
- Cons: fewer examples than Next.js.
- Pick when: you want full-stack React built on web fundamentals (forms, HTTP), or to compare with Next.js.
- What you'll learn: loader/action data flow, progressive enhancement, nested routes.

**SvelteKit** ◆
- Pros: compiles away the framework, less boilerplate, very readable components; full-stack with load functions and form actions; adapters for every host.
- Cons: smaller ecosystem and component libraries than React.
- Pick when: you want a different mental model from React with a polished full-stack framework.
- What you'll learn: compiler-based reactivity (runes), adapters, form actions.

**Astro** ◆
- Pros: ships zero JS by default; "islands" let you drop in React, Svelte or Vue components only where needed; content collections for Markdown.
- Cons: not built for app-like UIs with lots of shared client state.
- Pick when: blogs, docs, marketing and portfolio sites.
- What you'll learn: islands architecture, static generation, content collections.

**Nuxt** ◆
- Pros: the Vue equivalent of Next.js; auto-imports, server routes (Nitro), many modules.
- Cons: Vue instead of React.
- Pick when: learning Vue, or you prefer its template syntax.
- What you'll learn: Vue's reactivity, Nitro server engine.

**htmx + server templates** ✦
- Pros: HTML attributes trigger requests that swap in server-rendered fragments; works with any backend (FastAPI + Jinja, Go, Django, Hono + JSX); tiny client.
- Cons: complex client state is awkward; fewer UI component libraries.
- Pick when: CRUD apps, internal tools, or learning the hypermedia approach.
- What you'll learn: HTTP and HTML as the application model, server-side rendering without a JS framework.

**SolidStart** ✦
- Pros: fine-grained signals (no virtual DOM), very fast, JSX that feels like React.
- Cons: young, fewer libraries and examples.
- Pick when: curiosity about signals and performance.
- What you'll learn: signals-based reactivity, which also informs how modern React, Svelte and Angular are evolving.

**Expo (React Native)** ✦
- Pros: one React codebase for iOS, Android and web; Expo Router; EAS builds and over-the-air updates.
- Cons: app store accounts and review, native build pipeline, device testing.
- Pick when: the project is a mobile app.
- What you'll learn: native app structure, EAS Build/Submit, mobile auth and storage.

### How to choose

1. Mobile app → Expo.
2. Content site → Astro.
3. Client-only with a separate backend → Vite + React.
4. Full-stack React → Next.js by default; React Router v7 to learn a different model.
5. Want a non-React mental model → SvelteKit (full-stack), Nuxt (Vue) or htmx (no SPA at all).

## Configure

### Next.js (App Router)

**Setup**
- [ ] `npx create-next-app@latest` → TypeScript yes, ESLint yes, App Router yes, `src/` directory (optional, choose one convention and keep it), Tailwind optional.
- [ ] Keep `next.config.(mjs|ts)` minimal; add redirects, images or headers only when needed.
- [ ] Decide the auth gate location: `middleware.ts` for route protection, server-side checks in each data access path regardless.
- [ ] Put server-only code (database, secrets) in modules imported only from server components, route handlers or server actions. Add `import "server-only"` to those modules to make accidental client imports fail the build.
- [ ] Add scripts: `dev`, `build`, `start`, `lint`, and a `typecheck` script running `tsc --noEmit`.

**Env vars**
- Server-only by default. Only variables prefixed `NEXT_PUBLIC_` reach the browser, and they're inlined at build time, so never put secrets there.
- Local values go in `.env.local` (gitignored).

**Verify**
- [ ] `npm run build` succeeds (catches type errors and server/client boundary mistakes).
- [ ] `npm run dev` and click through main routes.

**Gotchas**
- `"use client"` marks a boundary; everything it imports becomes client code.
- Caching and revalidation defaults have changed between major versions; read the current docs for `fetch` caching and `revalidatePath`.
- `next build` needs build-time env vars only if code reads them at import time; keep database clients lazy where possible.

### Vite + React (SPA)

**Setup**
- [ ] `npm create vite@latest` → React + TypeScript.
- [ ] Add a router (React Router or TanStack Router) and a data-fetching library (TanStack Query).
- [ ] Point API calls at a base URL from `import.meta.env.VITE_API_URL`.
- [ ] Configure the host for SPA fallback (all paths serve `index.html`).

**Env vars**
- Only `VITE_`-prefixed variables are exposed, and all of them are public. Never put secrets in a Vite app.

**Verify**
- [ ] `npm run build` produces `dist/`; deep links work after deploy (SPA fallback configured).

### React Router v7 (framework mode)

**Setup**
- [ ] `npx create-react-router@latest`.
- [ ] Define routes in `app/routes.ts`; use `loader` for reads and `action` for writes.
- [ ] Pick the deploy target's adapter/template (Node, Vercel, Cloudflare).

**Verify**
- [ ] Forms work with JavaScript disabled (progressive enhancement), and the build runs on the target.

### SvelteKit

**Setup**
- [ ] `npx sv create` → SvelteKit, TypeScript.
- [ ] Install the adapter for the host (`@sveltejs/adapter-vercel`, `-netlify`, `-cloudflare`, `-node`) or keep `adapter-auto`.
- [ ] Server-only code goes in `+page.server.ts`, `+server.ts` or `$lib/server/`.

**Env vars**
- `$env/static/private` and `$env/dynamic/private` for secrets; `PUBLIC_`-prefixed variables via `$env/*/public`.

**Verify**
- [ ] `npm run build` and `npm run preview`; importing `$lib/server` from client code fails (by design).

### Astro

**Setup**
- [ ] `npm create astro@latest`.
- [ ] Add UI framework integrations only where needed (`npx astro add react`).
- [ ] For server routes or SSR, add the host adapter (`npx astro add vercel`) and set the output mode.
- [ ] Use content collections for Markdown content.

**Verify**
- [ ] `npm run build`; pages ship no JS unless an island uses `client:*`.

### Nuxt

**Setup**
- [ ] `npx nuxi@latest init`.
- [ ] Server routes in `server/api/`; config in `nuxt.config.ts` with `runtimeConfig` for env vars.

**Verify**
- [ ] `npm run build` and the host preset detected (Nitro picks presets for most hosts).

### htmx + server templates

**Setup**
- [ ] Pick the backend (FastAPI + Jinja2, Hono with JSX, Go `html/template`, Django).
- [ ] Include htmx with a `<script>` tag from a pinned version, or install it.
- [ ] Return full pages for normal navigation and fragments for `HX-Request` requests.

**Verify**
- [ ] Interactions swap fragments without full reloads; pages still work as plain links and forms.

### SolidStart

**Setup**
- [ ] `npm create solid@latest` → SolidStart.
- [ ] Set the Nitro preset for the host in `app.config.ts` if not auto-detected.

**Verify**
- [ ] `npm run build` and deploy on the target host.

### Expo (React Native)

**Setup**
- [ ] `npx create-expo-app@latest`; run on a device with Expo Go.
- [ ] Use Expo Router for navigation.
- [ ] Create an Expo account; `npx eas-cli build:configure`.
- [ ] Apple Developer and Google Play accounts are needed for store builds.
- [ ] Store tokens with `expo-secure-store`, not AsyncStorage.

**Verify**
- [ ] App runs on iOS and Android devices; an EAS development build installs.

**Gotchas**
- Anything in the app bundle is public; secrets stay on your backend.

## Related

- [`backend.md`](backend.md) for where server logic lives.
- [`hosting.md`](hosting.md) for framework presets per host.
- Skill: if a frontend or UI design skill is installed, use it for component and visual design work.
