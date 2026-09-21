---
name: paypal-disputes-refunds
description: PayPal disputes, chargebacks, refunds, evidence submission, dispute lifecycle (INQUIRY/CLAIM), and the Disputes API.
---

# Disputes & Refunds

**When to Use:** Developer asks about chargebacks, disputes, refunds, evidence submission, or the Disputes API.
**When NOT to Use:** Order capture/authorization issues (see checkout.md).

## Refunds

Proactively issue refunds via `POST /v2/payments/captures/{id}/refund` to prevent escalation. Add shipment tracking via `POST /v2/shipping/trackers` to strengthen dispute evidence. Always review a dispute via `GET /v1/customer/disputes/{id}` before accepting a claim — check `dispute_life_cycle_stage` and respond before `seller_response_due_date`. Never automatically accept a dispute claim without review.

## Dispute Categories

[Disputes](https://developer.paypal.com/disputes/overview.md) fall into two categories:
- **Internal disputes** — filed through PayPal's Resolution Center; parties resolve directly before PayPal adjudicates
- **External disputes** — chargebacks and ACH returns filed with banks; PayPal intermediates between merchant and issuer

## Lifecycle Stages

INQUIRY (up to 20 days) → CLAIM → CHARGEBACK → PRE_ARBITRATION → ARBITRATION

Buyers have 180 days from payment date to file. Pre-chargeback alerts give 20 hours to refund and avoid fees.

## API Actions

Beyond `accept-claim`: `POST .../send-message`, `POST .../make-offer`, `POST .../provide-evidence`, `POST .../escalate`, `POST .../provide-supporting-info`, `POST .../appeal`, `POST .../acknowledge-return-item`.

Always check the `links` array (HATEOAS) and `allowed_response_options` before calling action endpoints — available actions change by stage. Evidence is submitted via multipart/form-data (not JSON); check the `evidences` array for what PayPal has requested (`REQUESTED_FROM_SELLER` source).

## Critical Webhook Events

- `CUSTOMER.DISPUTE.CREATED`
- `PAYMENT.CAPTURE.REFUNDED`

## Live Documentation
- [Disputes API reference](https://developer.paypal.com/platforms/disputes/handle-disputes/use-disputes-api.md)
- [Disputes overview](https://developer.paypal.com/disputes/overview.md)
