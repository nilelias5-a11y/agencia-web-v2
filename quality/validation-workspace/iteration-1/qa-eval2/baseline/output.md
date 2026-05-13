# QA Report — Café Vermut Checkout Flow

**Date:** 2026-05-13
**Reviewed by:** quality-reviewer
**Project:** Café Vermut — specialty coffee e-commerce, Spain
**Stack:** Next.js 16 + Stripe (hosted Checkout, test mode) + Supabase + Sendcloud + Resend
**Flow tested:** Product page → Cart → Shipping details form → Stripe Checkout → Confirmation page
**Pages tested:** 4 (product detail, cart, shipping form, confirmation)

---

## Summary

| Severity | Count | Blocking delivery? |
|----------|-------|--------------------|
| CRITICAL | 3 | Yes |
| HIGH | 4 | Yes |
| MEDIUM | 5 | No |
| LOW | 3 | No |

**Overall verdict:** BLOCKED — fix all CRITICAL and HIGH issues before client tests tomorrow.

---

## Bug Log

### CRITICAL Issues

---

**BUG-001 — Confirmation page crashes on expired Stripe session**

- **Page/component:** `/confirmation?session_id=...`
- **Description:** When `session_id` in the URL belongs to an expired Stripe Checkout Session (sessions expire after 24 hours by default), the confirmation page throws a `TypeError` instead of rendering a graceful fallback. This crashes the entire page and the user sees a Next.js error screen.
- **Steps to reproduce:**
  1. Complete a Stripe test payment and note the `session_id` from the redirect URL.
  2. Wait 24 hours (or manually expire the session via the Stripe Dashboard or test API).
  3. Navigate to `/confirmation?session_id=<expired_id>`.
  4. Observe: `TypeError` crash.
- **Root cause:** The code reads properties directly from the Stripe session object without null-checking. If `stripe.checkout.sessions.retrieve()` returns a session with `payment_status: null` or if the call throws an error, any property access like `session.customer_details.email` will throw.
- **Expected behavior:** Page renders a message such as "We could not load your order. It may have expired. Check your email for confirmation or contact us."
- **Actual behavior:** `TypeError: Cannot read properties of null` — full page crash, no recovery.
- **Also triggered by:** Navigating to the confirmation page with a malformed or missing `session_id` query param (e.g., user bookmarks the URL and revisits it, bot crawlers).
- **Affected browsers/devices:** All — server-side crash.
- **Fix:** Wrap `stripe.checkout.sessions.retrieve()` in a try/catch. Check `if (!session || session.payment_status !== 'paid')` and render an appropriate error UI instead of crashing. Never read nested properties like `session.customer_details.email` without optional chaining (`session?.customer_details?.email`).
- **Assigned to:** Developer.

---

**BUG-002 — Cart is wiped on Stripe cancel / back navigation**

- **Page/component:** Cart → Stripe Checkout → `cancel_url` redirect
- **Description:** When a user enters Stripe Checkout but cancels or clicks the browser back button, they are returned to `cancel_url`. At this point, the cart is empty. The user loses their entire selection and must start from the product page.
- **Steps to reproduce:**
  1. Add one or more products to the cart.
  2. Proceed to Stripe Checkout.
  3. Click "Go back" or close the Stripe-hosted page.
  4. Observe: cart is now empty.
- **Root cause:** The cart is being cleared at the moment the user is redirected to Stripe (likely in the same server action or API route that creates the Checkout Session), rather than being cleared only after a confirmed `checkout.session.completed` webhook event.
- **Expected behavior:** Cart is preserved when the user returns from a cancelled payment. Cart is only cleared after a successful webhook confirms the order.
- **Actual behavior:** Cart is wiped on redirect to `cancel_url`.
- **Business impact:** High. A user who had an SCA 3D Secure challenge fail (very common in Spain — see SCA section below) or simply changed their mind will be forced to re-add all products. Most will abandon.
- **Affected browsers/devices:** All.
- **Fix:** Do not clear the cart in the route that creates the Checkout Session. Clear it only inside the `checkout.session.completed` webhook handler, keyed by `session.id` or `session.metadata.cartId`, after the order is persisted to Supabase.
- **Assigned to:** Developer.

---

