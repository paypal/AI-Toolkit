---
description: Explain a PayPal API error code or message and provide actionable solutions
argument-hint: <error_code or error_message>
allowed-tools: Read, Grep
---

If "$ARGUMENTS" is empty, respond only with: "Please provide a PayPal error code or message. Example: `/paypal:explain-error INSTRUMENT_DECLINED`" and stop.

If "$ARGUMENTS" does not match any known PayPal error code or message pattern, respond with: "Unknown error code or message: `$ARGUMENTS`. This may be a typo, a third-party error, or a new PayPal error not yet in this reference. Check https://developer.paypal.com/api/rest/reference/orders/v2/errors/ (Orders v2), https://developer.paypal.com/api/rest/responses/ (HTTP status codes), or the Errors section of the relevant API reference (Vault: https://developer.paypal.com/api/payment-tokens/v3, Payouts: https://developer.paypal.com/api/payments.payouts-batch/v1, Webhooks: https://developer.paypal.com/api/webhooks/v1) for the full list." Then do your best to explain what the error string suggests based on its name, and stop — do not fabricate retry behavior or related errors.

Otherwise, explain the PayPal error "$ARGUMENTS". Provide:

1. **What it means** — plain-language explanation
2. **Common causes** — most likely reasons this occurs
3. **How to fix it** — specific, actionable steps
4. **Code example** — detect the user's programming language by scanning any open files with Read/Grep; default to Node.js if no language is found. Show proper error handling including how to catch the error, display a user-friendly message, and whether to retry.
5. **Related errors** — only list errors from the Key error reference below that are meaningfully related (same HTTP status, same flow, or must be handled together). Do not suggest errors outside this list.

Key error reference:

**General (any API):**

- `INVALID_REQUEST` (400) — malformed body or missing fields; inspect the `details[]` array for the specific `field` and `issue`
- `UNAUTHORIZED` (401) — expired or missing access token; refresh via `POST /v1/oauth2/token`
- `AUTHENTICATION_FAILURE` (401) — missing `Authorization` header or invalid credentials; an expired bearer token usually comes back as `invalid_token` — mint a new token via `POST /v1/oauth2/token`
- `NOT_AUTHORIZED` (403) — valid token but the app lacks the scope or feature for this operation (`details[].issue` is usually `PERMISSION_DENIED`); on `GET /v1/reporting/balances` or `/v1/reporting/transactions` it means **Transaction Search** is not enabled for the app — enable it in the Developer Dashboard (Apps & Credentials → your app → Features) and mint a new token
- `PERMISSION_DENIED` (403) — account not eligible for this API; check Developer Dashboard feature eligibility
- `RESOURCE_NOT_FOUND` (404) — resource ID does not exist or belongs to a different account
- `INVALID_RESOURCE_ID` (404) — the `issue` behind most 404s (Orders, Payments, Payouts, Webhooks, Transaction Search): the ID is mistyped, belongs to another account, or was created in the other environment (sandbox vs. live)
- `DUPLICATE_TRANSACTION` (409) — idempotency key already succeeded; retrieve the existing resource instead
- `PREVIOUS_REQUEST_IN_PROGRESS` (409 `RESOURCE_CONFLICT`) — a request with the same `PayPal-Request-Id` is still being processed; wait, then retry with the **same** ID — do not generate a new one
- `RATE_LIMIT_REACHED` (429) — back off with exponential retry; honor the `Retry-After` header
- `INTERNAL_SERVER_ERROR` (500) — retry with the same `PayPal-Request-Id` idempotency key; log `debug_id`

**Orders v2:**

- `INSTRUMENT_DECLINED` (422) — buyer's payment method declined; ask for a different payment method, do not retry the same instrument
- `TRANSACTION_REFUSED` (422) — risk engine decline; do not retry automatically
- `ORDER_NOT_APPROVED` (422) — the buyer has not approved the order yet; send them to the `rel: approve` link (or wait for the JS SDK `onApprove` callback) before calling capture or authorize
- `ORDER_ALREADY_CAPTURED` / `ORDER_ALREADY_AUTHORIZED` (422) — only one capture (`intent=CAPTURE`) or one authorization (`intent=AUTHORIZE`) is allowed per order; `GET /v2/checkout/orders/{id}` to recover the existing capture or authorization ID instead of retrying
- `ORDER_ALREADY_COMPLETED` (422) — a completed order cannot be patched; create a new order instead
- `PAYER_ACTION_REQUIRED` (422) — the buyer must complete a step (3D Secure, bank or APM approval) before the order can be captured; redirect them to the `rel: payer-action` HATEOAS link, then retry
- `AMOUNT_MISMATCH` (422) — `amount.value` must equal `item_total + tax_total + shipping + handling + insurance - shipping_discount - discount` from `amount.breakdown`
- `PAYPAL_REQUEST_ID_REQUIRED` (400) — creating an order with a `payment_source` requires a `PayPal-Request-Id` header

