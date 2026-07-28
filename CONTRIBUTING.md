# Contributing to PayPal AI Toolkit

Thank you for your interest in contributing to the PayPal AI Toolkit. This document outlines the guidelines for reporting issues and submitting contributions.

## Reporting Issues and Bugs

If you find a bug, encounter an issue, or want to suggest an improvement, please open an issue in the GitHub Issues tracker for this repository. Provide clear steps to reproduce the issue, along with any relevant logs or screenshots.

## Local Development and Testing

Since this project is a Claude Code plugin consisting of commands, skills, and hooks:
1. Clone the repository to your local machine.
2. Load the plugin locally into Claude Code by running:
   ```bash
   claude --plugin-dir /path/to/AI-Toolkit
   ```
3. Manually test and verify the behavior of any commands or skills you modified or added.

## Submitting Pull Requests

1. Fork the repository and create your branch from main.
2. Implement your changes.
3. Ensure all modified files and directories follow the existing project layout.
4. Submit a Pull Request (PR) with a clear description of the changes and the problem they solve.
5. Include a source or citation for any factual claims when modifying files under `skills` to prevent unsourced claims from causing confusion.
