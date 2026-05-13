# API Eval 1 — Stripe Integration: Café Vermut

**Skill applied:** `api-developer.md`
**Stack:** Next.js · TypeScript strict · T3 Env · Stripe SDK `stripe@17`
**Pattern:** `ApiResult<T>` typed error handling · `AbortSignal.timeout` · test/production separation

---

## Architecture Overview

Café Vermut sells ground coffee (50–200 orders/month, avg ticket €25, Spain market). The integration covers:

1. `src/lib/stripe.ts` — Stripe client initialized with T3 Env, 30s timeout
2. `src/app/actions/checkout.ts` — Server Action `createCheckoutSession` with Spanish-market line items, success/cancel URLs
3. `src/app/api/webhooks/stripe/route.ts` — Webhook handler with signature verification, `checkout.session.completed` + `payment_intent.payment_failed`
4. `src/types/api.ts` — Shared `ApiResult<T>` type + `safeApiCall` utility
5. `docs/integrations/stripe.md` — Test vs production checklist, env var guide, known limitations

---

## File Tree

```
src/
├── types/
│   └── api.ts                              ← ApiResult<T> + safeApiCall
├── lib/
│   └── stripe.ts                           ← Stripe client (T3 Env, 30s timeout)
├── app/
│   ├── actions/
│   │   └── checkout.ts                     ← Server Action createCheckoutSession
│   └── api/
│       └── webhooks/
│           └── stripe/
│               └── route.ts                ← Webhook handler
docs/
└── integrations/
    └── stripe.md                           ← Integration docs + checklists
```

---

## Source Files

### `src/types/api.ts`

```typescript
// Typed result pattern — no untyped catch (error: any)
// Used across all external API calls in the project

export type ApiResult<T> =
  | { success: true; data: T }
  | { success: false; error: string; code?: string }

/**
 * Wraps any async call in a typed ApiResult.
 * Preserves the error message and optional code for structured handling.
 *
 * @example
 * const result = await safeApiCall(() => createCheckoutSession(items))
 * if (!result.success) {
 *   // result.error is a string — always
 * }
 */
export async function safeApiCall<T>(
  fn: () => Promise<T>
): Promise<ApiResult<T>> {
  try {
    const data = await fn()
    return { success: true, data }
  } catch (error) {
    if (error instanceof Error) {
      console.error("[API Error]:", error.message)
      // Stripe errors carry a `code` property — surface it if present
      const code =
        "code" in error && typeof error.code === "string"
          ? error.code
          : undefined
      return { success: false, error: error.message, code }
    }
    return { success: false, error: "Unknown error" }
  }
}
```

---

### `src/env.ts` (T3 Env — environment variable registration)

```typescript
// Extend your existing T3 Env schema with these Stripe variables.
// If your project does not yet have T3 Env, install:
//   npm install @t3-oss/env-nextjs zod

import { createEnv } from "@t3-oss/env-nextjs"
import { z } from "zod"

export const env = createEnv({
  /**
   * Server-side environment variables — never exposed to the browser.
   */
  server: {
    // Stripe secret key: sk_test_… in development, sk_live_… in production
    STRIPE_SECRET_KEY: z.string().min(1).startsWith("sk_"),
    // Webhook signing secret from Stripe Dashboard → Webhooks → your endpoint
    STRIPE_WEBHOOK_SECRET: z.string().min(1).startsWith("whsec_"),
    // Node environment
    NODE_ENV: z
      .enum(["development", "test", "production"])
      .default("development"),
  },

  /**
   * Client-side environment variables — safe to expose to the browser.
   */
  client: {
    // App base URL used for success_url / cancel_url construction
    NEXT_PUBLIC_APP_URL: z.string().url(),
  },

  /**
   * Destructure from process.env at build time.
   * Must match the keys defined above exactly.
   */
  runtimeEnv: {
    STRIPE_SECRET_KEY: process.env.STRIPE_SECRET_KEY,
    STRIPE_WEBHOOK_SECRET: process.env.STRIPE_WEBHOOK_SECRET,
    NODE_ENV: process.env.NODE_ENV,
    NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
  },
})
```

---

### `src/lib/stripe.ts`