**BUG-003 — Stripe webhook not tested end-to-end: order may not save to Supabase**

- **Page/component:** Stripe webhook endpoint (e.g., `/api/webhooks/stripe`)
- **Description:** There is no confirmed evidence that the `checkout.session.completed` webhook fires correctly in the test environment, that the Supabase write succeeds, and that Resend dispatches the confirmation email. If the webhook endpoint is broken, the client's very first live test tomorrow will result in a payment taken but no order in the system and no email sent — the most damaging possible outcome.
- **This is CRITICAL because:** Even if the hosted Checkout flow looks perfect in the browser, the order pipeline runs entirely in the webhook. If it fails silently, the client will see a confirmation page (because that reads from the URL `session_id` directly) but the order will be invisible in the database and no email will be sent.
- **Steps to verify (see Webhook Testing section below).**
- **Affected scope:** Order creation, email dispatch, inventory updates, any post-payment logic.
- **Fix:** Run the full webhook test protocol described below before client testing. Confirm Supabase row created, Resend email delivered, no 5xx from the webhook endpoint.
- **Assigned to:** Developer.

---

### HIGH Issues

---

**BUG-004 — "Add to cart" duplicate on rapid clicks (no debounce)**

- **Page/component:** Product Detail Page — "Añadir al carrito" button
- **Description:** Clicking "Añadir al carrito" multiple times in quick succession adds multiple units of the same item even when the user intends to add one. The button has no loading/disabled state during the cart mutation.
- **Steps to reproduce:**
  1. Open any product detail page.
  2. Click "Añadir al carrito" 3–4 times rapidly.
  3. Open the cart. Observe: 3–4 units added instead of 1.
- **Expected behavior:** After the first click, the button enters a disabled or loading state until the cart operation completes. Subsequent clicks are ignored.
- **Actual behavior:** Each click adds a separate unit; no visual feedback during mutation.
- **Affected browsers/devices:** All, most pronounced on desktop where double-clicking is habitual.
- **Fix options (pick one):**
  - Set `disabled={isLoading}` on the button while the cart server action / API call is in flight. Use `useTransition` (React) or a local `isPending` state.
  - Add a debounce of 300–500ms on the click handler.
  - Prefer the `disabled` approach — it also gives the user visual feedback.
- **Severity justification:** HIGH not CRITICAL because a duplicate can be removed from the cart, but a client demo will almost certainly hit this.
- **Assigned to:** Developer.

---

**BUG-005 — Spanish postal code validation missing**

- **Page/component:** Shipping details form — `código postal` field
- **Description:** The postal code field accepts any input. Invalid values (letters, wrong length, out-of-range prefixes) pass validation and reach the Stripe Checkout session creation, where Stripe may reject or silently misroute the address.
- **Spanish postal code rules:**
  - Exactly 5 digits.
  - First two digits = province code: valid range 01–52 (Álava to Melilla, including Ceuta 51 and Melilla 52).
  - Invalid examples that currently pass: `00000`, `53000`, `1234`, `ABCDE`, `99999`.
- **Steps to reproduce:**
  1. Navigate to the shipping form.
  2. Enter `00000` in the postal code field and submit.
  3. Observe: no validation error, form proceeds.
- **Expected behavior:** Field shows inline error "Introduce un código postal válido (5 dígitos, provincias 01–52)" if the value does not match `/^(0[1-9]|[1-4]\d|5[0-2])\d{3}$/`.
- **Actual behavior:** Invalid postal codes are accepted silently.
- **Impact:** Bad address data reaches Sendcloud, which can cause shipping label generation failures or delivery to wrong region.
- **Fix:**
  ```
  regex: /^(0[1-9]|[1-4]\d|5[0-2])\d{3}$/
  error message: "Código postal no válido. Debe tener 5 dígitos (01000–52999)."
  ```
  Validate both client-side (inline, on blur) and server-side (before creating the Stripe session).
- **Assigned to:** Developer.

---

**BUG-006 — No duplicate order guard in webhook handler**

