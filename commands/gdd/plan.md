---
name: plan
description: Plan a new requirement — update flowcharts and architecture diagrams to reflect the proposed changes, generating a draft for user approval
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
For a given requirement or feature request, analyze the existing GDD diagrams and produce a draft proposal showing exactly how the diagrams need to change. The draft is saved to `docs/gdd/drafts/` for user review before any code is written.

The core principle: **diagrams change first, code follows**.
</objective>

<process>

## Step 1: GDD Completeness Check

Before doing anything else, verify that GDD is properly initialized:

1. Check if `docs/gdd/` directory exists
2. Check if at least one `flow-*.md` file exists in `docs/gdd/`
3. Check if at least one `arch-*.md` file exists in `docs/gdd/`
4. Check if `docs/gdd/drafts/` contains any `draft-*.md` files

**If any of checks 1–3 fail:**
```
GDD is not initialized for this project.

Please run /gdd:init first to generate the initial diagram set,
then return to /gdd:plan for your requirement.
```
STOP — do not proceed.

**If check 4 finds existing drafts:**
List the existing draft files and their titles, then ask:
```
There are existing unreviewed draft proposals:
- docs/gdd/drafts/draft-plan-2026-01-23-10-30.md — "Add authentication flow"

Do you want to:
(a) Proceed with a new draft for the current requirement
(b) Review/apply the existing draft first

(Continuing with option a by default if you provide the requirement)
```
Note this in the draft but do not block.

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

## Step 5: Generate Draft Proposal

Create `docs/gdd/drafts/draft-plan-<YYYY-MM-DD-HH-MM>.md` with this exact structure:

```markdown
# Draft Plan: <Requirement Title>

> **Status**: PENDING_REVIEW
> **Created**: <YYYY-MM-DD HH:MM>
> **Requirement**: <one-line summary>
> **Affects**:
> - flow-request.md (major update)
> - arch-modules.md (minor update)

## Requirement Summary

<2–4 sentences describing what the requirement entails and why it's needed>

## Impact Analysis

<Brief explanation of what changes are needed and why>

## Diagram Changes

<!-- Repeat this block for each affected diagram -->

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

## Design Considerations

<Any trade-offs, alternatives considered, or open questions the user should think about>

## How to Apply This Draft

Once you've reviewed and approved this proposal:

1. For each modified diagram: replace the "Before" content in the actual file with the "After (Proposed)" content
2. For each new file: create the file with the "Proposed Diagram" content
3. Delete this draft file from docs/gdd/drafts/
4. Run /gdd:plan-review to validate the updated diagrams
5. Run /gdd:code to begin implementation
```

Make sure:
- The "Before" section contains the **exact current** diagram content (copy verbatim from the source file)
- The "After (Proposed)" section contains a **valid, complete** Mermaid diagram
- All proposed changes are grounded in the actual codebase, not invented

## Step 6: Confirmation Output

After writing the draft file, output:

```
Draft proposal created: docs/gdd/drafts/draft-plan-<timestamp>.md

Summary:
- Affects N diagram(s): [list affected files]
- Creates N new diagram(s): [list new files]

Next steps:
1. Review the draft: docs/gdd/drafts/draft-plan-<timestamp>.md
2. If approved, apply the changes to docs/gdd/ and delete the draft
3. Run /gdd:plan-review to validate the updated diagrams
4. Run /gdd:code to begin implementation

To apply the draft automatically, run:
  /gdd:plan --apply docs/gdd/drafts/draft-plan-<timestamp>.md
```

## Optional: --apply Flag

If called with `--apply <draft-file-path>`:

1. Read the specified draft file
2. Verify its status is `PENDING_REVIEW`
3. Apply all changes:
   - For "Modifying" entries: replace the diagram in the target file with the "After (Proposed)" content, update the "Last Updated" date
   - For "New File" entries: create the new file in `docs/gdd/`
4. Delete the draft file
5. Output confirmation of applied changes

</process>

<guidelines>
- The draft must be self-contained: a reader who hasn't seen the requirement should understand exactly what is changing and why
- Never modify files in docs/gdd/ (only drafts/) unless --apply flag is used
- Proposed diagrams must be syntactically valid Mermaid — test mentally by reading the graph structure
- If a requirement is too large (affects > 5 diagrams), consider splitting into sub-requirements
- "Before" sections must be copied verbatim from the actual current diagram files — never paraphrase
</guidelines>
