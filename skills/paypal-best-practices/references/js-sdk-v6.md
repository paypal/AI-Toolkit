---
name: paypal-js-sdk-v6
description: PayPal JavaScript SDK v6 - createInstance, payment sessions, web components, card fields, vaulting, Fastlane v6, and v5-to-v6 migration.
---

# JavaScript SDK v6

**When to Use:** Developer wants to integrate PayPal payments using the v6 Web SDK, upgrade from v5, use the component-based architecture, or work with v6-specific APIs (createInstance, payment sessions, web components, card fields).
**When NOT to Use:** Legacy v5 integrations where migration isn't planned (see checkout.md for v5 guidance). Native mobile SDKs (iOS/Android). Braintree-only integrations.

## What Changed in v6

The v6 SDK is a ground-up redesign of the PayPal JavaScript integration surface. Key differences from v5:

| Area | v5 (Legacy) | v6 (Current) |
|------|-------------|--------------|
| Script URL | `https://www.paypal.com/sdk/js?client-id=X` | `https://www.paypal.com/web-sdk/v6/core` |
| Authentication | Client ID in query string | `createInstance({ clientId })` or client token |
| Button rendering | `paypal.Buttons({ ... }).render('#container')` | `<paypal-button>` web components + event listeners |
| Callbacks | Inline in `paypal.Buttons()` options | Passed to payment session constructors |
| Eligibility | Implicit per render | Explicit via `findEligibleMethods()` |
| Components | `components=buttons,hosted-fields` in URL | `components: ["paypal-payments"]` in createInstance |
| Order return shape | `return orderId` (string) | `return { orderId }` (object) |
| Card fields | Hosted Fields (`paypal.HostedFields`) | `createCardFieldsComponent()` web components |

**Never load v5 (`sdk/js`) and v6 (`web-sdk/v6/core`) on the same page.** Remove the v5 script tag completely before adding v6 — they conflict and will cause unpredictable failures. Always use `async` on the script tag to avoid blocking rendering.

## Script Loading

```html
<!-- Production -->
<script async src="https://www.paypal.com/web-sdk/v6/core" onload="onPayPalWebSdkLoaded()"></script>

<!-- Sandbox -->
<script async src="https://www.sandbox.paypal.com/web-sdk/v6/core" onload="onPayPalWebSdkLoaded()"></script>
```

## Initialization

Use `window.paypal.createInstance()` to create an SDK instance. Two authentication modes:

### Client ID (recommended for most integrations)

```javascript
const sdkInstance = await window.paypal.createInstance({
  clientId: "YOUR_CLIENT_ID",
  components: ["paypal-payments"],
  pageType: "checkout",
  locale: "en-US",
});
```

### Client Token (required for vaulting and Fastlane)

Generate server-side via `POST /v1/oauth2/token` with `response_type=client_token` and `domains[]=YOUR_DOMAIN`. Pass the resulting `access_token` as `clientToken`:

```javascript
const sdkInstance = await window.paypal.createInstance({
  clientToken: await fetchClientToken(),
  components: ["paypal-payments", "fastlane"],
});
```

> **Critical — `clientId` vs `clientToken`:** These are mutually exclusive and NOT interchangeable.
> - Use `clientId` for standard checkout, BNPL, Venmo, and most integrations — no server-side token generation needed.
> - Use `clientToken` **only** when the integration requires vaulting or Fastlane.
> - Never substitute one for the other. If fetched reference material uses `clientId`, use `clientId`. If it uses `clientToken`, use `clientToken`. Do not change either based on assumptions about security or best practices.

### Available Components

| Component | Purpose |
|-----------|---------|
| `paypal-payments` | PayPal and Pay Later checkout |
| `venmo-payments` | Venmo (US only) |
| `paypal-guest-payments` | Standalone card button |
| `paypal-messages` | Pay Later promotional messaging |
| `card-fields` | Inline credit/debit card fields |
| `fastlane` | Accelerated guest checkout |
| `googlepay-payments` | Google Pay |
| `applepay-payments` | Apple Pay |
| `paypal-subscriptions` | Recurring billing / subscriptions |

### Partner Integrations

Partners processing on behalf of sellers must include `merchantId`:

```javascript
const sdkInstance = await window.paypal.createInstance({
  clientId: "PARTNER_CLIENT_ID",
  merchantId: "SELLER_MERCHANT_ID",
  components: ["paypal-payments"],
});
```

