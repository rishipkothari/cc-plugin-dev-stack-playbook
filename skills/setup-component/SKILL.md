---
name: setup-component
description: Guided, teaching-oriented setup of one stack component (hosting, database, auth, payments, email, testing, CI/CD, monitoring, etc.) by working through its Configure checklist from the stack playbook. Use when the user asks to set up, configure, wire up, connect or add a stack piece to a project (e.g. "set up auth", "add Stripe", "connect the database", "add tests and CI"), or to continue the setup checklist in docs/stack.md.
---

# Set up one stack component

You help the user implement one component of the project's stack, using the playbook's Configure checklist. The user is building skills: explain why each step exists, let them do steps themselves when they want to, and verify everything.

## Where the playbook files are

- Component files: `${CLAUDE_PLUGIN_ROOT}/components/<component>.md`; the `## Configure` section has one `### <Option>` per option with Setup, Env vars, Verify and Gotchas, plus a "Common to every option" block where present.
- If `${CLAUDE_PLUGIN_ROOT}` was not substituted or the path doesn't exist, look at `../../components/` relative to this skill's folder, then at the GitHub repo `rishipkothari/dev-stack-playbook` (`main`) via `https://raw.githubusercontent.com/rishipkothari/dev-stack-playbook/main/components/<component>.md` or the GitHub connector.

Component files: `hosting`, `framework`, `backend`, `database`, `migrations`, `auth`, `secrets`, `payments`, `apis`, `storage`, `email`, `jobs`, `domains`, `environments`, `testing`, `ci-cd`, `monitoring`, `costs`.

## Steps

### 1. Establish what to set up

- Read the project's `docs/stack.md` if it exists. Take the component and the chosen option from it; if the user named a component, use that. If unclear, show the next unchecked item in the Setup checklist and confirm.
- If there's no `docs/stack.md`, ask which option they're using. Offer the `plan-stack` skill if they haven't decided, but don't force it for a single component.
- Look at the actual codebase (framework, package manager, existing config) so steps fit what's there.

### 2. Read the checklist

Read the component file's `## Configure` → "Common to every option" (if present) and `### <Option>`. If the option isn't in the file, say so, offer the `add-option` skill afterwards, and work from the provider's current official docs.

### 3. Plan, then go step by step

Show the plan as a short numbered list, labeling each step:
- **You**: needs the user (creating accounts, dashboard clicks, payments, DNS at their registrar, pasting secrets).
- **Claude**: code, config files, commands you can run.
- **Either**: the user may want to do it to learn.

Then for each step:
1. Say in one or two sentences what it does and why it matters (what would break without it).
2. For **Either** steps, offer: "want to do this one yourself?" If yes, give precise instructions and review their result.
3. For **You** steps, give exact instructions. Dashboard layouts change: if what they see differs, ask what they see and adapt.
4. Never ask the user to paste secret values into the chat. Have them put secrets directly into `.env.local` and the host's env settings; refer to variables by name.
5. Check the step worked before moving on.

Follow the project's own instructions (e.g. its CLAUDE.md) on running checks and on asking before commits or pushes.

### 4. Verify

Run every item in the option's **Verify** list that can be run (build, tests, a request to the deployed app). For items only the user can check (an email arriving, a dashboard status), ask them to confirm. Call out any **Gotchas** that apply.

### 5. Record it

Update `docs/stack.md`:
- Tick the component in the Setup checklist.
- Fill in or correct the Environment variables table.
- Add anything learned (surprises, deviations from the plan) to Decision notes, Open questions, and a dated line in the Log.

### 6. Improve the playbook

If any step in the playbook was wrong, outdated or missing, tell the user and offer to fix the component file with the `add-option` skill (it also handles corrections). Suggest the next unchecked component.

## Related skills

Use specialist skills where they apply: `claude-api` for Claude API work, and any installed skill specific to the tool being set up.
