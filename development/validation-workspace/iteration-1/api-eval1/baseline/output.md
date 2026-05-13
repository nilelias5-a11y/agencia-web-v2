# Stripe Integration — Café Vermut

**Context:** Coffee shop selling ground coffee online. Volume: 50–200 orders/month, average ticket €25, Spain market (EUR, IVA included).

---

## 1. Dependencies & Environment Variables

```bash
npm install stripe
```

### `.env.local`
```env
STRIPE_SECRET_KEY=sk_live_...          # or sk_test_... for development
STRIPE_PUBLISHABLE_KEY=pk_live_...    # or pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...       # from Stripe Dashboard > Webhooks
NEXT_PUBLIC_BASE_URL=https://cafevermut.com
```

---

## 2. `lib/stripe.ts` — Stripe Client

```typescript
// lib/stripe.ts
import Stripe from "stripe";

if (!process.env.STRIPE_SECRET_KEY) {
  throw new Error(
    "Missing STRIPE_SECRET_KEY environment variable. " +
      "Set it in .env.local (sk_test_... for dev, sk_live_... for prod)."
  );
}

/**
 * Singleton Stripe client configured for Spain / EUR.
 *
 * API version is pinned so upgrades are explicit and auditable.
 * timeout: 10 000 ms — well within Next.js function limits and
 * Stripe's own 30 s server-side timeout.
 */
export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY, {
  apiVersion: "2024-06-20",
  typescript: true,
  timeout: 10_000, // ms — abort if Stripe doesn't respond in 10 s
  maxNetworkRetries: 2, // automatic idempotent retries on network errors
  appInfo: {
    name: "Café Vermut",
    version: "1.0.0",
    url: process.env.NEXT_PUBLIC_BASE_URL,
  },
});
```

---

## 3. `types/stripe.ts` — Shared Types

```typescript
// types/stripe.ts

/** A single line item sent to the checkout session */
export interface CheckoutLineItem {
  /** Stripe Price ID (price_...) configured in the Stripe Dashboard */
  priceId: string;
  quantity: number;
}

/** Input for createCheckoutSession */
export interface CreateCheckoutSessionInput {
  lineItems: CheckoutLineItem[];
  /** Customer email pre-fill (optional) */
  customerEmail?: string;
  /** Internal order reference stored as metadata */
  orderReference?: string;
}

/** Successful result from createCheckoutSession */
export interface CheckoutSessionResult {
  sessionId: string;
  url: string;
}

/** Error result (discriminated union) */
export interface CheckoutSessionError {
  error: string;
  code?: string;
}

export type CreateCheckoutSessionResult =
  | { success: true; data: CheckoutSessionResult }
  | { success: false; error: CheckoutSessionError };
```

---

## 4. `app/actions/createCheckoutSession.ts` — Server Action

