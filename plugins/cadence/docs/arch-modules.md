# cadence Plugin — Module Architecture

> **Type**: Architecture
> **Last Updated**: 2026-04-18
> **Covers**: Internal component layout of the cadence plugin and their dependencies

## Diagram

```mermaid
graph TD
    SessionStart["hooks/session-start<br>(SessionStart hook)"]
    UsingVivaxyWorkflow["skills/using-cadence<br>(routing skill)"]
    ClarifySkill["skills/main-clarify<br>(cadence:main:clarify)"]
    PlanSkill["skills/main-plan<br>(cadence:main:plan)"]
    SubtaskExecuteSkill["skills/subtask-execute<br>(cadence:subtask-execute)"]
    SubtaskReviewSkill["skills/subtask-review<br>(subagent)"]
    ReviewSkill["skills/main-review<br>(cadence:main:review)"]
    DeliverSkill["skills/main-deliver<br>(cadence:main:deliver)"]
    WorkflowDocs["docs/<br>(diagram files)"]
    ProjectSrc["Project source files"]

    SessionStart -->|injects routing context via| UsingVivaxyWorkflow
    UsingVivaxyWorkflow -->|invokes| ClarifySkill
    UsingVivaxyWorkflow -->|invokes| PlanSkill
    UsingVivaxyWorkflow -->|invokes| SubtaskExecuteSkill
    UsingVivaxyWorkflow -->|invokes| ReviewSkill
    UsingVivaxyWorkflow -->|invokes| DeliverSkill

    ClarifySkill -->|outputs clarification summary to conversation| UsingVivaxyWorkflow
    PlanSkill -->|reads| WorkflowDocs
    PlanSkill -->|updates diagrams| WorkflowDocs
    PlanSkill -->|outputs subtask plan to conversation| UsingVivaxyWorkflow
    SubtaskExecuteSkill -->|reads| WorkflowDocs
    SubtaskExecuteSkill -->|writes| ProjectSrc
    SubtaskExecuteSkill -->|records deviations in conversation| UsingVivaxyWorkflow
    SubtaskExecuteSkill -->|updates subtask status in conversation| UsingVivaxyWorkflow
    SubtaskExecuteSkill -->|spawns subagent| SubtaskReviewSkill
    SubtaskReviewSkill -->|reads| WorkflowDocs
    SubtaskReviewSkill -->|reads| ProjectSrc
    ReviewSkill -->|reads| WorkflowDocs
    ReviewSkill -->|reads| ProjectSrc
    DeliverSkill -->|reads| WorkflowDocs
    DeliverSkill -->|outputs retrospective to conversation| UsingVivaxyWorkflow
```

## Key Decisions

- Skills are instruction files, not executable code — Claude interprets them at runtime
- `using-cadence` is the single entry point — it detects feature tasks and routes to the correct phase automatically
- `cadence:subtask-execute` never modifies `flow-*.md` or `arch-*.md` — deviations are recorded in the conversation
- `cadence:subtask-review` is a read-only subagent — it never writes files
- Workflow state (subtask plan, deviations, retrospective) lives in the conversation context — Cadence is session-scoped

## Notes

- `hooks/run-hook.cmd` and `hooks/hooks.json` wire the SessionStart hook into Claude Code
- Plugin metadata lives in `.claude-plugin/` (not shown — not part of the Cadence workflow)
