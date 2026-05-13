# Fullstack Developer Memory

## Integration bugs found across projects

### Project: Ana López Fotógrafa — Contact Form (2026-05-13)

**Bug 1 — null initial state type mismatch with useActionState**
- Pattern: `useActionState(serverAction, null)` gives `state` the type `ReturnType<typeof serverAction> | null`
- Root cause: developers set `null` as initial state without adding null guards in render
- Fix: use `{ success: false }` as initial state, or add `state?.` null guards everywhere in JSX
- TypeScript only catches this in strict mode with explicit return type annotations

**Bug 2 — Unhandled exception in Server Action bypasses form error UI**
- Pattern: Server Action calls external API (Resend, Stripe, etc.) without try/catch
- Root cause: developers assume the return type covers all cases, forgetting that throws bypass the return
- Fix: always wrap the full Server Action body in try/catch; return `{ success: false, message: "..." }` on catch
- Impact: any third-party API outage causes full-page error boundary instead of inline form message

**Bug 3 — No rate limiting on public Server Actions**
- Pattern: contact/booking forms callable at arbitrary frequency
- Root cause: Next.js Server Actions have no built-in rate limiting
- Fix: `upstash/ratelimit` with IP key from `headers()`, or middleware-level rate limiting
- Standard threshold: 3 submissions per 10 minutes per IP for contact forms

**Bug 4 — Form fields not cleared after successful submission**
- Pattern: uncontrolled inputs under `useActionState` do not auto-clear on success
- Root cause: `useActionState` manages action state, not form field state
- Fix: change `key` prop on `<form>` when `state.success === true`, or call `formRef.current?.reset()` in `useEffect`

**Bug 5 — Missing shared ActionResult type**
- Pattern: Server Action return type defined inline, not as a named exported type
- Root cause: frontend and backend developed independently without a shared types file
- Fix: create `types/actions.ts` with named exported types; import in both Server Action and Client Component

## N+1 queries discovered

None yet — contact form flows have no database reads. Log here when list/detail pages are audited.

## Type issues TypeScript missed

- `Record<string, string[]>` does not guarantee key existence — `errors.name[0]` can be `undefined` without TypeScript error. Always use `errors?.name?.[0]`.
- `FormDataEntryValue | null` from `formData.get()` — TypeScript allows passing this directly to string-expecting functions without explicit narrowing in non-strict configurations.

## Edge cases that broke critical flows

- Resend API 429 (rate limit from provider) — unhandled in Server Action; throws instead of returning graceful error
- Double-click submit without `isPending` guard — sends duplicate emails
- Missing env var (`RESEND_API_KEY` undefined) — Resend SDK throws on initialization, not on send, making the error surface at import time in some configurations

## Recurring patterns to check on every project

1. Every public Server Action needs: try/catch, rate limiting, input validation before any side effect
2. `useActionState` initial state should never be `null` — use a typed empty state
3. Every named field in `errors: Record<string, string[]>` must be accessed with optional chaining
4. Form reset after success is always a manual step — uncontrolled inputs do not auto-clear
5. Shared action result types belong in `types/actions.ts` — never inline only in the Server Action file