## Eligibility Checking

Always check eligibility before rendering buttons. Eligibility depends on buyer location, currency, amount, and merchant configuration.

```javascript
const methods = await sdkInstance.findEligibleMethods({
  currencyCode: "USD",
  amount: "99.99",
});

if (methods.isEligible("paypal"))   { /* show PayPal button */ }
if (methods.isEligible("venmo"))    { /* show Venmo button */ }
if (methods.isEligible("paylater")) { /* show Pay Later button */ }
if (methods.isEligible("credit"))   { /* show PayPal Credit button */ }
```

For Pay Later and PayPal Credit, retrieve product details with `methods.getDetails("paylater")` and apply `productCode` and `countryCode` to the button element.

## Payment Sessions

v6 uses session objects to manage payment flows. Create a session, then start it on button click.

### One-Time PayPal Payment

```javascript
const session = sdkInstance.createPayPalOneTimePaymentSession({
  onApprove: async (data) => {
    const capture = await fetch(`/api/orders/${data.orderId}/capture`, { method: "POST" });
    if (capture.ok) window.location.href = "/success";
  },
  onCancel: () => {
    console.log("Payment cancelled by buyer");
  },
  onError: (error) => {
    console.error(error.code, error.message);
  },
  onShippingAddressChange: async (data) => {
    // Return a promise — resolve to accept the address, reject to force the buyer to pick another.
    const cost = await calculateShipping(data.shippingAddress);
    const res = await fetch(`/api/orders/${data.orderId}/shipping`, {
      method: "PATCH",
      body: JSON.stringify({ shippingCost: cost }),
    });
    if (!res.ok) throw new Error("Could not update shipping");
  },
});
```

### Starting the Session

```javascript
document.querySelector("paypal-button").addEventListener("click", async () => {
  await session.start(
    { presentationMode: "auto" },
    createOrder()  // must return Promise<{ orderId: string }>
  );
});
```

The `createOrder` function must return `{ orderId: "..." }` — this is different from v5 which returned a bare string.

> **Known SDK Bug**: The TypeScript types for `session.start()` declare the second argument as `(() => Promise<{ orderId: string }>) | Promise<{ orderId: string }>`, suggesting both a function reference and an invoked Promise are valid. However, the live SDK runtime rejects a function reference with `SdkInitError: .start() expects a Promise. Received 'function'`. Always invoke the function and pass the resulting Promise directly, as shown above (`createOrder()` not `createOrder`).

### Presentation Modes

| Mode | Use Case |
|------|----------|
| `auto` | Recommended — tries popup, falls back to modal |
| `popup` | Desktop browsers (may be blocked by popup blockers) |
| `modal` | WebView scenarios only — has cookie limitations on desktop |
| `redirect` | Mobile-optimized — full page redirect to PayPal |
| `payment-handler` | Experimental — browser Payment Handler API |
| `direct-app-switch` | Opens PayPal native app |

### Other Session Types

| Method | Purpose |
|--------|---------|
| `createPayPalOneTimePaymentSession()` | Standard PayPal payment |
| `createPayLaterOneTimePaymentSession()` | Pay Later / Pay in 4 |
| `createPayPalCreditOneTimePaymentSession()` | PayPal Credit (US) |
| `createVenmoOneTimePaymentSession()` | Venmo (US, USD only) |
| `createPayPalSavePaymentSession()` | Vault PayPal for future use |
| `createPayPalCreditSavePaymentSession()` | Vault PayPal Credit for future use |
| `createGooglePayOneTimePaymentSession()` | Google Pay |
| `createApplePayOneTimePaymentSession()` | Apple Pay |
| `createFastlane()` | Accelerated guest checkout |
| `createPayPalMessages()` | Pay Later promotional messaging |
| `createPayPalSubscriptionPaymentSession()` | Subscription / recurring payment |
| `createCardFieldsOneTimePaymentSession()` | Inline card fields one-time payment |
| `createCardFieldsSavePaymentSession()` | Vault card via card fields |

### Venmo Session Example

Venmo is US-only and USD-only. Check eligibility before rendering:

```javascript
if (methods.isEligible("venmo")) {
  const venmoSession = sdkInstance.createVenmoOneTimePaymentSession({
    onApprove: async (data) => {
      await fetch(`/api/orders/${data.orderId}/capture`, { method: "POST" });
    },
    onError: (error) => console.error("Venmo error:", error.message),
  });

  document.querySelector("venmo-button").addEventListener("click", async () => {
    await venmoSession.start({ presentationMode: "auto" }, createOrder());
  });
}
```

## Web Components (Buttons)

v6 uses native web components instead of `paypal.Buttons().render()`:

```html
<paypal-button type="pay" class="paypal-gold"></paypal-button>
<venmo-button type="pay" class="venmo-blue"></venmo-button>
<paylater-button hidden></paylater-button>
<paypal-credit-button hidden></paypal-credit-button>
```

### Button Attributes

| Attribute | Values |
|-----------|--------|
| `type` | `pay`, `checkout`, `buynow`, `subscribe` |
| `class` | `paypal-gold` (recommended), `paypal-blue`, `paypal-white` |

### CSS Customization

```css
paypal-button {
  --paypal-button-border-radius: 10px;
  width: 100%;
  max-width: 350px;
}
```

## Card Fields (Advanced Card Processing)

v6 replaces Hosted Fields with a component-based card fields API. Requires `components: ["card-fields"]`.

```javascript
const sdk = await window.paypal.createInstance({
  clientId: "YOUR_CLIENT_ID",
  components: ["card-fields"],
});

const methods = await sdk.findEligibleMethods();
if (methods.isEligible("advanced_cards")) {
  // Use createCardFieldsOneTimePaymentSession() for one-time payments
  // or createCardFieldsSavePaymentSession() for vaulting cards
  const cardSession = sdk.createCardFieldsOneTimePaymentSession();

  const numberField = cardSession.createCardFieldsComponent({
    type: "number", placeholder: "Card number",
  });
  const expiryField = cardSession.createCardFieldsComponent({
    type: "expiry", placeholder: "MM/YY",
  });
  const cvvField = cardSession.createCardFieldsComponent({
    type: "cvv", placeholder: "CVV",
  });

  // Card fields fill their parent container — ensure containers have defined height and width
  document.querySelector("#card-number").appendChild(numberField);
  document.querySelector("#card-expiry").appendChild(expiryField);
  document.querySelector("#card-cvv").appendChild(cvvField);
}
```

### Submit and Capture

```javascript
const orderId = await createOrder();

const { data, state } = await cardSession.submit(orderId, {
  billingAddress: { postalCode: "95131" },
});

switch (state) {
  case "succeeded":
    // data.liabilityShift: "POSSIBLE" (issuer liable), "NO" (merchant liable), "UNKNOWN"
    const capture = await captureOrder(data.orderId);
    break;
  case "canceled":
    // buyer dismissed 3DS — allow retry
    break;
  case "failed":
    console.error("Card submission failed:", data.message);
    break;
}
```

### Styling Card Fields

```javascript
const numberField = cardSession.createCardFieldsComponent({
  type: "number",
  style: {
    input: { fontSize: "16px", lineHeight: "24px" },
    ".invalid": { color: "orange" },
  },
});
```

## Pay Later

Requires `components: ["paypal-payments"]`. Check eligibility with `isEligible("paylater")` and retrieve product details with `getDetails("paylater")`.

```javascript
if (methods.isEligible("paylater")) {
  const paylaterDetails = methods.getDetails("paylater");

  const paylaterSession = sdkInstance.createPayLaterOneTimePaymentSession({
    onApprove: async (data) => {
      await fetch(`/api/orders/${data.orderId}/capture`, { method: "POST" });
    },
    onError: (error) => console.error(error.code, error.message),
  });

  const paylaterButton = document.querySelector("paylater-button");
  paylaterButton.productCode = paylaterDetails.productCode;
  paylaterButton.countryCode = paylaterDetails.countryCode;

  paylaterButton.addEventListener("click", async () => {
    await paylaterSession.start(
      { presentationMode: "auto" },
      createOrder(), // must return Promise<{ orderId }>
    );
  });
}
```

### Pay Later Messaging

Requires `components: ["paypal-messages"]`. Uses the `<paypal-message>` web component:

```html
<paypal-message auto-bootstrap amount="50" currency-code="USD"></paypal-message>
```

```javascript
const sdkInstance = await window.paypal.createInstance({
  clientToken,
  components: ["paypal-messages"],
});
sdkInstance.createPayPalMessages();
```

