# Integration Audit Report — Café Vermut (Checkout Flow)

**Date:** 2026-05-13
**Auditor:** Fullstack Developer
**Eval:** fullstack-eval2 — Type Mismatch between Server Action and Frontend

---

## Summary

The checkout integration between the `backend-developer`'s Server Action and the `frontend-developer`'s `OrderConfirmation.tsx` component is **critically broken** and cannot be shipped as-is. Three distinct failure classes were identified: (1) a complete structural type mismatch between the Server Action return shape and the component's expected props — the component will crash at runtime even though TypeScript may not catch it; (2) an N+1 query problem in the orders fetching logic that will degrade to catastrophic performance with any real order volume; and (3) an untyped `any[]` on `items` that defeats TypeScript's entire type system for the most business-critical field in the checkout domain. The integration is **blocked** until all Critical and High issues are resolved.

---

## PHASE 1 — Data Flow Audit

### Data Flow — Order (Checkout → Confirmation)

**Source:** Supabase table `orders` + `order_items`
**Fetched in:** Server Action (checkout) — unnamed function (backend-developer ownership)
**TypeScript type:** Inline — no shared type file identified
**Passed through:**
  1. Server Action returns `{ success: true, order: { id: string, status: string, items: any[], created_at: string } }`
  2. `OrderConfirmation.tsx` receives and destructures the response
**Rendered in:** `OrderConfirmation` component

**Checks:**
- [x] FAIL — Type at source does NOT match type at render (see Phase 4 for detail)
- [x] FAIL — `order.items` is `any[]` — null/undefined not guarded, item shape unknown
- [ ] Loading state — not audited (no loading skeleton confirmed in scope)
- [x] FAIL — Error state: if `success: false` is returned, the component has no fallback — it will attempt to destructure undefined fields
- [x] FAIL — No empty-state guard on items array before rendering

**Critical finding:** The Server Action returns `order.id` but the component reads `orderId`. The Server Action returns `order.created_at` (ISO timestamp string) but the component reads `estimatedDelivery` (different field, different semantic meaning). The Server Action returns no `total` field at all, but the component renders `total` as a `number`. These are three separate crash points.

---

## PHASE 2 — Form Flow Audit (Checkout)

### Form Flow — Checkout on /checkout

**Fields:** (inferred from domain — not fully audited, out of scope for this eval)
**Validation schema:** Not provided — not audited
**Server Action:** Checkout Server Action (backend-developer)

**Flow trace:**

1. User completes checkout → submits form
2. Server Action executes, writes to `orders` + `order_items`
3. Server Action returns `{ success: true, order: { id, status, items, created_at } }`
4. Frontend receives response
5. **CRASH POINT:** `OrderConfirmation.tsx` destructures `orderId`, `estimatedDelivery`, `total` — none of which exist at the top level of the response. Component renders `undefined` for all three fields or throws depending on how destructuring is applied.

**Type consistency check:**
- [x] FAIL — Client/consumer expects `{ orderId: string, estimatedDelivery: string, total: number }`
- [x] FAIL — Server Action returns `{ success: true, order: { id: string, status: string, items: any[], created_at: string } }`
- [x] FAIL — No shared type contract enforced between layers

**Specific field mismatches:**

| Server Action returns | Component expects | Issue |
|---|---|---|
| `order.id` | `orderId` | Wrong nesting level + renamed |
| `order.created_at` | `estimatedDelivery` | Different field entirely — semantic mismatch |
| *(not returned)* | `total` | Missing — will be `undefined`, rendered as `NaN` or blank |
| `order.status` | *(not consumed)* | Unused — not necessarily a bug but a gap |
| `order.items: any[]` | *(not consumed directly)* | `any[]` is a type safety failure regardless |

---

## PHASE 3 — Auth Flow Audit

Not in scope for this specific eval task (type mismatch + N+1 focus). Flag for separate audit pass.

---

## PHASE 4 — Type Safety End-to-End

### Issue 1 — Structural Type Mismatch (CRITICAL)

**Root cause:** The Server Action wraps the order under a `order` key, but the component expects the fields flat at the top level (or with different names). There is no shared TypeScript interface enforcing the contract between the two.

**Current Server Action return type (inferred):**
```typescript
type ServerActionReturn = {
  success: true
  order: {
    id: string
    status: string
    items: any[]
    created_at: string
  }
}
```

**What OrderConfirmation.tsx expects:**
```typescript
type OrderConfirmationProps = {
  orderId: string
  estimatedDelivery: string
  total: number
}
```

**The gap — field by field:**

