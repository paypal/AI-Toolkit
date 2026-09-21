---
name: paypal-expanded-checkout
description: PayPal Expanded Checkout- Advanced Card Fields, Apple Pay, Google Pay, and alternative payment methods (iDEAL, Bancontact, BLIK, Przelewy24).
---

# Expanded Checkout & Alternative Payment Methods

**When to Use:** Developer needs Advanced Card Fields, Apple Pay, Google Pay, or regional APMs beyond standard PayPal/Venmo buttons.
**When NOT to Use:** Standard PayPal button only (see checkout.md), Venmo standalone (see venmo.md).
**For v6 SDK:** See [js-sdk-v6.md](js-sdk-v6.md) for v6 card fields (`createCardFieldsComponent`), Apple Pay (`applepay-payments`), and Google Pay (`googlepay-payments`).

## Expanded Checkout

[Expanded Checkout](https://developer.paypal.com/docs/checkout/apm/) combines the JS SDK with customizable card payment forms and Alternative Payment Methods (APMs). Use when merchants need branded card UI (via Advanced Card Fields), local payment methods, or digital wallets. Supports: PayPal, Venmo, Pay Later, credit/debit cards, Apple Pay, Google Pay, and regional APMs.

## Bank Redirect APMs

Buyer is redirected to their bank to authenticate, then returned to merchant: iDEAL (Netherlands), Bancontact (Belgium), BLIK (Poland), Przelewy24 (Poland), EPS (Austria), MyBank (Italy), Multibanco (Portugal — voucher-based), Trustly (Austria, Germany, Denmark, Estonia, Spain, Finland, UK, Lithuania, Latvia, Netherlands, Norway, Sweden).

To render: add the APM's funding source to the SDK URL (`enable-funding=ideal,bancontact` etc.), render with `fundingSource: paypal.FUNDING.IDEAL`, and handle the redirect return on `onApprove` server-side. Refund window is 180 days (up to 365 for some).

## Apple Pay

Add `components=applepay` to the SDK URL. Host the domain association file at `/.well-known/apple-developer-merchantid-domain-association` for every domain. Register domains in the PayPal Developer Dashboard. Implement `onvalidatemerchant` (`paypal.Applepay().validateMerchant()`) and `onpaymentauthorized` (`paypal.Applepay().confirmOrder()`). Apple Pay only works on Safari/iOS/macOS with HTTPS — always test on real Apple devices.

## Google Pay

Add `components=googlepay` to the SDK URL. Also load `https://pay.google.com/gp/p/js/pay.js`. Call `paypal.Googlepay().config()` for allowed payment methods, check eligibility with `isReadyToPay()`, confirm orders in `onPaymentAuthorized`. Available in 36 countries and 22 currencies.

Both Apple Pay and Google Pay require enabling the feature in the PayPal Developer Dashboard (Apps & Credentials > Features) and completing production onboarding.

## Pay upon Invoice (Germany)

Deferred payment — Germany only, B2C only. Buyers pay within 30 days via bank transfer to Ratepay; merchants funded immediately by PayPal. Requires: German VAT ID, PayPal approval, €5–€2,500 range, shipment within 7 days, mandatory legal disclosures, shipment tracking via Add Tracking API. Not available for digital goods, vouchers, or gift cards.

## Live Documentation
- [Expanded Checkout](https://developer.paypal.com/v5/expanded/overview.md)
- [Apple Pay integration](https://developer.paypal.com/v5/apple-pay/integrate.md)
- [Google Pay integration](https://developer.paypal.com/v5/google-pay/integrate.md)
