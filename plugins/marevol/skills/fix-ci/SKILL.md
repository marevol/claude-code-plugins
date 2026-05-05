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

Diagnose and fix CI failures from GitHub Actions.

## Delegation Policy (MANDATORY)

This skill MUST delegate the heavy work to specialized subagents using the Agent tool. The main session is only allowed to:

- Run `gh` commands to fetch run metadata and logs (steps 1-2)
- Invoke subagents via the Agent tool (steps 3-4)
- Run local verification commands (step 5)
- Run `git` commit/push commands (steps 6-7)

The main session MUST NOT analyze logs in detail or edit source files inline. If you find yourself reading source files or planning fixes in the main session, STOP and dispatch the appropriate subagent instead.

## Instructions

1. **Fetch the latest failed run**
   Run `gh run list --status failure --limit 1 --json databaseId,name,headBranch,conclusion,event,createdAt` to find the most recent failed run.
   - Extract the run ID from the result
   - If no failed runs are found, inform the user and stop
   - If the user specified a particular run ID or branch, use that instead

2. **Get failure logs**
   Run `gh run view <run-id> --log-failed` to retrieve the logs for the failed steps.
   - If the output is very large, keep the last 200 lines per failed job
   - Note which job(s) and step(s) failed and the workflow file path

3. **REQUIRED — invoke `code-investigator` subagent for root-cause analysis**

   You MUST call the Agent tool with `subagent_type: code-investigator`. Do NOT analyze the logs or read source files yourself in the main session.

   Build the subagent prompt using this template (fill the placeholders):

   ```
   You are dispatched by the fix-ci skill to analyze a GitHub Actions CI failure.

   Run ID: {run_id}
   Workflow file: {workflow_path}
   Repository root: {pwd}

   Failed logs:
   <<<
   {log_excerpt}
   >>>

   Identify the root cause across these categories:
   - Build errors: compilation, type, syntax
   - Test failures: assertions, timeouts, flakiness
   - Lint/format: ESLint, Prettier, Ruff, Black
   - Dependency: missing packages, version conflicts, lockfile drift
   - Environment: missing env vars, runtime versions, Docker build

   Read implicated source / config files in the repo to confirm the cause.

   Return a structured report:
   - Root cause: <one or two sentences>
   - Affected files: <path:line list>
   - Suggested fix direction: <high-level approach, no code edits>
   ```

   After the subagent returns, show the root-cause summary to the user before continuing.

4. **REQUIRED — invoke `devops-engineer` subagent to apply the fix**

   You MUST call the Agent tool with `subagent_type: devops-engineer`. Do NOT use Edit / Write yourself in the main session for the fix; the subagent owns the edits.

   Build the subagent prompt using this template:

   ```
   You are dispatched by the fix-ci skill to apply a fix for a CI failure.

   Investigation summary (from code-investigator):
   <<<
   {summary_from_step_3}
   >>>

   Workflow file: {workflow_path}
   Repository root: {pwd}

   Apply the minimal fix following these rules:
   - Dependency failures → Dockerfiles, requirements.txt, package.json, lockfiles, version pinning
   - Test failures → read failing test and code under test, fix the real mismatch
   - Lint/format → run linter/formatter with auto-fix (e.g. `npx eslint --fix`, `ruff check --fix`) where possible
   - Multiple failures → fix in dependency order, smallest blast radius first
   - If the fix is unclear or risky, return analysis + options instead of editing files

   Return:
   - Changed files: <path list>
   - Summary of edits: <bullet list>
   - Or, if not applied: options for the main session to surface to the user
   ```

   If the subagent returns options instead of edits, present them to the user, get a decision, and re-invoke the subagent with the chosen option. Do NOT make the edits in the main session.

5. **Verify locally** (main session)
   Re-run the same checks that failed in CI on the local copy:
   - Re-run the specific test suite, build command, or lint check
   - If the local environment cannot replicate the CI step exactly, note any differences

6. **Commit the fix** (main session)
   Stage the files reported by the `devops-engineer` subagent and commit:
   ```
   fix: <concise description of CI fix>
   ```

7. **Push to the current branch** (main session)
   Push the commit to the remote so CI re-runs on the fix.
