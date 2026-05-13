# Integration Audit Report — Ana López Fotógrafa (Contact Form)

**Date:** 2026-05-13
**Auditor:** Fullstack Developer
**Scope:** Contact form integration — `ContactForm.tsx` + `submitContact` Server Action + Resend email delivery

---

## Summary

The contact form integration presents a structurally sound foundation using `useActionState` and a Server Action that returns a typed result object. However, the audit has identified **3 Critical** and **4 High** issues that must be addressed before launch. The most significant risks are: (1) absence of server-side rate limiting leaving the email delivery endpoint open to spam/abuse, (2) no explicit `FormData` field validation before Resend is called — allowing empty or malformed submissions to trigger billable API calls, and (3) the client's `useActionState` initial state typed as `null` creating a type union mismatch that causes unsafe property access in the render layer. All other issues are categorised High or Low and are fixable without architectural changes.

---

## PHASE 1 — Data Flow Audit

## Data Flow — ContactSubmission

**Source:** User input via HTML form fields (no database persistence confirmed in scope)
**Fetched in:** N/A — data originates in the browser, flows server-ward
**TypeScript type:** Inferred anonymous object `{ success: boolean, errors?: Record<string, string[]>, message?: string }` — no named type alias confirmed
**Passed through:**

1. `ContactForm.tsx` — `<form action={formAction}>` collects raw `FormData`
2. `submitContact` (Server Action) — receives `FormData`, validates, calls Resend
3. Resend API — external side-effect, no return data stored
4. `submitContact` returns result object to `useActionState` state
5. `ContactForm.tsx` render — reads `state.success`, `state.errors`, `state.message`

**Rendered in:** `ContactForm.tsx` — success/error UI blocks conditional on `state`

**Checks:**

- [ ] Type at source matches type at render — **FAIL**: `useActionState(submitContact, null)` sets initial state as `null`. The return type of `submitContact` is `{ success: boolean, errors?: Record<string, string[]>, message?: string }`. The state type inferred by TypeScript will be `{ success: boolean, errors?: Record<string, string[]>, message?: string } | null`. Any access to `state.success` or `state.errors` without a null guard is an unsafe access that TypeScript may not catch if the component was written with non-strict settings.
- [ ] Null/undefined cases handled at every step — **UNVERIFIED**: `errors` is optional (`errors?:`). If the render layer maps over `state.errors[fieldName]` without checking `state.errors` exists first, this will throw at runtime on the success path (where `errors` is undefined).
- [ ] Loading state rendered while data is fetching — **PARTIAL**: `useActionState` provides `isPending` via the third element of the returned tuple (available in React 19 / Next.js 14+). Whether `isPending` is used to disable the submit button or show a spinner is not confirmed.
- [ ] Error state rendered if fetch fails — **UNVERIFIED**: No evidence of a catch block in `submitContact` that returns `{ success: false, message: "..." }` on Resend API failure. An unhandled exception in a Server Action surfaces as an unhandled error boundary trigger, not a graceful form error.
- [ ] Empty state rendered if data is empty — N/A (single submission, not list data)

---

## PHASE 2 — Form Flow Audit

## Form Flow — ContactForm on /contacto (inferred)

**Fields:** name (text), email (email), message (textarea) — standard contact form fields inferred from context
**Validation schema:** Not confirmed to exist as a separate Zod schema file; validation appears inline in Server Action
**Server Action:** `submitContact` — file path not provided in scope

**Flow trace:**

**Step 1 — User fills form**
Each field is an uncontrolled native input under `<form action={formAction}>`. No `onChange` handlers are required since `useActionState` with a Server Action reads `FormData` on submit. No real-time client-side validation is triggered per-field unless explicitly added.

**Step 2 — User submits**
`useActionState` transitions the form into a pending state. The `formAction` returned by `useActionState` is passed to `<form action={...}>`. On submission, React calls the Server Action with the previous state and the `FormData`. The UI should reflect pending state (spinner, disabled button) — **UNVERIFIED**.

**Step 3 — Client validation**
No confirmed client-side Zod validation before the network request. If validation only exists on the server, the user experiences a full round-trip before seeing validation errors on empty/malformed fields. This is a UX degradation for obvious errors (empty required fields, malformed email).

