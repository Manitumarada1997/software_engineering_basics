# CDN (Content Delivery Network)

## What Is It?

A CDN is a **globally distributed fleet of reverse proxies** caching your content at the network edge, close to users:

```text
User (Mumbai) → CDN PoP (Mumbai, ~10ms) —hit→ served from cache
                                    —miss→ origin (us-east, ~220ms) → cached → next Mumbai user: 10ms
```

CloudFront, Cloudflare, Akamai, Fastly — same architecture: Points of Presence (PoPs) + cache + shielded origin.

## Why Does It Exist?

**Physics: distance = latency.** Light in fiber ≈ 200 km/ms round-trip accounting — a Mumbai user fetching from Virginia eats ~220ms+ *per round trip*, and a web page needs many. No code optimization fixes distance. The CDN's move: **move the bytes to the user** — and as a side effect, absorb load and DDoS traffic that never reaches your origin.

## Layer 1 — Simple Explanation

The CDN is a **chain of neighborhood branch warehouses** for a central factory: most customers get goods from the local branch (fast); branches restock from the factory occasionally; a rush at one branch doesn't exhaust the factory.

## Layer 2 — Engineer's View

**What's cacheable — the decision table:**

| Content | Policy |
|---|---|
| Static assets (JS/CSS/images) | immutable fingerprints (`app.a1b2c3.js`), cache forever |
| Personalized API responses | usually no edge cache (or short + keyed carefully) |
| Semi-dynamic | edge caching with `s-maxage` + `stale-while-revalidate` |
| HTML | TTL per tolerance for staleness |

The **cache-key** discipline (URL + headers you vary on) and **invalidation** ("only two hard things: cache invalidation and naming" — there are only two hard problems in CS) are the real engineering. Fingerprinted immutable assets + content-addressed keys (the same hashing idea as Git Internals!) make invalidation a non-problem: new content = new URL.

**The headers that drive everything:**

```http
Cache-Control: public, max-age=31536000, immutable     # fingerprinted asset
Cache-Control: no-cache                                 # always revalidate
Cache-Control: public, s-maxage=60, stale-while-revalidate=300   # API-ish
```

**CDN as a security & resilience layer (often its biggest ops value):**

| Function | Mechanism |
|---|---|
| DDoS absorption | volumetric traffic dies at the edge, terabytes away from origin |
| TLS edge | certificates + handshakes at PoPs near users |
| WAF | rules (OWASP CRS) evaluated per request at the edge |
| Bot management | fingerprinting/rate limits before origin |
| Origin shield | one internal cache layer — origin sees 1 load, not N PoPs' misses |

**Operational sharp edges:**

- **Cache poisoning**: unkeyed input influencing what others receive — treat cache keys and varied headers as security surface
- **Purge/propagation latency**: "deployed but users see old" — TTL vs purge API in your deploy pipeline
- **Debugging**: `X-Cache: Hit/Miss` headers, per-PoP behavior, cache keys you can inspect
- **Origin exposure**: origin IP discoverable → direct-to-origin DDoS; lock origin to CDN ranges (SG/allowlist)

## Real-World Example (DevOps flavored)

ShopEasy storefront: assets on CDN with immutable fingerprints (deploy pipeline hashes bundles — new deploy = new URLs = instant "invalidation"); product images cached at edge; `/api/` bypasses cache or uses 30s `stale-while-revalidate`. Origin locked to CloudFront prefix list in its SG. Result: origin load drops 85%, global p95 for assets < 40ms, and a 2M-req/min bot surge never left the edge.

## Common Mistakes

- Long TTLs on mutable URLs ("/app.js" not fingerprinted) — then "clearing cache" rituals
- Caching responses with auth/personalization baked in — cross-user leaks
- Origin left reachable directly — CDN security bypassed
- No `stale-while-revalidate` — thundering herds of misses on expiry
- Believing CDN fixes slow *APIs* — it fixes distance for cacheable bytes, not your database

## Mental Model

> A CDN is a **chain of neighborhood branches for your warehouse**: physics (distance) defeated by geography (move the stock), load absorbed at the counter, and the central warehouse (origin) shielded behind one restock door — with inventory control (cache keys/TTLs) being the actual engineering discipline.

## Remember This

1. CDNs defeat distance-latency by caching at PoPs near users
2. Cacheability by content class; fingerprinted immutable assets make invalidation a non-problem
3. `Cache-Control` semantics (`max-age`, `s-maxage`, `stale-while-revalidate`) are the control plane
4. Security side: DDoS absorption, edge TLS, WAF, and origin locking
5. Cache keys are attack surface (poisoning); origin must be CDN-only reachable

## One Sentence

A CDN places caches of your content near your users — turning distance-latency into cache hits, absorbing volumetric attacks at the edge, and shielding the origin behind cache keys, TTLs, and allowlists.

## Knowledge Check

1. Why do fingerprinted URLs eliminate cache invalidation as a problem?
2. Design cache policy for: hashed JS bundles, user-specific JSON, product images.
3. How does a CDN absorb a DDoS that would kill your origin?
4. What is cache poisoning, and which CDN design decision prevents it?

## Further Reading

- [Cloudflare — What is a CDN?](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/) (vendor-neutral fundamentals)
- MDN — `Cache-Control` documentation
- Fastly blog on surrogate keys/invalidation

---

**← Previous:** [Firewalls & VPNs](firewalls-vpns.md)
**Next:** [API Gateway](api-gateway.md) →
**Related:** [Proxies & Load Balancers](proxies-load-balancers.md) · [Caching (architecture)](../architecture/caching.md)
