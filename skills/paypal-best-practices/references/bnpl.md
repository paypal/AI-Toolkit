---
name: paypal-bnpl
description: PayPal Buy Now Pay Later (BNPL) - installments, Pay in 4, Pay Later messaging banners, financing, and eligibility.
---

# Buy Now, Pay Later (BNPL)

**When to Use:** Developer mentions installments, pay later, pay in 4, split payments, financing, or BNPL messaging banners.
**When NOT to Use:** One-time payments without installments (see checkout.md), subscriptions (see subscriptions.md).
**For v6 SDK:** See [js-sdk-v6.md](js-sdk-v6.md) for the v6 approach (`createPayLaterOneTimePaymentSession`, `createPayPalMessages`).

## Integration

[BNPL](https://developer.paypal.com/v5/pay-later/integrate.md) is surfaced through the JS SDK and Orders API v2. Add `components=messages` to the SDK URL to render promotional messaging banners using `paypal.Messages({ amount, pageType })` on product detail, cart, and checkout pages — highest-impact placement for conversion. Render the Pay Later button with `fundingSource: paypal.FUNDING.PAYLATER`.

PayPal automatically determines buyer eligibility — no separate API call needed. Merchants receive the full amount upfront. Always use `intent=CAPTURE` (not `intent=subscription`) for BNPL flows.

## Country Availability

| Country | Products | Limits |
|---------|----------|--------|
| **United States** | Pay in 4 (biweekly), Pay Monthly (3/6/12/24 mo) | $30–$1,500 (Pay in 4), $49–$10,000 (Monthly) |
| **United Kingdom** | Pay in 3 (monthly), PayPal Credit | £20–£3,000 (Pay in 3) |
| **Australia** | Pay in 4 (biweekly) | A$1–$1,999.99 |
| **Germany** | Ratenzahlung (3/6/12/24 mo), Pay in 30 | €99–€10,000 (installments), €1–€2,000 (Pay in 30) |
| **France** | Pay in 4 (over 90 days) | €30–€2,000 |
| **Italy** | Pay in 3, Pay in installments (6/12/24 mo) | €30–€2,000 (Pay in 3), €120–€5,000 (installments) |
| **Spain** | Pay in 3, Pay in installments (6/12/24 mo) | €30–€2,000 (Pay in 3), €120–€5,000 (installments) |
| **Canada** | Pay in 4 (biweekly) | C$30–$1,500 |

Always check the buyer's country and currency before rendering BNPL messaging or buttons — products and eligibility rules differ per market.

## Live Documentation
- [Pay Later / BNPL v6 — see js-sdk-v6.md](js-sdk-v6.md) (`createPayLaterOneTimePaymentSession`, `createPayPalMessages`)
- [Pay Later overview — v5 docs](https://developer.paypal.com/v5/pay-later/integrate.md)
- [BNPL messaging — v5 docs](https://developer.paypal.com/v5/pay-later/integrate.md)

For the latest country/currency availability, fetch the docs link above if WebSearch is available.
