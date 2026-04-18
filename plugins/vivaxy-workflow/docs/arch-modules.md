# vivaxy-workflow Plugin — Module Architecture

> **Type**: Architecture
> **Last Updated**: 2026-04-07
> **Covers**: Internal component layout of the vivaxy-workflow plugin and their dependencies

## Diagram

```mermaid
graph TD
    SessionStart["hooks/session-start<br>(SessionStart hook)"]
    UsingVivaxyWorkflow["skills/using-vivaxy-workflow/SKILL.md<br>(routing skill)"]
    PlanSkill["skills/plan/SKILL.md<br>(vivaxy-workflow:plan)"]
    CodeSkill["skills/code/SKILL.md<br>(vivaxy-workflow:code)"]
    PlanReview["skills/plan-review/SKILL.md<br>(diagram review subagent)"]
    CodeReview["skills/code-review/SKILL.md<br>(code review subagent)"]
    WorkflowDocs["docs/<br>(design documents + diagram files)"]
    ProjectSrc["Project source files"]

    SessionStart -->|injects routing context via| UsingVivaxyWorkflow
    UsingVivaxyWorkflow -->|invokes| PlanSkill
    UsingVivaxyWorkflow -->|invokes| CodeSkill

    PlanSkill -->|creates / updates| WorkflowDocs
    PlanSkill -->|spawns subagent| PlanReview
    PlanReview -->|reads| WorkflowDocs
    CodeSkill -->|reads| WorkflowDocs
    CodeSkill -->|writes| ProjectSrc
    CodeSkill -->|records deviations in| WorkflowDocs
    CodeSkill -->|spawns subagent| CodeReview
    CodeReview -->|reads| WorkflowDocs
    CodeReview -->|reads| ProjectSrc
```

## Key Decisions

- Skills are instruction files, not executable code — Claude interprets them at runtime
- `using-vivaxy-workflow` skill is the single entry point — it detects feature tasks and invokes the appropriate skill automatically
- `vivaxy-workflow:plan` writes documents and diagrams; `vivaxy-workflow:code` never writes docs/diagrams (except deviation records)
- Review skills (`plan-review`, `code-review`) are read-only subagents — they never write files
- Dependency direction: code depends on documents and diagrams, documents and diagrams do not depend on code

## Notes

- Dependency direction: arrows point from dependent to dependency
- `hooks/run-hook.cmd` and `hooks/hooks.json` wire the SessionStart hook into Claude Code
- Plugin metadata lives in `.claude-plugin/` (not shown — not part of the vivaxy Workflow workflow)
