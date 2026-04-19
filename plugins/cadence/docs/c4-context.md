# cadence Plugin — System Context

> **Type**: C4 Context
> **Last Updated**: 2026-04-19
> **Covers**: System boundary and external actors for the Cadence Claude Code plugin

## Diagram

```mermaid
C4Context
  title System Context — cadence Plugin

  Person(developer, "Developer", "Uses Claude Code to build software features")

  Enterprise_Boundary(claudeCode, "Claude Code") {
    System(cadence, "cadence Plugin", "Enforces spec-first development: design documents and diagrams before code")
  }

  System_Ext(projectDocs, "docs/", "Authoritative design documents and C4 diagram files for the user's project")
  System_Ext(projectSrc, "Project Source Code", "The user's application source files")

  Rel(developer, cadence, "Describes feature task to")
  Rel(cadence, projectDocs, "Reads and writes")
  Rel(cadence, projectSrc, "Reads and writes")
  Rel(projectDocs, projectSrc, "Constrains")
```

## Key Decisions

- The plugin has no runtime server — it is a set of instruction files interpreted by Claude Code
- `docs/` is the authoritative source of truth; code must conform to documents and diagrams, not the reverse
- The plugin operates on the user's project directory, not on its own source

## Notes

- See `c4-containers.md` for the internal decomposition of the cadence plugin
- See `c4-seq-execution.md` for the end-to-end skill execution flow