Update the amount dynamically: `document.querySelector("paypal-message").amount = "99.99"`.

## Subscriptions

Requires `components: ["paypal-subscriptions"]`. Uses `findEligibleMethods({ paymentFlow: "RECURRING_PAYMENT" })` instead of the default `ONE_TIME_PAYMENT`.

```javascript
const sdkInstance = await window.paypal.createInstance({
  clientId,
  components: ["paypal-subscriptions"],
  pageType: "checkout",
});

const methods = await sdkInstance.findEligibleMethods({
  paymentFlow: "RECURRING_PAYMENT",
  currencyCode: "USD",
});

if (methods.isEligible("paypal")) {
  const subscriptionSession = sdkInstance.createPayPalSubscriptionPaymentSession({
    onApprove: async (data) => {
      // data: { subscriptionId, payerId? }
      console.log("Subscription approved:", data.subscriptionId);
    },
    onError: (error) => console.error(error.message),
  });

  document.querySelector("paypal-button").addEventListener("click", async () => {
    // createSubscription must return Promise<{ subscriptionId }>
    await subscriptionSession.start(
      { presentationMode: "auto" },
      createSubscription(),
    );
  });
}
```

The button uses `type="subscribe"`: `<paypal-button type="subscribe"></paypal-button>`.

**Key difference from one-time payments:** `start()` takes `Promise<{ subscriptionId }>` instead of `Promise<{ orderId }>`. Create the subscription server-side via `POST /v1/billing/subscriptions` before starting the session.

## Apple Pay

Requires `components: ["applepay-payments"]` and Apple's SDK: `<script src="https://applepay.cdn-apple.com/jsapi/v1/apple-pay-sdk.js"></script>`.

```javascript
// Check native availability first
if (!window.ApplePaySession?.canMakePayments()) return;

const methods = await sdkInstance.findEligibleMethods({ currencyCode: "USD" });
if (methods.isEligible("applepay")) {
  const applePayDetails = methods.getDetails("applepay");
  const applePaySession = sdkInstance.createApplePayOneTimePaymentSession();

  // Render native Apple Pay button
  container.innerHTML = '<apple-pay-button id="apple-pay-button" buttonstyle="black" type="buy" locale="en">';

  button.addEventListener("click", () => {
    const paymentRequest = {
      ...applePaySession.formatConfigForPaymentRequest(applePayDetails.config),
      countryCode: "US",
      currencyCode: "USD",
      total: { label: "My Store", amount: "99.99", type: "final" },
      requiredBillingContactFields: ["name", "postalAddress"],
    };

    const nativeSession = new ApplePaySession(4, paymentRequest);

    nativeSession.onvalidatemerchant = (event) => {
      applePaySession.validateMerchant({ validationUrl: event.validationURL })
        .then((payload) => nativeSession.completeMerchantValidation(payload.merchantSession))
        .catch(() => nativeSession.abort());
    };

    nativeSession.onpaymentauthorized = async (event) => {
      const order = await createOrder(); // server-side
      await applePaySession.confirmOrder({
        orderId: order.orderId,
        token: event.payment.token,
        billingContact: event.payment.billingContact,
      });
      await captureOrder(order.orderId); // server-side
      nativeSession.completePayment({ status: ApplePaySession.STATUS_SUCCESS });
    };

    nativeSession.begin();
  });
}
```

**Prerequisites:** Domain association file at `/.well-known/apple-developer-merchantid-domain-association`, domains registered in PayPal Dashboard, Apple Pay enabled in sandbox Features. Safari/iOS/macOS only.

## Google Pay

Requires `components: ["googlepay-payments"]` and Google's SDK: `<script src="https://pay.google.com/gp/p/js/pay.js"></script>`.

