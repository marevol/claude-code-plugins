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

Generate appropriate documentation for code changes, including inline docs, CHANGELOG entries, and module documentation. Documentation drafting is delegated to the `tech-writer` subagent so style conventions (Javadoc, JSDoc, docstrings, godoc, rustdoc) are applied consistently.

## Instructions

1. **Identify changed files**
   - If the user specified files or a scope, use those
   - Otherwise, run `git diff --cached --name-only` to get staged files
   - If no staged files, run `git diff --name-only HEAD~1` to get the last commit's changes
   - If no changes found, ask the user what to document

2. **Determine documentation needs**
   Based on the type of changes, determine which documentation to generate:
   - **New or modified public APIs** → Inline documentation (Javadoc / JSDoc / docstring / rustdoc)
   - **New feature or significant change** → CHANGELOG entry
   - **New module or package** → Module-level documentation
   - Present the plan to the user and confirm which types of documentation to generate

3. **Delegate generation to `tech-writer`**
   Dispatch the `tech-writer` subagent with the changed file list and the confirmed documentation plan from step 2. Instruct it to perform steps 4-5 (inline docs, CHANGELOG entry) following the project's existing style and to return drafts for user confirmation before applying.

   The remaining steps describe what the subagent must cover.

4. **Generate inline documentation**
   - Scan changed files for undocumented public methods, classes, functions, and interfaces
   - Generate documentation following the project's existing style:
     - For Java: Javadoc with `@param`, `@return`, `@throws`
     - For TypeScript/JavaScript: JSDoc with `@param`, `@returns`, `@throws`
     - For Python: Google-style or NumPy-style docstrings (match existing convention)
     - For Go: Godoc-style comments
     - For Rust: `///` doc comments with examples
   - Include parameter descriptions, return values, exceptions, and usage examples where appropriate

5. **Generate CHANGELOG entry**
   - Detect existing CHANGELOG file (`CHANGELOG.md`, `CHANGES.md`, `HISTORY.md`)
   - If no CHANGELOG exists, ask the user if one should be created
   - Generate an entry using conventional commit categories:
     - `### Added` — new features
     - `### Changed` — changes to existing functionality
     - `### Fixed` — bug fixes
     - `### Removed` — removed features
     - `### Security` — security-related changes
   - Place the entry under the appropriate version heading (e.g., `## [Unreleased]`)

6. **Present and apply**
   - Show all generated documentation to the user before applying
   - Apply changes only after user confirmation
   - For inline docs, use Edit to insert documentation into existing files
   - For CHANGELOG, append or insert the entry at the correct position