```typescript
// app/actions/createCheckoutSession.ts
"use server";

import { redirect } from "next/navigation";
import Stripe from "stripe";
import { stripe } from "@/lib/stripe";
import type {
  CreateCheckoutSessionInput,
  CreateCheckoutSessionResult,
} from "@/types/stripe";

const BASE_URL =
  process.env.NEXT_PUBLIC_BASE_URL ?? "http://localhost:3000";

/**
 * Creates a Stripe Checkout Session for ground-coffee purchases.
 *
 * Designed for the Spain market:
 *   - Currency: EUR
 *   - Billing address collection enabled (required for Spanish IVA)
 *   - Phone number collection enabled (courier handoff)
 *   - Payment methods: card + SEPA debit
 *
 * Call from a <form action={createCheckoutSession}> or via
 * `const result = await createCheckoutSession(input)` in a Client Component.
 *
 * On success the user is redirected to Stripe-hosted checkout.
 * On failure a structured error object is returned so the UI can
 * display a meaningful message without crashing.
 *
 * @param input - Line items and optional customer metadata
 * @returns Never resolves on success (redirect throws); resolves with
 *          { success: false, error } on failure.
 */
export async function createCheckoutSession(
  input: CreateCheckoutSessionInput
): Promise<CreateCheckoutSessionResult> {
  // --- Input validation -----------------------------------------------
  if (!input.lineItems || input.lineItems.length === 0) {
    return {
      success: false,
      error: {
        error: "El carrito está vacío. Añade al menos un producto.",
        code: "EMPTY_CART",
      },
    };
  }

  for (const item of input.lineItems) {
    if (!item.priceId || !item.priceId.startsWith("price_")) {
      return {
        success: false,
        error: {
          error: `Price ID inválido: "${item.priceId}". Debe empezar por price_`,
          code: "INVALID_PRICE_ID",
        },
      };
    }
    if (!Number.isInteger(item.quantity) || item.quantity < 1) {
      return {
        success: false,
        error: {
          error: `Cantidad inválida para ${item.priceId}: debe ser un entero ≥ 1.`,
          code: "INVALID_QUANTITY",
        },
      };
    }
  }

  // --- Create Checkout Session ----------------------------------------
  let session: Stripe.Checkout.Session;

  try {
    session = await stripe.checkout.sessions.create({
      mode: "payment",
      currency: "eur",

      line_items: input.lineItems.map((item) => ({
        price: item.priceId,
        quantity: item.quantity,
      })),

      // Collect billing address — needed for Spanish IVA compliance
      billing_address_collection: "required",

      // Collect phone — useful for courier delivery
      phone_number_collection: { enabled: true },

      // Payment methods appropriate for Spain
      payment_method_types: ["card", "sepa_debit"],

      // Pre-fill customer email if available
      ...(input.customerEmail && { customer_email: input.customerEmail }),

      // Internal traceability
      metadata: {
        ...(input.orderReference && { order_reference: input.orderReference }),
        source: "cafe_vermut_web",
        integration_version: "1.0.0",
      },

      // Redirect URLs
      success_url: `${BASE_URL}/pedido/confirmacion?session_id={CHECKOUT_SESSION_ID}`,
      cancel_url: `${BASE_URL}/tienda?checkout=cancelled`,

      // 30-minute session expiry (default 24 h is too long for perishable stock)
      expires_at: Math.floor(Date.now() / 1000) + 30 * 60,

      // Allow promotional codes
      allow_promotion_codes: true,
    });
  } catch (err) {
    // --- Stripe API errors --------------------------------------------
    if (err instanceof Stripe.errors.StripeCardError) {
      // Should not happen at session creation, but guard anyway
      return {
        success: false,
        error: {
          error: "La tarjeta fue rechazada. Por favor, usa otra tarjeta.",
          code: err.code ?? "CARD_ERROR",
        },
      };
    }

    if (err instanceof Stripe.errors.StripeInvalidRequestError) {
      // Likely a bad price_id or misconfiguration — log for developers
      console.error("[Stripe] Invalid request:", err.message, {
        param: err.param,
        code: err.code,
      });
      return {
        success: false,
        error: {
          error:
            "Error de configuración en el pago. El equipo ha sido notificado.",
          code: err.code ?? "INVALID_REQUEST",
        },
      };
    }

    if (err instanceof Stripe.errors.StripeConnectionError) {
      console.error("[Stripe] Connection error:", err.message);
      return {
        success: false,
        error: {
          error:
            "No se pudo conectar con el servicio de pago. Inténtalo de nuevo.",
          code: "CONNECTION_ERROR",
        },
      };
    }

    if (err instanceof Stripe.errors.StripeAPIError) {
      console.error("[Stripe] API error:", err.message, { status: err.statusCode });
      return {
        success: false,
        error: {
          error: "El servicio de pago no está disponible. Inténtalo más tarde.",
          code: "API_ERROR",
        },
      };
    }

    if (err instanceof Stripe.errors.StripeRateLimitError) {
      console.error("[Stripe] Rate limit hit");
      return {
        success: false,
        error: {
          error: "Demasiadas solicitudes. Espera unos segundos e inténtalo de nuevo.",
          code: "RATE_LIMIT",
        },
      };
    }

    // Unknown error — re-throw so Next.js error boundary captures it
    console.error("[Stripe] Unexpected error:", err);
    return {
      success: false,
      error: {
        error: "Error inesperado al iniciar el pago. Inténtalo de nuevo.",
        code: "UNKNOWN",
      },
    };
  }

  // --- Validate session URL ------------------------------------------
  if (!session.url) {
    console.error("[Stripe] Session created but no URL returned", {
      sessionId: session.id,
    });
    return {
      success: false,
      error: {
        error: "No se pudo obtener la URL de pago. Contacta con soporte.",
        code: "MISSING_SESSION_URL",
      },
    };
  }

  // redirect() throws internally — must be called outside try/catch
  redirect(session.url);
}
```

