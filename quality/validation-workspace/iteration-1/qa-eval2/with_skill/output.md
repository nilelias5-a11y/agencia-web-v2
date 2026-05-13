## QA Report — Café Vermut

**Date:** 2026-05-13
**Reviewed by:** quality-reviewer
**Build version / commit:** Pre-client-testing build (Stripe test mode)
**Pages tested:** 5 (Product Page, Cart, Shipping Details Form, Stripe Checkout, Confirmation Page)

---

### Summary

| Severity | Count | Blocking delivery? |
|----------|-------|-------------------|
| 🔴 CRITICAL | 3 | Yes |
| 🟠 HIGH | 3 | Yes |
| 🟡 MEDIUM | 3 | No |
| 🟢 LOW | 2 | No |

**Overall verdict:** 🚫 Blocked — fix criticals first

---

### Bug Log

---

#### 🔴 CRITICAL Issues

---

**BUG-001**
- **Page/component:** Confirmation Page (`/confirmation?session_id=...`)
- **Description:** Page throws an uncaught `TypeError` when the Stripe Checkout Session referenced in the URL `session_id` param has expired (Stripe expires sessions after 24 hours). The component attempts to access properties on a `null` or `undefined` session object without a null guard, causing a full render crash instead of a graceful error state.
- **Steps to reproduce:**
  1. Complete a checkout flow and obtain a `session_id`.
  2. Wait 24 hours (or use a manually expired/invalid session_id in the URL).
  3. Navigate to `/confirmation?session_id=cs_test_expired123`.
  4. Observe page crash.
- **Expected behavior:** Page displays a user-friendly message such as "Tu sesión de pedido ha expirado. Consulta tu email de confirmación o contacta con soporte." Order lookup should fall back to the Supabase `orders` table using the session's associated email if the session itself is gone.
- **Actual behavior:** `TypeError: Cannot read properties of null (reading 'customer_details')` (or similar) — full page crash, white screen or unhandled error boundary.
- **Affected browsers/devices:** All browsers and devices — this is a server-side/data issue, not a rendering issue.
- **Fix recommendation:**
  - Wrap the Stripe session fetch in a `try/catch`. If the session is expired or not found (`null`), render a fallback UI instead of crashing.
  - Example guard: `if (!session) return <OrderExpiredMessage />`
  - Optionally, use the `session_id` to query the Supabase `orders` table as a fallback (the webhook should have persisted the order before session expiry).
- **Assigned to:** Developer (confirmation page component)

---

**BUG-002**
- **Page/component:** Cart (`/cart`) — post-payment-failure state
- **Description:** When Stripe returns the user to the `cancel_url` after a failed or abandoned payment, the cart is completely emptied. The user loses all their selected products and must start the shopping flow from scratch. This is a data-loss bug from the user's perspective.
- **Steps to reproduce:**
  1. Add one or more products to the cart.
  2. Proceed to checkout and reach the Stripe hosted payment page.
  3. Use the Stripe test card `4000 0000 0000 0002` (generic decline) or click "Back" / close the Stripe page.
  4. Observe the `cancel_url` destination (cart or homepage).
  5. Check cart contents — the cart is empty.
- **Expected behavior:** On cancellation or payment failure, the cart is preserved exactly as it was before the user was redirected to Stripe. The user is returned to the cart (or checkout summary) with their items intact and a clear message: "El pago no se ha completado. Tus productos siguen en el carrito."
- **Actual behavior:** Cart is emptied on redirect to `cancel_url`. User must re-add all products.
- **Affected browsers/devices:** All browsers/devices — logic bug in cart state management (likely cart cleared too early, before payment confirmation rather than after).
- **Fix recommendation:**
  - Do NOT clear the cart at the point of Stripe session creation. Only clear the cart after a confirmed successful webhook event (`checkout.session.completed`).
  - If using cookies or localStorage for cart state, ensure the cart data is not wiped during the redirect.
  - The `cancel_url` handler should restore the cart from the persisted pre-checkout snapshot, or simply never clear it until order confirmation is received.
- **Assigned to:** Developer (cart state management / Stripe redirect logic)

---

