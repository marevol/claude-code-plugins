---
name: security-scan
description: Scan code for security vulnerabilities and hardcoded secrets
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# Security Scan

Scan the codebase for security vulnerabilities, hardcoded secrets, and dependency risks.

## Instructions

1. **Determine scan scope**
   - If the user specified a directory or files, use those
   - If there are staged changes, scan `git diff --cached --name-only`
   - If on a feature branch, scan files changed vs the default branch with `git diff --name-only $(git merge-base HEAD origin/main)..HEAD`
   - Otherwise, ask the user for the scope

2. **Check OWASP Top 10 vulnerability patterns**
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

3. **Detect hardcoded secrets**
   Search for patterns matching:
   - API keys, tokens, passwords in source code (regex patterns for common formats)
   - AWS access keys (`AKIA...`), private keys (`-----BEGIN.*PRIVATE KEY-----`)
   - Connection strings with embedded credentials
   - `.env` files or similar that should be gitignored but are tracked
   - Verify `.gitignore` covers common secret files (`.env`, `*.pem`, `*.key`)

4. **Check dependency security**
   - Identify dependency manifest files (`package.json`, `pom.xml`, `requirements.txt`, `go.mod`, `Cargo.toml`, etc.)
   - Run available audit tools if installed (`npm audit`, `pip-audit`, `cargo audit`)
   - Flag dependencies with known vulnerabilities

5. **Generate structured report**
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
