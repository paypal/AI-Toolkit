---
name: paypal-subscriptions
description: PayPal Subscriptions - recurring billing, plan management, free trials, plan upgrades/downgrades, and revisions.
---

# Subscriptions

**When to Use:** Developer needs recurring billing, subscription plans, free trials, or plan upgrades/downgrades.
**When NOT to Use:** One-time payments (see checkout.md), BNPL installments (see bnpl.md).
**For v6 SDK:** See [js-sdk-v6.md](js-sdk-v6.md) for the v6 approach (`createPayPalSubscriptionPaymentSession`, `paypal-subscriptions` component, `paymentFlow: "RECURRING_PAYMENT"`).

## Three-Step Setup

[Subscriptions](https://developer.paypal.com/docs/subscriptions/) require server-side setup:

1. **Create a Product** — `POST /v1/catalogs/products`
2. **Create a Plan** — `POST /v1/billing/plans` with pricing and billing cycles
3. **Create a Subscription** — `POST /v1/billing/subscriptions` against an active plan

The plan must be in `ACTIVE` status before subscriptions can be created.

## JS SDK Integration

Use `vault=true&intent=subscription` in the SDK URL and `actions.subscription.create({ plan_id })` in `createSubscription`. In `onApprove`, send `data.subscriptionID` to your server — never grant access before verifying the subscription status via `GET /v1/billing/subscriptions/{id}`, because the client-side callback alone does not confirm that payment was actually collected.

## Lifecycle Operations

Support: suspend, cancel, and plan revision (upgrade/downgrade via `/revise`). When revising a plan, redirect the subscriber to the returned `approve` link to confirm. For free trials, add a `TRIAL` billing cycle with `sequence: 1` before the `REGULAR` cycle.

## Critical Webhook Events

- `BILLING.SUBSCRIPTION.ACTIVATED`
- `BILLING.SUBSCRIPTION.PAYMENT.FAILED`
- `BILLING.SUBSCRIPTION.CANCELLED`
- `BILLING.SUBSCRIPTION.SUSPENDED`
- `PAYMENT.SALE.COMPLETED` (each successful renewal)

## Live Documentation
- [Subscriptions guide](https://developer.paypal.com/subscriptions/about.md)
- [Billing Plans API](https://developer.paypal.com/docs/api/subscriptions/v1/)