```javascript
const methods = await sdkInstance.findEligibleMethods({ currencyCode: "USD" });
if (methods.isEligible("googlepay")) {
  const googlePayDetails = methods.getDetails("googlepay");
  const googlePaySession = sdkInstance.createGooglePayOneTimePaymentSession();
  const googlePayConfig = googlePaySession.formatConfigForPaymentRequest(googlePayDetails.config);

  const paymentsClient = new google.payments.api.PaymentsClient({
    environment: "TEST", // "PRODUCTION" for live
    paymentDataCallbacks: {
      onPaymentAuthorized: async (paymentData) => {
        const orderId = await createOrder(); // server-side, returns string ID
        const { status } = await googlePaySession.confirmOrder({
          orderId,
          paymentMethodData: paymentData.paymentMethodData,
        });
        if (status !== "PAYER_ACTION_REQUIRED") {
          await captureOrder({ orderId }); // server-side
        }
        return { transactionState: "SUCCESS" };
      },
    },
  });

  const isReady = await paymentsClient.isReadyToPay({
    allowedPaymentMethods: googlePayConfig.allowedPaymentMethods,
    apiVersion: googlePayConfig.apiVersion,
    apiVersionMinor: googlePayConfig.apiVersionMinor,
  });

  if (isReady.result) {
    const button = paymentsClient.createButton({
      onClick: () => paymentsClient.loadPaymentData({
        ...googlePayConfig,
        transactionInfo: {
          currencyCode: "USD",
          totalPriceStatus: "FINAL",
          totalPrice: "99.99",
        },
        callbackIntents: ["PAYMENT_AUTHORIZATION"],
      }),
    });
    container.appendChild(button);
  }
}
```

When `confirmOrder` returns `status === "PAYER_ACTION_REQUIRED"`, the buyer needs 3DS authentication before capture.

<!-- ─── Fastlane section (owned by the Fastlane team) ─── -->
## Fastlane (Accelerated Guest Checkout)

Requires `components: ["fastlane"]` and **`clientToken`** (not `clientId`). Quick Start uses `FastlanePaymentComponent`; Flexible uses `FastlaneCardComponent`. The choice depends on the integration type, not the SDK version — both components exist in v5 and v6.

### Common mistakes

| Mistake | Correct |
|---------|---------|
| `POST /v1/identity/generate-token` for the client token | `POST /v1/oauth2/token` with body `grant_type=client_credentials&response_type=client_token&intent=sdk_init` |
| `"domains[]": "localhost"` (or IP / unregistered hostname) | **Omit `domains[]` for sandbox/local dev.** PayPal returns `invalid_domain` for `localhost`. Production: list registered origins only. |
| Renaming JSON response field from `access_token` → `client_token` | PayPal returns the client-safe JWT in `access_token` even when `response_type=client_token`. Renaming reads `undefined`. Always use `const { access_token: clientToken } = await response.json();` |
| `payment_source: { token: { id, type: "SINGLE_USE" } }` in order create | `payment_source: { card: { single_use_token: "<token>" } }` |
| Using camelCase (`singleUseToken`) in a raw REST `fetch` body | REST API is snake_case (`single_use_token`); camelCase only applies inside `@paypal/paypal-server-sdk` typed methods |
| Diagnosing `idToken: null` as "merchant not provisioned" | Token was missing `intent=sdk_init` — fix the request body, don't escalate |

### 1. Client token (server-side)

```javascript
const auth = Buffer.from(`${PAYPAL_CLIENT}:${PAYPAL_SECRET}`).toString("base64");

const params = new URLSearchParams({
  grant_type: "client_credentials",
  response_type: "client_token",
  intent: "sdk_init",
});

// PRODUCTION ONLY: register every origin. OMIT for sandbox/localhost (PayPal rejects "localhost" with invalid_domain).
if (process.env.NODE_ENV === "production") {
  params.append("domains[]", "shop.example.com");
}

const response = await fetch("https://api-m.sandbox.paypal.com/v1/oauth2/token", {
  method: "POST",
  headers: {
    "Content-Type": "application/x-www-form-urlencoded",
    "Authorization": `Basic ${auth}`,
  },
  body: params,
});

// The field is `access_token`, not `client_token` — even with response_type=client_token.
const { access_token: clientToken } = await response.json();
```

> **About `intent=sdk_init`:** strictly required when you use Fastlane's identity lookup (`lookupCustomerByEmail` → `triggerAuthenticationFlow`). Without it, PayPal returns a plain `client_credentials` token whose decoded JWT has `idToken: null` and authentication silently fails. The v6 official sample omits it because its flow only renders the payment component without identity lookup — don't model your code on that omission if you call `identity.*`.

### 2. SDK init and component flow

