---
name: paypal-authentication
description: PayPal OAuth 2.0 access tokens, client credentials flow, idempotency headers (PayPal-Request-Id), and token caching guidance.
---

# Authentication & Idempotency

**When to Use:** Developer asks about OAuth, access tokens, credentials, API authentication, or idempotency headers.
**When NOT to Use:** MCP connection issues (use `/paypal:setup` command instead).

## OAuth 2.0

All PayPal API calls require [OAuth 2.0 authentication](https://developer.paypal.com/api/rest/authentication/). Use the client credentials flow: POST to `/v1/oauth2/token` with `grant_type=client_credentials` and HTTP Basic Auth using your Client ID and Secret.

Access tokens expire in up to 28,800 seconds (8 hours) depending on scope — always read the `expires_in` field from the token response rather than assuming a fixed value, cache the token, and refresh proactively rather than fetching a new token per request, since per-request token fetches add latency and risk hitting rate limits.

## Security Rules

- Never expose `client_secret` in client-side or frontend code
- Store credentials as environment variables (`PAYPAL_CLIENT_ID`, `PAYPAL_CLIENT_SECRET`)
- Use Sandbox (`api-m.sandbox.paypal.com`) for development, Production (`api-m.paypal.com`) for live traffic

## Idempotency

Always include a `PayPal-Request-Id` header with a unique UUID on every POST request — reuse the same value on retries to prevent duplicate transactions.

## Environment URLs

| Environment | Base URL                           |
| ----------- | ---------------------------------- |
| Sandbox     | `https://api-m.sandbox.paypal.com` |
| Production  | `https://api-m.paypal.com`         |

## Live Documentation

- [Apps, credentials & scopes — v6 docs](https://developer.paypal.com/apps-scopes-and-credentials.md)
- [Authentication guide — v5 docs](https://developer.paypal.com/api/rest/authentication/)
- [REST API reference](https://developer.paypal.com/api/rest/)
