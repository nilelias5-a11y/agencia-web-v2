# Integration Audit Report — Ana López Fotógrafa
## Contact Form Integration Audit (Fullstack Eval 1 — Baseline)

**Date:** 2026-05-13  
**Auditor:** Fullstack Developer  
**Project:** Ana López Fotógrafa — Photography portfolio website  
**Scope:** Contact form integration — `ContactForm.tsx` ↔ `submitContact` Server Action ↔ Resend

---

## Executive Summary

The contact form integration for Ana López Fotógrafa follows a sound architectural pattern: a React 19 `useActionState` Client Component drives a typed Server Action that validates with Zod, applies rate limiting via Upstash, and sends dual emails through Resend. The core happy path is solid. However, the audit surfaces **3 critical issues**, **4 high-priority issues**, and **5 low-priority issues** that must be addressed before the form is considered production-ready.

The most significant risk is a **type mismatch between the Server Action return type and the frontend error consumption pattern**: the backend returns `fieldErrors` as `Partial<Record<keyof ContactInput, string[]>>` but the frontend accesses `state.errors` (per the task description), while the implementation uses `state.fieldErrors`. This mismatch means field-level errors are silently swallowed in the current frontend reference — the form shows no per-field feedback on invalid input.

---

## Part 1 — Data Flow Audit: Entity `ContactSubmission`

```
## Data Flow — ContactSubmission

Source:          User input via HTML form (no database persistence)
Entry point:     ContactForm.tsx — browser FormData submitted via useActionState
Validated in:    src/app/actions/contact.ts — submitContact() Server Action
                 src/lib/validations/contact.ts — Zod contactSchema
Email sent via:  src/lib/email.ts — sendContactEmails()
Delivered to:    (1) Ana López — CONTACT_EMAIL_TO env var
                 (2) End user — data.email (from validated FormData)

TypeScript type chain:
  FormData (browser built-in)
    → raw object { name, email, sessionType, message } — untyped
      → contactSchema.safeParse(raw) — Zod validates
        → result.data — typed as ContactInput (z.infer<typeof contactSchema>)
          → sendContactEmails(data: ContactInput)
            → ContactNotificationEmail(props) — typed via ContactNotificationEmailProps
            → ContactConfirmationEmail(props) — typed via ContactConfirmationEmailProps
  Return value:
    → ContactActionResult (discriminated union)
      → useActionState state — typed as ContactActionResult | null
        → JSX conditional render — success / fieldErrors / global error
```

### Data Flow Checks

- [x] **Type at source matches type at render** — `ContactInput` is used consistently from Zod inference through email props. No silent widening.
- [FAIL] **Null/undefined cases handled at every step** — `formData.get()` returns `FormDataEntryValue | null`. This null is passed into `contactSchema.safeParse()` as the raw object. Zod will reject `null` values as validation failures (not crashes), so this is handled — but the `raw` object is typed as `{ name: FormDataEntryValue | null, email: FormDataEntryValue | null, ... }` which TypeScript accepts without complaint because `safeParse` takes `unknown`. However, if a field is missing from FormData entirely (programmatic submission), Zod catches it at validation. This is acceptable but worth noting.
- [x] **Loading state rendered** — `isPending` from `useActionState` correctly disables the submit button and shows "Enviando…" text while the Server Action is in flight.
- [FAIL] **Error state rendered correctly** — See Critical Issue #1 below. The frontend reference uses `state.fieldErrors` but the task specification states the Server Action returns `{ success: boolean, errors?: Record<string, string[]>, message?: string }`. There is a key name mismatch (`fieldErrors` vs `errors`) and a shape mismatch (`Partial<Record<keyof ContactInput, string[]>>` vs `Record<string, string[]>`).
- [x] **Empty/null state handled** — Initial state is `null`. All conditionals guard with `state?.` optional chaining. The form renders normally when `state` is null.

### Data Transformation Points