- **Page/component:** `/api/webhooks/stripe` — `checkout.session.completed` handler
- **Description:** Stripe webhooks are delivered at-least-once, not exactly-once. Under network failure, Stripe will retry the same event multiple times. If the webhook handler does not check whether an order for `session.id` already exists before inserting into Supabase, the same order will be created multiple times and Resend will send duplicate confirmation emails.
- **Steps to reproduce:**
  1. Simulate a webhook with `stripe trigger checkout.session.completed`.
  2. Trigger it a second time with the same event ID.
  3. Query Supabase orders table: duplicate rows for the same `session_id`.
- **Expected behavior:** Handler is idempotent. If an order with `stripe_session_id = session.id` already exists, skip insertion and return 200 immediately.
- **Actual behavior:** Likely inserts a duplicate order and sends a duplicate email (unconfirmed until webhook test is run, see BUG-003).
- **Fix:** Add an idempotency check before inserting:
  ```sql
  SELECT id FROM orders WHERE stripe_session_id = $1
  ```
  If a row is found, return 200 without re-processing. Alternatively, add a UNIQUE constraint on `stripe_session_id` in Supabase and handle the conflict gracefully.
- **Assigned to:** Developer.

---

**BUG-007 — Confirmation page does not handle missing `session_id` param**

- **Page/component:** `/confirmation`
- **Description:** If the user navigates to `/confirmation` without a `session_id` query param (direct URL visit, bot crawl, stale bookmark), the page crashes or shows a blank state rather than a meaningful message. This is distinct from BUG-001 (expired session) — here there is no session ID at all.
- **Steps to reproduce:**
  1. Navigate to `/confirmation` with no query params.
  2. Observe: crash or blank page.
- **Expected behavior:** Page renders "No se encontró información de tu pedido. Si realizaste una compra, revisa tu email." with a link back to the homepage.
- **Fix:** Check `if (!searchParams.session_id)` at the top of the page component and return the error UI early, before any Stripe API call is attempted.
- **Assigned to:** Developer.

---

### MEDIUM Issues

---

**BUG-008 — Phone field accepts international formats inconsistently**

- **Page/component:** Shipping form — `teléfono` field
- **Description:** Spanish mobile numbers are 9 digits starting with 6 or 7. Landlines start with 8 or 9. The field likely accepts any numeric string, including 4-digit values or numbers with country code prefix (`+34`) mixed with numbers without. Sendcloud may reject malformed phone numbers when creating shipments.
- **Recommendation:** Validate with `/^(\+34|0034)?[6-9]\d{8}$/` and show: "Introduce un número de teléfono español válido (9 dígitos)."
- **Severity:** MEDIUM — does not block payment but can break shipping label creation.

---

**BUG-009 — No loading state on shipping form submit button**

- **Page/component:** Shipping details form — submit/continue button
- **Description:** After clicking "Continuar al pago", there is a delay while the server creates the Stripe Checkout Session and redirects. During this time the button remains active, allowing double-submission. Similar to BUG-004 but on the form submit.
- **Recommendation:** Disable the submit button and show a spinner while the session creation request is in flight.
- **Severity:** MEDIUM — can result in duplicate Stripe Checkout Sessions being created.

---

**BUG-010 — Cart cookie/session not scoped to prevent cross-tab conflicts**

- **Page/component:** Cart (global state)
- **Description:** If a user opens two browser tabs, adds different products in each, and proceeds to checkout in both, the cart state may be inconsistent. This is a known issue in cookie-based or localStorage-based cart implementations without locking.
- **Recommendation:** Document this as a known limitation for v1. For v1.1, implement cart versioning or server-side cart with a unique `cartId` per session.
- **Severity:** MEDIUM — unlikely for a coffee shop, but can happen.

---

**BUG-011 — No stock/availability check before Stripe session creation**

- **Page/component:** Shipping form → Stripe session creation API route
- **Description:** If a product is out of stock (Supabase stock = 0) and the stock check only happens at "Add to Cart" time, a user who added a product when it was in stock could complete a payment for a product that is now unavailable. No guard exists at checkout time.
- **Recommendation:** Re-validate stock availability synchronously when creating the Stripe Checkout Session. If stock = 0, return an error to the form and prompt the user to update their cart.
- **Severity:** MEDIUM — not testable in the current Stripe test mode but must be verified before going live.

---

**BUG-012 — Resend email not verified for spam/deliverability in Spain**

