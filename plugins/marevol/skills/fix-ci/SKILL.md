---
name: fix-ci
description: Diagnose and fix CI failures by fetching failed run logs from GitHub Actions, analyzing root causes, applying fixes, verifying locally, and pushing. Use when CI is red, a GitHub Actions workflow has failed, or the user says "fix ci", "ci is broken", or "pipeline failed".
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - Write
  - Edit
  - Agent(code-investigator, devops-engineer)
---

# Fix CI

Diagnose and fix CI failures from GitHub Actions. Heavy log analysis is delegated to the `code-investigator` subagent and the actual fix is delegated to the `devops-engineer` subagent so the main session stays focused on orchestration.

## Instructions

1. **Fetch the latest failed run**
   Run `gh run list --status failure --limit 1 --json databaseId,name,headBranch,conclusion,event,createdAt` to find the most recent failed run.
   - Extract the run ID from the result
   - If no failed runs are found, inform the user and stop
   - If the user specified a particular run ID or branch, use that instead

2. **Get failure logs**
   Run `gh run view <run-id> --log-failed` to retrieve the logs for the failed steps.
   - If the output is very large, focus on the last 200 lines per failed job
   - Note which job(s) and step(s) failed

3. **Analyze the failure (delegate to `code-investigator`)**
   Dispatch the `code-investigator` subagent with a self-contained prompt containing the failed logs and the workflow file path. Ask it to:
   - Identify the root cause across these categories: build errors (compilation/type/syntax), test failures (assertions, timeouts, flakiness), lint/format issues (ESLint, Prettier, Ruff, Black), dependency problems (missing packages, version conflicts, lockfile drift), environment issues (missing env vars, runtime versions, Docker build failures)
   - Trace which source files / configs are implicated and report file paths and line numbers
   - Return a structured summary: root cause, affected files, and suggested fix direction
   - Present the root-cause summary to the user before proceeding

4. **Apply the fix (delegate to `devops-engineer`)**
   Dispatch the `devops-engineer` subagent with the investigation summary from step 3 plus the workflow context. Instruct it to:
   - For dependency-related failures, check Dockerfiles, `requirements.txt`, `package.json`, lockfiles, and version pinning
   - For test failures, read the failing test and the code under test to understand the mismatch
   - For lint/format issues, run the formatter/linter with auto-fix if available (e.g., `npx eslint --fix`, `ruff check --fix`)
   - If multiple failures exist, address them one at a time in order of dependency
   - If the fix is unclear or risky, return analysis + options to the main session for user confirmation before making changes
   - Apply edits and return a summary of changed files

5. **Verify locally**
   Run the same checks that failed in CI to confirm the fix works:
   - Re-run the specific test suite, build command, or lint check that failed
   - If the local environment cannot replicate the CI step exactly, note any differences

6. **Commit the fix**
   Stage the changed files and commit with the format:
   ```
   fix: <concise description of CI fix>
   ```

7. **Push to the current branch**
   Push the commit to the remote so CI re-runs on the fix.