| Step | Location | Transformation | Risk |
|------|----------|----------------|------|
| 1 | Browser | User input → FormData | None — native browser API |
| 2 | Server Action | `formData.get("email")` → string or null | Null if field missing — handled by Zod |
| 3 | Zod | `.toLowerCase()` and `.trim()` applied to email and name | Normalization happens server-side only — client shows user's original casing |
| 4 | `sendContactEmails` | `ContactInput` → `sessionLabel` via `SESSION_TYPE_LABELS` | Safe enum lookup — typed `Record<SessionType, string>` |
| 5 | Resend | React Email component → HTML string | React Email renders server-side — no XSS risk |
| 6 | Return | `ContactActionResult` discriminated union | Frontend must correctly narrow the union — see Issue #1 |

---

## Part 2 — Form Flow Audit: 8-Step Trace

```
## Form Flow — ContactForm on /contacto (or equivalent)

Fields:
  - name        string — text input, required
  - email       string — email input, required
  - sessionType "bodas" | "retratos" | "corporativo" — select, required
  - message     string — textarea, required

Validation schema: src/lib/validations/contact.ts — contactSchema (Zod)
Server Action:     src/app/actions/contact.ts — submitContact(formData: FormData)
```

### Step-by-Step Flow Trace

**Step 1 — User fills form fields**

The form is rendered as a standard HTML `<form action={formAction}>`. Each field has `id`, `name`, and `required` attributes. The `required` HTML attribute provides native browser validation as a first line of defense (triggers before the form submits). No client-side Zod validation is performed at this step.

- Risk: No `onChange` handlers or real-time validation feedback. User discovers errors only on submit. For a contact form, this is acceptable UX — not a bug.
- No `value`/`defaultValue` binding is present in the reference implementation. This means form fields do not repopulate with the user's data after a failed submission (fields reset to empty). This is a UX issue (see High Issue #3).

**Step 2 — User clicks "Enviar mensaje"**

React calls `formAction` (the bound Server Action). `useActionState` sets `isPending = true`. The submit button is disabled (`disabled={isPending}`) and shows "Enviando…". This correctly prevents double submission during the in-flight request.

