---
name: paypal-3d-secure
description: 3D Secure (3DS) card authentication, liability shift, SCA, and PSD2 Strong Customer Authentication for PayPal card payments.
---

# 3D Secure (3DS)

**When to Use:** Developer asks about card authentication, liability shift, SCA, PSD2, or 3DS integration.
**When NOT to Use:** Non-card payments (PayPal button, Venmo, BNPL — 3DS does not apply).
**For v6 SDK:** In v6, 3DS is triggered automatically via `cardSession.submit()` — see [js-sdk-v6.md](js-sdk-v6.md) Card Fields section. Returns `state: "succeeded"/"canceled"/"failed"` with `data.liabilityShift`.

## Overview

[3D Secure](https://developer.paypal.com/v5/expanded/3d-secure/integrate/card-fields.md) authenticates cardholders through their card issuer to reduce fraud and shift chargeback liability from merchant to issuer on success. Available in 36 countries across 22 currencies via Advanced Checkout (Advanced Card Fields). Only triggers for enrolled cards.

## Response Parameters

Evaluate before capturing:

| Parameter | Values | Guidance |
|-----------|--------|----------|
| `liability_shift` | `POSSIBLE` — proceed; `NO` — merchant bears liability, consider declining; `UNKNOWN` — issuer unavailable, ask buyer to retry | Primary decision field |
| `enrollment_status` | `Y` enrolled, `N` not enrolled, `U` unavailable, `B` bypassed | |
| `authentication_status` | `Y` success, `N` failed, `R` rejected, `A` attempted, `U` unable, `C` challenge required | |

When using the JS SDK, only `liability_shift` is returned — use the Orders API directly for full `authentication_result` detail.

## EU/UK Requirement

For European merchants, 3DS is required for Strong Customer Authentication (SCA) under PSD2 — always enable it for card payments in the EU/UK.

Never capture an order when `liability_shift` is `NO` unless you explicitly accept the fraud risk.

## Fastlane

For 3DS on Fastlane integrations, see [fastlane.md](fastlane.md) — the flow differs (uses `ThreeDomainSecureClient` or `attributes.verification` on the order, not Advanced Card Fields).

## Live Documentation
- [3D Secure guide](https://developer.paypal.com/v5/expanded/3d-secure/integrate/card-fields.md)
