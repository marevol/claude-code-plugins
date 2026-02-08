# Claude Code Plugin

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that provides specialized agents and development workflow skills for software engineering teams.

## Features

### Agents

Specialized agents that Claude Code can delegate tasks to based on context:

| Agent | Description |
|---|---|
| **solution-architect** | System design, architecture planning, and technology selection |
| **backend-engineer** | Server-side development across multiple languages and frameworks |
| **frontend-engineer** | Client-side development with modern web technologies |
| **fullstack-engineer** | End-to-end feature development spanning frontend and backend |
| **code-reviewer** | Comprehensive code review, quality analysis, and technical debt evaluation |
| **test-engineer** | Test strategy design, test implementation, and coverage analysis |
| **tester** | QA testing, test execution, browser-based validation, and accessibility auditing |
| **security-engineer** | Security assessment, vulnerability analysis, and threat modeling |
| **performance-engineer** | Performance profiling, optimization, and benchmarking |
| **devops-engineer** | CI/CD pipelines, infrastructure as code, and cloud operations |
| **ml-engineer** | Machine learning model development, training, and deployment |
| **ai-researcher** | AI/ML technology research, LLM evaluation, and RAG architecture |
| **uiux-designer** | UX design, accessibility standards, and design system methodology |
| **tech-writer** | Technical documentation, API docs, and architecture diagrams |

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
