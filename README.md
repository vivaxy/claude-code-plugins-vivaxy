# Graph Driven Development (GDD)

A Claude Code plugin that enforces development process consistency through Mermaid flowcharts and architecture diagrams.

**Core idea**: Diagrams are the source of truth. They change before code, and code is reviewed against them.

---

## Why GDD?

Without design documentation, coding agents drift: they fix the immediate problem but miss the broader system picture. Over time, the codebase diverges from intent, and each new feature makes the architecture harder to understand.

GDD addresses this by:

1. **Making design explicit** — Every module boundary and flow path is documented in `docs/gdd/`
2. **Enforcing design-first** — Code is only written after diagrams are updated and reviewed
3. **Closing the loop** — Code review checks implementation against diagrams, not just against itself

---

## Installation

### Via Claude Code Plugin Marketplace

Clone the repository to a local directory, then add the marketplace using a relative path:

```bash
# Clone the repository
git clone https://github.com/vivaxy/claude-code-plugin-graph-driven-development ~/claude-code-plugin-graph-driven-development
```

In Claude Code, register the marketplace using the local path (first time only):

```
/plugin marketplace add ~/claude-code-plugin-graph-driven-development/.claude-plugin/marketplace.json
```

Then install the plugin:

```
/plugin install gdd@gdd-marketplace
```

### Manual Installation (Global)

Copy the `commands/gdd/` directory to your global Claude config:

```bash
cp -r commands/gdd/ ~/.claude/commands/gdd/
cp CLAUDE.md ~/.claude/CLAUDE.md   # or append to existing ~/.claude/CLAUDE.md
```

### Manual Installation (Project-Level)

```bash
mkdir -p .claude/commands/gdd/
cp -r commands/gdd/ .claude/commands/gdd/
# Append CLAUDE.md content to your project's .claude/CLAUDE.md
```

---

## Quick Start

### 1. Initialize GDD for Your Project

Run this once when adopting GDD for a project:

```
/gdd:init
```

Claude will scan your codebase, detect the project type, and generate an appropriate set of diagrams in `docs/gdd/`. Review and correct them before proceeding.

### 2. Plan a New Feature

Before writing any code:

```
/gdd:plan Add user authentication with JWT tokens
```

Claude analyzes the current diagrams, presents the proposed changes in the conversation, and waits for your confirmation before writing anything to `docs/gdd/`.

### 3. Review the Plan

```
/gdd:plan-review
```

Claude checks the diagrams for completeness, consistency, and design quality. Fix any critical issues before coding.

### 4. Implement

```
/gdd:code Implement JWT authentication middleware
```

Claude reads the diagrams, extracts implementation constraints, then writes code that follows the defined boundaries and flows. Any necessary deviations are recorded.

### 5. Review the Code

```
/gdd:code-review
```

Claude verifies the implementation against the diagrams (Part 1) and reviews code quality (Part 2). The report tells you what to fix and whether diagram updates are needed.

---

## The GDD Workflow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────┐
│  /gdd:init  │────▶│  /gdd:plan  │────▶│ /gdd:plan-review │
│ (once/reset)│     │  (per req.) │     │   (per req.)    │
└─────────────┘     └─────────────┘     └────────┬────────┘
                                                  │ APPROVED
                                                  ▼
                                        ┌─────────────────┐
                                        │   /gdd:code     │
                                        │  (implement)    │
                                        └────────┬────────┘
                                                  │
                                                  ▼
                                        ┌─────────────────┐
                                        │ /gdd:code-review│
                                        │  (verify)       │
                                        └─────────────────┘
```

Diagrams live in `docs/gdd/`. They change first, code follows.

---

## docs/gdd Directory Layout

```
docs/gdd/
├── overview.md           # System context / big-picture boundary
├── flow-*.md             # Business process, request, data flow diagrams
├── arch-*.md             # Module dependency / component architecture diagrams
└── drafts/
    └── draft-deviation-*.md  # Recorded deviations from gdd:code
```

Each `.md` file contains Mermaid diagram(s) plus explanatory text:

```markdown
# Request Flow

