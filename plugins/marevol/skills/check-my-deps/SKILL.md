---
name: check-my-deps
description: Check dependency health, outdated packages, and security advisories
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - Agent(devops-engineer)
---

# Dependency Health Check

Analyze project dependencies for outdated packages, security vulnerabilities, and upgrade risks. The main session acts strictly as an **orchestrator**: it detects manifest files, dispatches the subagent, and presents the result. All audit, risk analysis, and reporting MUST be performed by the subagent.

## Subagent Delegation (MANDATORY)

This skill REQUIRES subagent delegation. The main session MUST NOT perform the following work inline:

- **Outdated checks, security audits, major-upgrade risk analysis, and report generation** — MUST dispatch `devops-engineer` via the `Agent` tool (step 2).

Rules:
- Do NOT skip the dispatch even if only one package manager is detected. Always delegate.
- Do NOT silently fall back to running audit tools (`npm audit`, `pip-audit`, etc.) yourself if the dispatch fails. Surface the error to the user and ask how to proceed.
- The main session may only detect manifest files. Running audit tooling and analyzing results is the subagent's job.

## Instructions

1. **Detect package managers** (main session)
   Scan the project root for dependency manifest files:
   - **npm/yarn/pnpm**: `package.json`, `yarn.lock`, `pnpm-lock.yaml`
   - **Maven**: `pom.xml`
   - **Gradle**: `build.gradle`, `build.gradle.kts`
   - **pip/poetry**: `requirements.txt`, `pyproject.toml`, `Pipfile`
   - **Go modules**: `go.mod`
   - **Cargo**: `Cargo.toml`
   - **Bundler**: `Gemfile`
   - If no dependency files found, inform the user and stop

2. **Run the audit — MUST dispatch `devops-engineer`**
   Invoke the `Agent` tool with `subagent_type: devops-engineer`. Do NOT run audit tooling yourself in the main session.

   Provide a self-contained prompt that includes:
   - The list of detected package managers and their manifest paths from step 1
   - The repository root path
   - Any user-specified focus (e.g., "security only", "skip major upgrades")

   Instruct the subagent to perform steps 3-6 below and return the final structured report verbatim. The main session passes the report through to the user without re-running any audit.

3. **Check for outdated dependencies** (subagent)
   Run the appropriate outdated check for each detected package manager:
   - npm: `npm outdated --json`
   - yarn: `yarn outdated --json`
   - pip: `pip list --outdated --format=json`
   - Maven: `mvn versions:display-dependency-updates` (if mvn available)
   - Gradle: `gradle dependencyUpdates` (if plugin available)
   - Go: `go list -u -m all`
   - Cargo: `cargo outdated` (if installed)
   - Classify updates as: **major** (breaking), **minor** (feature), **patch** (fix)

4. **Run security advisory checks** (subagent)
   Run available security audit tools:
   - npm: `npm audit --json`
   - yarn: `yarn audit --json`
   - pip: `pip-audit --format=json` (if installed)
   - Go: `govulncheck ./...` (if installed)
   - Cargo: `cargo audit` (if installed)
   - If audit tools are not installed, note this in the report

5. **Analyze major upgrade risks** (subagent)
   For dependencies with major version updates available:
   - Identify how many major versions behind the project is
   - Note dependencies that are unmaintained or deprecated
   - Flag dependencies with known breaking changes in newer versions

6. **Generate structured report** (subagent)
   Output findings organized by urgency:

   ```
   ## Dependency Health Report

   ### Security Vulnerabilities
   - [package@version]: [vulnerability description] → upgrade to [fixed version]

   ### Major Version Updates Available
   - [package]: [current] → [latest] (N major versions behind)
     - Breaking changes: [brief summary if known]

   ### Minor/Patch Updates Available
   - [package]: [current] → [latest]

   ### Up to Date
   - N dependencies are current

   ### Summary
   - Package managers detected: [list]
   - Total dependencies: N
   - Security issues: N (critical: N, high: N, moderate: N, low: N)
   - Outdated: N major, N minor, N patch
   - Recommended priority actions: [ordered list]
   ```
