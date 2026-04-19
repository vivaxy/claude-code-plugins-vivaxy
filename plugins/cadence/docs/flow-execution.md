# cadence Skill Execution Flow

> **Type**: Flow
> **Last Updated**: 2026-04-18
> **Covers**: End-to-end flow from user describing a feature to delivery

## Diagram

```mermaid
flowchart TD
    Start([User describes a feature task])

    Start --> UsingVivaxyWorkflow{"using-cadence<br>feature task?"}
    UsingVivaxyWorkflow -->|No — bug fix / docs| Passthrough([Proceed normally])
    UsingVivaxyWorkflow -->|Yes| CheckClarify{"Clarification<br>in conversation?"}

    CheckClarify -->|No| Clarify["cadence:main:clarify<br>Q&A with user,<br>outputs clarification summary"]
    Clarify --> CheckPlan{"Subtask plan<br>in conversation?"}

    CheckClarify -->|Yes| CheckPlan
    CheckPlan -->|No| Plan["cadence:main:plan<br>Decompose into subtasks,<br>update diagrams,<br>ExitPlanMode for approval,<br>output subtask plan to conversation"]
    Plan --> SubtaskLoop

    CheckPlan -->|Yes| SubtaskLoop{"Pending subtasks?"}
    SubtaskLoop -->|Yes| Execute["cadence:subtask-execute ST-XX<br>Read docs, TDD implementation,<br>update subtask status in conversation"]
    Execute --> SubtaskReview["cadence:subtask-review<br>(subagent)<br>Check acceptance criteria<br>+ diagram alignment"]
    SubtaskReview --> SubtaskVerdict{Verdict}
    SubtaskVerdict -->|NEEDS_WORK| FixSubtask[Fix issues in code]
    FixSubtask --> SubtaskReview
    SubtaskVerdict -->|ACCEPTED| MarkAccepted[Mark subtask ACCEPTED<br>in conversation]
    MarkAccepted --> SubtaskLoop

    SubtaskLoop -->|No — all ACCEPTED| Review["cadence:main:review<br>Verify all subtasks,<br>run test suite,<br>check success criteria"]
    Review --> FeatureVerdict{Verdict}
    FeatureVerdict -->|FEATURE_BLOCKED| FixFeature[Fix blocking issues]
    FixFeature --> Review
    FeatureVerdict -->|FEATURE_ACCEPTED| Deliver["cadence:main:deliver<br>Output retrospective to conversation,<br>deliver summary"]
    Deliver --> Done([Done])
```

## Key Decisions

- Clarification and subtask plan both live in conversation context — Cadence is session-scoped
- `cadence:main:plan` uses `EnterPlanMode`/`ExitPlanMode` as the user approval gate for the subtask plan
- Each subtask is independently executed and accepted before moving to the next
- `cadence:subtask-review` is a read-only subagent — it never writes files
- `cadence:main:review` runs the full test suite as part of end-to-end acceptance
- Deviations discovered during execution are recorded in the conversation rather than silently modifying diagrams

## Notes

- Cross-reference: `arch-modules.md` shows which files implement each step
- SessionStart hook injects Cadence routing guidance at the start of each session
