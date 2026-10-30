---
name: add-option
description: Extend or correct the stack playbook (rishipkothari/dev-stack-playbook) by adding a new option to a component file, adding a new component, or fixing outdated setup steps, keeping its Decide/Configure structure and README component map in sync. Use when a project used a tool the playbook doesn't cover, the user asks to add a framework, service or tool to the playbook, or a playbook setup step turned out wrong or outdated.
---

# Add or correct an option in the playbook

The playbook grows as the user's skill set grows. This skill edits the playbook's source in the GitHub repo `rishipkothari/dev-stack-playbook` (branch `main`), never the installed plugin copy.

## Getting the repo

Use the first method that works:

1. A local clone (check working directories and common locations; ask the user). `git pull` first.
2. In a Claude Code cloud session: attach the repo with the add-repo tool and clone it.
3. The GitHub connector or MCP tools (read each file with its SHA, write back with it).
4. The `gh` CLI contents API.

If none can write, produce the full edited files and tell the user where they go.

## The structure to keep

Every `components/<component>.md` has exactly these sections, in order:

```
# <Component>
<intro paragraph>

## Decide
### Questions to ask
### Options            (table: Option | Tier | Best for | Main trade-off)
### Option notes       (one bold-titled block per option: Pros, Cons, Pick when, What you'll learn)
### How to choose      (numbered rules)

## Configure
### Common to every option   (optional)
### <Option>                 (one per option: **Setup**, **Env vars**, **Verify**, **Gotchas** as needed, with - [ ] checklists)

## Related
```

Tiers: ★ default (at most one or two per component), ◆ alternative, ✦ distinctive (less common but teaches something the defaults don't).

## Steps

### 1. Pin down the change

Confirm with the user:
- Which component (or a new component), which option, and the tier.
- For a correction: which step was wrong and what actually worked (from the setup session if this follows `setup-component`).

### 2. Research

Check the option's current official docs (web search or fetch) for setup steps, env var names, pricing/free-tier shape and notable limits. Prefer stable facts; avoid exact prices that change often. Note anything you couldn't confirm.

### 3. Edit the component file

- Add a row to the Options table.
- Add an Option notes block (Pros, Cons, Pick when, What you'll learn). The "What you'll learn" line matters: it's what makes the playbook a learning tool.
- Update How to choose if the option changes the rules.
- Add a `### <Option>` block under Configure with Setup, Env vars, Verify and Gotchas.
- For a correction, fix the step in place and add a Gotcha if the mistake is easy to repeat.

For a **new component**, create `components/<name>.md` with the full structure, and add it to the component lists in `skills/plan-stack/SKILL.md`, `skills/setup-component/SKILL.md`, and the template table in `skills/plan-stack/stack-template.md`.

### 4. Update the README

Update the component's row in README.md → "Component map" (put the option in the right tier column), and the big-picture decisions if the option changes them.

### 5. Show, confirm, commit

Show the diff to the user and ask before committing. Commit with a message like `Add <option> to <component>` or `Fix <option> setup steps`, then push to `main`. Since the plugin has no `version` field, the push is the release; remind the user that installed copies pick it up on their next plugin update.
