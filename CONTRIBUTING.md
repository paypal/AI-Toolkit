# Contributing to PayPal AI Toolkit

Thank you for your interest in contributing to the PayPal AI Toolkit. This document outlines the guidelines for reporting issues and submitting contributions.

## Reporting Issues and Bugs

If you find a bug, encounter an issue, or want to suggest an improvement, please open an issue in the GitHub Issues tracker for this repository. Provide clear steps to reproduce the issue, along with any relevant logs or screenshots.

## Local Development and Testing

This project is an agent plugin consisting of commands, skills, and hooks. Clone the repository, then load it in the agent you are testing:

**Claude Code**

```bash
claude --plugin-dir /path/to/AI-Toolkit
```

**OpenAI Codex**

```bash
codex plugin marketplace add /path/to/AI-Toolkit
codex plugin add paypal@paypal-ai-toolkit
```

**Cursor**

```bash
cursor-agent --plugin-dir /path/to/AI-Toolkit
```

Then confirm skills, commands, and the sandbox MCP server in **Customize**. `plugin marketplace add` only accepts a git URL, not a local path. Copying into `~/.cursor/plugins/local/paypal` also works if **Allow Local Plugin Imports** is enabled for your team.

Then manually test and verify the behavior of any commands or skills you modified or added.

## Commit Messages

This project follows [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/). Commit messages and pull request titles should use this format:

```
<type>[optional scope]: <description>
```

For example:

```
feat: allow provided config object to extend other configs
```

Common types:

- `feat` — a new feature
- `fix` — a bug fix
- `docs` — documentation-only changes
- `chore` — maintenance or tooling
- `refactor` — a code change that neither fixes a bug nor adds a feature

Breaking changes should include `!` after the type (for example `feat!: switch MCP transport`) and describe the break in the commit body. See the [Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/) for the full format.

## Submitting Pull Requests

1. Fork the repository and create your branch from main.
2. Implement your changes.
3. Ensure all modified files and directories follow the existing project layout.
4. Submit a Pull Request (PR) with a Conventional Commits style title and a clear description of the changes and the problem they solve.
5. Include a source or citation for any factual claims when modifying files under `skills` to prevent unsourced claims from causing confusion.