```javascript
const sdkInstance = await window.paypal.createInstance({
  clientToken,
  components: ["fastlane"],
  pageType: "product-details",
});

const fastlane = await sdkInstance.createFastlane();
fastlane.setLocale("en_us");

// Render watermark
const watermark = await fastlane.FastlaneWatermarkComponent({ includeAdditionalInfo: true });
watermark.render("#watermark-container");

// Email lookup → authentication → member or guest experience
const { customerContextId } = await fastlane.identity.lookupCustomerByEmail(email);

if (customerContextId) {
  const { authenticationState, profileData } =
    await fastlane.identity.triggerAuthenticationFlow(customerContextId);

  if (authenticationState === "succeeded") {
    // Member: show saved shipping, render payment component
    const { selectedAddress, selectionChanged } =
      await fastlane.profile.showShippingAddressSelector();
    // Also available: await fastlane.profile.showCardSelector() — opens UI to switch the saved card

    const paymentComponent = await fastlane.FastlanePaymentComponent({
      shippingAddress: profileData.shippingAddress,
    });
    paymentComponent.render("#payment-container");

    // On submit: get single-use token → create order server-side
    const { id: singleUseToken } = await paymentComponent.getPaymentToken();
    await createOrderWithToken(singleUseToken);
  }
} else {
  // Guest: render card entry component
  const paymentComponent = await fastlane.FastlanePaymentComponent({});
  paymentComponent.render("#card-container");
  const { id: singleUseToken } = await paymentComponent.getPaymentToken();
  await createOrderWithToken(singleUseToken);
}
```

### 3. Create order (server-side, REST)

The single-use token goes under `payment_source.card.single_use_token`. Use a standard `client_credentials` access token for this call — NOT the client token from step 1.

```javascript
const payload = {
  intent: "CAPTURE",
  purchase_units: [{
    amount: {
      currency_code: "USD",
      value: "10.00",
      breakdown: { item_total: { currency_code: "USD", value: "10.00" } },
    },
    items: [{
      name: "Sample item",
      quantity: "1",
      category: "PHYSICAL_GOODS",
      unit_amount: { currency_code: "USD", value: "10.00" },
    }],
    soft_descriptor: "MYBIZ",
  }],
  payment_source: {
    card: {
      single_use_token: singleUseToken,
      experience_context: {
        brand_name: "My Store",
        shipping_preference: "GET_FROM_FILE",
        user_action: "PAY_NOW",
        payment_method_preference: "IMMEDIATE_PAYMENT_REQUIRED",
      },
    },
  },
};

await fetch(`${PAYPAL_API_BASE_URL}/v2/checkout/orders`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${accessToken}`,
    "PayPal-Request-Id": crypto.randomUUID(),
  },
  body: JSON.stringify(payload),
});
```

> Using `@paypal/paypal-server-sdk` instead? The field becomes `paymentSource.card.singleUseToken` (camelCase). Don't mix shapes in the same payload.

**Key APIs:** `fastlane.identity.lookupCustomerByEmail(email)`, `fastlane.identity.triggerAuthenticationFlow(contextId)`, `fastlane.profile.showShippingAddressSelector()`, `fastlane.FastlanePaymentComponent({ shippingAddress? })`, `component.getPaymentToken()` returns `{ id }` (single-use token, valid 3 hours).

### Troubleshooting

| Symptom | Cause |
|---|---|
| `ERR_INVALID_CLIENT_TOKEN` or `"invalid authorization token"` on `createInstance` | Token expired (3h TTL), wrong endpoint used, `domains[]` doesn't include the current origin, OR you read `data.client_token` from the OAuth response (it's `data.access_token`). |
| `invalid_domain` from the OAuth token request | Passed `"domains[]": "localhost"` / IP / unregistered hostname. Omit `domains[]` for sandbox; use registered origins only in production. |
| Decoded client token has `idToken: null` | Request was missing `response_type=client_token&intent=sdk_init` — got a plain access token, not a Fastlane-capable client token. |
| Order create returns `INVALID_PARAMETER_VALUE` on `payment_source` | Wrong shape. Verify `payment_source.card.single_use_token` (REST) or `paymentSource.card.singleUseToken` (server SDK). |
| Fastlane component never renders, origin error in console | Browser origin not in the `domains[]` list at client-token generation time. |
<!-- ─── End Fastlane section ─── -->

## Redirect Flow and Session Resumption

For redirect-based flows (mobile, WebView), use `hasReturned()` and `resume()` on page load:

```javascript
const session = sdkInstance.createPayPalOneTimePaymentSession(callbacks);

