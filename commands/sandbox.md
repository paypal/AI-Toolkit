---
description: Show PayPal sandbox setup, credentials, and developer dashboard resources
argument-hint: [topic — e.g. "setup", "credentials", "ngrok", "postman"]
allowed-tools: Read, Bash, mcp__paypal-sandbox__*
---

Show PayPal sandbox developer setup. If "$ARGUMENTS" matches a recognized topic, show only that section. If "$ARGUMENTS" is empty or unrecognized, show all sections below.

Topic aliases:
- "Getting Started" → aliases: setup, start, begin, quickstart, intro
- "Sandbox vs Production Credentials" → aliases: credentials, creds, auth, keys, secret, token
- "Postman" → aliases: postman, collection, api client

## Getting Started

1. Create a Developer account at https://developer.paypal.com
2. Go to **Dashboard → Apps & Credentials → Sandbox** — create an app to get your Client ID and Secret
3. Go to **Dashboard → Sandbox → Accounts** — create sandbox buyer and seller accounts for testing
4. Use **Sandbox base URL**: `https://api-m.sandbox.paypal.com`
5. Use **Sandbox MCP endpoint**: `https://mcp.sandbox.paypal.com/sse`

## Sandbox vs Production Credentials

| | Sandbox | Production |
|---|---|---|
| API base URL | `api-m.sandbox.paypal.com` | `api-m.paypal.com` |
| Buyer approval | `sandbox.paypal.com` | `paypal.com` |
| Credentials | Sandbox app in Dashboard | Live app in Dashboard |

Never mix sandbox and production credentials — they are completely separate environments. This plugin's MCP server only connects to the sandbox environment (`mcp.sandbox.paypal.com/sse`).

## Postman

PayPal publishes an official Postman collection with pre-built requests for all major APIs.

1. Open Postman and import the collection: https://www.postman.com/paypal
2. Set these collection variables:
   - `base_url` → `https://api-m.sandbox.paypal.com`
   - `client_id` → your sandbox Client ID
   - `client_secret` → your sandbox Client Secret
3. Run the **Get Access Token** request first — Postman will store the token automatically
4. You can now call Orders, Payments, Subscriptions, and other APIs directly from Postman

## Key Resources

- Developer Dashboard: https://developer.paypal.com/dashboard
- Sandbox accounts: https://developer.paypal.com/dashboard/accounts
- Webhooks Simulator: https://developer.paypal.com/dashboard/webhooksSimulator
- API Reference: https://developer.paypal.com/api/rest/
- PayPal Postman collection: https://www.postman.com/paypal
- JS SDK reference: https://developer.paypal.com/sdk/js/reference/

For testing specific scenarios (declines, BNPL, Venmo, subscriptions, disputes, webhooks), use `/paypal:test-accounts` for a detailed scenario guide.
