---
name: create-my-doc
description: Generate documentation for changed or specified code
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - Write
  - Edit
  - WebSearch
  - Agent(tech-writer)
---

# Generate Documentation

Generate appropriate documentation for code changes, including inline docs, CHANGELOG entries, and module documentation.

## Delegation Policy (MANDATORY)

This skill MUST delegate documentation drafting to the `tech-writer` subagent. The main session's only job is to:

- Identify changed files and the documentation plan (steps 1-2)
- Invoke the subagent via the Agent tool (step 3)
- Confirm drafts with the user before applying (step 4)

The main session MUST NOT draft docstrings, Javadoc, JSDoc, or CHANGELOG entries inline. If you find yourself writing `/** ... */` or `## [Unreleased]` content in the main session, STOP and dispatch the subagent.

## Instructions

1. **Identify changed files** (main session)
   - If the user specified files or a scope, use those
   - Otherwise, run `git diff --cached --name-only` to get staged files
   - If no staged files, run `git diff --name-only HEAD~1` to get the last commit's changes
   - If no changes found, ask the user what to document

2. **Determine documentation needs and confirm with user** (main session)
   Decide which documentation types apply:
   - **New or modified public APIs** → Inline documentation (Javadoc / JSDoc / docstring / rustdoc)
   - **New feature or significant change** → CHANGELOG entry
   - **New module or package** → Module-level documentation

   Present the plan to the user and confirm.

3. **REQUIRED — invoke `tech-writer` subagent**

   You MUST call the Agent tool with `subagent_type: tech-writer`. Do NOT draft docs yourself.

   Build the subagent prompt using this template:

   ```
   You are dispatched by the create-my-doc skill to draft documentation.

   Repository root: {pwd}
   Changed files: {file_list}
   Doc types requested: {inline|changelog|module} (per user-confirmed plan)

   Tasks:

   1) Inline documentation — for each changed file with undocumented public APIs:
      - Java: Javadoc with @param / @return / @throws
      - TypeScript / JavaScript: JSDoc with @param / @returns / @throws
      - Python: Google-style or NumPy-style docstrings (match existing convention)
      - Go: Godoc-style comments
      - Rust: /// doc comments with examples
      Include parameter descriptions, return values, exceptions, and usage examples where appropriate.
      Match the project's existing style — read 2-3 existing documented files to confirm.

   2) CHANGELOG entry (if requested):
      - Detect existing CHANGELOG file (CHANGELOG.md, CHANGES.md, HISTORY.md)
      - If none exists, return a recommendation to the main session instead of creating one
      - Use Keep-a-Changelog categories: Added / Changed / Fixed / Removed / Security
      - Place under the appropriate version heading (e.g., ## [Unreleased])

   Return drafts WITHOUT writing to disk yet:
   - For each inline doc: file path + the exact insertion (existing surrounding lines + new doc block)
   - For CHANGELOG: target file, target heading, exact entry text
   ```

4. **Confirm drafts and apply** (main session)
   Show the drafts returned by the subagent to the user. After confirmation, re-invoke `tech-writer` with instruction "User approved — apply the edits now and return the list of files written," or have the subagent itself perform the writes in a single round if the prompt permits.

   The main session MUST NOT modify the files itself.