**BUG-003**
- **Page/component:** Webhook handler (`/api/webhooks/stripe`) + Confirmation Page
- **Description:** The webhook flow has a race condition: the confirmation page may attempt to read order data from Supabase before the webhook has fired and written the order. If the user is redirected to the `success_url` faster than Stripe delivers the webhook (which can take several seconds), Supabase returns no order and the confirmation page either crashes or shows empty order details.
- **Steps to reproduce:**
  1. Use Stripe test card `4242 4242 4242 4242` for a successful payment.
  2. Monitor the confirmation page load time vs. webhook delivery time using the Stripe Dashboard event log.
  3. In slow network conditions or under Stripe webhook retry delays, observe the confirmation page loading before the order exists in Supabase.
- **Expected behavior:** The confirmation page either (a) waits for order data to be available with a loading state and polls/retries for a short window, or (b) reads order details directly from the Stripe session object rather than exclusively from Supabase. The user always sees complete order details.
- **Actual behavior:** The confirmation page may show an empty state, missing order details, or crash — depending on the timing of the webhook vs. the user's page load.
- **Affected browsers/devices:** All — this is a timing/architecture issue, not browser-specific.
- **Fix recommendation:**
  - The confirmation page should derive primary order data from the Stripe session (`/api/stripe/session?id=session_id`) rather than waiting for Supabase. The webhook write to Supabase is for persistence, not for powering the confirmation UI.
  - Alternatively, implement a short polling loop on the confirmation page (retry Supabase query every 2 seconds, up to 10 seconds) before showing a fallback.
  - Ensure the `success_url` confirmation page is resilient to a not-yet-written order.
- **Assigned to:** Developer (confirmation page + webhook architecture)

---

#### 🟠 HIGH Issues

---

**BUG-004**
- **Page/component:** Product Page (`/productos/[slug]`) — "Add to Cart" button
- **Description:** The "Añadir al carrito" button has no debounce or disable-after-click protection. Rapid successive clicks add the same product multiple times to the cart, resulting in inflated quantities the user did not intend.
- **Steps to reproduce:**
  1. Navigate to any product detail page.
  2. Click "Añadir al carrito" 5 times rapidly (e.g., double-click or triple-click quickly).
  3. Open the cart.
  4. Observe the product appears with quantity 5 (or however many clicks were registered).
- **Expected behavior:** Either (a) each click increments quantity by 1 with a visible quantity control the user can adjust, OR (b) the button is disabled for 500–1000ms after the first click to prevent duplicate additions while the cart state updates. The user sees immediate feedback that the item was added (button text changes to "Añadido ✓" or a toast notification appears).
- **Actual behavior:** Multiple rapid clicks each trigger a separate `addToCart()` call, adding the product N times with no feedback.
- **Affected browsers/devices:** All browsers and devices. Particularly likely on mobile where accidental double-taps are common.
- **Fix recommendation:**
  - Add a `disabled` state to the button during the cart update operation: `setIsAdding(true)` on click, `setIsAdding(false)` after state update resolves.
  - Optionally add a 500ms debounce using `useCallback` + a ref flag.
  - Show a success state after click: change button label to "Añadido" for 1.5s before resetting.
- **Assigned to:** Developer (Product Page / AddToCart component)

---

**BUG-005**
- **Page/component:** Shipping Details Form (`/checkout/envio` or equivalent)
- **Description:** The "Código postal" field does not validate that the entered value is a valid Spanish postal code. Spanish postal codes must be exactly 5 digits and begin with a province prefix between 01 and 52. The form currently accepts invalid values (e.g., `00000`, `99999`, `1234`, `ABCDE`, `53000`) and passes them to Sendcloud, which will either silently accept them or fail at the Sendcloud API level with no user-facing error.
- **Steps to reproduce:**
  1. Navigate to the shipping details form.
  2. Enter `00000` in the "Código postal" field.
  3. Fill all other fields with valid data and submit.
  4. Observe: form submits without an error on the postal code field.
  5. Repeat with `99999`, `1234`, and `ABCDE`.
- **Expected behavior:** The form validates the postal code in real time (on blur) and on submit. Invalid codes show: "El código postal debe tener 5 dígitos y corresponder a una provincia española (01–52)." The form does not proceed until a valid code is entered.
- **Actual behavior:** Invalid postal codes pass client-side validation. Server-side Sendcloud errors (if any) are not surfaced to the user.
- **Affected browsers/devices:** All browsers/devices.
- **Fix recommendation:**
  - Add a regex validator: `/^(0[1-9]|[1-4][0-9]|5[0-2])\d{3}$/`
  - Implement both client-side (React Hook Form / Zod schema) and server-side validation.
  - Error message (in Spanish): "Código postal no válido. Debe ser un código postal español de 5 dígitos."
  - Also validate that the field is exactly 5 numeric characters before running the province check.