> **Type**: Flow
> **Last Updated**: 2026-01-23
> **Covers**: How HTTP requests are processed end-to-end

## Diagram

\`\`\`mermaid
flowchart TD
    Client -->|HTTP POST /api/users| Router
    Router --> AuthMiddleware
    AuthMiddleware -->|valid token| UserHandler
    AuthMiddleware -->|invalid token| E_401[401 Unauthorized]
    UserHandler --> UserService
    UserService --> Database
    Database -->|result| UserService
    UserService -->|user object| UserHandler
    UserHandler -->|200 OK| Client
\`\`\`

## Key Decisions

- Auth is checked at middleware level, not inside handlers
- UserService owns all DB interactions for user data
```

---

## Command Reference

### `/gdd:init [--force]`

Initialize GDD for the current project.

- Scans codebase, detects project type
- Generates appropriate diagram set in `docs/gdd/`
- `--force`: Regenerate even if already initialized

**When to use**: First time adopting GDD, or when the project structure has fundamentally changed.

---

### `/gdd:plan <requirement>`

Propose diagram updates for a new requirement and apply them after confirmation.

- Reads current diagrams
- Analyzes impact of the requirement
- Presents the proposed Before/After diagram changes in the conversation
- Writes the approved changes directly to `docs/gdd/` after user confirmation

**When to use**: Before starting any new feature or change.

---

### `/gdd:plan-review`

Review diagrams for quality and completeness.

- Checks completeness, consistency, edge cases, and feasibility
- Outputs a report with `[CRITICAL]`, `[WARNING]`, `[SUGGESTION]` findings
- Gives a verdict: `APPROVED`, `APPROVED_WITH_WARNINGS`, `NEEDS_WORK`, or `BLOCKED`

**When to use**: After updating diagrams, before coding.

---

### `/gdd:code <task description>`

Implement code guided by GDD diagrams.

- Reads diagrams and extracts implementation constraints
- Writes code that follows defined module boundaries and flow order
- Records any necessary deviations in `docs/gdd/drafts/draft-deviation-*.md`

**When to use**: When diagrams are approved and you're ready to implement.

---

### `/gdd:code-review`

Review code against GDD diagrams and for quality.

- **Part 1 — Diagram Alignment**: Verifies code matches diagrams
- **Part 2 — Code Quality**: Identifies bugs, complexity, and maintainability issues
- Outputs verdict: `APPROVED`, `APPROVED_WITH_WARNINGS`, or `NEEDS_WORK`

**When to use**: After implementation, before merging.

---

## Best Practices

### Keep Diagrams at the Right Level of Abstraction

Too detailed → maintenance burden that slows you down  
Too abstract → no actual constraint on implementation

**Good**: Show module boundaries, decision points, external system interactions  
**Avoid**: Function signatures, variable types, implementation details

### One Source of Truth

Diagrams in `docs/gdd/` (not `drafts/`) are authoritative. When code and diagrams disagree:

1. If the code is right → update the diagram via `/gdd:plan`
2. If the diagram is right → fix the code

Never silently accept drift.

### Diagrams Are Living Documents

Update them when you learn something new. A diagram that accurately reflects a simpler system is better than one that aspires to a complex system that doesn't exist.

### Use GDD for New Features, Not Archaeology

Don't run `/gdd:init` on a legacy codebase expecting perfect diagrams. Use it to start a diagram set that's roughly accurate, then refine as you work on each area.

---

## Troubleshooting

**"GDD is not initialized"**  
Run `/gdd:init` first.

**The generated diagrams are inaccurate**  
This is expected for complex codebases. Correct them manually — they are Markdown files with Mermaid blocks, easy to edit. Accuracy improves over time as you run `/gdd:plan` for each change.

**Code review finds too many deviations**  
If deviations are consistently valid (the code is right, the diagram is wrong), the diagrams need to be updated via `/gdd:plan`. If deviations are consistently invalid (the code ignored the diagram), enforce the GDD workflow more strictly.

---

## License

MIT
