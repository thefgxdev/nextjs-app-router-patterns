# The server/client boundary

## Default

Everything is a Server Component until it needs one of three things:

1. **Browser state or events**: `useState`, `useEffect`, `onClick`, form interactivity beyond what a plain `<form action>` gives.
2. **Browser APIs**: `window`, `localStorage`, geolocation, canvas.
3. **A client-only library** (charts, editors, maps).

Push `'use client'` to the smallest leaf that needs it. A page is a Server Component that composes server-rendered content and a few interactive islands.

## Passing data across

- Server → Client: props must be serialisable. No functions, no class instances, no `Date` unless you accept a string on the other side.
- Client → Server: server actions or route handlers. Both are public endpoints (see [server-actions.md](server-actions.md)).
- Never pass secrets, tokens or full user records to a Client Component "because it is convenient". Anything in props ships to the browser.

## Composition trick

A Client Component can receive Server Components as `children`. Interactive shell outside, server-rendered content inside, no waterfall:

```tsx
// layout: server
<Sidebar>            {/* client: collapsible, remembers state */}
  <NavItems />       {/* server: fetched with the user's permissions */}
</Sidebar>
```

## Data fetching

- Fetch in Server Components, close to where the data is used. Parallelise with `Promise.all`; sequential awaits are the most common cause of slow pages.
- Deduplicate: the same `fetch` in the same request is memoised. For database calls, wrap in `cache()` from React.
- Stream slow parts with `<Suspense>` so the page shell renders first.

## Anti-patterns seen in audits

- `'use client'` at the top of a page file "to make it work", which ships the whole page to the browser and hides server-only errors.
- Fetching in `useEffect` what could have been fetched on the server, doubling latency and losing SEO.
- Reading cookies or headers in a Client Component through a prop that was set from a Server Component holding the full session.
