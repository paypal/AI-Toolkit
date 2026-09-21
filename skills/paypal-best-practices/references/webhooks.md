---
name: paypal-webhooks
description: PayPal webhooks - signature verification, event types (PAYMENT.CAPTURE.*, BILLING.SUBSCRIPTION.*), event handling, and webhook simulator testing.
---

# Webhooks

**When to Use:** Developer asks about webhook setup, signature verification, event handling, or specific webhook event types.
**When NOT to Use:** Non-webhook payment issues (see checkout.md), MCP connection problems (use `/paypal:setup`).

## Verification

Always verify webhook signatures using `POST /v1/notifications/verify-webhook-signature` before processing any event — never skip verification in production.

Return HTTP 200 immediately from your handler and process events asynchronously, because PayPal retries delivery if it doesn't receive a 200 within 30 seconds — slow processing causes duplicate events. Register explicit event types rather than wildcard subscriptions, since this prevents your handler from receiving irrelevant events and reduces noise.

## Critical Event Types

**Payment events:**
- `PAYMENT.CAPTURE.COMPLETED`
- `PAYMENT.CAPTURE.DENIED`
- `PAYMENT.CAPTURE.REFUNDED`

**Subscription events:**
- `BILLING.SUBSCRIPTION.ACTIVATED`
- `BILLING.SUBSCRIPTION.PAYMENT.FAILED`
- `BILLING.SUBSCRIPTION.CANCELLED`
- `BILLING.SUBSCRIPTION.SUSPENDED`
- `PAYMENT.SALE.COMPLETED` (each renewal)

**Dispute events:**
- `CUSTOMER.DISPUTE.CREATED`
- `CUSTOMER.DISPUTE.RESOLVED`

## Testing

Use the [Webhooks Simulator](https://developer.paypal.com/dashboard/webhooksSimulator) for testing without real transactions. Never use production credentials in test code.

## Live Documentation
- [Webhook signature verification — v6 docs](https://developer.paypal.com/api/webhooks/v1/definitions/verify_webhook_signature.md)
- [Webhook event format — v6 docs](https://developer.paypal.com/api/webhooks/v1/definitions/webhook.md)
- [Webhooks guide — v5 docs](https://developer.paypal.com/api/rest/webhooks/)
- [Webhooks Simulator](https://developer.paypal.com/dashboard/webhooksSimulator)
