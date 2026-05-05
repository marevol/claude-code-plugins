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

Scan the codebase for security vulnerabilities, hardcoded secrets, and dependency risks.

## Delegation Policy (MANDATORY)

This skill MUST delegate the actual scan to the `security-engineer` subagent. The main session's only job is to:

- Resolve the scan scope (step 1)
- Invoke the subagent via the Agent tool (step 2)
- Surface the subagent's report to the user

The main session MUST NOT grep / read source files for vulnerability patterns itself. If you start writing your own OWASP checks or secret-detection regexes in the main session, STOP and dispatch the subagent.

## Instructions

1. **Determine scan scope** (main session)
   - If the user specified a directory or files, use those
   - If there are staged changes, scan `git diff --cached --name-only`
   - If on a feature branch, scan files changed vs the default branch with `git diff --name-only $(git merge-base HEAD origin/main)..HEAD`
   - Otherwise, ask the user for the scope

2. **REQUIRED — invoke `security-engineer` subagent**

   You MUST call the Agent tool with `subagent_type: security-engineer`. Do NOT perform vulnerability checks, secret detection, or dependency audits yourself.

   Build the subagent prompt using this template:

   ```
   You are dispatched by the security-scan skill to perform a full security scan.

   Repository root: {pwd}
   Scan scope (file list): {file_list}

   Cover these areas and return findings classified by severity (CRITICAL / HIGH / MEDIUM / LOW):

   1) OWASP Top 10 patterns:
      - Injection (SQL string concatenation, unsanitized shell, template injection)
      - Broken Authentication (hardcoded credentials, weak password policy, missing rate limit)
      - Sensitive Data Exposure (logging of sensitive data, unencrypted storage, missing HTTPS enforcement)
      - XXE (unsafe XML parser config)
      - Broken Access Control (missing authz, IDOR, directory traversal)
      - Security Misconfiguration (debug mode, default creds, permissive CORS)
      - XSS (unescaped user input, unsafe DOM APIs)
      - Insecure Deserialization
      - Known Vulnerabilities (outdated libs with CVEs)
      - Insufficient Logging (missing audit trails)

   2) Hardcoded secrets:
      - API keys / tokens / passwords
      - AWS access keys (AKIA...), private keys (-----BEGIN ... PRIVATE KEY-----)
      - Connection strings with embedded credentials
      - Tracked .env / *.pem / *.key files; verify .gitignore coverage

   3) Dependency security:
      - Detect manifests (package.json, pom.xml, requirements.txt, go.mod, Cargo.toml, etc.)
      - Run audit tools if installed (npm audit, pip-audit, cargo audit, govulncheck)
      - Flag known vulnerable dependencies

   Output the report exactly in this format:

   ## Security Scan Report

   ### CRITICAL
   - <file:line> — <description> — <remediation>

   ### HIGH
   - ...

   ### MEDIUM
   - ...

   ### LOW
   - ...

   ### Summary
   - Total findings: N
   - Files scanned: N
   - Recommendations: <prioritized action items>

   If no issues are found, report a clean scan result.
   ```

   Surface the subagent's report verbatim to the user.