- **Page/component:** Confirmation email (Resend)
- **Description:** The sending domain used by Resend must have SPF, DKIM, and DMARC DNS records configured. Without these, confirmation emails will land in spam or be rejected by major Spanish ISPs (Movistar, Vodafone). This is not a code bug but a configuration gap that will affect client perception on day one.
- **Recommendation:** Verify in Resend dashboard that the sending domain is fully verified (all DNS records green). Send a test email to a Gmail and a Hotmail/Outlook address and confirm delivery to inbox, not spam.
- **Severity:** MEDIUM — does not break the checkout flow but confirmation emails not arriving is a significant client concern.

---

### LOW Issues

---

**BUG-013 — "Volver al carrito" link on Stripe Checkout missing or broken**

- **Page/component:** Stripe hosted Checkout page
- **Description:** Stripe Checkout shows a "Back" link to return to the merchant site. The `cancel_url` must be set to the cart page, not the homepage or shipping form. Confirm the `cancel_url` is set to `/cart` so the flow makes sense to the user. Note: once BUG-002 is fixed, this becomes more important.
- **Severity:** LOW — cosmetic/UX, not functional.

---

**BUG-014 — Confirmation page title and meta description are likely generic**

- **Page/component:** `/confirmation` page `<head>`
- **Description:** The confirmation page likely has a generic `<title>` (e.g., "Café Vermut" or "Confirmación"). This page should not be indexable (add `<meta name="robots" content="noindex">`). If it is indexable, bots hitting it without a `session_id` compound BUG-007.
- **Recommendation:** Add `noindex` to the confirmation page.
- **Severity:** LOW.

---

**BUG-015 — No explicit error message when Stripe session creation fails**

- **Page/component:** Shipping form → API route
- **Description:** If the Stripe API returns an error during Checkout Session creation (rate limit, invalid price ID, etc.), the user likely sees a generic 500 or a silent failure. There is no user-facing error message with instructions.
- **Recommendation:** Catch errors from `stripe.checkout.sessions.create()` and return a user-facing message: "No pudimos procesar tu pago en este momento. Inténtalo de nuevo o contáctanos."
- **Severity:** LOW — rare scenario but poor experience when it occurs.

---

## Stripe Test Cards — Spain / SCA

Stripe test mode. All test cards use any future expiry date and any 3-digit CVC.

### Standard success flows

| Scenario | Card number | Expected result |
|----------|-------------|-----------------|
| Instant payment success | `4242 4242 4242 4242` | Payment succeeds, webhook fires, confirmation page loads |
| Debit card success (common in Spain) | `4000 0566 5566 5556` | Payment succeeds |
| Visa success | `4000 0072 4000 0007` | Payment succeeds (Spain BIN) |

### SCA / 3D Secure flows (critical for Spain)

Spain is under PSD2 SCA mandate. Most consumer cards from Spanish banks (BBVA, Santander, CaixaBank, Sabadell) will trigger 3DS. These are the most important cases to test.

| Scenario | Card number | Expected result |
|----------|-------------|-----------------|
| 3DS required — authentication succeeds | `4000 0027 6000 3184` | User completes challenge in modal, payment succeeds |
| 3DS required — user fails authentication | `4000 0084 2000 1629` | Challenge shown, user cancels/fails, returned to `cancel_url` with cart (after BUG-002 fix) |
| 3DS — authentication unavailable (issuer down) | `4000 0008 4000 1629` | Stripe declines, user returned to `cancel_url` |
| 3DS required — authentication not attempted | `4000 0025 2000 3155` | Simulates card that requires 3DS but 3DS not initiated |

### Decline flows

| Scenario | Card number | Expected result |
|----------|-------------|-----------------|
| Generic card decline | `4000 0000 0000 0002` | Stripe shows decline error, user can retry |
| Insufficient funds | `4000 0000 0000 9995` | Stripe shows "funds insuficientes" equivalent |
| Expired card | `4000 0000 0000 0069` | Stripe shows expired card error |
| CVC check fail | `4000 0000 0000 0101` | Stripe shows CVC error |
| Card declined — fraud | `4100 0000 0000 0019` | Stripe declines, no retry path |
| Do not honor | `4000 0000 0000 0119` | Generic bank decline |

### Payment method — Bizum / SEPA (if applicable)