- `orderId` — Server returns `order.id`. Consumer reads `orderId`. At the component boundary, this resolves to `undefined`. If rendered inside JSX it silently renders nothing. If used in a fetch call or passed to a child component expecting `string`, it throws at runtime.

- `estimatedDelivery` — Server returns `order.created_at` (order creation timestamp). Consumer reads `estimatedDelivery` (an estimated delivery date — completely different semantic). Even if the field names were reconciled, the data would be semantically wrong: showing the order creation time as "estimated delivery" is a product bug, not just a type bug.

- `total` — Server Action does not return any `total` field. The component expects a `number`. At runtime this is `undefined`. When rendered (`{total}` in JSX), it displays as blank. If passed to any arithmetic (`total * taxRate`), it produces `NaN` silently.

**Fix required (backend-developer must implement):**

Option A — Align the Server Action return shape to what the component expects:
```typescript
// Server Action — updated return shape
return {
  success: true,
  orderId: order.id,
  estimatedDelivery: calculateEstimatedDelivery(order.created_at), // real logic needed
  total: order.total_amount, // must exist in DB schema
}
```

Option B — Create a shared contract type in a shared types file and enforce it on both sides:
```typescript
// lib/types/checkout.ts
export interface CheckoutResult {
  success: boolean
  orderId: string
  estimatedDelivery: string   // ISO string, formatted at render layer
  total: number
}
```

Option B is strongly preferred. Without a shared type, this mismatch will recur every time either developer modifies their code independently.

---

### Issue 2 — `any[]` on `items` (HIGH)

**Location:** Server Action return type — `order.items: any[]`

**Problem:** `any[]` completely disables TypeScript for the entire items array. This means:
- No autocomplete on item fields
- No compile-time error if an item field is renamed in the DB and the frontend still reads the old name
- No guarantee that item shape is consistent between what Supabase returns and what any consuming component renders
- `any` propagates — every variable that receives a value from `items[n]` also becomes `any`, silently spreading the type hole

**Impact:** The `items` field is the most data-rich part of an order. It typically contains product name, quantity, unit price, subtotal. If any of these fields are missing, renamed, or null, no TypeScript error will occur — the bug surfaces only at runtime, potentially after a customer has completed payment.

**Fix required:**

Step 1 — Define the item type (from Supabase schema):
```typescript
// lib/types/order.ts
export interface OrderItem {
  id: string
  order_id: string
  product_id: string
  product_name: string
  quantity: number
  unit_price: number
  subtotal: number
}
```

Step 2 — Replace `any[]` with the typed array:
```typescript
// Server Action return type — corrected
order: {
  id: string
  status: 'pending' | 'confirmed' | 'preparing' | 'delivered' | 'cancelled'  // enum, not string
  items: OrderItem[]
  created_at: string
}
```

Step 3 — Generate Supabase types via CLI and use them as the source of truth:
```bash
supabase gen types typescript --project-id <project-id> > lib/types/supabase.ts
```

Then derive `OrderItem` from the generated types rather than writing it manually:
```typescript
import { Database } from '@/lib/types/supabase'
export type OrderItem = Database['public']['Tables']['order_items']['Row']
```

---

### Issue 3 — `status: string` instead of union type (MEDIUM)

**Location:** Server Action return — `status: string`

**Problem:** The `status` field in an orders domain is an enum with a fixed set of values (`pending`, `confirmed`, `preparing`, `delivered`, `cancelled` — or similar). Typing it as `string` means:
- A typo like `'confirmd'` compiles without error
- Exhaustive switch/case on status receives no TypeScript warning when a new status value is added to the DB
- UI components that conditionally render based on status (e.g., show "Your order is being prepared" banner) have no type guard

**Fix:**
```typescript
type OrderStatus = 'pending' | 'confirmed' | 'preparing' | 'delivered' | 'cancelled'
```

---

## PHASE 5 — N+1 Query Detection and Fix

### N+1 Query — Orders + Order Items

**Location:** Orders query (backend-developer ownership)

**Current code:**
```typescript
const orders = await supabase.from('orders').select('*')
for (const order of orders.data) {
  const items = await supabase.from('order_items').eq('order_id', order.id)
}
```

**Problem analysis:**

This executes `1 + N` database queries where N is the number of orders:
- 1 query to fetch all orders
- 1 query PER order to fetch its items

With 10 orders: 11 queries
With 100 orders: 101 queries
With 1,000 orders: 1,001 queries

