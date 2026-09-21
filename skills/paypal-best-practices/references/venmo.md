---
name: paypal-venmo
description: Pay with Venmo - Venmo button, eligibility check (isFundingEligible), and Venmo standalone checkout for US merchants and buyers.
---

# Pay with Venmo

**When to Use:** Developer mentions Venmo, Venmo button, or Venmo payments. US merchants and buyers only, USD only.
**When NOT to Use:** Non-US merchants (Venmo is not available). Venmo payouts (see payouts.md).
**For v6 SDK:** See [js-sdk-v6.md](js-sdk-v6.md) for the v6 component-based approach (`createVenmoOneTimePaymentSession`, `venmo-payments` component).

## Integration

[Pay with Venmo](https://developer.paypal.com/v5/venmo/integrate.md) is available for US merchants and buyers via the JS SDK. Add `enable-funding=venmo` to the SDK URL and render a button with `fundingSource: paypal.FUNDING.VENMO`.

Venmo only renders when the buyer is eligible — always call `paypal.isFundingEligible(paypal.FUNDING.VENMO)` before rendering and provide a standard PayPal button as the fallback.

On desktop, Venmo requires a Chrome browser with a Venmo cookie. On mobile, it deep-links to the Venmo native app. Venmo uses USD only and flows through the same Orders API v2 — no separate API integration is required. After capture, confirm the payment method via `payment_source.venmo` in the response.

## Live Documentation
- [Pay with Venmo v6 — see js-sdk-v6.md](js-sdk-v6.md) (`createVenmoOneTimePaymentSession`, `venmo-payments` component)
- [Pay with Venmo — v5 docs](https://developer.paypal.com/v5/venmo/integrate.md)