If Bizum or SEPA Direct Debit is enabled in the Stripe Dashboard for this account, test them separately. Bizum in Stripe test mode uses test phone numbers documented in the Stripe docs. SEPA test IBAN: `DE89370400440532013000`.

### Key test scenario to run before client demo

Run all four of these in sequence and confirm the full pipeline works:

1. `4242 4242 4242 4242` — Happy path: payment → webhook → Supabase order → Resend email → confirmation page.
2. `4000 0027 6000 3184` — SCA success: 3DS challenge completes, same pipeline as above.
3. `4000 0084 2000 1629` — SCA failure: user is returned to cart, cart is NOT empty (this will fail until BUG-002 is fixed).
4. `4000 0000 0000 0002` — Decline: Stripe shows error, user can update card and retry without losing cart.

---

## Webhook Testing Protocol

Stripe webhooks do not fire in the browser. You must test them separately.

### Step 1 — Install and authenticate Stripe CLI

```bash
stripe login
```

### Step 2 — Forward webhooks to local dev server

```bash
stripe listen --forward-to localhost:3000/api/webhooks/stripe
```

This prints a webhook signing secret (starts with `whsec_`). Confirm this matches the value in your `.env.local` as `STRIPE_WEBHOOK_SECRET`. If it does not match, webhook signature verification will fail and your handler will reject all events.

### Step 3 — Trigger a test `checkout.session.completed` event

```bash
stripe trigger checkout.session.completed
```

