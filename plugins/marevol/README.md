# Claude Code Plugin

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that provides specialized agents and development workflow skills for software engineering teams.

## Features

### Agents

Specialized agents that Claude Code can delegate tasks to based on context:

| Agent | Description |
|---|---|
| **solution-architect** | C4 architecture diagrams, ADRs, and technology comparison matrices |
| **backend-engineer** | Production-quality server-side code with Spring Boot, FastAPI, Go |
| **frontend-engineer** | Accessible, performant UI with React/Next.js, Vue/Nuxt, TypeScript |
| **fullstack-engineer** | End-to-end features spanning API and UI with type-safe integration |
| **code-reviewer** | Structured code audit reports with severity-classified findings (quality gate) |
| **test-engineer** | Comprehensive test suites with coverage analysis (test coverage gate) |
| **tester** | Test execution, browser-based QA, and exploratory testing with bug reports |
| **security-engineer** | STRIDE threat modeling and OWASP vulnerability assessment (security gate) |
| **performance-engineer** | Bottleneck identification and optimization reports (performance gate) |
| **devops-engineer** | Terraform modules, Dockerfiles, GitHub Actions, and monitoring configs |
| **ml-engineer** | ML pipelines, model serving, RAG systems, and LLM integrations |
| **ai-researcher** | AI/ML technology evaluation and RAG/agent architecture design |
| **uiux-designer** | User flow diagrams, wireframe specs, and accessibility audit reports |
| **tech-writer** | README, API docs, ADRs, and developer guides (documentation gate) |

### Skills

Slash commands for common development workflows:

| Command | Description |
|---|---|
| `/create-my-commit` | Commit staged files with conventional commit messages |
| `/create-my-pr` | Create a branch from staged files and submit a pull request |
| `/create-my-test` | Generate tests for staged or specified source files |
| `/create-my-doc` | Generate documentation for changed or specified code |
| `/create-my-adr` | Generate an Architecture Decision Record (ADR) |
| `/check-my-deps` | Check dependency health, outdated packages, and security advisories |
| `/security-scan` | Scan code for security vulnerabilities and hardcoded secrets |
| `/codex-review` | Review gate using Codex CLI (read-only) to iterate review and fix until convergence |
| `/gemini-review` | Review gate using Gemini CLI (read-only) to iterate review and fix until convergence |

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
