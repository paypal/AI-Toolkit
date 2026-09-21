---
description: Configure PayPal plugin — verify credentials, test MCP server connection, and show active environment
argument-hint: [mode — e.g. "refresh", "status"]
allowed-tools: Read, Bash, mcp__paypal-sandbox__*
---

# PayPal Setup

Walk the user through PayPal plugin configuration for Claude Code. Verify everything works before they use other commands.

## Workflow

### Step 1: Check MCP Connection

Two checks, sandbox-first:

1. **Is the sandbox MCP connected?** Count tools whose name matches `mcp__paypal-sandbox__*` in this session. If any are present, run one lightweight probe (`mcp__paypal-sandbox__list_invoices` with `page_size: 1`). Report the tool count from the session.
2. **Is the sandbox token in `~/.claude/settings.json`?** Read the file and check whether the `"env"` block has a non-empty `"PAYPAL_SANDBOX_ACCESS_TOKEN"`.

#### Report

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  PayPal Plugin Setup
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  [✓] Sandbox MCP                — connected (31 tools)
  [✓] Sandbox token in settings  — set

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If the MCP isn't connected OR the probe returns an error (401, SSE header invalid, etc.), report `needs setup` and go to Step 2. If the MCP is connected but tool count looks low (less than 10), note it briefly and proceed to Step 2 to help the user upgrade.

```
  [!] Sandbox MCP                — needs setup
  [!] Sandbox token in settings  — not set
```

**If MCP tools aren't loaded at all** (no `mcp__paypal-sandbox__*` tools in the session and `/mcp` doesn't list `paypal-sandbox`): the plugin isn't installed correctly. Fix:

- Marketplace: `/plugin marketplace add paypal/AI-Toolkit` then `/plugin install paypal@paypal-ai-toolkit`
- Local: `claude --plugin-dir /path/to/AI-Toolkit`

Set up the sandbox connection using the env-var token route (`PAYPAL_SANDBOX_ACCESS_TOKEN` in `~/.claude/settings.json`) — this is the standard path and gives the full sandbox tool surface.

The `SSE error: Header has invalid value` failure mode means the env var is set but contains whitespace/newlines (usually from a multi-line `~/.zshrc` export). Fix: regenerate, put on a single line in `settings.json`.

### Step 2: Diagnose why a set token still fails

Reach this step only when Step 1 reports the sandbox token IS set in `~/.claude/settings.json` but the MCP server still fails to connect (probe returns 401, SSE error, or no tools loaded).

Likely causes, in order of frequency:

1. **Token expired.** PayPal access tokens last up to 8 hours depending on scope. Most common. → Step 3 to regenerate.
2. **Token has stray whitespace or a newline.** Usually from a copy-paste that picked up the surrounding quote characters, or from a multi-line `~/.zshrc` export that was migrated to `settings.json` without flattening. JSON should reject this, but if it survived, the `Authorization` header will be rejected by Node's HTTP layer (`SSE error: Header has invalid value`). → Open `settings.json`, confirm the value is a single line with no whitespace, regenerate if unsure.
3. **Gateway rate-limited (`HTTP 429`).** You've been reconnecting too often during testing. Auth is fine — just wait 1–2 minutes. Do not regenerate.
4. **Network issue.** Behind a corporate VPN/proxy. Check connectivity to `mcp.sandbox.paypal.com`.

If none of the above resolve it, go to Step 3 and generate a fresh token from scratch.

**If the token was NOT set in Step 1:** skip Step 2 and go straight to Step 3.

### Step 3: Credential Setup

Guide the user through generating a sandbox access token.

#### Sandbox Setup

```
Let's set up your PayPal sandbox credentials.

1. Go to: https://developer.paypal.com/dashboard/applications/sandbox
2. Create an app (or use the default "My Testing Application")
3. Copy your Client ID and Client Secret

Then generate an access token (the resulting value is one line):

  curl -X POST https://api-m.sandbox.paypal.com/v1/oauth2/token \
    -u "YOUR_CLIENT_ID:YOUR_CLIENT_SECRET" \
    -d "grant_type=client_credentials" \
    | jq -r .access_token

Paste the returned A21AA… string into ~/.claude/settings.json under the
"env" block (recommended over ~/.zshrc — GUI-launched Claude Code doesn't
read shell profiles, and a multi-line export breaks the Authorization
header):

  {
    "env": {
      ... other keys ...,
      "PAYPAL_SANDBOX_ACCESS_TOKEN": "A21AA…"
    },
    ...
  }

Then FULLY QUIT Claude Code (close the app — not just /clear) and reopen.
```

Wait for the user to confirm they've set the token. Then re-verify by attempting an MCP tool call.

**If 401 / Unauthorized:** "Token was rejected. It may be malformed, expired, or copied with a stray newline. Regenerate with the curl command above — copy the value as a single line."

**If timeout / network error:** "Check your network connection. If you're behind a corporate proxy or VPN, you may need to allowlist `mcp.sandbox.paypal.com`."

### Step 3.5: Project Environment Check

After MCP verification, scan the user's project for **their own** server-side credentials (used by their app to call the PayPal REST API directly — separate from the plugin's MCP auth).

**Check for `.env` files:**

```bash
ls .env .env.sample .env.example 2>/dev/null
```

