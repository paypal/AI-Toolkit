---
name: paypal-mcp-tools
description: PayPal MCP server tool inventory - orders, invoices, subscriptions, disputes, shipment tracking, catalog, and merchant insights.
---

# PayPal MCP Server Tools

**When to Use:** Developer asks what MCP tools are available, how to use a specific MCP tool, or wants to execute a live PayPal API operation.
**When NOT to Use:** Architecture decisions or code generation (use the relevant reference file instead).

## Overview

The [PayPal MCP server](https://mcp.paypal.com) ([quickstart](https://developer.paypal.com/ai-tools/mcp-server.md)), when connected, exposes tools for orders, payments, invoices, subscriptions, disputes, catalog, shipment tracking, and reporting.

## Tool Inventory

| Category | Tools |
|----------|-------|
| Orders/Payments | `create_order`, `pay_order` |
| Invoices | 7 invoice tools (create, send, list, etc.) |
| Subscriptions | 7 subscription tools (create plan, create subscription, etc.) |
| Disputes | `list_disputes`, `get_dispute` |
| Catalog | Product management tools |
| Shipment tracking | Tracking tools |
| Reporting | `list_transactions`, merchant insights |

## Commerce Tools (Remote-only)

Three additional tools — `search_product`, `create_cart`, `checkout_cart` — are available for agentic shopping flows but require the request header `x-feature-flags: commerce:true`.

## Guidance

Prefer MCP tools over raw API calls when the MCP server is available in the agent context. For full control over request structure, fall back to the REST API directly.

## Live Documentation
- [MCP quickstart](https://developer.paypal.com/ai-tools/mcp-server.md)
- [MCP server](https://mcp.paypal.com)
