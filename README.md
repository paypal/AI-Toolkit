# PayPal AI Toolkit

A plugin that integrates PayPal's APIs and MCP server into your AI coding workflow. Get AI-assisted help with PayPal payments, subscriptions, invoices, disputes, and more — in Claude Code, OpenAI Codex, and other compatible agents.

## Features

- **Best practices Skill** — auto-injects PayPal API knowledge when you're working on payment integrations
- **Commands** — quick reference commands for common developer tasks (Claude Code)
- **Hooks** — automatically checks PayPal best practices before writing integration code
- **MCP Server integration** — connects your agent to PayPal's sandbox MCP servers for direct API operations via natural language

![Demo of installation and usage](https://github.com/user-attachments/assets/787a2b4c-4276-422c-9124-fd535571a68c)

## Installation

### Prerequisites

- A PayPal Developer account at https://developer.paypal.com
- A PayPal sandbox access token
- Claude Code (`claude --version`) and/or OpenAI Codex (`codex --version`)

### Claude Code

PayPal AI Toolkit is available on the official Anthropic marketplace, which Claude Code adds automatically. Install it with:

```
/plugin install paypal@claude-plugins-official
```

Then run `/reload-plugins` to activate it.

#### Local / development install

To work on the plugin itself, clone the repo and load it directly instead:

```bash
git clone https://github.com/paypal/AI-Toolkit.git
```

Then start Claude Code with the plugin loaded:

```bash
cd your-project/
claude --plugin-dir /path/to/AI-Toolkit
```

### OpenAI Codex

Add the PayPal marketplace, then install the plugin:

```bash
codex plugin marketplace add paypal/AI-Toolkit
codex plugin add paypal@paypal-ai-toolkit
```

#### Local / development install

```bash
git clone https://github.com/paypal/AI-Toolkit.git
codex plugin marketplace add /path/to/AI-Toolkit
codex plugin add paypal@paypal-ai-toolkit
```

Restart Codex after installing so skills and the sandbox MCP server load.

### Configure your sandbox access token

1. Generate a sandbox access token:

   ```bash
   curl -X POST https://api-m.sandbox.paypal.com/v1/oauth2/token \
     -u "YOUR_SANDBOX_CLIENT_ID:YOUR_SANDBOX_CLIENT_SECRET" \
     -d "grant_type=client_credentials" \
     | jq -r .access_token
   ```

   Get the client ID and secret from the [PayPal Developer Dashboard](https://developer.paypal.com/dashboard/applications/sandbox).

2. Paste the single-line value into your agent's config.

   **Claude Code** — merge into the existing `"env"` block in `~/.claude/settings.json`:

   ```json
   "env": {
     "PAYPAL_SANDBOX_ACCESS_TOKEN": "A21AA…"
   }
   ```

   **OpenAI Codex** — merge into `~/.codex/config.toml`:

   ```toml
   [shell_environment_policy.set]
   PAYPAL_SANDBOX_ACCESS_TOKEN = "A21AA…"
   ```

3. **Fully quit and reopen** the agent (close the app — not just `/clear`).

4. In Claude Code, run `/paypal:setup` to verify. In Codex, ask the agent to list PayPal MCP tools or create a sandbox invoice.

> **Use the agent config file, not `~/.zshrc`.** GUI launches don't source `~/.zshrc`, and a line-wrapped `export` embeds a newline in the token that breaks the HTTP header.

Tokens expire in up to 8 hours depending on scope — check the `expires_in` field in the response. Run `/paypal:setup refresh` when you hit a 401.

## Commands

Slash commands are available in Claude Code. Codex uses the same skills and MCP server through natural language.

| Command                         | Description                                       |
| ------------------------------- | ------------------------------------------------- |
| `/paypal:doctor [symptom]`      | Scan your codebase for PayPal integration issues  |
| `/paypal:explain-error <code>`  | Explain a PayPal error code with causes and fixes |
| `/paypal:sandbox [topic]`       | Sandbox setup, test accounts, and testing tips    |
| `/paypal:setup [mode]`          | Configure the plugin, verify MCP, refresh tokens  |
| `/paypal:test-accounts [topic]` | Test cards, BNPL, Venmo, webhooks, and scenarios  |

### Examples

```
/paypal:explain-error INSTRUMENT_DECLINED
/paypal:explain-error 422 UNPROCESSABLE_ENTITY
/paypal:sandbox subscriptions
/paypal:sandbox webhooks
/paypal:setup
/paypal:doctor webhooks not firing
```

## MCP Server

The plugin connects to PayPal's sandbox MCP server, which exposes tools for:

- **Orders & Payments** — create orders, capture payments, process refunds
- **Invoices** — create, send, manage, and generate QR codes for invoices
- **Subscriptions** — manage billing plans and subscriptions
- **Disputes** — list and respond to buyer disputes
- **Catalog** — manage products and pricing
- **Shipment** — create and track shipments
- **Reporting** — list transactions, get merchant insights

### Transport and environments

The server uses **SSE** at the `/sse` path. `paypal-sandbox` activates once you set `PAYPAL_SANDBOX_ACCESS_TOKEN` in `~/.claude/settings.json` (Claude Code) or `~/.codex/config.toml` (Codex).

## Skills

The `paypal-best-practices` skill is automatically invoked when your agent detects you're working on PayPal integrations. It provides:

- Recommended APIs and deprecated APIs to avoid
- Authentication and credential best practices
- Orders, Subscriptions, and Invoices API flows
- Webhook verification and security guidance
- Error handling patterns
- Sandbox testing guidance

The `paypal-routing` skill routes PayPal-related questions to the right command or reference file and is not directly user-invocable.

## Hooks

A `PreToolUse` hook checks whether a file edit touches PayPal SDK, API, or checkout code. If it does, the agent is required to consult the `paypal-best-practices` skill and the correct SDK version reference (v5 or v6) before writing the code.

## Plugin Structure

```
AI-Toolkit/
├── .agents/
│   └── plugins/
│       └── marketplace.json     # Codex marketplace catalog
├── .claude-plugin/
│   ├── marketplace.json     # Claude Code marketplace registry
│   └── plugin.json          # Claude Code plugin manifest
├── .codex-plugin/
│   └── plugin.json          # Codex plugin manifest
├── .mcp.json                # PayPal sandbox MCP server (SSE)
├── skills/
│   ├── paypal-best-practices/
│   │   ├── SKILL.md         # Auto-injected best practices context
│   │   └── references/      # Topic-specific integration guides
│   └── paypal-routing/
│       └── SKILL.md         # Intent routing (non-user-invocable)
├── commands/
│   ├── doctor.md            # /paypal:doctor
│   ├── explain-error.md     # /paypal:explain-error
│   ├── sandbox.md           # /paypal:sandbox
│   ├── setup.md             # /paypal:setup
│   └── test-accounts.md     # /paypal:test-accounts
├── hooks/
│   └── hooks.json           # Pre-write best-practices check
├── LICENSE                  # Apache-2.0
├── CONTRIBUTING.md          # Contribution and commit guidelines
└── README.md
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to report issues, develop locally, and submit pull requests.

Commit messages and PR titles should follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/), for example:

```
feat: allow provided config object to extend other configs
```

## Resources

- [PayPal Developer Docs](https://developer.paypal.com/docs/)
- [PayPal REST API Reference](https://developer.paypal.com/api/rest/)
- [PayPal MCP Server](https://developer.paypal.com/ai-tools/mcp-server.md)
- [PayPal Agent Toolkit](https://github.com/paypal/agent-toolkit)
- [PayPal Developer Dashboard](https://developer.paypal.com/dashboard)
