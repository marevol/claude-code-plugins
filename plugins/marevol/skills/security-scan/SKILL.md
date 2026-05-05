---
name: security-scan
description: Scan code for security vulnerabilities and hardcoded secrets
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - Agent(security-engineer)
---

# Security Scan

Scan the codebase for security vulnerabilities, hardcoded secrets, and dependency risks. The main session acts strictly as an **orchestrator**: it resolves the scan scope, dispatches the subagent, and presents the result. All scanning, classification, and reporting MUST be performed by the subagent.

## Subagent Delegation (MANDATORY)

This skill REQUIRES subagent delegation. The main session MUST NOT perform the following work inline:

- **Vulnerability pattern scanning, secret detection, dependency audit, severity classification, and report generation** — MUST dispatch `security-engineer` via the `Agent` tool (step 2).

Rules:
- Do NOT skip the dispatch even if the scope is small (e.g., one file). Always delegate.
- Do NOT silently fall back to inline scanning if the dispatch fails. Surface the error to the user and ask how to proceed.
- The main session may only run scope-resolution commands (`git diff`, `git merge-base`, listing files). Reading file contents for scanning is the subagent's job.

## Instructions

1. **Determine scan scope** (main session)
   - If the user specified a directory or files, use those
   - If there are staged changes, scan `git diff --cached --name-only`
   - If on a feature branch, scan files changed vs the default branch with `git diff --name-only $(git merge-base HEAD origin/main)..HEAD`
   - Otherwise, ask the user for the scope

2. **Run the full scan — MUST dispatch `security-engineer`**
   Invoke the `Agent` tool with `subagent_type: security-engineer`. Do NOT scan files yourself in the main session.

   Provide a self-contained prompt that includes:
   - The resolved scan scope (explicit list of files or directories) from step 1
   - The repository's primary language(s) / framework(s) if known
   - Any user-provided focus areas (e.g., "focus on auth code")

   Instruct the subagent to perform steps 3-6 below and return the final structured report verbatim. The main session passes the report through to the user without re-running any scan.

3. **Check OWASP Top 10 vulnerability patterns** (subagent)
   Scan source files for common vulnerability patterns including:
   - **Injection**: SQL string concatenation, unsanitized shell commands, template injection
   - **Broken Authentication**: Hardcoded credentials, weak password policies, missing rate limiting
   - **Sensitive Data Exposure**: Logging of sensitive data, unencrypted storage, missing HTTPS enforcement
   - **XXE**: Unsafe XML parser configuration
   - **Broken Access Control**: Missing authorization checks, IDOR patterns, directory traversal
   - **Security Misconfiguration**: Debug mode enabled, default credentials, overly permissive CORS
   - **XSS**: Unescaped user input in HTML output, unsafe DOM manipulation APIs
   - **Insecure Deserialization**: Unsafe dynamic code execution, untrusted data parsing via unsafe serialization libraries
   - **Known Vulnerabilities**: Outdated library versions with known CVEs
   - **Insufficient Logging**: Missing audit trails for security-critical operations

4. **Detect hardcoded secrets** (subagent)
   Search for patterns matching:
   - API keys, tokens, passwords in source code (regex patterns for common formats)
   - AWS access keys (`AKIA...`), private keys (`-----BEGIN.*PRIVATE KEY-----`)
   - Connection strings with embedded credentials
   - `.env` files or similar that should be gitignored but are tracked
   - Verify `.gitignore` covers common secret files (`.env`, `*.pem`, `*.key`)

5. **Check dependency security** (subagent)
   - Identify dependency manifest files (`package.json`, `pom.xml`, `requirements.txt`, `go.mod`, `Cargo.toml`, etc.)
   - Run available audit tools if installed (`npm audit`, `pip-audit`, `cargo audit`)
   - Flag dependencies with known vulnerabilities

6. **Generate structured report** (subagent)
   Output findings organized by severity:

   ```
   ## Security Scan Report

   ### CRITICAL
   - [Finding with file:line and description]

   ### HIGH
   - [Finding with file:line and description]

   ### MEDIUM
   - [Finding with file:line and description]

   ### LOW
   - [Finding with file:line and description]

   ### Summary
   - Total findings: N
   - Files scanned: N
   - Recommendations: [prioritized action items]
   ```

   - If no issues found, report a clean scan result
   - For each finding, include the file path, line number, and a brief remediation suggestion
