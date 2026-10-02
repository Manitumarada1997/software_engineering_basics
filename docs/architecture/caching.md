# Caching

## What Is It?

Caching is **remembering answers** to avoid recomputing/refetching them — at every layer you've already met: CPU caches, page cache (Memory page), CDN (Networking), Redis, app-level memoization, DNS TTLs. The architecture skill is doing it *deliberately*: where, what invalidates, and what staleness costs.

```text
The two questions every cache answers:
1. What's the key?        (identity — collision = data leak between users!)
2. What invalidates it?   (TTL? event? version? — the two hard problems joke)
```

## Why Does It Exist?

Because of the economics of locality: fetching/computing is expensive, remembering is cheap — a cache converts *latency and load* into *memory and complexity*. The wins cascade: DB read 8ms → cache hit 0.2ms → and the DB's freed capacity is *your scalability headroom* (Capacity page's ceiling list — the cache moves it).

The bill: **staleness** (CAP page's cheapest consistency — usually fine, sometimes not) and **complexity** (invalidation, coherence, cold starts).

## Layer 1 — Simple Explanation

The cache is the **notepad by the phone**: frequent answers ( pizza number, gate codes) written down for instant retrieval — with the rule that matters: *when the gate code changes, who updates the notepad, and how long is the wrong code acceptable?*

TTL is "rewrite the note weekly whatever happens"; event-based invalidation is "the office updates it when codes change" — and a stale note is better than no note... until someone's locked out at 2 AM.

## Layer 2 — Engineer's View

**The patterns, by position:**

| Pattern | Where | Traits |
|---|---|---|
| **Cache-aside** | app checks cache → miss → DB → fill | default; app owns logic; stale window = TTL |
| **Write-through** | writes go cache+DB together | fresh reads, write latency |
| **Write-behind** | cache first, DB async | fast writes, **durability risk** |
| **Read-through** | cache itself loads misses | infra-managed cache-aside |

**The failure modes to design against (each a famous outage shape):**

| Failure mode | Scenario | Defense |
|---|---|---|
| **Stampede** | hot key expires → 500 DB hits at once | request coalescing (single-flight), jittered TTLs, stale-while-revalidate |
| **Cold cache** | restart/flush → load hits DB at full depth | pre-warm, gradual traffic, capacity for cold |
| **Key collisions** | user A's data served to B | user-scoped keys — the security bug in caching |
| **Big-key/eviction** | one 50MB value evicting everything | size limits, small values |
| **Cache as SPOF** | Redis down → DB down (stampede) | cache resilience ≠ optional; degrade gracefully |

**Cache coherence across layers** (the distributed-caching truth): your system has *five* caches (browser, CDN, gateway, service, DB pool) — each with its own TTL. Total staleness = sum of windows; debugging "I changed it but see old" = walking the layers (`Cache-Control` headers, surrogate keys — the CDN page).

**Invalidation strategies ranked by honesty:**

```text
1. Content addressing: new content = new key (app.v123.js, image@sha256) — invalidation
   becomes unnecessary (Git/Images pages' idea — the gold standard)
2. Event-driven: write → invalidate key (correct; coupling to events)
3. TTL (short): eventually right, always simple — the default
4. TTL (long) + hope: the incident you're planning
```

**Redis specifically (the cache you operate):** single-threaded execution (big keys block everything — the capacity page's ceiling), eviction policies (allkeys-lru vs volatile-ttl), persistence optional (cache ≠ data store — but everyone eventually stores *something* there; make that a decision, not an accident).

## Real-World Example (DevOps flavored)

ShopEasy's cache stack, layered with intent:

```text
Product page:  CDN (immutable asset URLs, fingerprinted) + API cache 30s + stale-while-revalidate
Inventory:     NOT cached (CP feature — CAP page's quorum write) — a deliberate exception
Sessions:      Redis (the actual data store tier — sized, HA'd, *not* treated as cache)
Rankings:      precomputed projection refreshed hourly (write-heavy origin)
Stampede armor: single-flight on hot keys, TTL jitter ±10%
Incident prevented: cache flush at 10:00 would have been a DB outage — graceful
                    degradation mode (serve stale on error) shipped after the game day
```

The architecture-review question set: what's stale, for how long, per feature — the consistency sheet (CAP page) has a caching column now.

## Common Mistakes

- Unscoped keys — cross-user leakage (security incident)
- No stampede armor on hot keys — the expiry-synchronized avalanche
- Caching the CP features — consistency requirements violated by a TTL
- Redis as both cache and durable store without deciding which
- Ignoring the layer-sum: fixing one cache's TTL while four others serve old
- No cold-start capacity plan — the flush that became an outage

## Mental Model

> The cache is the **notepad by the phone** — until it's five notepads (layers) with different rewrite rules, and the office's answer to "how stale is stale?" becomes *the* consistency decision. Content addressing is the gold standard (new answer, new page); TTL+hope is the 2 AM lockout.

## Remember This

1. Every cache = key + invalidation policy + staleness budget (a per-feature decision)
2. Patterns: cache-aside default; write-through fresh; write-behind risky
3. Design against: stampedes (coalescing/jitter), cold starts, key collisions, SPOF-cache
4. Total staleness = sum of all layers' windows — debug them as a chain
5. Content addressing beats invalidation; TTL-short beats TTL-hope
6. Redis: single-threaded, eviction policy, and cache-vs-store must be a *choice*

## One Sentence

Caching trades recomputation for memory by remembering answers under an explicit staleness policy — valuable exactly in proportion to how deliberately the key, the invalidation, and the failure modes (stampedes, cold starts, collisions) are designed.

## Knowledge Check

1. Compute the staleness budget across: browser 60s, CDN 300s, API cache 30s — what does the user see after a price change?
2. Design the stampede defense for a hot product-detail key.
3. Which ShopEasy feature is deliberately uncached, and which consistency model forbids it?
4. Why does fingerprinting make cache invalidation a non-problem?

## Further Reading

- *Designing Data-Intensive Applications* ch. 3-4 · Redis docs (eviction, patterns)
- The CDN page (this page's edge chapter)

---

**← Previous:** [Event-Driven, Queues & Kafka](event-driven-kafka.md)
**Next:** [Replication & Sharding](replication-sharding.md) →
**Related:** [CDN](../networking/cdn.md) · [CAP & Consistency](cap-consistency.md)