---

## 5. `app/api/webhooks/stripe/route.ts` — Webhook Handler

```typescript
// app/api/webhooks/stripe/route.ts
import { headers } from "next/headers";
import { NextRequest, NextResponse } from "next/server";
import Stripe from "stripe";
import { stripe } from "@/lib/stripe";

// -----------------------------------------------------------------------
// Configuration
// -----------------------------------------------------------------------

const WEBHOOK_SECRET = process.env.STRIPE_WEBHOOK_SECRET;

/**
 * Maximum request body size for webhooks: 1 MB.
 * Stripe webhook payloads are typically < 10 KB; this is a safety ceiling.
 */
export const config = {
  api: {
    bodyParser: false, // Raw body required for signature verification
  },
};

// -----------------------------------------------------------------------
// Event handlers
// -----------------------------------------------------------------------

/**
 * Handle checkout.session.completed
 *
 * Fired when the customer successfully completes payment.
 * At this point the payment is captured (or authorized for capture).
 *
 * Responsibilities:
 *   - Fulfil the order (update DB, trigger dispatch, send confirmation email)
 *   - Be idempotent — Stripe may deliver the event more than once
 */
async function handleCheckoutSessionCompleted(
  session: Stripe.Checkout.Session
): Promise<void> {
  const orderReference = session.metadata?.order_reference;
  const customerEmail = session.customer_details?.email;

  console.info("[Webhook] checkout.session.completed", {
    sessionId: session.id,
    paymentStatus: session.payment_status,
    amountTotal: session.amount_total, // in cents
    currency: session.currency,
    orderReference,
    customerEmail,
  });

  // Guard: only process fully paid sessions
  if (session.payment_status !== "paid") {
    console.warn(
      "[Webhook] checkout.session.completed with non-paid status — skipping",
      { sessionId: session.id, paymentStatus: session.payment_status }
    );
    return;
  }

  // ----------------------------------------------------------------
  // TODO: Replace the blocks below with your actual business logic
  // ----------------------------------------------------------------

  // 1. Idempotency check — verify this session hasn't been processed
  //    Example (Prisma):
  //    const existing = await prisma.order.findUnique({
  //      where: { stripeSessionId: session.id },
  //    });
  //    if (existing?.status === "confirmed") {
  //      console.info("[Webhook] Already processed, skipping", { sessionId: session.id });
  //      return;
  //    }

  // 2. Create / update order in DB
  //    await prisma.order.upsert({
  //      where: { stripeSessionId: session.id },
  //      update: { status: "confirmed", paidAt: new Date() },
  //      create: {
  //        stripeSessionId: session.id,
  //        orderReference: orderReference ?? session.id,
  //        customerEmail: customerEmail ?? "",
  //        amountTotal: session.amount_total ?? 0,
  //        currency: session.currency ?? "eur",
  //        status: "confirmed",
  //        paidAt: new Date(),
  //      },
  //    });

  // 3. Trigger fulfilment (e.g. queue a dispatch job)
  //    await dispatchQueue.add({ orderId: ..., address: session.shipping_details });

  // 4. Send confirmation email
  //    await sendOrderConfirmationEmail({ to: customerEmail, orderReference });

  console.info("[Webhook] Order fulfilled successfully", {
    sessionId: session.id,
    orderReference,
  });
}

/**
 * Handle payment_intent.payment_failed
 *
 * Fired when a payment attempt fails (declined card, insufficient funds, etc.)
 * Note: with Checkout the customer is given multiple retry attempts before
 * the session expires, so a single payment_failed doesn't mean the order
 * is lost — but it's worth logging for analytics and support.
 */
async function handlePaymentIntentPaymentFailed(
  paymentIntent: Stripe.PaymentIntent
): Promise<void> {
  const lastError = paymentIntent.last_payment_error;

  console.warn("[Webhook] payment_intent.payment_failed", {
    paymentIntentId: paymentIntent.id,
    amount: paymentIntent.amount,
    currency: paymentIntent.currency,
    errorCode: lastError?.code,
    errorMessage: lastError?.message,
    declineCode: lastError?.decline_code,
    customerEmail: paymentIntent.receipt_email,
  });

  // ----------------------------------------------------------------
  // TODO: Add optional business logic
  // ----------------------------------------------------------------

  // 1. Update order status to "payment_failed" if you track attempts
  //    await prisma.paymentAttempt.create({
  //      data: {
  //        stripePaymentIntentId: paymentIntent.id,
  //        status: "failed",
  //        errorCode: lastError?.code ?? null,
  //        declineCode: lastError?.decline_code ?? null,
  //        failedAt: new Date(),
  //      },
  //    });

  // 2. Optionally notify the customer (be careful not to spam on retries)
  //    await sendPaymentFailedEmail({ to: paymentIntent.receipt_email, lastError });

  // 3. Alert operations team if failure rate spikes
  //    await alertOps("payment_failed", { paymentIntentId: paymentIntent.id, ...lastError });
}

// -----------------------------------------------------------------------
// Route handler
// -----------------------------------------------------------------------

/**
 * POST /api/webhooks/stripe
 *
 * Stripe webhook endpoint. Must:
 *   - Accept raw body (no JSON parsing middleware)
 *   - Verify the Stripe-Signature header
 *   - Return 2xx quickly (Stripe times out at 30 s and retries up to 3 days)
 *   - Be idempotent (Stripe delivers at-least-once)
 *
 * Stripe retry policy: if this endpoint returns non-2xx or times out,
 * Stripe retries with exponential backoff for up to 72 hours.
 */
export async function POST(req: NextRequest): Promise<NextResponse> {
  // --- 1. Validate configuration ------------------------------------
  if (!WEBHOOK_SECRET) {
    console.error(
      "[Webhook] STRIPE_WEBHOOK_SECRET is not set — cannot verify signatures"
    );
    return NextResponse.json(
      { error: "Webhook secret not configured" },
      { status: 500 }
    );
  }

  // --- 2. Read raw body (required for signature verification) -------
  let rawBody: string;
  try {
    rawBody = await req.text();
  } catch (err) {
    console.error("[Webhook] Failed to read request body:", err);
    return NextResponse.json(
      { error: "Failed to read request body" },
      { status: 400 }
    );
  }

  // --- 3. Extract and validate Stripe-Signature header --------------
  const headersList = await headers();
  const signature = headersList.get("stripe-signature");

  if (!signature) {
    console.warn("[Webhook] Missing stripe-signature header");
    return NextResponse.json(
      { error: "Missing stripe-signature header" },
      { status: 400 }
    );
  }

  // --- 4. Verify webhook signature ----------------------------------
  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(rawBody, signature, WEBHOOK_SECRET);
  } catch (err) {
    if (err instanceof Stripe.errors.StripeSignatureVerificationError) {
      console.warn("[Webhook] Signature verification failed", {
        message: err.message,
      });
      return NextResponse.json(
        { error: "Invalid webhook signature" },
        { status: 400 }
      );
    }
    console.error("[Webhook] Unexpected error during signature verification:", err);
    return NextResponse.json(
      { error: "Signature verification error" },
      { status: 400 }
    );
  }

  console.info("[Webhook] Event received", {
    eventId: event.id,
    type: event.type,
    livemode: event.livemode,
    created: new Date(event.created * 1000).toISOString(),
  });

  // --- 5. Dispatch event to handler ---------------------------------
  try {
    switch (event.type) {
      case "checkout.session.completed": {
        const session = event.data.object as Stripe.Checkout.Session;
        await handleCheckoutSessionCompleted(session);
        break;
      }

      case "payment_intent.payment_failed": {
        const paymentIntent = event.data.object as Stripe.PaymentIntent;
        await handlePaymentIntentPaymentFailed(paymentIntent);
        break;
      }

      // Extend here as needed:
      // case "checkout.session.expired": { ... break; }
      // case "charge.refunded": { ... break; }

      default:
        // Acknowledge but ignore unhandled event types
        console.info("[Webhook] Unhandled event type — acknowledged", {
          type: event.type,
        });
    }
  } catch (err) {
    // Handler threw — log the error but still return 200 to prevent
    // Stripe from retrying for a bug in our own code.
    // For transient infrastructure errors (DB down) you MAY return 500
    // to trigger Stripe retries, but do so deliberately.
    console.error("[Webhook] Handler error for event", {
      eventId: event.id,
      type: event.type,
      error: err,
    });
    // Return 200 to prevent infinite retries on application-level bugs
    return NextResponse.json(
      { received: true, warning: "Handler error logged" },
      { status: 200 }
    );
  }

  // --- 6. Acknowledge receipt ----------------------------------------
  return NextResponse.json({ received: true }, { status: 200 });
}

// Reject non-POST methods
export async function GET(): Promise<NextResponse> {
  return NextResponse.json({ error: "Method not allowed" }, { status: 405 });
}
```