**Check for v6 integration signals:**

```bash
grep -rl "web-sdk/v6/core\|createInstance\|paypal-payments" --include="*.js" --include="*.jsx" --include="*.ts" --include="*.tsx" --include="*.html" . 2>/dev/null | head -5
```

**Logic:**

- If `.env.sample` or `.env.example` exists but no `.env` → flag it:

  ```
  [!] Project Environment
      Found .env.sample but no .env file.
      Copy it and fill in your credentials:

        cp .env.sample .env

      Required variables for v6 server endpoints:
        PAYPAL_CLIENT_ID=your_client_id
        PAYPAL_CLIENT_SECRET=your_client_secret
        PAYPAL_ENVIRONMENT=sandbox

      These are for your APP's server-side calls — separate from the plugin's
      MCP auth (which uses PAYPAL_*_ACCESS_TOKEN in settings.json).
  ```

- If `.env` exists → check it has `PAYPAL_CLIENT_ID` and `PAYPAL_CLIENT_SECRET` set (non-empty), without ever printing the values:

  ```bash
  grep -q "^PAYPAL_CLIENT_ID=.\+" .env && echo "PAYPAL_CLIENT_ID: set" || echo "PAYPAL_CLIENT_ID: missing"
  grep -q "^PAYPAL_CLIENT_SECRET=.\+" .env && echo "PAYPAL_CLIENT_SECRET: set" || echo "PAYPAL_CLIENT_SECRET: missing"
  ```

  If missing → flag which keys are absent. Never echo, log, or otherwise surface the actual value of `.env` contents.

- If no `.env.sample` and no `.env` but v6 signals detected → flag it:

  ```
  [!] Project Environment
      v6 SDK integration detected but no .env file found.
      Your server endpoints need PAYPAL_CLIENT_ID and PAYPAL_CLIENT_SECRET.
      Create a .env file in the project root with these variables.
  ```

- If no v6 signals and no `.env.sample` → skip silently.

Include the project environment status in the Step 4 summary output.

### Step 4: Environment Summary

After successful connection, present a summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  PayPal Plugin — Ready
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  [✓] Sandbox MCP                — connected (31 tools)
  [✓] Sandbox token in settings  — set

  Your project
  [✓] .env — PAYPAL_CLIENT_ID and PAYPAL_CLIENT_SECRET set
       — or —
  [!] .env.sample found but no .env (copy it and fill in credentials)
       — or —
  [ ] No server-side PayPal code detected — skipped

  Token expires in up to 8 hours depending on scope. If you see 401s later, run /paypal:setup refresh.

  Useful commands
  /paypal:doctor [symptom]       — Scan your code for integration issues
  /paypal:explain-error <code>   — Explain a PayPal error code
  /paypal:sandbox [topic]        — Sandbox setup reference
  /paypal:test-accounts [topic]  — Test scenarios and accounts

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 5: Suggest Next Steps

Detect PayPal code in the project root:

```bash
grep -rl "paypal\|@paypal/\|api-m.paypal.com" --include="*.js" --include="*.jsx" --include="*.ts" --include="*.tsx" --include="*.html" --include="*.py" --include="*.java" --include="*.rb" --include="*.php" . 2>/dev/null | head -5
```

**If the project has PayPal-related code:**

```
Your project has PayPal integration code. Try:
  /paypal:doctor — Scan your integration for issues
```

**If the project has no PayPal code:**

```
No PayPal code detected in this project. Try:
  - Ask me to "create a PayPal checkout flow" and I'll help you build one
  - /paypal:sandbox — Learn about the sandbox environment
  - /paypal:test-accounts — See test scenarios and test card numbers
```

**If the user already has everything working:**

```
You're all set. Try asking me to:
  - Create an order or capture a payment (via MCP tools)
  - Build a checkout page with the PayPal JS SDK
  - Set up webhook handling for your server
```

## Error Reference (LLM-facing)

| Error                                                | Likely Cause                                                                                                                                                  | Fix                                                                                                                                            |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| No `mcp__paypal-sandbox__*` tools in session         | Plugin not loaded                                                                                                                                             | Install via marketplace or restart: `claude --plugin-dir /path/to/AI-Toolkit`                                                                  |
| 401 Unauthorized                                     | Token expired or malformed                                                                                                                                    | Regenerate via the curl in Step 3 and paste a single-line value into `~/.claude/settings.json`, then restart Claude Code                       |
| `SSE error: Header has invalid value`                | Token in env var contains a newline/whitespace (multi-line `~/.zshrc` export)                                                                                 | Move the token to `~/.claude/settings.json` as a single line                                                                                   |
| `SSE error: Non-200 status code (429)` or `HTTP 429` | PayPal's hosted gateway is rate-limiting reconnects from the same client (commonly hit when toggling Enable/Authenticate/Reconnect repeatedly during testing) | Wait 1–2 minutes before retrying. Don't change tokens or `.mcp.json` — the auth is fine, the gateway just needs the request rate to cool down. |
| Timeout / connection refused                         | Network, proxy, or VPN blocking                                                                                                                               | Allowlist `mcp.sandbox.paypal.com`                                                                                                             |
| 403 Forbidden                                        | Token valid but missing API permissions                                                                                                                       | Check app permissions in the PayPal Developer Dashboard                                                                                        |