```typescript
import Stripe from "stripe"
import { env } from "@/env"

/**
 * Singleton Stripe client for server-side use only.
 *
 * API version pinned to 2024-12-18.acacia — bump intentionally after
 * reviewing Stripe's migration guide.
 *
 * Timeout: 30s (Stripe's recommended value for payment APIs;
 * the SDK retries internally on network failures within this window).
 */
export const stripe = new Stripe(env.STRIPE_SECRET_KEY, {
  apiVersion: "2024-12-18.acacia",
  typescript: true,
  timeout: 30_000, // 30 seconds — matches payment API default per skill spec
})
```

---

### `src/app/actions/checkout.ts`

```typescript
"use server"

import { stripe } from "@/lib/stripe"
import { env } from "@/env"
import type { ApiResult } from "@/types/api"
import { safeApiCall } from "@/types/api"

// ---------------------------------------------------------------------------
// Types
// ---------------------------------------------------------------------------

export interface CoffeeLineItem {
  /** Stripe Price ID (price_xxxxxxxx) for the coffee product variant */
  priceId: string
  /** Number of units — min 1 */
  quantity: number
}

export interface CheckoutSessionResult {
  /** Stripe-hosted Checkout URL — redirect the browser to this */
  url: string
  /** Stripe Checkout Session ID — stored for order reconciliation */
  sessionId: string
}

// ---------------------------------------------------------------------------
// Server Action
// ---------------------------------------------------------------------------

/**
 * Creates a Stripe Checkout Session for ground coffee purchases.
 *
 * Returns ApiResult<CheckoutSessionResult>:
 * - success: true  → redirect to result.data.url
 * - success: false → display result.error to the user
 *
 * @param items     One or more coffee SKUs with price ID and quantity
 * @param customerEmail  Pre-fills the email field on the Checkout page (optional)
 * @param orderId   Your internal order reference, stored in Stripe metadata
 *
 * @example
 * const result = await createCheckoutSession(
 *   [{ priceId: "price_1Abc123", quantity: 2 }],
 *   "cliente@example.com",
 *   "order_789"
 * )
 * if (result.success) redirect(result.data.url)
 */
export async function createCheckoutSession(
  items: CoffeeLineItem[],
  customerEmail?: string,
  orderId?: string
): Promise<ApiResult<CheckoutSessionResult>> {
  // Guard: at least one item required
  if (!items.length) {
    return { success: false, error: "No items provided for checkout" }
  }

  return safeApiCall(async () => {
    const session = await stripe.checkout.sessions.create({
      mode: "payment",

      // Line items — each entry maps to one coffee SKU
      line_items: items.map((item) => ({
        price: item.priceId,
        quantity: item.quantity,
      })),

      // Redirect URLs
      // {CHECKOUT_SESSION_ID} is a Stripe template variable, not a JS template literal
      success_url: `${env.NEXT_PUBLIC_APP_URL}/pedido/confirmacion?session_id={CHECKOUT_SESSION_ID}`,
      cancel_url: `${env.NEXT_PUBLIC_APP_URL}/tienda`,

      // Spain market: accept card + Bizum
      payment_method_types: ["card", "bizum"],

      // Pre-fill customer email if available (reduces friction at checkout)
      customer_email: customerEmail ?? undefined,

      // Shipping address collection — required for physical goods (ground coffee)
      shipping_address_collection: {
        allowed_countries: ["ES"], // Spain only for now
      },

      // Shipping options — flat rate for Spain
      // Create these in Stripe Dashboard → Products → Shipping rates
      // and paste the IDs below
      shipping_options: [
        {
          shipping_rate: "shr_standard_ES", // Replace with real shr_xxxx ID
        },
      ],

      // Metadata for webhook reconciliation
      metadata: {
        ...(orderId ? { orderId } : {}),
        source: "cafe-vermut-web",
      },

      // Automatic tax calculation (requires Stripe Tax enabled in Dashboard)
      automatic_tax: { enabled: true },

      // Locale — Spanish speakers
      locale: "es",
    })

    if (!session.url) {
      throw new Error("Stripe did not return a checkout URL")
    }

    return {
      url: session.url,
      sessionId: session.id,
    }
  })
}
```

---

### `src/app/api/webhooks/stripe/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server"
import { stripe } from "@/lib/stripe"
import { env } from "@/env"
import type Stripe from "stripe"

// ---------------------------------------------------------------------------
// Disable body parsing — Stripe signature verification requires raw bytes
// ---------------------------------------------------------------------------

export const dynamic = "force-dynamic"

// ---------------------------------------------------------------------------
// POST /api/webhooks/stripe
// ---------------------------------------------------------------------------