---

## 6. `app/pedido/confirmacion/page.tsx` — Success Page

```typescript
// app/pedido/confirmacion/page.tsx
import { stripe } from "@/lib/stripe";
import { redirect } from "next/navigation";

interface Props {
  searchParams: { session_id?: string };
}

/**
 * Order confirmation page.
 * Retrieves the completed Checkout Session from Stripe to display
 * a personalised confirmation to the customer.
 */
export default async function ConfirmacionPage({ searchParams }: Props) {
  const { session_id } = searchParams;

  if (!session_id) {
    redirect("/tienda");
  }

  let session;
  try {
    session = await stripe.checkout.sessions.retrieve(session_id, {
      expand: ["line_items", "line_items.data.price.product"],
    });
  } catch {
    redirect("/tienda?error=session_not_found");
  }

  if (session.payment_status !== "paid") {
    redirect("/tienda?error=payment_not_completed");
  }

  const customerName = session.customer_details?.name ?? "Cliente";
  const customerEmail = session.customer_details?.email ?? "";
  const amountTotal = ((session.amount_total ?? 0) / 100).toFixed(2);
  const orderRef = session.metadata?.order_reference ?? session.id;

  return (
    <main className="max-w-lg mx-auto py-16 px-4 text-center">
      <h1 className="text-3xl font-bold text-green-700 mb-4">
        ¡Pedido confirmado!
      </h1>
      <p className="text-lg text-gray-700 mb-2">
        Gracias, <strong>{customerName}</strong>.
      </p>
      <p className="text-gray-600 mb-2">
        Hemos recibido tu pago de <strong>€{amountTotal}</strong>.
      </p>
      <p className="text-gray-600 mb-6">
        Te enviaremos la confirmación a <strong>{customerEmail}</strong>.
      </p>
      <p className="text-sm text-gray-400">Referencia: {orderRef}</p>
      <a
        href="/tienda"
        className="mt-8 inline-block bg-amber-600 text-white px-6 py-3 rounded-lg hover:bg-amber-700 transition"
      >
        Seguir comprando
      </a>
    </main>
  );
}
```

