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
---

# Fix CI

Diagnose and fix CI failures from GitHub Actions.

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

3. **Analyze the failure**
   Identify the root cause from the logs. Common categories:
   - **Build errors**: compilation failures, type errors, syntax errors
   - **Test failures**: failing assertions, timeouts, flaky tests
   - **Lint/format issues**: ESLint, Prettier, Ruff, Black violations
   - **Dependency problems**: missing packages, version conflicts, lockfile drift
   - **Environment issues**: missing env vars, wrong runtime versions, Docker build failures
   - Present a brief summary of the root cause before proceeding

4. **Apply the fix**
   Edit the relevant files to resolve the failure.
   - For dependency-related failures, check Dockerfiles, `requirements.txt`, `package.json`, lockfiles, and version pinning
   - For test failures, read the failing test and the code under test to understand the mismatch
   - For lint/format issues, run the formatter/linter with auto-fix if available (e.g., `npx eslint --fix`, `ruff check --fix`)
   - If multiple failures exist, address them one at a time in order of dependency
   - If the fix is unclear or risky, present your analysis and proposed options to the user before making changes

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