Each query has network latency to Supabase (typically 20–80ms). At 100 orders, this is 2–8 seconds of cumulative latency just for data fetching — before any rendering. At 1,000 orders, this is a timeout in production.

Additional bugs in the current code:
- `.eq('order_id', order.id)` is missing `.select('*')` — this call is malformed and may return an error silently depending on the Supabase client version
- `orders.data` is accessed without null check — if the query fails, `orders.data` is `null` and the `for...of` loop throws a `TypeError: null is not iterable`

**Fix — Solution A: Supabase relational join (preferred)**

Use Supabase's built-in foreign key traversal to fetch everything in a single query:

```typescript
const { data: orders, error } = await supabase
  .from('orders')
  .select(`
    id,
    status,
    created_at,
    total_amount,
    order_items (
      id,
      product_id,
      product_name,
      quantity,
      unit_price,
      subtotal
    )
  `)

if (error) throw new Error(`Failed to fetch orders: ${error.message}`)
if (!orders) return []
```

This produces exactly 1 query. Supabase handles the join at the PostgREST layer.

**Result type** (automatically inferred by Supabase client):
```typescript
Array<{
  id: string
  status: string
  created_at: string
  total_amount: number
  order_items: Array<{
    id: string
    product_id: string
    product_name: string
    quantity: number
    unit_price: number
    subtotal: number
  }>
}>
```

**Fix — Solution B: Two queries + in-memory join (fallback if foreign key not set up)**

```typescript
// Query 1 — fetch all orders
const { data: orders, error: ordersError } = await supabase
  .from('orders')
  .select('*')

if (ordersError || !orders) throw new Error('Failed to fetch orders')

// Query 2 — fetch ALL items for all orders in a single IN query
const orderIds = orders.map(o => o.id)
const { data: allItems, error: itemsError } = await supabase
  .from('order_items')
  .select('*')
  .in('order_id', orderIds)

if (itemsError || !allItems) throw new Error('Failed to fetch order items')

// In-memory join — O(n) with Map
const itemsByOrderId = new Map<string, typeof allItems>()
for (const item of allItems) {
  const existing = itemsByOrderId.get(item.order_id) ?? []
  existing.push(item)
  itemsByOrderId.set(item.order_id, existing)
}

const enrichedOrders = orders.map(order => ({
  ...order,
  items: itemsByOrderId.get(order.id) ?? [],
}))
```

This is exactly 2 queries regardless of how many orders exist. Solution A is preferred when the foreign key relationship exists in the schema.

**Performance comparison:**

| Approach | Orders=10 | Orders=100 | Orders=1000 |
|---|---|---|---|
| Current (N+1) | 11 queries | 101 queries | 1001 queries |
| Solution A (join) | 1 query | 1 query | 1 query |
| Solution B (2 queries) | 2 queries | 2 queries | 2 queries |

---

## PHASE 6 — Error Boundaries

Not fully audited in scope of this eval. The following gaps are noted from the type mismatch analysis:

- If `OrderConfirmation.tsx` crashes due to `undefined` props (which it will with current code), there is no evidence of an `error.tsx` boundary wrapping the checkout confirmation route segment. Without it, the entire page throws a white screen with no recovery path.

**Minimum required:**
```
app/
├── error.tsx                    # Root boundary — must exist
├── checkout/
│   ├── error.tsx                # Catches checkout-specific failures
│   └── confirmation/
│       └── error.tsx            # Catches confirmation render failures
```

---

## PHASE 7 — Manual Testing Notes

These flows would fail immediately under manual testing with the current code:

### Manual Test — Checkout Confirmation Flow

**Happy Path Result: FAIL**
- Step 1: User completes checkout form → Server Action executes successfully
- Step 2: Server Action returns `{ success: true, order: { id, status, items, created_at } }`
- Step 3: `OrderConfirmation.tsx` renders → `orderId` is `undefined`, `estimatedDelivery` is `undefined`, `total` is `undefined`
- Step 4: Either silent blank fields (broken UX) or component crash (TypeError) depending on implementation

**Root cause:** The structural type mismatch means the happy path produces broken output even when everything else works correctly.

---

## Issues Found

### Critical (must fix before launch)

- [ ] **Type mismatch — orderId**: Server Action returns `order.id` (nested) but component reads `orderId` (flat). Runtime value is `undefined`. File: Server Action + `OrderConfirmation.tsx` — Component renders blank order ID or crashes.

- [ ] **Type mismatch — estimatedDelivery**: Server Action returns `order.created_at` (creation timestamp) but component reads `estimatedDelivery` (different field, different semantic). Runtime value is `undefined`. Even if fixed to the same field, showing creation time as delivery estimate is a product bug. File: Server Action + `OrderConfirmation.tsx`.