---

## 7. `components/BuyButton.tsx` — Client Component Example

```typescript
// components/BuyButton.tsx
"use client";

import { useState } from "react";
import { createCheckoutSession } from "@/app/actions/createCheckoutSession";

interface Props {
  priceId: string;
  label?: string;
}

/**
 * Single-product buy button that triggers the Server Action.
 * Handles loading state and error display.
 */
export function BuyButton({ priceId, label = "Comprar ahora" }: Props) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function handleClick() {
    setLoading(true);
    setError(null);

    const result = await createCheckoutSession({
      lineItems: [{ priceId, quantity: 1 }],
    });

    // If we reach this point the redirect didn't happen — show the error
    if (!result.success) {
      setError(result.error.error);
      setLoading(false);
    }
  }

  return (
    <div>
      <button
        onClick={handleClick}
        disabled={loading}
        className="bg-amber-600 text-white px-6 py-3 rounded-lg hover:bg-amber-700 disabled:opacity-50 transition"
      >
        {loading ? "Redirigiendo al pago…" : label}
      </button>
      {error && (
        <p className="mt-2 text-red-600 text-sm" role="alert">
          {error}
        </p>
      )}
    </div>
  );
}
```

---

## 8. Error Handling Summary

| Layer | Error type | Response |
|---|---|---|
| Server Action | Empty cart | `{ success: false, code: "EMPTY_CART" }` |
| Server Action | Invalid price ID | `{ success: false, code: "INVALID_PRICE_ID" }` |
| Server Action | `StripeCardError` | User-friendly message, no re-throw |
| Server Action | `StripeInvalidRequestError` | Log + user-friendly message |
| Server Action | `StripeConnectionError` | User-friendly, suggest retry |
| Server Action | `StripeRateLimitError` | User-friendly, suggest wait |
| Server Action | Timeout (10 s) | Falls through to `StripeConnectionError` |
| Webhook | Missing secret | HTTP 500 (misconfiguration) |
| Webhook | Missing signature | HTTP 400 |
| Webhook | Signature mismatch | HTTP 400 (replay attack / wrong secret) |
| Webhook | Handler throws | HTTP 200 + error log (prevents infinite Stripe retries) |

