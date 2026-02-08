# Claude Code Plugins Marketplace

A curated marketplace of plugins for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), providing specialized agents and development workflow skills to enhance your software engineering experience.

## Quick Start

1. Add this marketplace to Claude Code:

```bash
/plugin marketplace add marevol/claude-code-plugins
```

2. Install a plugin:

```bash
/plugin install marevol@marevol-plugins
```

## Available Plugins

### marevol

A comprehensive Claude Code plugin providing 14 specialized agents and 9 development workflow skills for software engineering teams.

#### Agents

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

#### Skills

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

For more details, see the [marevol plugin README](plugins/marevol/README.md).

## Repository Structure

```
claude-code-plugins/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace configuration
├── plugins/
│   └── marevol/
│       ├── .claude-plugin/
│       │   └── plugin.json       # Plugin configuration
│       ├── agents/               # 14 specialized agent definitions
│       ├── skills/               # 9 development workflow skills
│       └── README.md             # Plugin documentation
├── LICENSE                       # Apache License 2.0
└── README.md                     # This file
```

## Contributing

Contributions are welcome! To add a new plugin:

1. Create a directory under `plugins/` with your plugin name
2. Add a `.claude-plugin/plugin.json` with your plugin metadata
3. Add agents and/or skills following the existing structure
4. Update `.claude-plugin/marketplace.json` to register your plugin
5. Submit a pull request

## License

This project is licensed under the [Apache License 2.0](LICENSE).