- **Assigned to:** Developer (Shipping Details Form / validation schema)

---

**BUG-006**
- **Page/component:** Stripe Checkout → Confirmation Page (3DS flow)
- **Description:** The 3D Secure authentication flow for cards requiring SCA (Strong Customer Authentication, mandatory for Spanish-issued cards under PSD2) has not been explicitly tested. If the `success_url` and `cancel_url` are not correctly configured for 3DS redirect handling, users with Spanish bank cards that require 3DS will either land on the wrong page after authentication or experience a broken post-authentication state.
- **Steps to reproduce:**
  1. Add a product to cart and proceed to Stripe Checkout.
  2. Enter test card `4000 0025 0000 3155` (3DS required — the correct Stripe test card for SCA/3DS challenge).
  3. Complete the 3DS challenge popup/redirect.
  4. Verify the redirect lands correctly on the `success_url` with a valid `session_id`.
  5. Verify the confirmation page loads correctly with order details.
- **Expected behavior:** After completing the 3DS challenge, Stripe redirects to `success_url?session_id=...`. The confirmation page loads with complete order details. The webhook has fired and the order is persisted in Supabase.
- **Actual behavior:** Unknown — this flow has not been verified. Risk of broken post-3DS redirect or missing `session_id` in the return URL.
- **Affected browsers/devices:** All browsers — particularly Mobile Safari (iOS) which handles 3DS redirects differently and is the most common browser for Spanish mobile users.
- **Fix recommendation:**
  - Run the full 3DS test flow before client testing using `4000 0025 0000 3155`.
  - Also test `4000 0027 6000 3184` (3DS2 with frictionless flow — should auto-authenticate without a challenge popup, but still exercises the SCA pathway).
  - Verify the `success_url` includes `{CHECKOUT_SESSION_ID}` as a template variable in the Stripe session creation call: `success_url: 'https://example.com/confirmation?session_id={CHECKOUT_SESSION_ID}'`
- **Assigned to:** Developer (Stripe session creation config + confirmation page)

---

#### 🟡 MEDIUM Issues

---

**BUG-007**
- **Page/component:** Shipping Details Form — field-level UX
- **Description:** The form fields for "Teléfono" do not enforce a Spanish phone number format. Spanish mobile numbers must start with 6 or 7 and be 9 digits total. Landlines start with 8 or 9. Currently any string is accepted. This leads to bad data in Sendcloud and potentially failed shipping label generation.
- **Steps to reproduce:**
  1. Enter `123` in the teléfono field and submit.
  2. Observe: no validation error.
- **Expected behavior:** Validation error shown for non-Spanish phone formats. Accept both `+34XXXXXXXXX` and `XXXXXXXXX` (9-digit) formats.
- **Actual behavior:** Any value accepted.
- **Fix recommendation:** Regex: `/^(\+34|0034)?[6-9]\d{8}$/`. Show error: "Introduce un teléfono español válido (ej: 612 345 678)."
- **Assigned to:** Developer

---

**BUG-008**
- **Page/component:** Cart → Checkout transition
- **Description:** There is no loading/processing state shown to the user between clicking "Proceder al pago" and the Stripe Checkout page appearing. The Stripe session creation API call can take 1–3 seconds. During this time the button appears unresponsive, creating a perception of a broken interaction. Users may click the button multiple times.
- **Steps to reproduce:**
  1. Add a product to cart.
  2. Click "Proceder al pago".
  3. Observe the button state during the 1–3 second API call to create the Stripe session.
- **Expected behavior:** Button shows a loading spinner or "Procesando..." text immediately on click, and is disabled to prevent duplicate calls.
- **Actual behavior:** Button appears static with no feedback until the Stripe redirect occurs.
- **Fix recommendation:** Set button to disabled with a loading indicator on click. Reset if the API call fails.
- **Assigned to:** Developer

---

