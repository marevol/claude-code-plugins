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
  - Agent(solution-architect)
---

# Generate Architecture Decision Record

Create a new ADR following the project's existing conventions or standard ADR format.

## Delegation Policy (MANDATORY)

This skill MUST delegate the ADR drafting to the `solution-architect` subagent. The main session's only job is to:

- Detect the ADR location and numbering convention (steps 1-2)
- Gather decision inputs from the user (step 3)
- Invoke the subagent via the Agent tool (step 4)
- Write the returned Markdown to disk (step 5)

The main session MUST NOT draft the Context / Decision / Consequences / Alternatives sections itself. If you start writing prose for those sections, STOP and dispatch the subagent.

## Instructions

1. **Detect ADR directory and conventions** (main session)
   - Search for existing ADR directories: `docs/adr/`, `docs/decisions/`, `doc/adr/`, `adr/`, `docs/architecture/decisions/`
   - If found, analyze existing ADRs for naming convention (e.g., `0001-use-react.md`, `ADR-001-use-react.md`)
   - If no ADR directory exists, ask the user for the preferred location (default: `docs/adr/`) and create it

2. **Determine next ADR number** (main session)
   - List existing ADR files and extract the highest number
   - Increment by 1 for the new ADR
   - If no existing ADRs, start at `0001`

3. **Gather decision details from the user** (main session)
   Ask the user for (if not already provided):
   - **Title**: a short noun phrase (e.g., "Use PostgreSQL for primary database")
   - **Context**: the issue motivating this decision
   - **Decision**: what is being proposed or decided
   - **Alternatives considered**: other options evaluated

4. **REQUIRED — invoke `solution-architect` subagent**

   You MUST call the Agent tool with `subagent_type: solution-architect`. Do NOT draft the ADR yourself.

   Build the subagent prompt using this template:

   ```
   You are dispatched by the create-my-adr skill to draft an ADR.

   Repository root: {pwd}
   ADR directory: {adr_dir}
   Naming convention: {naming_pattern_or_default}
   Target file path: {full_target_path}
   ADR number: {number}
   Date (YYYY-MM-DD): {today}

   User-supplied inputs:
   - Title: {title}
   - Context: {context}
   - Decision: {decision}
   - Alternatives considered: {alternatives}

   Produce the full ADR Markdown body using this template, fleshing out each
   section with substantive trade-off analysis grounded in the user's inputs:

   # {number}. {title}

   Date: {date}

   ## Status

   Proposed

   ## Context

   <expand the user-supplied context: forces at play — technical, organizational, project-specific>

   ## Decision

   <state the decision in full sentences, active voice, with concrete specifics>

   ## Consequences

   <what becomes easier, what becomes harder, what trade-offs are accepted>

   ## Alternatives Considered

   <each alternative with one paragraph explaining why it was not chosen>

   Return ONLY the final Markdown body of the ADR. The main session will write
   it to {full_target_path}.
   ```

5. **Write ADR file** (main session)
   - Write the Markdown returned by the subagent to the target file path
   - Display the file path and a one-line summary of the ADR