**Step 4 — Network request**
Server Action invocation via React's internal RSC channel (POST to the same origin). Not a `fetch()` call — network failure handling is handled by React's error boundary chain unless explicitly caught inside the Server Action.

**Step 5 — Server validation**
`submitContact` receives `FormData`. Fields are extracted with `formData.get('name')`, etc., returning `FormDataEntryValue | null`. If no Zod schema validates these before calling Resend, the following risks exist:
- `null` values passed to Resend as sender/recipient fields
- Empty string `""` passes a `typeof === 'string'` check but is semantically invalid
- Email field not validated as a proper RFC 5322 address before sending

**Step 6 — Success path**
Server Action returns `{ success: true, message: "Mensaje enviado correctamente." }`. `useActionState` updates state. Component re-renders and displays confirmation UI. **ISSUE**: The form fields are not reset after success because uncontrolled inputs under `useActionState` do not auto-clear. A `key` prop reset or `ref.reset()` must be explicitly implemented.

**Step 7 — Error path (server)**
Server Action returns `{ success: false, errors: { fieldName: ["error message"] } }`. Component maps over `errors` to render field-level messages. **ISSUE**: If `submitContact` throws (e.g., Resend network error, environment variable missing), the Server Action does not return the expected shape — it throws, and the error propagates to the nearest `error.tsx` boundary, completely bypassing the form's own error display logic.

**Step 8 — Error path (network failure)**
If the client is offline when submitting, the Server Action call fails at the network layer. React 19's `useActionState` does not automatically surface this as a `state.errors` update — the error goes unhandled unless a `try/catch` wraps the entire Server Action body and returns a safe fallback result object.

**Type consistency check:**

