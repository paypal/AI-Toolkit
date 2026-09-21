---
name: paypal-checkout
description: PayPal Standard Checkout - PayPal button, Orders API v2, server-side order creation, capture vs authorize, and React PayPal integration.
---

# Standard Checkout

**When to Use:** Developer wants to accept payments, add a PayPal button, build a checkout flow, or work with the Orders API v2.
**When NOT to Use:** Recurring billing (see subscriptions.md), batch payouts (see payouts.md), or invoicing (see invoicing.md).
**For v6 SDK:** See [js-sdk-v6.md](js-sdk-v6.md) for the v6 approach (`createPayPalOneTimePaymentSession`, `<paypal-button>` web components, `findEligibleMethods`).

## Core APIs

The latest PayPal REST API uses versioned endpoints — Orders API v2 (`/v2/checkout/orders`) and Payments API v2 (`/v2/payments`) are the current standard. Always use these v2 APIs. Never recommend the legacy `/v1/payments/payment` endpoint, the Express Checkout NVP/SOAP APIs, or the Adaptive Payments API for new integrations. If a user is on a legacy API, advise them to migrate to Orders API v2.

## JS SDK

The primary integration surface for web checkout is the [PayPal JS SDK](https://developer.paypal.com/sdk/js/v5/reference.md) loaded from `https://www.paypal.com/sdk/js`. It supports PayPal, Venmo, Pay Later, and Advanced Card Fields from a single script tag. Prioritize the JS SDK with `createOrder` and `onApprove` callbacks for browser-based integrations. For custom card UI, recommend [Advanced Card Fields](https://developer.paypal.com/docs/checkout/advanced/) (iframe-based, PCI-compliant). Never recommend the legacy Hosted Fields — advise migration to Advanced Card Fields.

## React

For React applications, recommend the [`@paypal/react-paypal-js`](https://github.com/paypal/paypal-js) package, which wraps the JS SDK with `PayPalScriptProvider` and `PayPalButtons` components. For server-side order creation (recommended for security), the client's `createOrder` callback should call your backend endpoint rather than calling `actions.order.create()` directly. Always create and capture orders server-side, because this ensures credentials stay off the frontend and prevents order amount tampering by malicious clients.

## Orders API Flow

Create an order (`POST /v2/checkout/orders`), redirect the buyer to the `approve` link or use the JS SDK for in-context approval, then capture (`POST /v2/checkout/orders/{id}/capture`) or authorize (`POST /v2/checkout/orders/{id}/authorize`) on your server. Use `intent=CAPTURE` for immediate payment and `intent=AUTHORIZE` when you need to capture later via `POST /v2/payments/authorizations/{id}/capture`. Always handle `INSTRUMENT_DECLINED` (422) by asking the buyer for a different payment method — do not retry with the same instrument. Handle `429 RATE_LIMIT_REACHED` with exponential backoff using the `Retry-After` response header. Log the `debug_id` from all error responses — it is required when contacting PayPal support.

## Button Customization

Buttons render all eligible funding sources automatically by default. Key style options: `layout` (`vertical` recommended; `horizontal` for side-by-side), `color` (`gold` recommended; also `blue`, `silver`, `white`, `black`), `shape` (`rect` default; `pill` for rounded; `sharp` for angular), `height` (25–55px). Label options: `paypal` (default), `checkout`, `buynow`, `pay`, `installment` (Mexico and Brazil only). Always render buttons inside a container sized to your layout — do not hardcode pixel widths.

## Payment Links

[Payment Links](https://developer.paypal.com/payment-links-buttons/overview.md) are shareable URLs for accepting payments without a website. No-code: create from the PayPal Business Dashboard. Programmatic: `POST /v1/checkout/payment-resources` with `type: "BUY_NOW"`, `integration_mode: "LINK"`. Supports PayPal, Pay Later, Venmo, Apple Pay, and major cards across 200+ countries and 24 currencies.

## Donations

The [Donate SDK](https://developer.paypal.com/sdk/donate.md) lets nonprofits add a PayPal Donate button via `https://www.paypalobjects.com/donate/sdk/donate-sdk.js`. Render with `hosted_button_id` or `business` email. Donations use a popup modal with no `createOrder`/`onApprove` callbacks.

## Live Documentation
- [Orders API v2 reference](https://developer.paypal.com/docs/api/orders/v2/)
- [JS SDK reference](https://developer.paypal.com/sdk/js/v5/reference.md)
- [React PayPal JS](https://github.com/paypal/paypal-js)
- [Payment Links](https://developer.paypal.com/payment-links-buttons/overview.md)