if (session.hasReturned()) {
  await session.resume();
} else {
  setupPayPalButton(session);
}
```

## Browser Compatibility

| Browser | Minimum Version |
|---------|----------------|
| Chrome | 69 |
| Safari | 12 |
| Firefox | 63 |
| Samsung Internet | 10 |
| Edge | 79 |

Check at runtime:

```javascript
if (window.isBrowserSupportedByPayPal()) {
  // safe to initialize
}
```

## Content Security Policy (CSP)

If your site uses CSP headers, add the following to avoid blocked scripts and iframes:

- `script-src`: `https://*.paypal.com https://*.paypalobjects.com`
- `frame-src`: `https://*.paypal.com`
- `connect-src`: `https://*.paypal.com`
- `img-src`: `https://*.paypal.com https://*.paypalobjects.com`

Omitting these is a common production blocker — the SDK loads and renders from PayPal-hosted domains.

## Common Error Codes

| Code | Meaning | Action |
|------|---------|--------|
| `ERR_INVALID_CLIENT_TOKEN` | Token expired or invalid | Regenerate server-side and reinitialize |
| `ERR_DOMAIN_MISMATCH` | Domain not in token's domain list | Check `domains[]` in token request |
| `ERR_DEV_UNABLE_TO_OPEN_POPUP` | Popup blocked | Fall back to `modal` or `redirect` |
| `INSTRUMENT_DECLINED` | Payment method declined | Ask buyer for a different method |
| `NETWORK_ERROR` | Network failure | Retry with backoff |

## Migration from v5

The [PayPal Upgrade Hub](https://developer.paypal.com/upgrade/ec/guide/Web%20SDK%20v6/) provides a step-by-step migration guide. Key steps:

1. Replace the `sdk/js?client-id=X` script tag with `web-sdk/v6/core`
2. Add a server endpoint returning a browser-safe client token (if using vaulting/Fastlane)
3. Replace `paypal.Buttons({ ... }).render()` with `createInstance()` + web components
4. Move `createOrder` / `onApprove` callbacks into payment session constructors
5. Return `{ orderId }` objects instead of bare orderId strings
6. Add explicit `findEligibleMethods()` calls before rendering buttons
7. Replace Hosted Fields with `createCardFieldsComponent()`

## React + v6

`@paypal/react-paypal-js` **does support v6**. The monorepo includes an active v6 Storybook (`packages/react-paypal-js-storybook/v6`). Do not warn users that React v6 support is unavailable — it is available. React v6 patterns will be added to RulesHub — refer there for authoritative code examples.

## Sample Integration

Official v6 sample repository with JavaScript, TypeScript, and React examples:
[github.com/paypal-examples/v6-web-sdk-sample-integration](https://github.com/paypal-examples/v6-web-sdk-sample-integration)

## Best Practices

1. Never expose client secrets in frontend code — use client ID or server-generated client tokens
2. Always check `findEligibleMethods()` before rendering buttons — do not assume availability
3. Use `presentationMode: "auto"` for maximum compatibility across browsers and devices
4. Create and capture orders server-side — never trust client-supplied amounts
5. Include `PayPal-Request-Id` (idempotency key) on all server-side POST requests
6. Handle `INSTRUMENT_DECLINED` by prompting for a different payment method — do not auto-retry
7. Log `debug_id` from all error responses — required for PayPal support escalation
8. Use `async` on the script tag to avoid blocking page rendering
9. For card fields, ensure containers have defined `height` and `width` before mounting
10. Cache access tokens server-side — do not generate per request

## Live Documentation
- [v6 Setup Guide](https://developer.paypal.com/sdk/js/set-up.md)
- [v6 API Reference](https://developer.paypal.com/sdk/js/reference.md)
- [v6 Card Fields One-Time Checkout](https://developer.paypal.com/expanded/card-fields.md)
- [v5 to v6 Upgrade Hub](https://developer.paypal.com/upgrade/ec/guide/Web%20SDK%20v6/)
- [v6 Sample Integration (GitHub)](https://github.com/paypal-examples/v6-web-sdk-sample-integration)
- [Save Cards with v6](https://developer.paypal.com/platforms/checkout/save-payment-methods/during-purchase/js-sdk/cards.md)