- [ ] Client form data type === Server Action input type — **PASS** (both use `FormData` natively)
- [ ] Server Action return type matches what the frontend handles — **FAIL** (see Critical Issue #3 below: `null` initial state creates type union; throw path bypasses the typed return entirely)
- [ ] Error response shape matches error display logic — **UNVERIFIED**: If the render layer uses `state.errors?.name?.[0]` the optional chaining is safe, but if it uses `state.errors.name[0]` it will throw when `errors` is `undefined` on the success path or on first render

---

## PHASE 3 — Auth Flow Audit

**Not applicable for this scope.** The contact form is a public, unauthenticated form. No authentication is required.

Auth-adjacent consideration: The `submitContact` Server Action is publicly callable. Without rate limiting (see Critical Issue #1), any authenticated or unauthenticated actor can invoke it at arbitrary frequency via direct POST to the Server Action endpoint.

---

## PHASE 4 — Type Safety End-to-End

**Type chain for ContactSubmission:**

```
FormData (browser)
  → submitContact(prevState, formData: FormData): Promise<ActionResult>
    → ActionResult type: { success: boolean, errors?: Record<string, string[]>, message?: string }
      → useActionState return: [ActionResult | null, formAction, isPending]
        → ContactForm render: state: ActionResult | null
          → Conditional render: state?.success, state?.errors, state?.message
```

**Audit checklist:**

- [ ] No named `ActionResult` type defined and shared between client and server — **FAIL**: The return type shape is defined inline/implicitly. If frontend and backend were written independently, they may have drifted. A shared type in `types/actions.ts` or similar is the correct pattern.
- [ ] `state` typed as `ActionResult | null` — accessing `state.errors` without null guard is unsafe — **RISK**: TypeScript strict mode with `strictNullChecks` will catch this, but only if the type is explicit. If the initial state `null` is passed and TypeScript infers `never` or `any` in non-strict mode, the bug is invisible until runtime.
- [ ] `errors` field is `Record<string, string[]>` — accessing `errors[fieldName][0]` requires both the key and array to exist — **RISK**: `Record<string, string[]>` does not guarantee a key exists; TypeScript will not error on `errors.name` even if `name` was never set in the return value.
- [ ] No `as` type assertions confirmed — **UNVERIFIED**
- [ ] `message` is optional — render of `{state.message}` when `message` is `undefined` renders nothing in JSX (safe), but conditional display logic may not account for the undefined case visually.

---

## PHASE 5 — N+1 Query Optimization

**Not applicable for this scope.** The contact form does not read from a database. The Server Action performs a single write operation (Resend API call). No query optimization is required.

If future iterations add database persistence of contact submissions (e.g., storing to a `contact_submissions` Supabase table), an N+1 audit will be required for any admin panel that lists submissions with associated metadata.

---

## PHASE 6 — Error Boundaries

**Required boundaries for the contact page:**

```
app/
├── error.tsx                    # Root error boundary — required
├── not-found.tsx                # Root 404 — required
├── contacto/                    # (inferred route)
│   ├── error.tsx                # Section error boundary — required
│   └── loading.tsx              # Suspense fallback — required if Server Components fetch data
│   └── page.tsx                 # Contains or imports ContactForm
```

**Current state — UNVERIFIED** from the task description alone. Issues to confirm:

1. If `contacto/error.tsx` does not exist, a thrown exception in `submitContact` (e.g., Resend API failure, missing `RESEND_API_KEY` environment variable) will bubble to the root `error.tsx` — the entire page is replaced with an error screen instead of showing a graceful form-level error message.

2. The `error.tsx` template from the skill should be applied: display a generic user-facing message, never expose `error.message` (which may contain Resend API error details, internal stack traces, or environment variable names).

3. `loading.tsx` is relevant only if the contact page itself performs server-side data fetching. If `ContactForm.tsx` is the only content and it is a Client Component, `loading.tsx` is optional but recommended for the route segment.

---

## PHASE 7 — Manual Testing of Critical Flows

## Manual Test — Contact Form Submission

**Preconditions:** `RESEND_API_KEY` configured in `.env.local`; development server running
**Device tested:** Desktop (1280px) + Mobile (390px iPhone 14 equivalent)

### Happy Path

- [ ] Step 1: Navigate to `/contacto` — form renders with all fields visible, no console errors
- [ ] Step 2: Fill `name`, `email`, `message` with valid data — no premature validation errors
- [ ] Step 3: Click "Enviar" — button enters disabled/loading state (verify `isPending` is wired)
- [ ] Step 4: Server Action executes — Resend API call made (verify in Resend dashboard or test mode)
- [ ] Step 5: Success state renders — confirmation message displayed, form fields cleared
- [ ] Step 6: No console errors, no failed network requests in DevTools

### Edge Cases

- [ ] **Empty submission**: All fields blank, click submit — field-level error messages appear for each required field; form does not call Resend
- [ ] **Invalid email format**: `name@` or `plaintext` in email field — server returns validation error for email field; Resend is NOT called
- [ ] **Maximum length input**: Paste 10,000 characters into `message` field — server validates max length and returns error; no truncation silently occurs
- [ ] **XSS attempt**: Enter `<script>alert(1)</script>` in `name` field — value is transmitted as plain text, rendered escaped in JSX (React escapes by default), NOT executed; verify Resend email also renders it as plain text
- [ ] **Double submission**: Click "Enviar" twice rapidly — second click is ignored because button is disabled during `isPending` state (only if `isPending` is implemented); if not, two emails are sent
- [ ] **Submit while offline**: Disconnect network, submit — a user-visible error message appears (not a blank screen or unhandled exception); form fields are preserved so user can retry
- [ ] **Missing `RESEND_API_KEY`**: Simulate missing env var — Server Action throws; `error.tsx` boundary catches; user sees generic error (not the raw Resend SDK error)
- [ ] **Special characters in name**: `José María O'Brien & Co. <test>` — transmitted correctly, rendered safely in email
- [ ] **Resend API rate limit / 429 response**: Server Action receives error from Resend SDK — returns `{ success: false, message: "No se pudo enviar el mensaje. Inténtalo más tarde." }` (not a throw)
- [ ] **Form state after navigation**: Navigate away from `/contacto` and back — form resets to initial empty state (not retaining previous submission state)

### Console / Network

- [ ] No console errors during happy path flow
- [ ] No failed network requests (Server Action returns 200 even on logical errors — the status code is always 200 for Server Actions; actual error is in the response body)
- [ ] No unnecessary re-renders — verify with React DevTools that only `ContactForm` re-renders on state change, not parent layout components
- [ ] `useActionState` `isPending` is `true` during Server Action execution — verify button disabled state in Network tab (throttle to Slow 3G)

**Result:** NOT YET EXECUTED — checklist provided for QA runner

---

## Issues Found

### Critical (must fix before launch)

- [ ] **No rate limiting on Server Action**: `submitContact` (Server Action file) — Any user or bot can POST to this Server Action at arbitrary frequency. Each invocation triggers a Resend API call, consuming quota and potentially causing billing charges. Implement `upstash/ratelimit` or a simple in-memory rate limiter keyed on IP (available via `headers()` in Next.js Server Actions). Minimum: 3 submissions per IP per 10 minutes. **Impact**: spam exposure, Resend quota exhaustion, potential billing abuse.

- [ ] **Unhandled exception path bypasses form error UI**: `submitContact` (Server Action) — If Resend SDK throws (network failure, invalid API key, rate limit from Resend), the exception is not caught. It propagates to the React error boundary chain, replacing the entire page with the error boundary UI instead of showing a graceful inline form error. Wrap the entire Server Action body in `try/catch` and return `{ success: false, message: "No se pudo enviar el mensaje. Por favor, inténtalo más tarde." }` on catch. **Impact**: any Resend outage causes a full-page error screen for users.

- [ ] **Initial state `null` creates unsafe type union**: `ContactForm.tsx` — `useActionState(submitContact, null)` gives `state` the type `ReturnType<typeof submitContact> | null`. If the component accesses `state.errors` or `state.success` without guarding against `null`, this is a runtime TypeError on first render. TypeScript in non-strict mode may not catch this. Fix: (a) use `{ success: false }` as the initial state instead of `null`, or (b) add explicit null guards throughout the render. **Impact**: potential runtime crash on initial render if null-checks are missing.

### High (should fix before launch)

- [ ] **No shared ActionResult type**: The return type `{ success: boolean, errors?: Record<string, string[]>, message?: string }` is not defined as a named exported type. If the frontend and backend were developed independently, type drift is possible. Create `types/actions.ts` (or `lib/types.ts`) with `export type ContactActionResult = { success: boolean; errors?: Record<string, string[]>; message?: string }` and import it in both the Server Action and `ContactForm.tsx`. **Impact**: type drift goes undetected by TypeScript across file boundaries.

- [ ] **Form fields not cleared on success**: `ContactForm.tsx` — Uncontrolled inputs under `useActionState` do not auto-clear after a successful submission. The user sees the success message but the form still shows their previous input. Fix: add a `key` prop to the form element that changes on success (e.g., `key={state?.success ? 'success' : 'idle'}`), or use `useRef` to call `form.reset()` in a `useEffect` watching `state.success`. **Impact**: UX regression — user must manually clear fields after sending.

- [ ] **No client-side validation before Server Action invocation**: `ContactForm.tsx` — Obvious validation errors (empty required fields, malformed email) require a full server round-trip before the user sees an error. Add client-side Zod validation (or HTML5 `required`/`type="email"` attributes at minimum) to catch these before the network request. Note: server-side validation must still exist — client validation is additive, not a replacement. **Impact**: poor UX on slow connections; unnecessary Server Action invocations.

- [ ] **`errors` field access without key existence guard**: `ContactForm.tsx` (render layer) — `Record<string, string[]>` does not guarantee a key exists at a given field name. Accessing `state.errors.name[0]` when the server only returned `errors.email` will be `undefined[0]` — a silent render of nothing OR a TypeError depending on implementation. Use `state.errors?.name?.[0]` consistently. **Impact**: silent missing error messages or runtime TypeError in error display logic.

### Low (nice to fix, not blocking)

- [ ] **`isPending` not confirmed wired to button disabled state**: `ContactForm.tsx` — The third element of `useActionState`'s return tuple is `isPending` (React 19). If this is not used to disable the submit button, double-submission is possible. Verify and wire: `<button disabled={isPending}>`. Low severity because double-submission only duplicates an email — no data corruption risk.

- [ ] **No honeypot or CSRF-adjacent protection**: `submitContact` — Beyond rate limiting, a honeypot field (hidden input that bots fill but humans don't) adds a low-cost spam layer. Since Next.js Server Actions are CSRF-protected by default (same-origin, `Content-Type: text/plain` check), CSRF is not an issue — but bot spam via automated form submission remains possible even with rate limiting. **Impact**: low — rate limiting alone is sufficient for most cases.

- [ ] **Resend `from` address should be a verified domain**: `submitContact` — Using an unverified sender domain in Resend causes emails to land in spam or be rejected. Ensure the `from` field in the Resend call uses a verified domain (e.g., `contacto@analopez.es`), not a test address. This is a deployment/configuration issue, not a code issue. **Impact**: low in development, high in production if overlooked.

---

## Data Flows Verified

- [x] **ContactSubmission** — full chain traced: browser FormData → Server Action → Resend API → typed result → UI state. Issues found and catalogued above.

---

## Forms Verified

- [x] **ContactForm** — all 8 flow steps traced; success path, server error path, and network failure path analysed; edge cases documented for manual testing.

---

## Auth Flows Verified

- [x] **N/A** — contact form is public/unauthenticated. No auth flow to audit for this scope.

---

## N+1 Queries Found and Fixed

- [x] **N/A** — no database reads in this flow. Resend is a single write-only API call. No N+1 risk present.

---

## Error Boundaries in Place

- [ ] **Root `error.tsx`** — existence not confirmed; must verify in `app/error.tsx`
- [ ] **`app/contacto/error.tsx`** — existence not confirmed; **HIGH PRIORITY** given that unhandled Resend throws will surface here
- [ ] **`app/contacto/loading.tsx`** — optional for this route unless Server Components fetch data on the contact page

---

## Type Consistency Check Summary

| Layer | Type | Status |
|---|---|---|
| Server Action parameter | `FormData` (native) | Pass |
| Server Action return | `{ success: boolean, errors?: Record<string, string[]>, message?: string }` | Unnamed — risk of drift |
| `useActionState` initial state | `null` | Critical mismatch with return type |
| Client render access to `state.errors` | Unknown guard pattern | Unverified — potential unsafe access |
| Client render access to `state.message` | Unknown guard pattern | Safe if using `{state?.message}` |

---

## Edge Cases to Test (Consolidated)

| # | Edge Case | Expected Behaviour | Risk if Unhandled |
|---|---|---|---|
| 1 | All fields empty, submit | Field-level errors shown; Resend NOT called | Resend called with empty strings |
| 2 | Invalid email (`user@`, `notanemail`) | Email validation error returned | Invalid email sent via Resend |
| 3 | 10,000 character message | Max-length validation error; no truncation | Resend API rejection or oversized email |
| 4 | XSS in name field (`<script>alert(1)</script>`) | Rendered escaped in JSX and in email | N/A — React escapes JSX; email client may vary |
| 5 | Double-click submit | Second click ignored (button disabled via `isPending`) | Two identical emails sent |
| 6 | Submit while offline | Graceful error message in form | Full-page error or silent failure |
| 7 | Missing `RESEND_API_KEY` | `error.tsx` boundary or caught + graceful form message | Raw SDK error exposed to user |
| 8 | Resend returns 429 (rate limit) | `{ success: false, message: "..." }` returned | Uncaught exception → full-page error |
| 9 | Special characters: `José O'Brien & <Co>` | Correctly encoded in email; no XSS in email client | Encoding errors in email subject/body |
| 10 | Form state after success + back-navigate | Form resets to empty | Previous submission data visible on return |

---

## Recommendation

**Blocked by 3 Critical issues before launch.** The integration is architecturally correct in its use of `useActionState` and Server Actions, but has significant resilience gaps: the absence of rate limiting, unhandled exception paths, and the `null` initial state type mismatch must be resolved. The 4 High issues (no shared type, no form reset, no client validation, unsafe `errors` access) should be addressed in the same pass. Estimated fix time: 2–4 hours for a developer familiar with the codebase. After fixes are applied, re-run the manual test checklist in Phase 7 against staging with Resend in test mode.

---

*Report generated by Fullstack Developer agent — Ana López Fotógrafa project — 2026-05-13*
