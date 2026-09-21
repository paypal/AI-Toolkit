---
name: paypal-invoicing
description: PayPal Invoicing API - create, send, and track itemized invoices, reminders, partial payments, and QR codes.
---

# Invoicing

**When to Use:** Developer asks about creating, sending, or tracking invoices programmatically.
**When NOT to Use:** Payment Links (see checkout.md), one-time checkout (see checkout.md).

## Overview

The [Invoicing API](https://developer.paypal.com/invoicing/about.md) (`/v2/invoicing/invoices`) lets merchants programmatically create, send, and track itemized invoices.

Two primary steps:
1. **Create a draft** — `POST /v2/invoicing/invoices`
2. **Send it** — `POST /v2/invoicing/invoices/{id}/send`

The creation payload includes invoicer and recipient details, line items with quantities/amounts/taxes/discounts, payment terms (net days), and configuration for partial payments, tips, and custom charges.

Customers pay via a PayPal-hosted URL using PayPal, cards, Venmo, or ACH. Unlike Payment Links, invoices are per-customer (not reusable), support payment reminders, status tracking, and partial payments.

For offline payments (check, wire transfer), use the manual payment recording endpoint.

The Invoicing API is distinct from the MCP `create_invoice` tool — use the REST API directly for full control over invoice structure.

## Live Documentation
- [Invoicing overview](https://developer.paypal.com/invoicing/about.md)
- [Invoicing API reference](https://developer.paypal.com/docs/api/invoicing/v2/)
