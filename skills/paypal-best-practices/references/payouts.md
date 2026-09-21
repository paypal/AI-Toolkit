---
name: paypal-payouts
description: PayPal Payouts API - batch payments to sellers, contractors, and claimants across 96 countries and 24 currencies, plus 1099 reporting.
---

# Payouts

**When to Use:** Developer needs to send money to sellers, contractors, freelancers, or claimants via batch payments.
**When NOT to Use:** Accepting payments from buyers (see checkout.md), invoicing (see invoicing.md).

## Overview

The [Payouts API](https://developer.paypal.com/payouts/overview.md) enables batch payments to multiple recipients in 96 countries across 24 currencies. Access requires approval via the PayPal Developer Dashboard.

Prerequisites: PayPal Business account with verified identity, confirmed email, and sufficient balance.

Two tiers: Standard Payouts (API, web upload, FTP) and Advanced Payouts (50+ currencies, 240+ countries, prepaid cards, 1099 reporting).

## Core API

`POST /v1/payments/payouts` with:
- `sender_batch_header` — batch ID, email subject, message
- `items[]` — each with `recipient_type` (`EMAIL`, `PHONE`, `PAYPAL_ID`, or `VENMO_HANDLE`), `amount`, `receiver`, optional `note` and `sender_item_id`

Poll status with `GET /v1/payments/payouts/{payout_batch_id}` or individual items with `GET /v1/payments/payouts-item/{id}`. Webhooks also supported.

Rate limit: 400 POST requests per minute — handle HTTP 429 with backoff. Venmo payouts (`VENMO_HANDLE`) are US-only, USD-only. Per-item maximum is $20,000 USD.

Always store `sender_item_id` per recipient for idempotency — reuse on retries to avoid duplicates.

## Live Documentation
- [Payouts overview](https://developer.paypal.com/payouts/overview.md)
- [Payouts API reference](https://developer.paypal.com/docs/api/payments.payouts-batch/v1/)