Observe:
- Your terminal running the dev server should log the incoming webhook.
- The handler should return HTTP 200 within 30 seconds (Stripe's timeout).
- A new row should appear in the Supabase `orders` table.
- Resend should dispatch the confirmation email (check Resend logs/dashboard).

### Step 4 — Test idempotency (BUG-006)

Trigger the same event twice with the same event ID:

```bash
stripe trigger checkout.session.completed
stripe trigger checkout.session.completed
```

Query Supabase: confirm only one order row exists for the `stripe_session_id`.

### Step 5 — Test webhook failure recovery

Stop your local server mid-request (simulate a crash). Observe that Stripe retries the webhook. Confirm that when the server comes back up and processes the retry, the idempotency check prevents a duplicate order.

### Step 6 — Test with a real completed Checkout Session

1. Complete a full checkout in the browser using card `4242 4242 4242 4242`.
2. Note the `session_id` from the confirmation URL.
3. Verify in Supabase: `SELECT * FROM orders WHERE stripe_session_id = '<session_id>'` returns exactly one row.
4. Verify in Resend: email was sent to the address entered in the shipping form.

### Step 7 — Verify webhook endpoint is registered in Stripe Dashboard

Before going live (even for client testing), confirm:
- Dashboard → Developers → Webhooks → Endpoint for this project exists.
- Event `checkout.session.completed` is listed.
- Endpoint URL matches the deployed URL (not `localhost`).
- For client testing tomorrow (if on a preview deployment): the webhook endpoint must point to the preview URL, not production and not localhost.

---

## Edge Cases for the Payment Flow

### User behaviour edge cases

| Scenario | Expected | Risk |
|----------|----------|------|
| User opens checkout in two tabs simultaneously | Each tab creates its own Stripe session; only the first payment should create an order (idempotency via BUG-006 fix handles this) | Duplicate orders if not idempotent |
| User hits browser back from Stripe Checkout | Returns to `cancel_url`; cart must be intact (BUG-002) | Cart loss |
| User refreshes confirmation page | Page re-fetches session from Stripe; must be idempotent (do not re-send email on page load) | Duplicate emails |
| User bookmarks and revisits confirmation page next day | Session expired → must show graceful error (BUG-001) | Crash |
| User pays but closes browser before redirect | Payment succeeded in Stripe, but user never reaches confirmation page. Webhook must have already fired and created the order. Confirmation email is the source of truth. | Silent success — no crash, but user may call support thinking payment failed |
| User submits shipping form with script injection in address fields | All fields must be sanitised before writing to Supabase | XSS / SQL injection |
| Product added to cart, then sold out before checkout | No stock re-validation at session creation (BUG-011 MEDIUM) | Oversell |
| Stripe session creation succeeds but redirect fails (flaky network) | User is stuck on shipping form with no feedback; retry must be possible | Dead end |
| SCA challenge times out | Stripe handles this; user is returned to `cancel_url`. Cart must be intact. | Cart loss (BUG-002) |

### Data edge cases

| Field | Edge case to test |
|-------|------------------|
| Email | `test+alias@domain.es`, very long email, no TLD |
| Nombre / Apellidos | Accented characters (José, Martínez), double surnames (García López), all-caps |
| Dirección | Very long street name (100+ chars), apartment number with slash (`3º 2ª`) |
| Código postal | `00000` (invalid), `01000` (valid, Álava), `52999` (valid, Melilla), `53000` (invalid), `1234` (too short) |
| Teléfono | `600000000` (valid mobile), `912345678` (valid landline), `+34600000000` (valid with prefix), `123` (invalid) |
| Cart quantity | 0 items (should not be able to reach shipping form), 99 items, fractional quantity |

---

## What Is Blocking Delivery vs What Can Ship with a Bug

### BLOCKING — Must fix before client tests tomorrow

| Bug | Why it blocks |
|-----|--------------|
| BUG-001 — Confirmation page crashes on expired session | Client will click a confirmation link, it will crash. Guaranteed to be seen. |
| BUG-002 — Cart wiped on cancel/back | Client will test the cancel flow. Losing the cart is the most visible UX failure possible. |
| BUG-003 — Webhook pipeline not verified | Without this, a payment can succeed with no order in Supabase and no email. This is the worst possible outcome for a demo. |
| BUG-004 — Add to cart no debounce | Client will double-click during demo. Duplicates in cart will look unprofessional. |
| BUG-005 — Spanish postal code validation | Client will enter a real address; invalid codes must be caught. Sendcloud will likely reject bad codes silently. |
| BUG-006 — No idempotency in webhook | If Stripe retries during client testing and a duplicate order appears, trust in the system is destroyed. |
| BUG-007 — Confirmation page crashes without session_id | Direct URL visit (which the client will likely do to share with their team) will crash the page. |

### CAN SHIP with documented bug (fix in v1.1)

| Bug | Why it can wait |
|-----|----------------|
| BUG-008 — Phone validation | Sendcloud accepts some non-standard formats; not blocking payment. |
| BUG-009 — No loading state on form submit | Double-submission is unlikely in a demo context; not a crash. |
| BUG-010 — Cross-tab cart conflict | Coffee shop use case makes this extremely unlikely. |
| BUG-011 — No stock re-check at checkout | Only a risk if stock changes between add-to-cart and checkout; low probability for demo. |
| BUG-012 — Resend domain deliverability | Test email deliverability now; if DNS is already configured this is a non-issue. |
| BUG-013 — cancel_url destination | Minor UX. |
| BUG-014 — Confirmation page not noindexed | SEO concern, not functional. |
| BUG-015 — No error on Stripe session creation failure | Rare scenario; not expected during demo. |

---

## Recommended Pre-Demo Checklist (Tomorrow Morning)

- [ ] BUG-001 fixed and tested: visit `/confirmation` with expired session_id → graceful error shown
- [ ] BUG-002 fixed and tested: cancel Stripe Checkout → return to cart with items intact
- [ ] BUG-003 verified: run `stripe trigger checkout.session.completed` → Supabase row created, Resend email received
- [ ] BUG-004 fixed and tested: rapid-click "Añadir al carrito" → only 1 unit added
- [ ] BUG-005 fixed and tested: enter `00000` → validation error shown; enter `08001` → proceeds
- [ ] BUG-006 fixed and tested: trigger webhook twice → only 1 Supabase order row
- [ ] BUG-007 fixed and tested: visit `/confirmation` with no params → graceful message shown
- [ ] Full happy-path test with `4242 4242 4242 4242`: product → cart → form → Stripe → confirmation → email received
- [ ] SCA test with `4000 0027 6000 3184`: 3DS challenge appears, completes, order created
- [ ] Cancel test with `4000 0084 2000 1629`: cancelled 3DS returns to cart with items intact
- [ ] Stripe webhook endpoint registered in Dashboard pointing to preview deployment URL
- [ ] Resend sending domain DNS verified (check Resend dashboard)

---

*Report generated by quality-reviewer — Café Vermut v1 pre-client QA, 2026-05-13*
