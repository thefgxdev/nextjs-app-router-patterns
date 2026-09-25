# Structure: the domain does not import the framework

Ask a team where "an order above a certain amount needs manager approval" lives, and watch four people point at four places. The fix is a structure with one answer.

```
src/
  domain/              # business rules; plain TypeScript; no next, no db client
    orders/
      order.ts         # types and invariants
      approve.ts       # the rule lives here, once
      index.ts
  application/         # use cases: orchestrate domain + ports
    orders/cancel-order.ts
  infra/               # adapters: database, email, payment provider, queue
    db/
    email/
    payments/
  app/                 # Next.js: routes, layouts, server actions, route handlers
    (shop)/orders/[id]/page.tsx
    (shop)/orders/[id]/actions.ts
  components/          # UI, client and server
  lib/                 # framework glue: auth, logging, config
```

## Rules

1. `domain/` imports nothing from `app/`, `infra/` or `next`. A lint rule enforces it (eslint `no-restricted-imports` or dependency-cruiser).
2. `application/` depends on `domain/` and on **interfaces** for infrastructure (ports), not on `infra/` directly. Adapters are injected or resolved in one composition file.
3. `app/` is thin: parse input, call a use case, render. If a route handler is 600 lines long, the rules leaked.
4. Side effects (email, webhooks, search index) are not called inline in a request. They are events handled after commit, through an outbox or a queue.

## What you get

- The domain has the highest test coverage and the fastest tests, with no database.
- Switching an email provider touches one adapter file and nothing else.
- A new business rule touches the domain and one use case, never a route.
- A new engineer finds "refund eligibility" by file name in under a minute.

## Migrating an existing app

Do not rewrite. Pick one rule that lives in three places, move it into `domain/`, make the three places call it, add the lint rule for that module. Repeat with the next rule. The structure emerges from the moves, and every step ships.