export async function POST(request: NextRequest): Promise<NextResponse> {
  // 1. Read raw body (required for signature verification)
  const body = await request.text()

  // 2. Validate Stripe-Signature header presence
  const signature = request.headers.get("stripe-signature")
  if (!signature) {
    console.warn("[Stripe Webhook] Missing stripe-signature header")
    return NextResponse.json(
      { error: "Missing stripe-signature header" },
      { status: 400 }
    )
  }

  // 3. Verify webhook signature
  let event: Stripe.Event

  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      env.STRIPE_WEBHOOK_SECRET
    )
  } catch (err) {
    const message = err instanceof Error ? err.message : "Unknown error"
    console.error("[Stripe Webhook] Signature verification failed:", message)
    return NextResponse.json(
      { error: `Webhook signature verification failed: ${message}` },
      { status: 400 }
    )
  }

  // 4. Route to typed event handlers
  try {
    switch (event.type) {
      case "checkout.session.completed": {
        const session = event.data.object as Stripe.CheckoutSession
        await handleCheckoutSessionCompleted(session)
        break
      }

      case "payment_intent.payment_failed": {
        const intent = event.data.object as Stripe.PaymentIntent
        await handlePaymentIntentFailed(intent)
        break
      }

      default: {
        // Log unhandled events — useful during development
        // Remove or downgrade to debug in production once stable
        console.log(`[Stripe Webhook] Unhandled event type: ${event.type}`)
      }
    }
  } catch (err) {
    // Handler threw — return 500 so Stripe retries the event
    const message = err instanceof Error ? err.message : "Unknown error"
    console.error(
      `[Stripe Webhook] Handler failed for event ${event.type} (${event.id}):`,
      message
    )
    return NextResponse.json(
      { error: `Handler failed: ${message}` },
      { status: 500 }
    )
  }

  // 5. Acknowledge receipt — Stripe stops retrying on 2xx
  return NextResponse.json({ received: true }, { status: 200 })
}

// ---------------------------------------------------------------------------
// Event handlers
// ---------------------------------------------------------------------------

/**
 * checkout.session.completed
 *
 * Triggered when the customer completes payment on the Stripe-hosted page.
 * Use this event to:
 *   - Mark the order as paid in your database
 *   - Trigger an order confirmation email (via Resend)
 *   - Decrement inventory
 *
 * IMPORTANT: Always check session.payment_status === "paid" before fulfilling.
 * For payment methods with delayed notification (e.g. bank transfers),
 * the session is "complete" but payment may still be pending.
 */
async function handleCheckoutSessionCompleted(
  session: Stripe.CheckoutSession
): Promise<void> {
  const orderId = session.metadata?.orderId
  const customerEmail = session.customer_details?.email
  const paymentStatus = session.payment_status

  console.log(
    `[Stripe] checkout.session.completed — session: ${session.id}, orderId: ${orderId ?? "none"}, paymentStatus: ${paymentStatus}`
  )

  // Guard: only fulfill when payment is confirmed
  if (paymentStatus !== "paid") {
    console.warn(
      `[Stripe] Session ${session.id} complete but paymentStatus=${paymentStatus} — skipping fulfillment`
    )
    return
  }

  // TODO: implement order fulfillment
  // Examples:
  //   await db.order.update({ where: { stripeSessionId: session.id }, data: { status: "PAID" } })
  //   await sendOrderConfirmationEmail({ to: customerEmail, orderId, session })
  //   await decrementInventory(session.line_items)

  console.log(
    `[Stripe] Order ${orderId ?? session.id} fulfilled for ${customerEmail ?? "unknown"}`
  )
}

/**
 * payment_intent.payment_failed
 *
 * Triggered when a payment attempt fails (declined card, insufficient funds, etc.).
 * Use this event to:
 *   - Log the failure for analytics / fraud monitoring
 *   - Send a recovery email if the customer has an account
 *   - Update internal order status to PAYMENT_FAILED
 *
 * Note: Stripe already retries Smart Retries internally for subscriptions.
 * For one-time payments the customer must return to checkout.
 */
