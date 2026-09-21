---
name: paypal-agentic-commerce
description: PayPal Agentic Commerce - AI shopping agents, Store Sync, Agent Ready, delegated payment tokens, and ChatGPT commerce integration.
---

# Agentic Commerce

**When to Use:** Developer asks about AI shopping agents, Store Sync, Agent Ready, delegated payment tokens, or ChatGPT commerce integration.
**When NOT to Use:** Standard checkout (see checkout.md), regular MCP tool usage (see mcp-tools.md).

## Overview

[Agentic Commerce](https://developer.paypal.com/agentic-commerce-services/about.md) enables AI shopping assistants to discover products, build carts, and complete PayPal purchases on behalf of buyers.

Two components:
- **Store Sync** — syncs your product catalog and order management system so AI agents can access inventory and place orders directly
- **Agent Ready** — accepts payments through AI shopping platforms like ChatGPT

## Delegated Payment Tokens

Agent Ready uses Braintree-based delegated payment tokens — secure, one-time-use credentials bound to your merchant ID, a max amount, currency, and expiry.

Your MCP server must implement a `complete_checkout` tool that receives the token and processes it via Braintree SDK or GraphQL. The checkout session endpoint (`/checkout_sessions`) must return Braintree payment provider configuration per the ACP spec, and your MCP server must be publicly hosted.

Supported payment methods: `card`, `applepay`, `googlepay`. Transactions initiated via ChatGPT are tagged with `facilitator_details` for filtering in the Braintree Control Panel.

Agentic Commerce is early-access — request access via [this form](https://www.paypal.com/us/business/ai#form) before building.

## Live Documentation
- [Agentic Commerce overview](https://developer.paypal.com/agentic-commerce-services/about.md)
