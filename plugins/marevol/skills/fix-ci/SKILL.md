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

Diagnose and fix CI failures from GitHub Actions. The main session acts strictly as an **orchestrator**: it gathers logs, dispatches subagents, and reports results. The actual analysis and fix work MUST be performed by subagents.

## Subagent Delegation (MANDATORY)

This skill REQUIRES subagent delegation. The main session MUST NOT perform the following work inline:

- **Log analysis / root-cause investigation** — MUST dispatch `code-investigator` via the `Agent` tool (step 3).
- **Applying the fix** — MUST dispatch `devops-engineer` via the `Agent` tool (step 4).

Rules:
- Do NOT skip these dispatches even if the failure looks "obvious" or "small". Always delegate.
- Do NOT silently fall back to inline analysis if a dispatch fails. Surface the error to the user and ask how to proceed.
- The main session may only run `gh`, `git`, and minimal verification commands directly. Reading source files for the purpose of diagnosis or editing files to apply a fix is the subagent's job.

## Instructions

1. **Fetch the latest failed run** (main session)
   Run `gh run list --status failure --limit 1 --json databaseId,name,headBranch,conclusion,event,createdAt` to find the most recent failed run.
   - Extract the run ID from the result
   - If no failed runs are found, inform the user and stop
   - If the user specified a particular run ID or branch, use that instead

2. **Get failure logs** (main session)
   Run `gh run view <run-id> --log-failed` to retrieve the logs for the failed steps.
   - If the output is very large, focus on the last 200 lines per failed job
   - Note which job(s) and step(s) failed
   - Identify the workflow file path (e.g., `.github/workflows/<name>.yml`)

3. **Analyze the failure — MUST dispatch `code-investigator`**
   Invoke the `Agent` tool with `subagent_type: code-investigator`. Do NOT analyze the logs yourself in the main session.

   Provide a self-contained prompt that includes:
   - The failed run ID, job name(s), and step name(s)
   - The relevant log excerpts (last 200 lines per failed job)
   - The workflow file path and any related config files
   - The repository's primary language(s) / build system if known

   Instruct the subagent to:
   - Identify the root cause across these categories: build errors (compilation/type/syntax), test failures (assertions, timeouts, flakiness), lint/format issues (ESLint, Prettier, Ruff, Black), dependency problems (missing packages, version conflicts, lockfile drift), environment issues (missing env vars, runtime versions, Docker build failures)
   - Trace which source files / configs are implicated and report file paths and line numbers
   - Return a structured summary: `root_cause`, `affected_files`, `suggested_fix_direction`

   Present the returned root-cause summary to the user before proceeding.

4. **Apply the fix — MUST dispatch `devops-engineer`**
   Invoke the `Agent` tool with `subagent_type: devops-engineer`. Do NOT edit files yourself in the main session.

   Provide a self-contained prompt that includes:
   - The full investigation summary returned in step 3
   - The workflow file path and the failed job/step
   - Any project-specific constraints the user mentioned

   Instruct the subagent to:
   - For dependency-related failures, check Dockerfiles, `requirements.txt`, `package.json`, lockfiles, and version pinning
   - For test failures, read the failing test and the code under test to understand the mismatch
   - For lint/format issues, run the formatter/linter with auto-fix if available (e.g., `npx eslint --fix`, `ruff check --fix`)
   - If multiple failures exist, address them one at a time in order of dependency
   - If the fix is unclear or risky, return analysis + options to the main session for user confirmation BEFORE making changes
   - Apply edits and return a summary of changed files

5. **Verify locally** (main session)
   Run the same checks that failed in CI to confirm the fix works:
   - Re-run the specific test suite, build command, or lint check that failed
   - If the local environment cannot replicate the CI step exactly, note any differences
   - If verification fails, loop back to step 3 with the new evidence (re-dispatch `code-investigator`)

6. **Commit the fix** (main session)
   Stage the changed files and commit with the format:
   ```
   fix: <concise description of CI fix>
   ```

7. **Push to the current branch** (main session)
   Push the commit to the remote so CI re-runs on the fix.
