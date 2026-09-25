# Server actions are public endpoints

A server action is an HTTP endpoint with a generated URL. Anyone with the id can call it with any payload. Treat it exactly like a route handler.

## The shape of a safe action

```ts
'use server';
import { z } from 'zod';
import { getSession } from '@/lib/auth';
import { orders } from '@/domain/orders';

const Input = z.object({ orderId: z.string().uuid(), key: z.string().uuid() });

export async function cancelOrder(raw: unknown) {
  const input = Input.parse(raw);                  // 1. validate at the boundary
  const session = await getSession();             // 2. who is calling
  if (!session) return { ok: false, error: 'unauthenticated' as const };
  const result = await orders.cancel({            // 3. domain decides, with authorisation inside
    orderId: input.orderId, actor: session.user, idempotencyKey: input.key,
  });
  return result;                                  // 4. typed result, never a thrown internal error
}
```

## Rules

1. **Validate the input.** `FormData` and JSON are untrusted. Parse with a schema; reject on failure with a typed error.
2. **Authenticate and authorise inside the action**, not in the component that renders the button. The button is not a security boundary.
3. **Authorise the resource, not only the user.** "Logged in" is not "may cancel this order".
4. **Make it idempotent.** Generate an idempotency key when the form is shown; send it with the action; store it before the side effect. Double submit and lost response are normal.
5. **Return, do not throw**, for expected failures. Thrown errors become digests in production and the client cannot act on them.
6. **Revalidate what changed.** `revalidateTag('orders')` or `revalidatePath`, scoped. Revalidating the whole layout on every action is how sites become slow.
7. **Log with context.** Actor, action, input ids, result, duration, request id.

## Progressive enhancement

`<form action={cancelOrder}>` works without JavaScript. `useActionState` adds pending and error states. Keep the no-JS path working; it is also the path that makes the action easy to test with `curl`.

## What to test

- Unauthenticated call returns the typed error.
- Authenticated user calling on another user's resource returns the typed error.
- The same key twice returns the same result and performs the side effect once.
- Malformed input fails validation without touching the database.