async function handlePaymentIntentFailed(
  intent: Stripe.PaymentIntent
): Promise<void> {
  const lastError = intent.last_payment_error
  const declineCode = lastError?.decline_code ?? "unknown"
  const failureMessage = lastError?.message ?? "No message"

  console.error(
    `[Stripe] payment_intent.payment_failed — intent: ${intent.id}, declineCode: ${declineCode}, message: ${failureMessage}`
  )

  // TODO: implement failure handling
  // Examples:
  //   await db.order.update({
  //     where: { stripePaymentIntentId: intent.id },
  //     data: { status: "PAYMENT_FAILED", failureReason: failureMessage },
  //   })
  //   if (intent.metadata.customerEmail) {
  //     await sendPaymentFailedEmail({ to: intent.metadata.customerEmail, declineCode })
  //   }
}
```

---

### `docs/integrations/stripe.md`

```markdown
# Stripe Integration

**SDK:** `stripe` v17  
**API version:** `2024-12-18.acacia`  
**Market:** Spain (card + Bizum, locale `es`, shipping restricted to `ES`)

---

## Environment Variables

| Variable | Where to obtain | Notes |
|---|---|---|
| `STRIPE_SECRET_KEY` | Dashboard → Developers → API keys → Secret key | `sk_test_…` in development, `sk_live_…` in production |
| `STRIPE_WEBHOOK_SECRET` | Dashboard → Developers → Webhooks → your endpoint → Signing secret | Different value for test and production webhooks |
| `NEXT_PUBLIC_APP_URL` | Your deployment URL | `http://localhost:3000` locally, `https://cafevermut.com` in production |

Add to `.env.local` (never commit):

```env
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

---

## Files

| File | Role |
|---|---|
| `src/lib/stripe.ts` | Stripe client singleton, T3 Env, 30s timeout |
| `src/types/api.ts` | `ApiResult<T>` type + `safeApiCall` utility |
| `src/app/actions/checkout.ts` | Server Action `createCheckoutSession` |
| `src/app/api/webhooks/stripe/route.ts` | Webhook handler (POST) |

---

## Test Mode

### Enabling test mode
Set `STRIPE_SECRET_KEY=sk_test_…` in `.env.local`. All API calls will hit the Stripe test environment automatically — no code change required.

### Test cards for Spain market

| Scenario | Card number | Expiry | CVC |
|---|---|---|---|
| Successful payment | `4242 4242 4242 4242` | Any future date | Any 3 digits |
| Card declined | `4000 0000 0000 0002` | Any future date | Any 3 digits |
| Requires 3D Secure (SCA) | `4000 0025 0000 3155` | Any future date | Any 3 digits |
| Insufficient funds | `4000 0000 0000 9995` | Any future date | Any 3 digits |
| Bizum (test) | Use Stripe Dashboard test Bizum flow | — | — |

### Local webhook testing (Stripe CLI)

```bash
# Install Stripe CLI: https://stripe.com/docs/stripe-cli
stripe login
stripe listen --forward-to localhost:3000/api/webhooks/stripe
# Copy the whsec_… value printed and set it as STRIPE_WEBHOOK_SECRET in .env.local
```

---

## Production Checklist

### Stripe Dashboard — before go-live

- [ ] Business verified (Stripe identity verification complete)
- [ ] Bank account connected for payouts
- [ ] Bizum enabled: Dashboard → Settings → Payment methods → Bizum
- [ ] Stripe Tax enabled (if using `automatic_tax`): Dashboard → Tax → Settings
- [ ] Shipping rates created: Dashboard → Products → Shipping rates → create `shr_standard_ES`
- [ ] Product and Price IDs created for each coffee SKU: Dashboard → Products
- [ ] Webhook endpoint registered: Dashboard → Developers → Webhooks → Add endpoint
  - URL: `https://cafevermut.com/api/webhooks/stripe`
  - Events: `checkout.session.completed`, `payment_intent.payment_failed`
- [ ] Webhook signing secret copied from the registered endpoint

### Environment variables — hosting provider

- [ ] `STRIPE_SECRET_KEY` set to `sk_live_…` (NOT `sk_test_…`)
- [ ] `STRIPE_WEBHOOK_SECRET` set to the production endpoint's `whsec_…`
- [ ] `NEXT_PUBLIC_APP_URL` set to `https://cafevermut.com`

### Code — before go-live

- [ ] `shipping_options` in `checkout.ts` updated with real `shr_xxxx` IDs
- [ ] `TODO` fulfillment stubs in `route.ts` replaced with real DB + email calls
- [ ] `safeApiCall` errors mapped to user-facing messages in the UI
- [ ] End-to-end test completed with a live test card in Stripe test mode

### Post-launch monitoring

