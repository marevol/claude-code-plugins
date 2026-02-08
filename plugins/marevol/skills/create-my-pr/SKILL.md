---
name: create-my-pr
description: Create a new branch from staged files and submit a pull request
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# Create Branch and Pull Request from Staged Files

Create a new branch from staged files, commit them, and submit a pull request.

## Instructions

1. **Check for staged files**
   - Run `git diff --cached --name-only` to get the list of staged files
   - If there are no staged files, inform the user and stop immediately
   - Run `git diff --cached` to understand the actual changes

2. **Detect default branch and current branch state**
   - Detect the default branch with `git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@'`
   - If the command fails, fall back to `main` (or ask the user)
   - Check the current branch name with `git branch --show-current`
   - If already on a feature branch (not the default branch), ask the user whether to:
     - Use the current branch as-is (skip branch creation)
     - Create a new branch from the current position

3. **Analyze changes and generate branch name** (if creating a new branch)
   - Determine the change type: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `style`, `perf`, `ci`, `build`
   - Generate a branch name using the type directly as prefix: `{type}/{short-description}` (e.g., `feat/user-authentication`, `fix/payment-validation`)
   - Create and switch to the new branch

4. **Commit staged files**
   - Commit only the staged files (do NOT add any new files with `git add`)
   - Use conventional commit format: `type(scope): description`

5. **Generate pull request content**
   - **Title**: Use a conventional, descriptive title. If the user provided one, use it.
   - **Description** in the following format:

     ```
     ## Summary
     Brief overview of changes

     ## Changes Made
     - List specific modifications
     - Include technical details

     ## Testing
     - Testing approach (if applicable)

     ## Breaking Changes
     - List breaking changes (if applicable)

     ## Additional Notes
     - Considerations for reviewers
     ```

   - Write in English
   - Write as a human developer (natural tone, not robotic)

6. **Push and create pull request**
   - Push the branch to remote with `git push -u origin {branch-name}`
   - Create a pull request using `gh pr create` targeting the detected default branch (or the user-specified target branch)
   - If the user requested a draft PR, use `--draft` flag
   - Display the resulting PR URL
