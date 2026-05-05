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

Analyze project dependencies for outdated packages, security vulnerabilities, and upgrade risks.

## Delegation Policy (MANDATORY)

This skill MUST delegate the audit to the `devops-engineer` subagent. The main session's only job is to:

- Detect package managers in the repo (step 1)
- Invoke the subagent via the Agent tool (step 2)
- Surface the report to the user

The main session MUST NOT run `npm outdated`, `npm audit`, `pip-audit`, `cargo audit`, `govulncheck`, etc. itself or interpret their output. If you start parsing audit output in the main session, STOP and dispatch the subagent.

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

   If no dependency files found, inform the user and stop.

2. **REQUIRED — invoke `devops-engineer` subagent**

   You MUST call the Agent tool with `subagent_type: devops-engineer`. Do NOT run dependency tooling or analyze versions yourself.

   Build the subagent prompt using this template:

   ```
   You are dispatched by the check-my-deps skill to perform a full dependency health audit.

   Repository root: {pwd}
   Detected package managers: {list_with_manifest_paths}

   Tasks:

   1) Outdated check — run the appropriate command per package manager:
      - npm: npm outdated --json
      - yarn: yarn outdated --json
      - pip: pip list --outdated --format=json
      - Maven: mvn versions:display-dependency-updates (if available)
      - Gradle: gradle dependencyUpdates (if plugin available)
      - Go: go list -u -m all
      - Cargo: cargo outdated (if installed)
      Classify updates as major / minor / patch.

   2) Security audit — run available tools:
      - npm: npm audit --json
      - yarn: yarn audit --json
      - pip: pip-audit --format=json (if installed)
      - Go: govulncheck ./... (if installed)
      - Cargo: cargo audit (if installed)
      Note tools not installed in the report (do NOT install them).

   3) Major upgrade risk analysis:
      - How many major versions behind for each
      - Unmaintained / deprecated packages
      - Known breaking changes in newer versions

   Output the report exactly in this format:

   ## Dependency Health Report

   ### Security Vulnerabilities
   - <package@version>: <description> → upgrade to <fixed version>

   ### Major Version Updates Available
   - <package>: <current> → <latest> (N major versions behind)
     - Breaking changes: <brief summary if known>

   ### Minor/Patch Updates Available
   - <package>: <current> → <latest>

   ### Up to Date
   - N dependencies are current

   ### Summary
   - Package managers detected: [list]
   - Total dependencies: N
   - Security issues: N (critical: N, high: N, moderate: N, low: N)
   - Outdated: N major, N minor, N patch
   - Recommended priority actions: <ordered list>
   ```

   Surface the subagent's report verbatim to the user.
