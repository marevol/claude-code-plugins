---
name: create-my-adr
description: Generate an Architecture Decision Record (ADR)
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - Write
---

# Generate Architecture Decision Record

Create a new ADR following the project's existing conventions or standard ADR format.

## Instructions

1. **Detect ADR directory and conventions**
   - Search for existing ADR directories: `docs/adr/`, `docs/decisions/`, `doc/adr/`, `adr/`, `docs/architecture/decisions/`
   - If found, analyze existing ADRs for naming convention (e.g., `0001-use-react.md`, `ADR-001-use-react.md`)
   - If no ADR directory exists, ask the user for the preferred location (default: `docs/adr/`) and create it

2. **Determine next ADR number**
   - List existing ADR files and extract the highest number
   - Increment by 1 for the new ADR
   - If no existing ADRs, start at `0001`

3. **Gather decision details from the user**
   Ask the user for (if not already provided):
   - **Title**: A short noun phrase describing the decision (e.g., "Use PostgreSQL for primary database")
   - **Context**: What is the issue that motivates this decision?
   - **Decision**: What is the change being proposed or decided?
   - **Alternatives considered**: What other options were evaluated?

4. **Generate ADR document**
   Create the ADR with the following sections:

   ```markdown
   # {number}. {title}

   Date: {YYYY-MM-DD}

   ## Status

   Proposed

   ## Context

   {context — the forces at play, including technical, political, social, and project-specific}

   ## Decision

   {decision — the change being proposed or decided, stated in full sentences with active voice}

   ## Consequences

   {consequences — what becomes easier or harder as a result of this decision}

   ## Alternatives Considered

   {alternatives — other options that were evaluated and why they were not chosen}
   ```

   - Use the naming convention detected from existing ADRs
   - If no convention exists, use `{number}-{kebab-case-title}.md` (e.g., `0001-use-postgresql.md`)

5. **Write ADR file**
   - Write the file to the ADR directory
   - Display the file path and a summary of the ADR