**Vault (Payment Method Tokens v3) and saved payment methods:**

- `NOT_ENABLED_TO_VAULT_PAYMENT_SOURCE` (403 on `/v3/vault/*`, 422 on Orders) — the API caller or merchant is not allowed to vault this payment source; enable **Vault** for the app in the Developer Dashboard or contact PayPal support — not retryable
- `VAULT_INSTRUCTION_REQUIRED` (422) — `payment_source.*.attributes.vault` was sent without an instruction; add `store_in_vault: "ON_SUCCESS"`
- `INVALID_VAULT_ID` (422) — `vault_id` is invalid or not found: wrong environment, deleted token, or a setup token used where a payment token is required
- `MISMATCHED_VAULT_ID_TO_PAYMENT_SOURCE` (422) — the token does not match the `payment_source` key it was sent under (e.g. a PayPal token inside `payment_source.card`)
- `PAYER_ACTION_REQUIRED` (422, Vault v3) — the setup token has not been approved by the payer; complete the approval flow before `POST /v3/vault/payment-tokens`
- `SETUP_TOKEN_ALREADY_TOKENIZED` (422) — the setup token was already exchanged for a payment token; reuse that payment token
- `TOKEN_NOT_FOUND` (404) — the token ID does not exist (deleted, other account, or other environment)
- `PAYPAL_REQUEST_ID_PREVIOUSLY_USED` (422) — the `PayPal-Request-Id` was already used for a different request; use a fresh UUID per request and reuse it only when retrying that same request

**Merchant-initiated (stored credential) charges:**

- `REQUIRED_PARAMETER_FOR_CUSTOMER_INITIATED_PAYMENT` (422) — a field required for a customer-present payment is missing; for an unattended charge send `stored_credential.payment_initiator: "MERCHANT"`
- `MERCHANT_INITIATED_WITH_SECURITY_CODE` (422) — do not send `security_code` on a merchant-initiated card payment; likewise `MERCHANT_INITIATED_WITH_AUTHENTICATION_RESULTS` (no 3DS results) and `MERCHANT_INITIATED_WITH_MULTIPLE_PURCHASE_UNITS` (one purchase unit per order)
- `MISSING_PREVIOUS_REFERENCE` (422) — merchant-initiated network-token charges must include `stored_credential.previous_network_transaction_reference` (or `previous_transaction_reference`)

**Payouts v1:**

- `AUTHORIZATION_ERROR` / `SENDER_EMAIL_UNCONFIRMED` (rejected at `POST /v1/payments/payouts`) — the sending account is not enabled for Payouts or its email is unconfirmed; enable Payouts for the app and confirm the account email in the Developer Dashboard — not retryable, not a code bug
- `REQUIRED_SCOPE_MISSING` — the access token lacks the Payouts scope; mint a token from an app that has Payouts enabled
- `INSUFFICIENT_FUNDS` — the sender's PayPal balance cannot cover the batch; in sandbox, add funds to the business sandbox account
- `RECEIVER_UNREGISTERED` / `RECEIVER_UNCONFIRMED` — not an HTTP error: the batch is accepted and the item shows `transaction_status: UNCLAIMED` with this code in `errors.name`; the recipient has no (confirmed) PayPal account, and unclaimed funds return to the sender after 30 days
- Sandbox tip: set `items[0].note` to `ERRPYO002` (`SENDER_EMAIL_UNCONFIRMED`), `ERRPYO003` (`AUTHORIZATION_ERROR`) or `ERRPYO005` (`INSUFFICIENT_FUNDS`) to simulate these responses — see https://developer.paypal.com/api/payouts/test-go-live

**Webhooks v1:**

- `WEBHOOK_URL_ALREADY_EXISTS` (400) — a webhook for that URL already exists on this app; list with `GET /v1/notifications/webhooks` and reuse or `PATCH` it instead of creating another
- `WEBHOOK_NUMBER_LIMIT_EXCEEDED` (400) — an app can have at most 10 webhooks; delete stale ones
- `verification_status: "FAILURE"` from `POST /v1/notifications/verify-webhook-signature` (HTTP 200, not an error code) — the body was re-serialized, the wrong `webhook_id` was sent, or the event came from the Webhooks Simulator (simulated events never verify); verify the raw bytes of a real delivery

**Subscriptions v1:**

- `PLAN_STATUS_INVALID` — plan must be `ACTIVE` before creating subscriptions
- `SUBSCRIPTION_STATUS_INVALID` — subscription is not in a state that allows this operation

Always include the full error response body and `debug_id` when contacting PayPal support at https://developer.paypal.com/support/.
