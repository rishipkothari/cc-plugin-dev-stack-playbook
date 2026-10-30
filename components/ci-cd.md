# CI/CD

Continuous integration (automatic checks on every push and pull request) and continuous deployment (shipping when checks pass). On Vercel-style hosts, deployment is already wired to git; CI adds the checks that should block a bad deploy.

## Decide

### Questions to ask

- Which checks must pass before code reaches `main`? (Type check, lint, tests, build.)
- Do you work through pull requests, or push straight to `main`?
- Should migrations run automatically on deploy?
- Do you want automated dependency updates?
- What do you want to learn: GitHub Actions basics, branch protection, or release automation?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| GitHub Actions + host git integration | ★ default | Checks in Actions, deploys by the host | Two systems to look at |
| Dependabot / Renovate | ◆ alternative | Automated dependency update PRs | PR noise without grouping |
| Branch protection + required checks | ✦ distinctive | Making CI actually block merges | Slower for solo hotfixes |
| Deploy from Actions (host CLI) | ◆ alternative | Hosts without git integration, or full control | More workflow code |
| Release automation (release-please, Changesets) | ✦ distinctive | Versioned releases and changelogs | Overkill for most apps |
| Pre-commit hooks (Husky / lefthook) | ◆ alternative | Catching issues before pushing | Can be bypassed; slows commits |

### Option notes

**GitHub Actions + host integration** ★
- Pros: Actions is free for public repos and has free minutes for private ones; the host deploys on push regardless.
- What you'll learn: workflow YAML, job matrices, caching, secrets in CI.

**Dependabot / Renovate** ◆
- Pros: keeps dependencies current and patched; Renovate groups and schedules updates flexibly.
- What you'll learn: semver, lockfiles, update hygiene.

**Branch protection** ✦
- Pros: PRs can't merge until checks pass; Vercel can also wait for checks before promoting.
- What you'll learn: PR-based workflow, required status checks, rulesets.

**Deploy from Actions** ◆
- Pros: deploy only after tests pass in the same pipeline.
- What you'll learn: CLI deploys, deploy tokens, environments with approvals.

**Release automation** ✦
- Pros: conventional commits drive version bumps and changelogs.
- What you'll learn: semantic versioning, release pipelines.

**Pre-commit hooks** ◆
- Pros: fast feedback.
- What you'll learn: git hooks, lint-staged.

### How to choose

1. Every project: an Actions workflow running typecheck, lint, tests and build on PRs and pushes.
2. Add Dependabot (simplest) or Renovate (more control).
3. Working through PRs → add branch protection with required checks.
4. Libraries or versioned products → release automation.

## Configure

### GitHub Actions + host integration

**Setup**
- [ ] `.github/workflows/ci.yml`:
  - `on: [push, pull_request]` (or `push` to main + `pull_request`).
  - `actions/checkout`, `actions/setup-node` with `node-version` from `.nvmrc` and `cache: npm`.
  - `npm ci`, then `npm run typecheck`, `npm run lint`, `npm run test:run`, `npm run build`.
- [ ] Provide dummy build-time env vars in the workflow if the build needs them (e.g. `DATABASE_URL: postgres://dummy`), real secrets only when a job truly needs them.
- [ ] E2E job: install browsers (`npx playwright install --with-deps`), run against a local build or the preview URL; upload the report as an artifact.
- [ ] Python: `astral-sh/setup-uv`, `uv sync`, `uv run pytest`.
- [ ] Keep the host's git integration on for deploys.

**Verify**
- [ ] A PR shows the CI checks; a failing test turns the check red.

**Gotchas**
- Pin action versions (`@v4` or a commit SHA).
- Secrets aren't available to workflows triggered by PRs from forks.

### Dependabot / Renovate

**Setup**
- [ ] Dependabot: `.github/dependabot.yml` with `package-ecosystem: npm` (and `github-actions`), weekly schedule, grouped updates.
- [ ] Or Renovate: install the GitHub app; `renovate.json` with `extends: ["config:recommended"]` and grouping/automerge for patch updates.

**Verify**
- [ ] An update PR appears and runs CI.

### Branch protection + required checks

**Setup**
- [ ] Repo **Settings → Rules → Rulesets** (or Branches): protect `main`, require PRs and the CI status checks.
- [ ] Optionally require linear history and block force pushes.
- [ ] On Vercel, optionally enable deployment checks so production waits for CI.

**Verify**
- [ ] A PR with failing CI can't be merged.

### Deploy from Actions

**Setup**
- [ ] Disable auto-deploy on the host (or keep previews only).
- [ ] Add the host token as an Actions secret (e.g. `VERCEL_TOKEN`, plus org/project IDs).
- [ ] Deploy job depends on the test job: `vercel pull`, `vercel build --prod`, `vercel deploy --prebuilt --prod`.
- [ ] Optionally use a GitHub Environment with required reviewers for production.

**Verify**
- [ ] A push with failing tests doesn't deploy.

### Release automation

**Setup**
- [ ] release-please: `googleapis/release-please-action` workflow; use conventional commit messages.
- [ ] Or Changesets: `npx @changesets/cli init`; add a changeset per change.

**Verify**
- [ ] A release PR with a changelog is opened automatically.

### Pre-commit hooks

**Setup**
- [ ] `npm i -D husky lint-staged` (or lefthook); `npx husky init`.
- [ ] Hook runs `lint-staged` (format + lint changed files) and optionally `typecheck`.

**Verify**
- [ ] Committing a lint error is blocked.

## Related

- [`testing.md`](testing.md), [`migrations.md`](migrations.md), [`hosting.md`](hosting.md).