**BUG-009**
- **Page/component:** Webhook handler (`/api/webhooks/stripe`) — Resend email
- **Description:** If the Resend confirmation email call fails (network error, Resend API key invalid, rate limit), there is likely no retry mechanism and no error logging. The order may be saved to Supabase correctly but the customer receives no confirmation email. This is silent from the user's perspective.
- **Steps to reproduce:**
  1. Temporarily set an invalid Resend API key in environment variables.
  2. Complete a checkout with `4242 4242 4242 4242`.
  3. Trigger the webhook manually via Stripe CLI or Dashboard.
  4. Observe: order saved to Supabase, but no confirmation email sent and no error surfaced.
- **Expected behavior:** Resend failures are caught, logged to the server console/monitoring, and ideally queued for retry. At minimum, the webhook returns a 200 to Stripe (so Stripe doesn't retry the entire webhook) but the email failure is logged.
- **Actual behavior:** Silent failure — email send errors likely uncaught or swallowed.
- **Fix recommendation:** Wrap the Resend call in a try/catch. Log failures. Consider a separate email retry queue (even a simple Supabase `email_queue` table with a cron job) for production.
- **Assigned to:** Developer

---

#### 🟢 LOW Issues

---

**BUG-010**
- **Page/component:** Cart
- **Description:** Cart item quantity displayed as an integer but no upper bound is enforced. A user could theoretically add 999 units of a single product. No maximum quantity validation exists client-side.
- **Recommendation:** Add a `max` constraint (e.g., 10 units per product) with a user-visible warning. Minor UX issue but low priority.

---

**BUG-011**
- **Page/component:** Confirmation Page
- **Description:** The page title (`<title>`) is likely generic ("Café Vermut" or "Confirmación") and does not include the order reference number. This makes it harder for users to bookmark or identify the tab.
- **Recommendation:** Set page title dynamically: "Pedido #XXXX confirmado — Café Vermut". Low SEO impact but good UX practice.

---

### Stripe Test Cards Reference

Use the following test cards for the full checkout flow QA:

| Scenario | Card Number | Expiry | CVC | Notes |
|---|---|---|---|---|
| Successful payment | `4242 4242 4242 4242` | Any future | Any 3 digits | Happy path — use for all baseline flow testing |
| Generic card decline | `4000 0000 0000 0002` | Any future | Any 3 digits | Tests BUG-002 (cart cleared on failure) |
| Insufficient funds | `4000 0000 0000 9995` | Any future | Any 3 digits | More realistic decline reason |
| 3DS required (SCA) | `4000 0025 0000 3155` | Any future | Any 3 digits | **Primary 3DS test for Spain** — shows challenge popup |
| 3DS2 frictionless | `4000 0027 6000 3184` | Any future | Any 3 digits | SCA auto-authenticated, no popup |
| 3DS — decline after auth | `4000 0084 0000 1629` | Any future | Any 3 digits | Tests 3DS flow where card declines after challenge |
| Expired card | `4000 0000 0000 0069` | Any future | Any 3 digits | Stripe-returned error, not webhook-triggered |
| Incorrect CVC | `4000 0000 0000 0127` | Any future | Any 3 digits | Tests CVC validation error state |

> **Note on Spain / PSD2:** The majority of Spanish-issued cards (BBVA, CaixaBank, Santander, ING España) trigger 3DS under PSD2. The card `4000 0025 0000 3155` is the correct Stripe test card for this scenario and must pass before client testing.

---

### Stripe Dashboard Verification Checklist

Before client testing, verify the following in the Stripe Dashboard (test mode):

**Payments tab:**
- [ ] Successful test payments show status `Succeeded` (not `Requires capture` or `Incomplete`)
- [ ] Payment intents created via Stripe Checkout show the correct amount in EUR
- [ ] Declined test payments show the correct decline reason code (e.g., `card_declined`, `insufficient_funds`)

**Webhooks tab:**
- [ ] Webhook endpoint URL is configured and active (`/api/webhooks/stripe`)
- [ ] `checkout.session.completed` event is listed in the subscribed events
- [ ] Recent webhook deliveries show HTTP 200 responses — no failed deliveries
- [ ] If any webhook shows a non-200 response, inspect the event payload and server logs
- [ ] Webhook signing secret (`STRIPE_WEBHOOK_SECRET`) matches the one in the environment — Stripe CLI (`stripe listen`) uses a different secret than the Dashboard endpoint; confirm the correct one is in production/test env

**Events tab:**
- [ ] `checkout.session.completed` events are present for all completed test payments
- [ ] Each event has a `customer_details` object with the test email and name
- [ ] `payment_intent.succeeded` events correspond 1:1 with `checkout.session.completed` events

**Checkout Sessions tab (Developers > Logs):**
- [ ] Sessions created with the correct `success_url` (must include `{CHECKOUT_SESSION_ID}` template variable)
- [ ] Sessions created with the correct `cancel_url` (must point to cart/checkout, not homepage)
- [ ] Session `locale` is set to `es` (Spanish) for proper Stripe Checkout localization
- [ ] Session `shipping_address_collection` is restricted to `ES` only (if using Stripe's built-in shipping form — verify Sendcloud integration is the sole address handler)

**Test mode confirmation:**
- [ ] The Stripe Dashboard banner shows "Test mode" — confirm no test payments have been made on a live key
- [ ] Environment variables `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` and `STRIPE_SECRET_KEY` both begin with `pk_test_` and `sk_test_` respectively

---

### Webhook Edge Case Test Scenarios

| Scenario | How to trigger | Expected result |
|---|---|---|
| Webhook fires before confirmation page loads | Use `stripe trigger checkout.session.completed` via Stripe CLI | Order in Supabase, confirmation page shows details |
| Confirmation page loads before webhook fires | Introduce artificial delay in webhook handler | Confirmation page must not crash — show loading state or derive data from Stripe session directly |
| Webhook fires twice (Stripe retry) | Manually resend a webhook event from Dashboard | Order must NOT be duplicated in Supabase — idempotency check on `session_id` required |
| Webhook with invalid signature | Tamper with `stripe-signature` header | Handler must return 400 and log the error — must NOT process the payload |
| Webhook for a `payment_intent.payment_failed` event | Use decline card, check if event is received | Confirm cart is NOT cleared on this event (only on `checkout.session.completed`) |
| Resend email failure during webhook | Temporarily break Resend API key | Webhook must return 200 to Stripe, failure must be logged, order must still be saved |

> **Idempotency check (critical):** The Supabase order insert must check if an order with the given `stripe_session_id` already exists before inserting. Without this, Stripe's automatic webhook retries (which occur if the endpoint returns non-200) will create duplicate orders.

---

### Passed Checks (Expected — Verify Before Sign-off)

- Navigation: ⬜ Not verified — test all product → cart → checkout routing links
- Forms (shipping): ❌ BUG-005 (postal code), BUG-007 (phone) — validation incomplete
- Responsive (all breakpoints): ⬜ Not verified — test at 320px, 375px, 768px, 1024px, 1280px, 1440px, 1920px
- Console errors: ⬜ Not verified — run through full flow with DevTools open; zero errors expected in production
- Accessibility baseline: ⬜ Not verified — confirm all form fields have `<label>` elements; check focus order through checkout form
- SEO signals: ⬜ Not verified — confirm each page has unique `<title>` and `<meta name="description">`
- Payment flow (happy path): ❌ BUG-001, BUG-002, BUG-003 — CRITICAL issues present
- Payment flow (3DS): ❌ BUG-006 — not tested
- Cart behavior: ❌ BUG-004 (debounce), BUG-002 (cart cleared on failure)
- Webhook flow: ❌ BUG-003 (race condition), BUG-009 (email failure handling)
- Confirmation page: ❌ BUG-001 (expired session crash), BUG-003 (race condition)

---

### Notes for Developer Before Client Testing

1. **Fix priority order:** BUG-001 (confirmation crash) → BUG-002 (cart cleared on failure) → BUG-003 (race condition) → BUG-004 (debounce) → BUG-005 (postal code validation) → BUG-006 (3DS test).
2. **Stripe CLI is required** for local webhook testing. Run `stripe listen --forward-to localhost:3000/api/webhooks/stripe` during development and use `stripe trigger checkout.session.completed` to test the webhook handler in isolation.
3. **The 3DS card (`4000 0025 0000 3155`) must be tested on Mobile Safari (iOS)** specifically, as the 3DS challenge popup behavior differs from desktop and has historically caused issues on iOS with Stripe Checkout redirects.
4. **Do not test with live Stripe keys** — confirm `pk_test_` and `sk_test_` prefixes on all configured keys before running any tests.
5. **Supabase RLS:** Confirm Row Level Security policies on the `orders` table allow the webhook service role to insert and the authenticated user to read their own orders only.
