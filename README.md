# Next.js App Router Patterns

Patterns for Next.js products that have to last: where the server/client boundary goes, how to keep server actions safe to retry, how caching actually behaves, how errors reach the user and the logs, and how to structure a codebase so the business rules have one home.

By [Felipe Guedes](https://fgxdev.com). Notes from building and auditing Next.js systems in production, not from the tutorial.

## Contents

- [`docs/boundaries.md`](docs/boundaries.md): Server Components by default, Client Components at the leaves, and the three questions that decide which is which.
- [`docs/server-actions.md`](docs/server-actions.md): validation, authorisation, idempotency and error contracts for server actions. They are public HTTP endpoints; treat them like it.
- [`docs/caching.md`](docs/caching.md): request memoisation, the data cache, the full route cache, tags and revalidation. Where stale data comes from and how to reason about it.
- [`docs/errors.md`](docs/errors.md): error boundaries, `error.tsx`, digests, what the user sees versus what the log gets.
- [`docs/structure.md`](docs/structure.md): a folder layout where the domain does not import from the framework, and a new engineer finds "refund eligibility" in under a minute.

## Principles

1. **The framework is an adapter.** Business rules live in plain TypeScript modules with no `next` import. Route handlers, server actions and components call them.
2. **Validate at the boundary, type inward.** Every input crosses a schema once; the validated type travels with the value.
3. **Every mutation is safe to retry.** Double submits and lost responses are normal. Idempotency keys on anything that costs money.
4. **Cache on purpose.** Every cached read has a tag and a revalidation story. "Why is this stale" must have an answer in under a minute.
5. **Errors have two audiences.** The user gets a way forward; the log gets the message and the digest.

## Em português

Padrões para produtos em Next.js que precisam durar: fronteira servidor/cliente, server actions seguras de repetir, cache explicável, erros com dois públicos e uma estrutura em que a regra de negócio tem um lugar só. Desenvolvimento com Next.js em [fgxdev.com/pt/desenvolvimento-de-software-cascavel](https://fgxdev.com/pt/desenvolvimento-de-software-cascavel/).

## License

Apache-2.0. Copyright (c) 2026 Felipe Guedes (fgxdev.com). Redistributions must keep the NOTICE file and mark any changes.
