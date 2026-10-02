# API Design

## What Is It?

An API is a **contract**: the interface through which consumers and providers agree to interact. The design discipline makes contracts that are useful, evolvable, and hard to misuse — REST, gRPC, GraphQL, and async event schemas (Events page) all share the underlying rules.

## Why Does It Exist?

Because of the one-way door principle (Thinking page): **public API shape is nearly irreversible** — every consumer's code encodes your choices, and every change ripples. A good contract enables independent evolution of both sides (the entire microservices bet rests on it); a bad one couples deployments tighter than the monolith ever was.

```text
HTTP page: the envelope. This page: the letter — resources, verbs,
           errors, versions, and the compatibility rules that let both sides change.
```

## Layer 1 — Simple Explanation

The API is the **restaurant's menu contract**: what's orderable, how it's described, what arrives. Great menus are organized by *diner intent* (not the kitchen's org chart), describe what you get (not the stove's internals), change without confusing regulars (the special disappears — the number-5 never gets silently spicier), and have a stated policy when things are off (86'd = 404, "we're out" = 409, not "mystery plate").

## Layer 2 — Engineer's View

**REST done as design (not just HTTP):**

```text
Resources as nouns:      /orders/{id}, not /getOrder
Stateless calls;         representation in, representation out
Status codes as contract: 200/201/202(accepted-async) · 400(your fault) · 401/403 ·
                          404/409/422 · 429(rate) · 5xx(our fault — the HTTP page's table)
Pagination/limits built-in from day one:  ?cursor=...&limit=50
Idempotency keys on writes:  Idempotency-Key: uuid → safe retries (Fundamentals page)
Errors as machines read them: { "code": "OUT_OF_STOCK", "order_id": "...", "retry": false }
```

**The evolution rules — compatibility as policy (the SemVer of interfaces):**

| Change | Compatible? |
|---|---|
| Add optional field/request param | ✅ additive = safe |
| Add endpoint / response field | ✅ |
| Remove/rename anything | ❌ breaking — new version or deprecation window |
| Add required field | ❌ breaking |
| Change semantics silently (field stays, meaning shifts) | ❌ worst kind |

Techniques for breaking changes: **expand/contract** (add new → migrate consumers → remove old — the same pattern as DB migrations), versioned paths (`/v2/`) as last resort, deprecation headers + sunset dates. Async events: schema registry + enforced compatibility modes (the Events page's contract).

**Choosing the style (the tradeoff table):**

| Style | Sweet spot | Cost |
|---|---|---|
| **REST/HTTP-JSON** | public APIs, ubiquity, tooling | chattiness, over/under-fetch |
| **gRPC** | internal service-to-service: typed contracts, streaming, speed | browser friction, contract-first culture needed |
| **GraphQL** | client-shaped queries, frontend agility | server complexity: caching, N+1, authZ granularity |

**The design-for-failure ruleset** (marrying Fundamentals): document timeouts guidance per endpoint; define rate limits + `Retry-After` behavior in the contract; idempotency on all mutating ops; error taxonomy that distinguishes retryable (`retry: true`) from permanent. An API without failure semantics is a contract with no fine print — consumers invent behavior (the retry storm).

**Design-first as process:** OpenAPI/proto definitions *before* implementation, reviewed like code, linted by policy (PaC for contracts: naming, pagination, error shape enforced in CI), mock-driven consumer development. The contract in the repo is the source of truth — code generates docs, not the reverse.

## Real-World Example (DevOps flavored)

ShopEasy's checkout v2 migration — compatibility in practice:

```text
v1: POST /orders {card, address}          — 14 consumers
v2 need: split payment methods, idempotent retries
Plan (expand/contract):
  Phase 1 (additive): POST /v1/orders accepts new fields; Idempotency-Key honored;
        responses gain payment_status field — zero consumer breakage
  Phase 2 (migrate): 12 consumers move; sunset header on remaining two; metrics
        track per-consumer usage of old fields (usage data drives deprecation)
  Phase 3 (contract): remove old fields after usage = 0 for 30 days
Rollback at every phase = feature flag off; no version cliff ever existed
```

## Common Mistakes

- Verbs in URLs, chatty APIs, kitchen-internal shapes leaking into representations
- Breaking changes shipped silently — the version cliff + consumer outage
- Errors as strings humans must parse ("Something went wrong") — no machine taxonomy
- No pagination on day one — the collection that grew
- Non-idempotent writes without keys — retries corrupting
- API docs generated-after-facto, drifting from reality (contract theater)

## Mental Model

> The API is the **menu as legal contract**: diner-intent organization, honest descriptions, additive-only changes, stated sunset policies, and fine print for failure (86'd, out-of-stock, come-back-tuesday). Kitchens reorganize nightly behind a stable menu — that stability *is* the independence both microservices and restaurants are buying.

## Remember This

1. API = contract = one-way door: design beats retrofit, additive-only evolution
2. Resources/verbs/status-codes/errors-as-taxonomy; pagination + idempotency from day one
3. Expand/contract for breaking changes; usage metrics drive deprecation, not dates
4. Style by context: REST public, gRPC internal, GraphQL frontend-shaped
5. Contracts include failure semantics — timeouts, retryability, limits
6. Design-first: spec reviewed/linted in CI (PaC for contracts)

## One Sentence

API design is contract engineering — defining resources, errors, and evolution rules so consumers and providers can change independently, with additive compatibility and explicit failure semantics making integration safe under distributed physics.

## Knowledge Check

1. Classify as breaking/safe: new optional field; new required header; enum value added to a response; enum value *removed*; field renamed.
2. Why must idempotency and retryability be contract-level, not implementation details?
3. Choose REST/gRPC/GraphQL for: public partner API, internal fraud-scoring service, mobile frontend — defend each.
4. Run the expand/contract for "rename user_id to customer_id" across 14 consumers.

## Further Reading

- *API Design Patterns* — JJ Geewax; Google's API design guide (resource-first, aip.dev)
- OpenAPI / protobuf specs; Next: [Design Patterns & ADRs](patterns-adrs.md)

---

**← Previous:** [Replication & Sharding](replication-sharding.md)
**Next:** [Design Patterns & ADRs](patterns-adrs.md) →
**Related:** [HTTP & HTTPS](../networking/http-https.md) · [Semantic Versioning](../development-practices/versioning.md)
