---
name: check-my-deps
description: Check dependency health, outdated packages, and security advisories
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# Dependency Health Check

Analyze project dependencies for outdated packages, security vulnerabilities, and upgrade risks.

## Instructions

1. **Detect package managers**
   Scan the project root for dependency manifest files:
   - **npm/yarn/pnpm**: `package.json`, `yarn.lock`, `pnpm-lock.yaml`
   - **Maven**: `pom.xml`
   - **Gradle**: `build.gradle`, `build.gradle.kts`
   - **pip/poetry**: `requirements.txt`, `pyproject.toml`, `Pipfile`
   - **Go modules**: `go.mod`
   - **Cargo**: `Cargo.toml`
   - **Bundler**: `Gemfile`
   - If no dependency files found, inform the user and stop

2. **Check for outdated dependencies**
   Run the appropriate outdated check for each detected package manager:
   - npm: `npm outdated --json`
   - yarn: `yarn outdated --json`
   - pip: `pip list --outdated --format=json`
   - Maven: `mvn versions:display-dependency-updates` (if mvn available)
   - Gradle: `gradle dependencyUpdates` (if plugin available)
   - Go: `go list -u -m all`
   - Cargo: `cargo outdated` (if installed)
   - Classify updates as: **major** (breaking), **minor** (feature), **patch** (fix)

3. **Run security advisory checks**
   Run available security audit tools:
   - npm: `npm audit --json`
   - yarn: `yarn audit --json`
   - pip: `pip-audit --format=json` (if installed)
   - Go: `govulncheck ./...` (if installed)
   - Cargo: `cargo audit` (if installed)
   - If audit tools are not installed, note this in the report

4. **Analyze major upgrade risks**
   For dependencies with major version updates available:
   - Identify how many major versions behind the project is
   - Note dependencies that are unmaintained or deprecated
   - Flag dependencies with known breaking changes in newer versions

5. **Generate structured report**
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
