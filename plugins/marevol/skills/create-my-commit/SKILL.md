---
name: create-my-commit
description: Commit staged files to the current branch
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# Commit Staged Files

Commit only the currently staged files to the current branch using conventional commit format.

## Instructions

1. **Check for staged files**
   - Run `git diff --cached --name-only` to get the list of staged files
   - If there are no staged files, inform the user and stop immediately
   - Run `git diff --cached` to understand the actual changes

2. **Analyze commit style and changes**
   - Run `git log --oneline -10` to review the project's recent commit message style and tone
   - Determine the change type: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `style`, `perf`, `ci`, `build`
   - Identify the scope from the changed files (e.g., module name, component name)
   - Check if the staged changes contain logically distinct groups (e.g., a refactor + a bug fix, changes to unrelated modules)

3. **Evaluate split commit**
   - If the staged changes can be logically separated into independent commits, suggest splitting them to the user
   - Explain how the changes could be grouped (e.g., "The refactor of module A and the bug fix in module B could be separate commits")
   - If the user agrees, guide them through `git reset HEAD <files>` and committing in stages
   - If the user prefers a single commit, proceed as normal

4. **Generate commit message**
   - If the user provided a message, use it as-is
   - Otherwise, auto-generate a commit message based on the staged changes
   - Use conventional commit format: `type(scope): description`
   - Match the tone and style observed in the project's recent commit history
   - The description should be concise and written in English
   - Add a body with more details if the changes are complex

5. **Commit staged files**
   - Commit only the staged files (do NOT add any new files with `git add`)
   - Display the committed files and the resulting commit hash