- [ ] **Missing field — total**: Server Action does not return any `total` field. Component reads `total: number`. Runtime value is `undefined`. If rendered: blank. If used in math: `NaN`. File: Server Action — field must be added from `orders.total_amount` or computed from items.

- [ ] **N+1 query**: `for (const order of orders.data) { await supabase.from('order_items').eq(...) }` executes 1+N database queries. Will timeout in production at scale. File: Orders query function (backend-developer). Fix: use Supabase relational join `.select('*, order_items(*)')` or 2-query in-memory join pattern.

- [ ] **Null dereference on orders.data**: `orders.data` accessed without null check before `for...of` loop. If Supabase query fails (network error, RLS policy rejection), this throws `TypeError: null is not iterable` — unhandled, crashes the request. File: Orders query function.

### High (should fix before launch)

- [ ] **`any[]` on items**: `order.items: any[]` disables TypeScript for the entire items array. Silent type hole that will cause runtime errors when item fields change. File: Server Action return type. Fix: define `OrderItem` interface from Supabase generated types.

- [ ] **No shared type contract**: Server Action return type and component prop type are defined independently with no shared interface. Any independent change to either will re-create the mismatch silently. File: needs new `lib/types/checkout.ts` with `CheckoutResult` interface enforced on both sides.

- [ ] **Malformed Supabase query**: `.eq('order_id', order.id)` missing `.select('*')` call — may return unexpected shape or silent error depending on client version. File: Orders query function (N+1 loop).

### Medium (should fix, not launch-blocking)

- [ ] **`status: string` instead of union type**: Order status should be `'pending' | 'confirmed' | 'preparing' | 'delivered' | 'cancelled'`. Typed as `string` prevents exhaustive checks and allows typos. File: Server Action return type + any component that conditionally renders based on status.

- [ ] **`created_at` not formatted**: Raw ISO timestamp string passed through without formatting. If any component renders it directly, user sees `2026-05-13T14:32:00.000Z` instead of a human-readable date. File: Rendering layer — formatting should happen in one place only.

### Low (nice to fix)

- [ ] **`order.status` returned but not consumed**: Server Action returns `status` but `OrderConfirmation.tsx` doesn't use it. Either the component is missing a status display (product gap) or the field is unnecessary noise in the response.

- [ ] **No error boundary on confirmation route**: If `OrderConfirmation.tsx` throws (which it will currently), no `error.tsx` is in place to catch it gracefully. User sees white screen with no recovery option.

---

## Data Flows Verified

- [ ] Order (checkout → confirmation) — FAILED: type mismatch at Server Action → component boundary
- [ ] Order Items — FAILED: N+1 query + `any[]` type hole

## Forms Verified

- [ ] Checkout form — NOT verified (out of scope for this eval — separate audit needed for form validation, Zod schema, and submission states)

## Auth Flows Verified

- [ ] Not in scope for this eval

## N+1 Queries Found and Fixed

- [ ] `orders` + `order_items` loop — FOUND, fix documented above (Solution A: Supabase join; Solution B: 2-query in-memory map). Fix must be implemented by backend-developer.

## Error Boundaries in Place

- [ ] Root `error.tsx` — not confirmed
- [ ] `/checkout/confirmation/error.tsx` — not confirmed, required

---

## Recommendation

**BLOCKED — not ready for QA.**

Three critical runtime failures will occur on the checkout confirmation page even under perfect network conditions and correct user input:
1. `orderId` renders as `undefined`
2. `estimatedDelivery` renders as `undefined`
3. `total` renders as `undefined` / `NaN`

Additionally, the N+1 query is a production incident waiting to happen — it is harmless in development with seeded test data but will cause timeouts with real order volume.

**Required before QA handoff:**
1. Backend-developer must align Server Action return shape to `CheckoutResult` shared interface (or both sides adapt to the shared contract)
2. Backend-developer must add `total` field to Server Action return
3. Backend-developer must replace N+1 loop with Supabase relational join
4. Backend-developer must add null guard on `orders.data` before iteration
5. Frontend-developer or backend-developer must create `lib/types/checkout.ts` with shared `CheckoutResult` + `OrderItem` interfaces
6. `any[]` on items must be replaced with typed `OrderItem[]`
7. Error boundary (`error.tsx`) must be added to the checkout confirmation route segment

Once all Critical and High issues are resolved, re-audit the checkout flow end-to-end before passing to quality-reviewer.