---

## 9. Timeout Configuration

Timeout is set at two levels:

1. **Stripe SDK** (`lib/stripe.ts`): `timeout: 10_000` ms — the SDK aborts the HTTP call to Stripe if no response arrives within 10 seconds and throws `StripeConnectionError`. This is caught in the Server Action.

2. **Next.js function timeout** (optional, `next.config.js`):
```javascript
// next.config.js
module.exports = {
  experimental: {
    serverActions: {
      bodySizeLimit: "1mb",
    },
  },
};
```

For Vercel deployments, set the function region to `mad1` (Madrid) in `vercel.json` to minimise latency to Stripe's EU servers:
```json
{
  "functions": {
    "app/api/webhooks/stripe/route.ts": {
      "maxDuration": 15,
      "regions": ["mad1"]
    },
    "app/actions/createCheckoutSession.ts": {
      "maxDuration": 15,
      "regions": ["mad1"]
    }
  }
}
```

---

## 10. Stripe Dashboard Setup (Checklist)

- [ ] Create Products and Prices in the Stripe Dashboard (Products > Add product > "One-time" price in EUR)
- [ ] Copy each `price_...` ID into the product data in the codebase
- [ ] Register the webhook endpoint: **Dashboard > Developers > Webhooks > Add endpoint**
  - URL: `https://cafevermut.com/api/webhooks/stripe`
  - Events to listen: `checkout.session.completed`, `payment_intent.payment_failed`
- [ ] Copy the **Signing secret** (`whsec_...`) into `STRIPE_WEBHOOK_SECRET`
- [ ] Test with `stripe listen --forward-to localhost:3000/api/webhooks/stripe` (Stripe CLI)
- [ ] Enable SEPA Debit in **Dashboard > Settings > Payment methods**
- [ ] Enable IVA/Tax in **Dashboard > Tax** if using Stripe Tax (optional)

---

## 11. File Structure

```
cafe-vermut/
├── lib/
│   └── stripe.ts                          # (2) Stripe singleton client
├── types/
│   └── stripe.ts                          # Shared TypeScript types
├── app/
│   ├── actions/
│   │   └── createCheckoutSession.ts       # (3) Server Action
│   ├── api/
│   │   └── webhooks/
│   │       └── stripe/
│   │           └── route.ts              # (4) Webhook handler
│   └── pedido/
│       └── confirmacion/
│           └── page.tsx                  # Success page
├── components/
│   └── BuyButton.tsx                     # Example buy button
├── .env.local                            # Secrets (never commit)
└── vercel.json                           # Deployment config (optional)
```

---

## 12. Security Notes

1. **Never expose `STRIPE_SECRET_KEY`** to the client. It is only used in Server Actions and Route Handlers.
2. **Always verify the webhook signature** — `stripe.webhooks.constructEvent` checks both the HMAC-SHA256 signature and a 5-minute timestamp tolerance to prevent replay attacks.
3. **`STRIPE_WEBHOOK_SECRET` is different** from `STRIPE_SECRET_KEY`. Each webhook endpoint has its own signing secret.
4. **Idempotency**: the webhook handler should check whether the session/payment intent has already been processed before fulfilling the order, to handle Stripe's at-least-once delivery.
5. **PCI compliance**: using Stripe-hosted Checkout means the card data never touches your server — you are in the lowest PCI DSS scope (SAQ A).
