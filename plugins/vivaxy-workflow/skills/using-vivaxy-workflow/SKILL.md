---
name: vivaxy-workflow:using-vivaxy-workflow
description: Guide for when to apply the vivaxy Workflow — read at session start to understand when vivaxy Workflow applies and what to do first
---

<SUBAGENT-STOP>
If you were dispatched as a subagent to execute a specific task, skip this skill.
</SUBAGENT-STOP>

# Using vivaxy Workflow

vivaxy Workflow enforces a spec-first development process: write/update design documents and architecture diagrams before writing code.

## When vivaxy Workflow Applies

| Task type | vivaxy Workflow required? |
|-----------|--------------|
| Implement a new feature | YES — invoke `vivaxy-workflow:plan` skill |
| Add a new API endpoint or route | YES — invoke `vivaxy-workflow:plan` skill |
| Refactor a module's structure | YES — invoke `vivaxy-workflow:plan` skill |
| Add a new component / service | YES — invoke `vivaxy-workflow:plan` skill |
| Fix a bug in existing code | NO |
| Fix a typo or rename | NO |
| Write or update tests | NO |
| Answer a question | NO |
| Update documentation | NO |
| Update a config value | NO |

**Rule**: If the task adds new behavior or changes how modules interact — vivaxy Workflow applies. If it corrects something that was supposed to work already — vivaxy Workflow does not apply.

## Routing Logic

1. Is this a feature/new-functionality task?
   - NO → proceed normally
   - YES → check project state:

2. Does `docs/` exist with at least one `flow-*.md` AND one `arch-*.md`?
   - NO → invoke `vivaxy-workflow:plan` skill directly (it will auto-initialize `docs/` as needed)
   - YES → invoke `vivaxy-workflow:plan` skill directly with the user's requirement as the argument

## When vivaxy Workflow Is Not Initialized

Do not wait for user confirmation. Immediately invoke the `vivaxy-workflow:plan` skill — it will proactively create the missing `docs/` files as part of the plan.

## When vivaxy Workflow Is Initialized

Do not wait for user confirmation. Immediately:

1. Say one line: "vivaxy Workflow is active — running `vivaxy-workflow:plan` for your requirement."
2. Invoke the `vivaxy-workflow:plan` skill with the user's requirement as the argument.

## Instruction Priority

1. User's explicit instructions (CLAUDE.md, direct requests) — highest
2. vivaxy Workflow — for feature development tasks
3. Default behavior — for everything else

If the user says "just implement it, skip the spec" — respect that.
