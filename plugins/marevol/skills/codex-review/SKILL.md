---
name: codex-review
description: Review gate using Codex CLI (read-only) to iterate review and fix until convergence
user-invocable: true
argument-hint: "[diff_range]"
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - Write
  - Edit
---

# Codex Review Gate

Iterative review gate: Codex audits (read-only), Claude Code fixes, re-review until `ok: true`.

## When to Use

- After creating/updating specifications, SPEC, PRD, requirements, or design documents
- After creating/updating implementation plans (e.g., PLANS.md)
- After completing major steps (>=5 files, new modules, public APIs, infra/config changes)
- Before git commit, PR, merge, or release

## Instructions

1. **Assess change size**
   - Use `$ARGUMENTS` as the diff range; default to `HEAD` if omitted
   - Run `git diff <diff_range> --stat` and `git diff <diff_range> --name-status --find-renames`
   - Classify: **small** (<=3 files, <=100 lines), **medium** (4-10 files, 100-500 lines), **large** (>10 files, >500 lines)

2. **Run architecture review** (medium and large only)
   - Build the following prompt, replacing `{placeholders}` with actual values, and append the output schema from the "Output Schema" section below:
   ```
   Review the architectural consistency of the following changes.
   Output exactly one JSON object using the schema appended below.

   This is a review gate. Set ok: false if any blocking issues exist.
   Issues will be fixed and re-reviewed until convergence.

   diff_range: {diff_range}
   Focus: Dependencies, separation of concerns, breaking changes, security design
   Previous notes: {notes_for_next_review}

   [APPEND OUTPUT SCHEMA]
   ```
   - Execute `codex exec --sandbox read-only "<PROMPT>"` and wait for completion

3. **Run diff review**
   - Build the following prompt, replacing `{placeholders}` with actual values, and append the output schema:
   ```
   Review the following changes.
   Output exactly one JSON object using the schema appended below.

   This is a review gate. Set ok: false if any blocking issues exist.
   Issues will be fixed and re-reviewed until convergence.

   diff_range: {diff_range}
   Target files: {target_files}
   Focus: {review_focus}
   Previous notes: {notes_for_next_review}

   [APPEND OUTPUT SCHEMA]
   ```
   - For **large** changes: split into groups of max 5 files / 300 lines each, run 3-5 sub-agents in parallel
   - Wait for all executions to complete

4. **Run cross-check review** (large only)
   - Build the following prompt with aggregated group results, and append the output schema:
   ```
   Integrate parallel review results and perform cross-cutting review.
   Output exactly one JSON object using the schema appended below.

   This is a review gate. Set ok: false for any cross-cutting blocking issues
   (e.g., interface mismatches, authorization gaps, API compatibility breaks).

   Overall stats: {stat_output}
   Group results: {group_jsons}
   Focus: Interface consistency, error handling patterns, authorization, API compatibility, test coverage

   [APPEND OUTPUT SCHEMA]
   ```

5. **Fix blocking issues** (fix loop, max 5 iterations)
   - If any review returns `ok: false`:
     1. Parse the `issues` array and create a fix plan
     2. Apply minimal fixes (defer spec changes to unresolved issues)
     3. Run tests/linters if available
     4. Re-review with updated diff, including `notes_for_next_review`
   - Stop when `ok: true`, max iterations reached, or tests fail twice consecutively

6. **Handle errors**
   - If `codex exec` fails: retry once (for timeout, halve the file count)
   - On second failure: skip the phase and record reason in report
   - If arch skipped: continue with diff only
   - If diff skipped: mark affected files as "unreviewed" in report

7. **Generate final report**
   Output a structured report:
   ```
   ## Codex Review Results
   - Size: {size} ({file_count} files, {line_count} lines)
   - Iterations: {n}/{max_iters} | Status: ok / not ok

   ### Fix History
   - {file}: {description of fix}

   ### Advisory (Reference)
   - {file}: {advisory issue}

   ### Unreviewed (Errors Only)
   - {file}: {reason}, manual review recommended

   ### Unresolved (If Any)
   - {file}: {issue}, risk assessment, recommended action
   ```

## Output Schema

Instruct Codex to output exactly one JSON object. Append this schema to the end of every review prompt where `[APPEND OUTPUT SCHEMA]` appears.

```json
{
  "ok": true,
  "phase": "arch|diff|cross-check",
  "summary": "Brief review summary",
  "issues": [
    {
      "severity": "blocking|advisory",
      "category": "correctness|security|perf|maintainability|testing|style",
      "file": "src/example.py",
      "lines": "42-45",
      "problem": "Description of the issue",
      "recommendation": "Suggested fix"
    }
  ],
  "notes_for_next_review": "Notes for subsequent reviews"
}
```

| Field | Description |
|-------|-------------|
| `ok` | `true` if zero blocking issues, `false` if one or more |
| `severity` | `blocking` = must fix (triggers `ok: false`), `advisory` = recommendation only |
| `category` | Issue classification for prioritization |
| `notes_for_next_review` | Context to include in the next review prompt |