- The `noValidate` attribute on the form suppresses native browser validation in favor of server-side validation. This is intentional in the implementation but creates an accessibility gap: if JavaScript fails to load, the form submits with no client-side validation at all (see Low Issue #5).

**Step 3 — Client validation**

No client-side Zod validation exists. The only client-side gate is HTML `required` attributes — but these are disabled by `noValidate`. In practice, with JavaScript enabled and React mounted, all validation is deferred entirely to the server.

- This is a deliberate architectural choice (server-only validation). Acceptable for simple contact forms, but means every validation error requires a round-trip.

**Step 4 — Network request (Server Action)**

`useActionState` serializes the form's `FormData` and sends it to the Next.js Server Action runtime via a POST request to the Next.js internal endpoint. This is not a standard `fetch` — it is handled by the React/Next.js framework.

- The request is encrypted in transit (HTTPS in production). The `FormData` is transmitted as multipart/form-data.
- No CSRF token is needed — Next.js Server Actions are protected by the framework's own CSRF mechanism (Origin header validation).

**Step 5 — Server validation (Zod + rate limit)**

Two server-side gates execute in sequence:

1. **Rate limit check** (Upstash sliding window, 3 req / 10 min per IP):
   - IP extracted from `x-forwarded-for` header (first IP to prevent spoofing)
   - Fail-open on Upstash unreachability — logged, not thrown
   - Returns `{ success: false, error: "Demasiadas solicitudes..." }` on limit exceeded

2. **Zod validation** (`contactSchema.safeParse(raw)`):
   - `name`: min 2, max 100, trimmed
   - `email`: email format, lowercased, trimmed
   - `sessionType`: enum literal `"bodas" | "retratos" | "corporativo"`
   - `message`: min 10, max 2000, trimmed
   - Returns `{ success: false, error: "...", fieldErrors: {...} }` on failure

**Step 6 — Success path**

If both gates pass and `sendContactEmails` succeeds (notification sent to Ana):
- Returns `{ success: true }`
- `useActionState` updates `state` to `{ success: true }`
- Frontend renders the success message: `"¡Mensaje enviado! Te confirmaré por email en breve."`
- The form is replaced by the success paragraph — correct, prevents re-submission
- No redirect occurs — user stays on the page with inline success feedback

**Step 7 — Error path (server-side validation failures)**

There are three distinct server error scenarios:

| Scenario | Return shape | Frontend rendering |
|----------|-------------|-------------------|
| Rate limit exceeded | `{ success: false, error: string }` | Global error paragraph |
| Zod field validation fails | `{ success: false, error: string, fieldErrors: {...} }` | BROKEN — see Critical Issue #1 |
| Email send fails (notification) | `{ success: false, error: string }` | Global error paragraph |
| Unexpected exception | `{ success: false, error: string }` | Global error paragraph |

The global error is conditionally rendered only when `!state.success && !state.fieldErrors`. However, since `state.fieldErrors` is defined when Zod validation fails, this condition prevents the global summary message from showing — the user would see per-field errors but no general message. For the rate-limit and email-failure cases (where `fieldErrors` is undefined), the global message shows correctly.

**Step 8 — Error path (network / infrastructure failure)**

If the Next.js Server Action runtime itself fails (network timeout, server crash):

- `useActionState` does not automatically catch this — the promise rejects
- Without an error boundary, this throws an unhandled promise rejection
- React 19 will propagate the error to the nearest `error.tsx` boundary
- If no `error.tsx` exists on the contact page route, it bubbles to the root `error.tsx`
- The form never shows a user-friendly error — the page replaces with the error boundary UI

This is **not handled at the form level** — it requires an `error.tsx` on the route (see Critical Issue #2).

---

## Part 3 — Type Consistency Check

### Server Action Return Type vs. Task Specification vs. Implementation

The task specification states the Server Action returns:
```typescript
{ success: boolean, errors?: Record<string, string[]>, message?: string }
```

The actual implementation in `src/app/actions/contact.ts` returns:
```typescript
type ContactActionResult =
  | { success: true }
  | { success: false; error: string; fieldErrors?: Partial<Record<keyof ContactInput, string[]>> }
```

The frontend reference in `backend-eval1` uses:
```typescript
state?.fieldErrors?.name   // uses "fieldErrors" key
state?.error               // uses "error" key (not "message")
```

**Discrepancies identified:**

| Property | Task spec | Implementation | Frontend access | Match? |
|----------|-----------|---------------|----------------|-------|
| Field errors key | `errors` | `fieldErrors` | `state.fieldErrors` | MISMATCH vs spec |
| Global error key | `message` | `error` | `state.error` | MISMATCH vs spec |
| Field errors type | `Record<string, string[]>` | `Partial<Record<keyof ContactInput, string[]>>` | `.name[0]`, `.email[0]`, etc. | Shape compatible, but `Partial` adds undefined possibility |

**Assessment:**

The backend implementation and the frontend reference are internally consistent with each other — both use `fieldErrors` and `error`. The mismatch is between the task specification's described contract (`errors`, `message`) and what was actually implemented. This is a contract documentation drift issue. If other consumers (future admin panels, monitoring integrations) are coded against the spec as written, they would receive `undefined` for both `errors` and `message`.

**TypeScript type safety of the discriminated union:**

The `ContactActionResult` discriminated union is well-formed. TypeScript can narrow it with `if (state.success)`. However:

1. The `success: false` branch contains both `error: string` (always present) and `fieldErrors?: Partial<...>` (optional). This means consumers must check `state.fieldErrors` to determine if it's a validation error vs. a general error — which the frontend correctly does.

2. `Partial<Record<keyof ContactInput, string[]>>` means each field key is `string[] | undefined`. The frontend accesses `state.fieldErrors.name[0]` without guarding against `state.fieldErrors.name` being `undefined`. Under `noUncheckedIndexedAccess: true` (required tsconfig), this would be a compile error. Under standard strict mode, it silently fails at runtime if the key is absent.

### `useActionState` Signature Compatibility

The `useActionState` hook in React 19 has the signature:
```typescript
useActionState<State, Payload>(
  action: (state: Awaited<State>, payload: Payload) => State | Promise<State>,
  initialState: Awaited<State>,
  permalink?: string
): [state: Awaited<State>, dispatch: (payload: Payload) => void, isPending: boolean]
```

The Server Action signature in the implementation is:
```typescript
submitContact(formData: FormData): Promise<ContactActionResult>
```

For `useActionState` to work with this, it expects the action to accept `(previousState: ContactActionResult | null, formData: FormData)` — a two-argument signature where the first argument is the previous state.

**CRITICAL:** The current implementation of `submitContact` only accepts one argument: `formData: FormData`. It does not accept `previousState` as the first parameter. React 19's `useActionState` will pass `previousState` as the first argument and `formData` as the second. This means `formData` is actually received in the `previousState` parameter slot — the action will fail to extract form fields correctly.

This is **Critical Issue #1** — the most severe bug in the integration.

### Zod Schema vs. Form Field Names

The Zod schema fields and HTML `name` attributes must match exactly for `formData.get("fieldName")` to retrieve the correct value:

| Zod field | HTML `name` attribute | `formData.get()` key | Match? |
|-----------|----------------------|---------------------|--------|
| `name` | `name` | `"name"` | YES |
| `email` | `name="email"` | `"email"` | YES |
| `sessionType` | `name="sessionType"` | `"sessionType"` | YES |
| `message` | `name="message"` | `"message"` | YES |

All field names match. No mismatch at the FormData extraction layer.

### Email Template Props vs. `sendContactEmails` Arguments

| Prop passed | Expected by template | Type | Match? |
|-------------|---------------------|------|--------|
| `name` | `ContactNotificationEmailProps.name` | `string` | YES |
| `email` | `ContactNotificationEmailProps.email` | `string` | YES |
| `sessionType` | `ContactNotificationEmailProps.sessionType` | `SessionType` | YES |
| `sessionLabel` | `ContactNotificationEmailProps.sessionLabel` | `string` | YES |
| `message` | `ContactNotificationEmailProps.message` | `string` | YES |
| `name` | `ContactConfirmationEmailProps.name` | `string` | YES |
| `sessionLabel` | `ContactConfirmationEmailProps.sessionLabel` | `string` | YES |

All email template props are correctly typed and consistently passed.

---

## Part 4 — Edge Cases to Test

### Group A: Input Validation Edge Cases

1. **Empty form submission**  
   Submit with all fields empty. Expected: Zod returns errors for all 4 fields. Actual behavior to verify: field errors appear per-field (blocked by Critical Issue #1 — `previousState` bug must be fixed first).

2. **`name` field at boundary values**  
   - 1 character: "A" → should fail (`min(2)`)
   - 2 characters: "Ab" → should pass
   - 100 characters: "A".repeat(100) → should pass
   - 101 characters: "A".repeat(101) → should fail (`max(100)`)

3. **`message` field at boundary values**  
   - 9 characters: "Hello!!" → should fail (`min(10)`)
   - 10 characters: "Hello!!!!!" → should pass
   - 2000 characters: "A".repeat(2000) → should pass
   - 2001 characters: "A".repeat(2001) → should fail (`max(2000)`)
   - Paste 10,000 characters → should fail with `max(2000)` error, form does not crash

4. **Invalid email formats**  
   - "notanemail" → should fail
   - "user@" → should fail
   - "user@domain" → Zod's email validator may or may not accept this (TLD-less domains) — verify behavior
   - "user+tag@domain.com" → should pass (plus addressing)
   - Unicode email "üser@domain.com" → should fail or pass depending on Zod version — document expected behavior

5. **`sessionType` manipulation**  
   - Submit with an unlisted value (e.g., programmatic POST with `sessionType=hacking`) → Zod enum validation rejects it with a controlled error — verify this is tested and does not cause a crash

6. **XSS attempts in text fields**  
   - `name`: `<script>alert('xss')</script>` → Zod accepts (no HTML stripping), React Email renders as escaped text, no XSS in email — SAFE because React escapes JSX expressions. Verify the notification email does not render raw HTML from user input.
   - `message`: Same as above — verify email templates use `{message}` JSX expression (safe) not `dangerouslySetInnerHTML` (unsafe). Current implementation uses `{message}` in a `<Text>` component — CORRECT.

7. **SQL injection attempts** (not applicable — no database, but note for completeness)  
   No database writes in this flow. Resend receives the raw string as email body content — no injection vector.

8. **Unicode and special characters**  
   - Spanish characters: "García Ñoño" → must pass (`.trim()` handles surrounding whitespace, unicode chars are valid)
   - Emoji in name: "Ana 📸 López" → Zod string accepts emoji — verify email rendering handles it correctly
   - Null byte injection: `"field\x00value"` → verify Zod does not crash (it should not — JavaScript strings handle null bytes)
   - CRLF injection in message: `"line1\r\nContent-Type: text/html"` — relevant for email header injection. The `message` field is only used in the email body (not headers), so no injection risk. Verify.

### Group B: Concurrency and Network Edge Cases

9. **Double submission (rapid click)**  
   The submit button is disabled during `isPending`. Verify:
   - Click submit → button disabled immediately → second click has no effect
   - Via DevTools, manually re-enable button and click again → second FormData POST dispatched → verify server handles it gracefully (rate limiter is the backstop)
   - Result: rate limiter allows first 3 submissions within 10 minutes. A burst of 2 rapid legitimate submissions would succeed. This is acceptable.

10. **Submit while offline**  
    Disconnect network → submit form → Server Action POST fails at network level → `useActionState` receives a rejected promise → without error boundary, React propagates to nearest `error.tsx`. Verify:
    - Does the page show a blank error screen or a useful message?
    - Does the form remain mounted (ideal) or replace with error UI (current behavior with no form-level error handling)?
    - Expected fix: wrap `useActionState` dispatch in try/catch — but React 19's `useActionState` handles this through React's error mechanism, not try/catch. An `error.tsx` boundary is required.

11. **Slow network (latency > 10s)**  
    - `isPending` should remain `true` for the entire duration — button stays disabled
    - No visual timeout after N seconds — no loading spinner auto-cancels
    - Verify no timeout is configured on the Server Action (Next.js default timeout applies — ~30s for Vercel)

12. **Upstash Redis unavailable**  
    The implementation uses a fail-open strategy: if `contactRatelimit.limit(ip)` throws, it catches the error, logs it, and continues with `rateLimitResult = { success: true, ... }`. Verify:
    - The catch block correctly populates all required fields of the mock result object (`limit`, `remaining`, `reset`, `pending`)
    - `pending: Promise.resolve()` is valid but may cause TypeScript errors if the Upstash return type requires a specific Promise shape — verify compilation

13. **Resend API unavailable or returns 5xx**  
    `sendContactEmails` uses `Promise.allSettled`. If Resend returns an error:
    - `notification.status === "rejected"` → `notificationSent = false`
    - Server Action returns `{ success: false, error: "No hemos podido enviar..." }` (for notification failure)
    - Verify: does the user see the actionable error message including the direct email address?

14. **RESEND_API_KEY invalid or expired**  
    - T3 Env validates the key starts with `re_` at build time — this catches obviously wrong keys at startup
    - A valid-format but expired key passes T3 Env validation and fails at runtime
    - Failure is caught by the try/catch in the Server Action and returns a generic error to the client

### Group C: Accessibility and UX Edge Cases

15. **Form submission with JavaScript disabled**  
    - With `noValidate` set and no `action` URL configured (Server Actions bind to JS-only dispatch), the form does nothing on submit without JS
    - Progressive enhancement is not implemented — this is a known limitation for Server Actions with `useActionState`
    - Verify: does the form at minimum show a usable state for JS-disabled users, or does it submit to a broken endpoint?

16. **Screen reader experience on error**  
    - When field errors appear, do screen readers announce them?
    - The error `<p>` elements have no `role="alert"` or `aria-live` attribute — errors are rendered inline but not announced dynamically
    - Users navigating with VoiceOver/NVDA will not hear the error message unless they navigate back to the field
    - See High Issue #4

17. **Focus management after submission**  
    - After a failed submission, where is keyboard focus?
    - After a successful submission (form replaced by success message), where is keyboard focus?
    - Neither is managed in the current implementation

---

## Part 5 — Issues Report

---

### CRITICAL (must fix before launch)

---

**CRITICAL-1: Server Action signature incompatible with `useActionState`**

- **Location:** `src/app/actions/contact.ts` — `submitContact` function signature
- **Description:** React 19's `useActionState(action, initialState)` passes the action function with signature `(previousState: State, payload: Payload)`. The current implementation defines `submitContact(formData: FormData)` with only one parameter. When invoked by `useActionState`, `previousState` (type `ContactActionResult | null`) is passed as the first argument and `FormData` as the second. The function then calls `formData.get("name")` on what is actually a `ContactActionResult | null` — this throws a runtime TypeError: `formData.get is not a function`.
- **Impact:** The form cannot submit at all. Every submission attempt results in an unhandled server error. This is a complete functional failure.
- **Fix:**
  ```typescript
  // src/app/actions/contact.ts
  export async function submitContact(
    _previousState: ContactActionResult | null,  // required first param
    formData: FormData
  ): Promise<ContactActionResult> {
    // ... rest of implementation unchanged
  }
  ```
  The `_previousState` parameter is prefixed with `_` to signal intentional non-use. The function body requires no other changes.

---

**CRITICAL-2: No error boundary on the contact form route**

- **Location:** Route containing the contact form (e.g., `app/contacto/error.tsx` or equivalent)
- **Description:** If the Server Action runtime throws (network failure, Next.js internal error, unhandled exception escaping the try/catch), React propagates the error to the nearest `error.tsx`. If none exists for the contact route, it bubbles to the root `error.tsx`. The user sees the full error boundary page — the contact form disappears. There is no way to recover without a page reload.
- **Impact:** Any infrastructure hiccup during form submission destroys the form context. Users who were mid-message lose their input and see a confusing error screen.
- **Fix:** Create `app/contacto/error.tsx` (or the relevant route segment) with a recoverable error boundary that shows a friendly message and a "Try again" button. The `reset()` function from the error boundary remounts the form.
  ```tsx
  // app/contacto/error.tsx
  "use client"
  export default function ContactError({
    error,
    reset,
  }: {
    error: Error & { digest?: string }
    reset: () => void
  }) {
    return (
      <div>
        <p>No hemos podido procesar tu mensaje. Por favor, inténtalo de nuevo.</p>
        <button onClick={reset}>Reintentar</button>
      </div>
    )
  }
  ```

---

**CRITICAL-3: Type contract drift between specification and implementation**

- **Location:** `src/app/actions/contact.ts` — `ContactActionResult` type definition
- **Description:** The established API contract specifies the Server Action returns `{ success: boolean, errors?: Record<string, string[]>, message?: string }`. The implementation returns `{ success: true } | { success: false, error: string, fieldErrors?: Partial<Record<keyof ContactInput, string[]>> }`. The key names differ (`errors` vs `fieldErrors`, `message` vs `error`). Any consumer coded against the documented contract (including future integrations, tests, or a different frontend developer's component) will receive `undefined` for both `errors` and `message`.
- **Impact:** Breaks the documented API contract. Creates a maintenance trap where the specification and implementation diverge silently.
- **Options:**
  - Option A: Update the implementation to match the spec (`errors`, `message`). Refactor the frontend reference component accordingly.
  - Option B: Update the spec to match the implementation (`fieldErrors`, `error`). Simpler but requires updating all documentation.
  - Recommendation: Option A — align implementation to spec, as the spec's naming (`errors` for field errors, `message` for global message) is more semantically clear and matches the shape expected by many form libraries.

---

### HIGH (should fix before launch)

---

**HIGH-1: Field-level errors not connected to `aria-describedby`**

- **Location:** `src/components/contact-form.tsx` — all field error `<p>` elements
- **Description:** Error messages for each field are rendered as sibling `<p>` elements but are not linked to their inputs via `aria-describedby`. Screen readers announce the input label but not the error message when the field receives focus. Users relying on assistive technology cannot identify which field has an error without navigating the DOM manually.
- **Fix:**
  ```tsx
  <input
    id="name"
    name="name"
    type="text"
    required
    aria-describedby={state?.fieldErrors?.name ? "name-error" : undefined}
    aria-invalid={!!state?.fieldErrors?.name}
  />
  {state?.fieldErrors?.name && (
    <p id="name-error" role="alert" className="text-red-600">
      {state.fieldErrors.name[0]}
    </p>
  )}
  ```
  Apply pattern to all 4 fields. Add `role="alert"` to error paragraphs so they are announced dynamically when they appear.

---

**HIGH-2: Form fields lose user input on validation failure**

- **Location:** `src/components/contact-form.tsx` — all `<input>`, `<select>`, `<textarea>` elements
- **Description:** No `defaultValue` is bound to form fields. When the Server Action returns a validation error and the component re-renders, all fields reset to empty. The user must retype their entire message after a validation error (e.g., they mistyped their email and now lose a 500-word message).
- **Impact:** Severe UX regression, especially for the `message` textarea.
- **Fix:** The `useActionState` approach with Server Actions does not easily support repopulating fields (unlike controlled inputs with `useState`). Two options:
  - Option A: Use a hidden `<input>` or `FormData`-preserving pattern — not straightforward with Server Actions.
  - Option B: Switch the message field to a controlled `<textarea>` using a separate `useState` hook for persistence only, independent of the form action state.
  - Option C (recommended): Use a client-side form library (e.g., `react-hook-form`) for field state management, and call the Server Action from `handleSubmit` via `startTransition`. This gives full field control while keeping server validation.

---

**HIGH-3: Global error message hidden when `fieldErrors` present**

- **Location:** `src/components/contact-form.tsx` — global error conditional
- **Description:** The global error is rendered only when `state && !state.success && !state.fieldErrors`. However, the Zod validation failure case always includes a general error message (`"Por favor, revisa los campos del formulario."`) in addition to `fieldErrors`. Because `fieldErrors` is truthy, the condition `!state.fieldErrors` evaluates to `false` and the global summary message is suppressed. The user sees per-field errors but no summary.
- **Impact:** Minor UX issue in isolation, but for users who scroll past fields quickly or use screen readers, the summary message provides important context.
- **Fix:** Render the global error message unconditionally when it exists (independent of `fieldErrors`):
  ```tsx
  {state && !state.success && state.error && (
    <p className="text-red-600" role="alert">{state.error}</p>
  )}
  ```

---

**HIGH-4: Rate limit error indistinguishable from validation error in the UI**

- **Location:** `src/components/contact-form.tsx` — error rendering logic
- **Description:** Rate limit errors (`"Demasiadas solicitudes..."`) are returned as `{ success: false, error: string }` without `fieldErrors`. The frontend renders them as a global error paragraph — the same UI used for email send failures and unexpected errors. There is no visual distinction between "you submitted too fast, wait 10 minutes" and "our server is down, try again later."
- **Impact:** Users who hit the rate limit may repeatedly retry (worsening their frustration) because the message appears similar to a transient error.
- **Fix:** Extend `ContactActionResult` with an error code or type:
  ```typescript
  | { success: false; errorCode: "rate_limited"; error: string }
  | { success: false; errorCode: "validation_error"; error: string; fieldErrors?: ... }
  | { success: false; errorCode: "send_failed"; error: string }
  | { success: false; errorCode: "unexpected"; error: string }
  ```
  The frontend can then render a countdown or specific guidance for rate-limit errors.

---

### LOW (nice to fix, not blocking)

---

**LOW-1: `noUncheckedIndexedAccess` violation on `fieldErrors` access**

- **Location:** `src/components/contact-form.tsx` — all `state.fieldErrors?.name[0]` accesses
- **Description:** `fieldErrors` is typed as `Partial<Record<keyof ContactInput, string[]>>`. Under `noUncheckedIndexedAccess: true` (required tsconfig), each field access returns `string[] | undefined`. Accessing `[0]` on `string[] | undefined` is a type error. The current code `state.fieldErrors?.name[0]` is missing the additional optional chain on `name`:
  ```typescript
  // Current (TypeScript error with noUncheckedIndexedAccess):
  state.fieldErrors?.name[0]
  
  // Correct:
  state.fieldErrors?.name?.[0]
  ```
- **Impact:** TypeScript compile error in strict mode. Not a runtime crash (JavaScript accesses `undefined[0]` as `undefined`, which renders nothing), but it means the codebase fails to compile with the required tsconfig.

---

**LOW-2: `pending` property in rate limit fallback uses non-Upstash type**

- **Location:** `src/app/actions/contact.ts` — rate limit catch block
- **Description:** The fallback `rateLimitResult` object sets `pending: Promise.resolve()`. The Upstash `Ratelimit.limit()` return type defines `pending` as a `Promise<unknown>`. `Promise.resolve()` returns `Promise<void>`. TypeScript may flag this as an assignability issue if `Promise<unknown>` does not accept `Promise<void>`. In practice, `Promise<void>` is assignable to `Promise<unknown>` in TypeScript, so this is likely not a compile error — but it should be explicitly typed:
  ```typescript
  pending: Promise.resolve() as Promise<unknown>
  ```

---

**LOW-3: Email hardcodes `ana@analopez.com` in confirmation template**

- **Location:** `src/emails/contact-confirmation.tsx` — the `<a href="mailto:...">` link
- **Description:** The email template hardcodes `ana@analopez.com`. This should reference `env.CONTACT_EMAIL_TO` (the configurable env variable) to avoid divergence if the contact email changes. However, `env` cannot be imported in email templates rendered by React Email (they run in a separate context). The `sendContactEmails` function should pass the contact email as a prop:
  ```typescript
  // In email.ts
  react: ContactConfirmationEmail({
    name: data.name,
    sessionLabel,
    contactEmail: env.CONTACT_EMAIL_TO,  // pass from env
  }),
  
  // In contact-confirmation.tsx
  interface Props { name: string; sessionLabel: string; contactEmail: string }
  ```

---

**LOW-4: No `loading.tsx` on the contact route**

- **Location:** `app/contacto/` (or equivalent route)
- **Description:** No `loading.tsx` exists for the contact page route segment. If the page has any server-side data fetching (even if minimal), users on slow connections see a blank page during navigation to the contact page. The `loading.tsx` provides an instant Suspense fallback.
- **Impact:** Minor — contact pages are typically lightweight static renders. Low priority unless the page has server-side data fetching.

---

**LOW-5: Progressive enhancement not implemented**

- **Location:** `src/components/contact-form.tsx`
- **Description:** The form uses `noValidate` which disables native browser validation. With `action={formAction}` (a JavaScript function), if JavaScript fails to load, the form submits to a void function. The form is completely non-functional without JavaScript. While this is a known limitation of `useActionState` forms, it could be partially mitigated by providing a `<noscript>` fallback with a direct mailto link or a static form action endpoint.
- **Impact:** Affects a small percentage of users (JS disabled, or JS failed to parse). Not a blocking issue for a portfolio contact form.

---

## Summary Dashboard

| Category | Count | Items |
|----------|-------|-------|
| Critical | 3 | CRITICAL-1 (useActionState signature), CRITICAL-2 (error boundary), CRITICAL-3 (type contract drift) |
| High | 4 | HIGH-1 (aria-describedby), HIGH-2 (field repopulation), HIGH-3 (global error visibility), HIGH-4 (rate limit UX) |
| Low | 5 | LOW-1 (noUncheckedIndexedAccess), LOW-2 (pending type), LOW-3 (hardcoded email), LOW-4 (loading.tsx), LOW-5 (progressive enhancement) |

## Data Flows Verified

- [PARTIAL] `ContactSubmission` — chain traced end-to-end; broken at Server Action invocation due to CRITICAL-1

## Forms Verified

- [FAIL] `ContactForm` on contact page — blocked by CRITICAL-1 (useActionState signature mismatch)

## Auth Flows Verified

- [N/A] Contact form is a public form — no authentication required

## N+1 Queries Found

- [N/A] No database queries in this flow — email-only path

## Error Boundaries in Place

- [ ] `app/contacto/error.tsx` — MISSING (CRITICAL-2)
- [ ] Root `error.tsx` — unverified, assumed present but not confirmed

## Recommendation

**BLOCKED. Do not ship to production until CRITICAL-1 is resolved.**

The `useActionState` signature mismatch (CRITICAL-1) means the contact form cannot function at all — every submission will throw a TypeError on the server. This is a complete functional failure that would be immediately visible to any user who tries to contact Ana López.

After CRITICAL-1 is fixed, the form will function for the happy path. CRITICAL-2 (error boundary) and CRITICAL-3 (type contract) should be addressed before launch. HIGH-1 through HIGH-4 are strongly recommended before launch to meet accessibility standards and production UX quality.

The backend implementation itself (Zod validation, rate limiting, dual-email pattern, fail-open rate limiting, T3 Env typed secrets) is well-structured and follows the agency's standards. The infrastructure is sound — the failure points are at the integration seam between frontend and backend.

---

*Fullstack Developer — Integration Audit complete*  
*Next step: Frontend Developer to fix CRITICAL-1 (add `_previousState` parameter). Backend Developer to update `ContactActionResult` type to align with documented spec (CRITICAL-3).*
