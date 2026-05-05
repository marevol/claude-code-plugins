# Claude Code Plugin

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that provides specialized agents and development workflow skills for software engineering teams.

## Features

### Agents

Specialized agents that Claude Code can delegate tasks to based on context:

Agents are model-tuned for their cognitive load: deep-reasoning roles use **Opus**, code/document generation roles use **Sonnet**.

| Agent | Model | Description |
|---|---|---|
| **code-investigator** | sonnet | Map architecture, call graphs, and conventions in existing codebases (investigation gate) |
| **solution-architect** | opus | C4 architecture diagrams, ADRs, and technology comparison matrices |
| **backend-engineer** | sonnet | Production-quality server-side code with Spring Boot, FastAPI, Go |
| **frontend-engineer** | sonnet | Accessible, performant UI with React/Next.js, Vue/Nuxt, TypeScript |
| **code-reviewer** | opus | Structured code audit reports with severity-classified findings (quality gate) |
| **test-engineer** | sonnet | Comprehensive test suites with coverage analysis (test coverage gate) |
| **security-engineer** | opus | STRIDE threat modeling and OWASP vulnerability assessment (security gate) |
| **performance-engineer** | opus | Bottleneck identification and optimization reports (performance gate) |
| **devops-engineer** | sonnet | Terraform modules, Dockerfiles, GitHub Actions, and monitoring configs |
| **ml-engineer** | sonnet | ML pipelines, model serving, RAG systems, and LLM integrations |
| **ai-researcher** | opus | AI/ML technology evaluation and RAG/agent architecture design |
| **tech-writer** | sonnet | README, API docs, ADRs, and developer guides (documentation gate) |
| **tech-translator** | sonnet | Translate technical documents between languages preserving accuracy and formatting |

### Skills

Slash commands for common development workflows:

| Command | Description |
|---|---|
| `/create-my-commit` | Commit staged files with conventional commit messages |
| `/create-my-pr` | Create a branch from staged files and submit a pull request |
| `/check-my-deps` | Check dependency health, outdated packages, and security advisories |
| `/security-scan` | Scan code for security vulnerabilities and hardcoded secrets |
| `/codex-review` | Review gate using Codex CLI (read-only) to iterate review and fix until convergence |
| `/gemini-review` | Review gate using Gemini CLI (read-only) to iterate review and fix until convergence |
| `/fix-ci` | Diagnose and fix GitHub Actions CI failures by fetching logs and applying fixes |
| `/java-source-code` | Fetch and display Java source for third-party dependency classes not in the project |

## Installation

1. Add the marketplace:

```bash
/plugin marketplace add marevol/claude-code-plugins
```

2. Install the plugin:

```bash
/plugin install marevol@marevol-plugins
```

## License

Apache License 2.0