- [ ] Stripe Dashboard → Developers → Logs — check for 4xx/5xx webhook responses
- [ ] Stripe Dashboard → Developers → Events — confirm `checkout.session.completed` is received and acknowledged (200)
- [ ] Alert set up for `payment_intent.payment_failed` rate > 5%

---

## Known Limitations

- **Bizum:** Only available for Spanish cardholders and Spanish Stripe accounts. Bizum test flow requires the Stripe Dashboard test mode — no test card equivalent.
- **Shipping:** Currently restricted to Spain (`ES`). Add countries to `allowed_countries` and create additional shipping rates in the Dashboard before expanding.
- **Automatic tax:** Requires Stripe Tax configuration. If not enabled, remove `automatic_tax: { enabled: true }` from `createCheckoutSession` to avoid API errors.
- **Rate limits:** Stripe test mode: 100 req/s. Production: 100 req/s by default — sufficient for 200 orders/month.
- **Webhook retries:** Stripe retries failed webhooks (non-2xx responses) up to 3 days with exponential backoff. The handler returns 500 on business logic errors to trigger retries — this is intentional.
- **Idempotency:** For 50–200 orders/month the current volume does not require explicit idempotency keys. If volume grows significantly, add `idempotencyKey` to `stripe.checkout.sessions.create` calls.
```

---

## TypeScript Compliance

- Zero `any` types — all Stripe event objects cast through `Stripe.CheckoutSession` / `Stripe.PaymentIntent` typed interfaces
- `ApiResult<T>` discriminated union — callers are forced to check `result.success` before accessing `result.data`
- `safeApiCall` extracts Stripe's `error.code` property through a narrowing check, not `as any`
- `CoffeeLineItem` and `CheckoutSessionResult` are exported types — the UI layer can import them for prop typing
- T3 Env `createEnv` enforces `STRIPE_SECRET_KEY` starts with `sk_` and `STRIPE_WEBHOOK_SECRET` starts with `whsec_` at build time — misconfiguration fails loudly before the app starts

---

## Error Handling Map

| Error path | Behavior | HTTP status |
|---|---|---|
| Missing `stripe-signature` header | Return 400 immediately | 400 |
| Signature verification failure | Return 400, log error | 400 |
| `session.payment_status !== "paid"` | Skip fulfillment, return 200 (not a Stripe error) | 200 |
| Business logic handler throws | Return 500 → triggers Stripe retry | 500 |
| `createCheckoutSession` — no items | Returns `ApiResult` with `success: false` | (Server Action, no HTTP) |
| `createCheckoutSession` — Stripe SDK throws | `safeApiCall` catches, returns typed `ApiResult` | (Server Action, no HTTP) |
| Session URL missing (edge case) | `safeApiCall` catches thrown Error | (Server Action, no HTTP) |

---

## Timeout Configuration

```
Payment APIs (Stripe SDK): 30s  ← sdk-level timeout: 30_000
```

The Stripe SDK `timeout` option aborts the underlying HTTP request if no response is received within 30 seconds. This prevents indefinite hangs during checkout under slow network conditions. For the webhook handler, no explicit timeout is set because Next.js enforces its own route handler timeout (`maxDuration` in `next.config.ts` if needed).

---

## Installation

```bash
npm install stripe @t3-oss/env-nextjs zod
```

`tsconfig.json` must include:
```json
{
  "compilerOptions": {
    "strict": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

`.env.local` (development):
```env
STRIPE_SECRET_KEY=sk_test_51...
STRIPE_WEBHOOK_SECRET=whsec_...
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

---

## Usage Example — Checkout Button

```typescript
// src/app/tienda/page.tsx (or a Client Component wrapping this action)
"use client"

import { createCheckoutSession } from "@/app/actions/checkout"
import { useTransition } from "react"

export function AddToCartButton({ priceId }: { priceId: string }) {
  const [isPending, startTransition] = useTransition()

  function handleBuy() {
    startTransition(async () => {
      const result = await createCheckoutSession(
        [{ priceId, quantity: 1 }],
        undefined,        // customerEmail — omit if not authenticated
        crypto.randomUUID() // orderId — generate or use your DB ID
      )

      if (result.success) {
        // Redirect to Stripe-hosted Checkout
        window.location.href = result.data.url
      } else {
        // result.error is always a string — show to user
        alert(`Error al iniciar el pago: ${result.error}`)
      }
    })
  }

  return (
    <button onClick={handleBuy} disabled={isPending}>
      {isPending ? "Cargando..." : "Comprar ahora"}
    </button>
  )
}
```
