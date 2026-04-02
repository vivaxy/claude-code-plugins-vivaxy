---
name: gdd:plan
description: Plan a new requirement — update flowcharts and architecture diagrams to reflect the proposed changes, with user confirmation before writing
argument-hint: "<requirement description>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - TodoWrite
---

<objective>
For a given requirement or feature request, analyze the existing GDD diagrams and produce a proposal showing exactly how the diagrams need to change. Present the proposal to the user for confirmation, then directly write the approved changes to `docs/gdd/`.

The core principle: **diagrams change first, code follows**.
</objective>

<process>

## Step 1: GDD Completeness Check

Before doing anything else, verify that GDD is properly initialized:

1. Check if `docs/gdd/` directory exists
2. Check if at least one `flow-*.md` file exists in `docs/gdd/`
3. Check if at least one `arch-*.md` file exists in `docs/gdd/`

**If any of checks 1–3 fail:**
```
GDD is not initialized for this project.

Please run /gdd:init first to generate the initial diagram set,
then return to /gdd:plan for your requirement.
```
STOP — do not proceed.

## Step 2: Read Current Diagrams

Read all files in `docs/gdd/` (excluding `drafts/`):
- `overview.md`
- All `flow-*.md` files
- All `arch-*.md` files

Build a mental model of the current system design.

## Step 3: Understand the Requirement

The requirement is provided as `$ARGUMENTS`. If it's too brief or ambiguous:
- Ask one focused clarifying question
- Do not ask multiple questions at once
- If the requirement is clear enough, proceed

## Step 4: Impact Analysis

Analyze which diagrams are affected by this requirement:

For each existing diagram, determine:
- **No change needed**: requirement has no impact
- **Minor update**: add/rename a node, add an edge
- **Major update**: new subgraph, new flow path, new module
- **New diagram needed**: entirely new flow or architecture area

Also determine if:
- New `flow-*.md` files need to be created
- New `arch-*.md` files need to be created

## Step 5: Present Proposal and Wait for Confirmation

Output the full proposal in the conversation using this structure:

```
## Proposed Diagram Changes

**Requirement**: <one-line summary>
**Affects**:
- flow-request.md (major update)
- arch-modules.md (minor update)

### Requirement Summary

<2–4 sentences describing what the requirement entails and why it's needed>

### Impact Analysis

<Brief explanation of what changes are needed and why>

---

### [Modifying] flow-request.md

**Change type**: Major update
**Reason**: <Why this change is needed>

#### Before

\`\`\`mermaid
<full current diagram content>
\`\`\`

#### After (Proposed)

\`\`\`mermaid
<full proposed diagram content>
\`\`\`

#### What Changed

- Added node X to represent Y
- New edge from A to B because Z
- Renamed C to D for clarity

---

### [New File] flow-auth.md

**Reason**: <Why a new diagram is needed>

#### Proposed Diagram

\`\`\`mermaid
<full new diagram content>
\`\`\`

#### Key Decisions

- Decision 1: rationale
- Decision 2: rationale

---

### Design Considerations

<Any trade-offs, alternatives considered, or open questions the user should think about>
```

Then ask:
```
Please review the proposed diagram changes above.

Reply with:
- "yes" or "confirm" to apply the changes directly to docs/gdd/
- Any feedback or corrections to revise the proposal
```

**Wait for user confirmation before proceeding.**

## Step 6: Apply Confirmed Changes

Once the user confirms, directly write all changes to `docs/gdd/`:

1. For each "Modifying" entry: replace the diagram content in the actual file with the "After (Proposed)" content, update the "Last Updated" date
2. For each "New File" entry: create the new file in `docs/gdd/` using the standard diagram file format

Output confirmation:
```
Changes applied to docs/gdd/:

- Updated: flow-request.md
- Created: flow-auth.md

Next steps:
1. Run /gdd:plan-review to validate the updated diagrams
2. Run /gdd:code to begin implementation
```

</process>

<guidelines>
- The proposal must be self-contained: a reader who hasn't seen the requirement should understand exactly what is changing and why
- Directly write to docs/gdd/ only after explicit user confirmation
- Proposed diagrams must be syntactically valid Mermaid — test mentally by reading the graph structure
- If a requirement is too large (affects > 5 diagrams), consider splitting into sub-requirements
- "Before" sections must be copied verbatim from the actual current diagram files — never paraphrase
</guidelines>
