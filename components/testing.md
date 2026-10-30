# Testing

Automated checks that the code does what you think. A small personal project still benefits from three layers: static checks (types, lint), unit tests for logic, and a few end-to-end tests for the critical flows.

## Decide

### Questions to ask

- What would hurt most if it broke? (Money math, auth, data loss.) Test that first.
- Is there pure logic (calculations, parsing) that's easy to unit-test?
- Which user flows must always work (sign in, the main form, checkout)?
- Do tests need a database? A real one, a branch, or mocks?
- What do you want to learn: unit testing, browser automation, mocking networks, or property-based testing?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| TypeScript `tsc --noEmit` + ESLint | ★ default | Catching whole classes of bugs for free | Not behavior tests |
| Vitest | ★ default | Unit and integration tests for TS/Vite/Next projects | — |
| Playwright | ★ default | End-to-end browser tests of real flows | Slower; needs a running app |
| Testing Library | ◆ alternative | Testing React components the way users use them | Needs a DOM environment (jsdom/happy-dom) |
| Jest | ◆ alternative | Existing projects already on Jest | Slower and more config than Vitest for ESM/TS |
| pytest | ◆ alternative | Python backends | — |
| MSW (Mock Service Worker) | ✦ distinctive | Mocking HTTP APIs in tests and dev | Mocks can drift from reality |
| Property-based testing (fast-check) | ✦ distinctive | Finding edge cases in logic automatically | Thinking in properties takes practice |
| Visual regression (Playwright screenshots) | ✦ distinctive | Catching unintended UI changes | Flaky across environments if not pinned |

### Option notes

**Type check + lint** ★
- Pros: zero-maintenance safety net; catches typos, null errors, wrong props.
- What you'll learn: strict TypeScript settings, lint rules worth having.

**Vitest** ★
- Pros: fast, Jest-compatible API, native TS/ESM, watch mode, coverage.
- What you'll learn: unit tests, test doubles, coverage reports.

**Playwright** ★
- Pros: real browsers (Chromium, Firefox, WebKit); auto-waiting; codegen records tests; trace viewer for debugging; runs against preview URLs.
- What you'll learn: E2E testing, selectors by role, test fixtures, CI browsers.

**Testing Library** ◆
- Pros: queries by role/label encourage accessible UI.
- What you'll learn: component tests, accessibility-driven selectors.

**Jest** ◆
- Pick when: the project already uses it.

**pytest** ◆
- Pros: fixtures, parametrization; FastAPI's `TestClient`.

**MSW** ✦
- Pros: intercepts fetch at the network level in tests, Node and the browser.
- What you'll learn: network mocking, contract thinking.

**fast-check** ✦
- Pros: generates many inputs to find counterexamples (great for pay calculations or date math).
- What you'll learn: invariants, shrinking counterexamples.

**Visual regression** ✦
- Pros: `expect(page).toHaveScreenshot()` catches layout regressions.
- What you'll learn: snapshot baselines, deterministic rendering.

### How to choose

1. Always: type check + lint.
2. Logic-heavy code → Vitest (add fast-check for math/date logic).
3. Critical flows → 3–10 Playwright tests, not hundreds.
4. Component-heavy UI → Testing Library on top of Vitest.
5. Python backend → pytest.

## Configure

### Type check + lint

**Setup**
- [ ] `tsconfig.json` with `"strict": true`.
- [ ] Scripts: `"typecheck": "tsc --noEmit"`, `"lint": "eslint ."` (or Biome: `npx @biomejs/biome init`, `biome check`).
- [ ] Run both in CI (see [ci-cd](ci-cd.md)).

**Verify**
- [ ] Introducing a type error makes `npm run typecheck` fail.

### Vitest

**Setup**
- [ ] `npm i -D vitest` (plus `@vitejs/plugin-react` and `jsdom` or `happy-dom` for component tests).
- [ ] `vitest.config.ts` with path aliases matching `tsconfig` (`vite-tsconfig-paths`).
- [ ] Tests next to code (`calc.test.ts`) or in `tests/`.
- [ ] Scripts: `"test": "vitest"`, `"test:run": "vitest run"`, coverage with `--coverage`.

**Verify**
- [ ] `npm run test:run` passes; a deliberately broken function makes a test fail.

### Playwright

**Setup**
- [ ] `npm init playwright@latest` (creates config, example test, optional GitHub workflow).
- [ ] `webServer` in `playwright.config.ts` starts the app for local runs; `baseURL` from env for running against previews.
- [ ] Auth: log in once in a setup project and reuse `storageState`.
- [ ] Use role-based locators (`getByRole`, `getByLabel`).
- [ ] Keep test data isolated (seeded database or per-test cleanup).

**Verify**
- [ ] `npx playwright test` passes headless; `npx playwright show-report` shows results; traces captured on failure.

### Testing Library

**Setup**
- [ ] `npm i -D @testing-library/react @testing-library/user-event @testing-library/jest-dom`.
- [ ] Vitest `environment: "jsdom"` and a setup file importing `@testing-library/jest-dom/vitest`.

**Verify**
- [ ] A component test renders and interacts via `userEvent`.

### pytest

**Setup**
- [ ] `uv add --dev pytest httpx`; tests in `tests/`.
- [ ] FastAPI: `TestClient(app)`; override dependencies for the database.

**Verify**
- [ ] `uv run pytest` passes.

### MSW

**Setup**
- [ ] `npm i -D msw`; define handlers; `setupServer(...handlers)` in the Vitest setup file.

**Verify**
- [ ] Tests that call external APIs run offline.

### fast-check

**Setup**
- [ ] `npm i -D fast-check`; write properties with `fc.assert(fc.property(...))` inside Vitest tests.

**Verify**
- [ ] A seeded bug is found and shrunk to a minimal counterexample.

### Visual regression

**Setup**
- [ ] `await expect(page).toHaveScreenshot()` in Playwright; commit baseline images.
- [ ] Run in a fixed environment (the same Docker image in CI) to avoid font/rendering differences.

**Verify**
- [ ] A CSS change produces a diff image.

## Related

- [`ci-cd.md`](ci-cd.md), [`environments.md`](environments.md).
