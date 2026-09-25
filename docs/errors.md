# Errors have two audiences

The user needs a way forward. The log needs the truth. Designing for both is the whole job.

## What the user sees

`error.tsx` at the segment level renders when a Server Component or a nested route throws. In production the framework strips the message and passes a **digest**; the boundary cannot show what went wrong and should not try. Its job:

- Say what happened in plain words ("we could not load your orders").
- Offer an action: retry (`reset()`), go back, contact support with the digest attached.
- Keep the rest of the page alive: a boundary per segment, not one for the whole app.

`not-found.tsx` for missing resources, with `notFound()` from the server; `global-error.tsx` for the root layout failing.

## What the log gets

- The full error, the digest, the request id, the route, the user (id, not email), the input ids.
- Structured, so the digest a user pastes in a ticket finds the log line in one search.
- Wired to an error tracker (Sentry or similar) with source maps uploaded, not served.

## Expected failures are not exceptions

Validation errors, "not authorised", "already cancelled": return typed results from actions and route handlers. Reserve `throw` for the unexpected. The client renders a message for a typed error; it cannot do anything useful with a digest.

## Timeouts and partial failure

- Slow data behind `<Suspense>` with a fallback; a slow widget must not block the page.
- Outbound calls with timeouts shorter than the platform's function timeout, so you return an error page instead of a platform 504.
- When a non-critical part fails, render the rest and show a small "temporarily unavailable" in its place.

## Checklist

- [ ] `error.tsx` per segment that can fail independently.
- [ ] Digest shown to the user with a copy button; digest searchable in logs.
- [ ] Expected failures returned, not thrown.
- [ ] Every outbound call has a timeout.
- [ ] Error tracker receives server and client errors with release version.
