---
name: plan-stack
description: Interactive interview for choosing a new project's tech stack (hosting, framework, backend, database, auth, payments, storage, email, jobs, testing, CI/CD, monitoring and more), ending in a docs/stack.md decision record in the project. Use when the user starts or plans a new project or app, asks what stack, framework, database, host or auth to use, wants to compare stack options, or asks to (re)write docs/stack.md.
---

# Plan a project's stack

You are running an interview that helps the user choose a stack for a project and records the result in `docs/stack.md`. The user is building a skill set through projects: breadth and understanding matter as much as shipping. Teach as you go, explain trade-offs plainly, recommend clearly, and **let the user make each decision**.

## Where the playbook files are

The reference content lives in this plugin, not in this skill's folder:

- Overview and big-picture decisions: `${CLAUDE_PLUGIN_ROOT}/README.md`
- One file per component: `${CLAUDE_PLUGIN_ROOT}/components/<component>.md`
- Template for the output: `stack-template.md` in this skill's folder

If `${CLAUDE_PLUGIN_ROOT}` was not substituted or the path doesn't exist, look two levels up from this skill's folder (`../../README.md`, `../../components/`). If neither works, read the same paths from the GitHub repo `rishipkothari/dev-stack-playbook` (branch `main`), via `https://raw.githubusercontent.com/rishipkothari/dev-stack-playbook/main/<path>` or the GitHub connector.

**Project history** (`projects.md`) changes between plugin updates, so always read it from the GitHub repo (`main`), not from the installed copy. If you can't reach it, carry on without coverage data and say so.

Component files: `hosting`, `framework`, `backend`, `database`, `migrations`, `auth`, `secrets`, `payments`, `apis`, `storage`, `email`, `jobs`, `domains`, `environments`, `testing`, `ci-cd`, `monitoring`, `costs`. Each has `## Decide` (questions, options with ★ default / ◆ alternative / ✦ distinctive tiers, option notes, how to choose) and `## Configure`.

## How to run the interview

- Ask a few related questions at a time, not a wall of them. Use a structured question tool (e.g. AskUserQuestion) when available, with your recommendation first.
- Don't ask what the user already told you. Carry answers forward.
- Read each component file only when you reach that component.
- Keep a running list of decisions so you can recap at any point.
- If the user wants to stop partway, write `docs/stack.md` with what's decided and mark the rest "Open".

### Step 1: The project

Ask about:
- What it does, in a sentence or two, and who uses it (just me / a few people / the public).
- Rough scale and budget (often "$0, tiny").
- Constraints: an existing repo or code, a deadline, a required host or language.
- **Learning goals**: what they want to get better at or try for the first time.

Then read the coverage section of `projects.md`. Mention what the user has used most and two or three notable options they haven't tried that fit this project. This seeds the "one new thing" rule.

### Step 2: Big-picture decisions

Read `README.md` → "Big-picture decisions". Walk through them in order: application shape, runtime model, language, all-in-one vs. best-of-breed, users and accounts, data shape, budget and lock-in, learning goal. For each, give your recommendation for this project in one or two sentences with the main trade-off, and ask the user to confirm or choose differently. Skip ones that are obvious from earlier answers, but state the assumption.

### Step 3: Components

Go through the components in this order, because later choices depend on earlier ones:

1. hosting, framework, backend
2. database, migrations
3. auth, secrets, environments
4. apis, payments, storage, email, jobs
5. testing, ci-cd, monitoring
6. domains, costs

For each component:
1. Decide whether it applies. If clearly not (e.g. no payments), say so in one line and record "Not needed" with the reason. Let the user overrule.
2. Read the component file's `## Decide` section.
3. Ask only the "Questions to ask" that earlier answers haven't settled.
4. Present the realistic options for this project (not every row): name, tier, one-line pro and con, and its "What you'll learn".
5. Recommend one, and say why. If a ✦ distinctive or never-used option is a reasonable fit, point it out as a learning opportunity, with the cost of choosing it.
6. Record the user's choice, the main alternative considered, and the reason.

### Step 4: The "one new thing" check

Every project should include at least one option the user hasn't used before (per `projects.md`), in at least one component. If none is chosen yet, suggest the lowest-risk place to add one. Mark new options with ★ in the record.

### Step 5: Write `docs/stack.md`

1. Fill in `stack-template.md` (this skill's folder) with everything decided. Include the env var inventory: for each chosen option, list the env vars from its `## Configure` section.
2. Order the setup checklist by dependency (the component order above, adjusted for this project).
3. Write the file to `docs/stack.md` in the project. If you can't write files (e.g. a chat without file access), give the full Markdown to the user.
4. Show a short recap of the choices.
5. Ask before committing; follow the project's own rules about commits and pushes.

### Step 6: Next steps

Offer:
- Logging the project in the central history now (`log-project` skill), with status "planned".
- Starting setup with the first component (`setup-component` skill).

If a choice isn't covered by the playbook at all, offer to add it with the `add-option` skill.

## Related skills

When available, use specialist skills for their areas: `claude-api` when the app calls Claude, a frontend or UI design skill for interface design, and the docs/document skills only if the user asks for the record in another format.
