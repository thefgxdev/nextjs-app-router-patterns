# Caching, explained by where stale data comes from

There are four layers. Every "why is this stale" question is answered by naming the layer.

| Layer | Scope | Lifetime | You control it with |
|---|---|---|---|
| Request memoisation | One request | The request | Same `fetch` URL and options, or `cache()` for non-fetch |
| Data cache | The server, across requests | Until revalidated | `fetch` options `cache`, `next.revalidate`, `next.tags`; `unstable_cache` |
| Full route cache | Static routes rendered at build | Until revalidated or redeployed | Dynamic APIs (`cookies()`, `headers()`, `searchParams`) opt out |
| Router cache | The browser | Session, with a short TTL | `router.refresh()`, `revalidatePath` |

Defaults have changed across Next.js versions. Check the version you run; do not trust memory or old blog posts.

## Rules that survive version changes

1. **Every cached read has a tag.** `fetch(url, { next: { tags: ['orders', `order:${id}`] } })`. Mutations call `revalidateTag`. No tag, no cache.
2. **Personalised data is never in the data cache** unless the key includes the user. Cookies and headers make a route dynamic; that is the framework protecting you.
3. **Time-based revalidation is for content that may be stale**, like a product catalogue. Tag-based is for content that must be fresh after a write.
4. **Static where possible, dynamic where necessary.** A marketing page is static. A dashboard is dynamic. A product page is static with tag revalidation.
5. **The browser router cache surprises people.** After a mutation, call `router.refresh()` or revalidate the path, or the user sees the old data on back navigation.

## Debugging staleness

1. Is it the browser? Hard-reload. If fixed, router cache.
2. Is it the server data cache? Hit the API directly. If the API is fresh but the page is not, data cache or route cache.
3. Is it upstream? Your database is fine, but a CDN in front of the API has a TTL.
4. Log the cache status header your host exposes; Vercel and others mark `HIT`, `MISS`, `STALE`.

## A checklist for a review

- [ ] Every `fetch` on a cached path has tags and a revalidation story.
- [ ] Mutations revalidate the tags they affect, and only those.
- [ ] No user-specific data cached under a shared key.
- [ ] Static pages are actually static (check the build output).
- [ ] Back-navigation after a mutation shows fresh data.
