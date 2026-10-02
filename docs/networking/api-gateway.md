# API Gateway

## What Is It?

A **policy-enforcing front door for APIs**: a specialized reverse proxy (Proxies page) that centralizes cross-cutting concerns — authentication, authorization, rate limiting, routing, transformation, metrics — so individual services don't reimplement them.

```text
Client → Gateway → [authN → rate limit → route → transform → meter] → service A / B / C
```

Products: Kong, Apigee, AWS API Gateway, Azure API Management, Envoy Gateway.

## Why Does It Exist?

Microservices created an N×M problem: every service needs auth, rate limiting, TLS, logging, versioning, documentation — repeated in every language and team, inconsistently. The gateway extracts these into **one governed boundary**:

- Clients get one endpoint and one contract style (vs knowing 40 services' quirks)
- Security gets one place to enforce "valid token? within quota?" (shift-left for attackers)
- The org gets metering, API discovery, and deprecation policy

**The trade-off it buys with:** the gateway is a potential bottleneck and single point of failure — everything passes through one policy layer. (Hence "gateway as a managed HA service" and the microgateway/sidecar variants.)

## Layer 1 — Simple Explanation

The gateway is the **hotel's main entrance desk**: one door for all guests; verifies identity (authN), checks the booking list (authZ — which wings you may enter), enforces visitor limits (rate limiting), gives directions (routing), and logs everyone through (metering). Individual floors don't each employ their own bouncer.

## Layer 2 — Engineer's View

**Cross-cutting concerns, centralized:**

| Concern | Gateway implementation |
|---|---|
| AuthN | JWT/OIDC validation, API keys, mTLS termination |
| AuthZ | coarse route-level scopes (`scope=payments:read`) — fine-grained stays in services |
| Rate limiting | per key/IP/route quotas + bursting (token bucket) — protecting backends |
| Routing/versioning | `/v1/payments` → service-a v1; canary by header |
| Transformation | request/response mapping, protocol bridging (REST↔gRPC) |
| Observability | uniform access logs, latency metrics, request IDs injected |
| Documentation | OpenAPI publication + developer portal |

**Rate limiting algorithms (choose consciously):**

| Algorithm | Behavior |
|---|---|
| Token bucket | steady refill, allows bursts — the default good choice |
| Leaky bucket | smooths to constant rate |
| Fixed window | simple, burst-at-boundary artifacts |
| Sliding window | accurate, more state |

Distributed rate limiting (shared counters in Redis vs per-node limits) is the scaling decision — per-node limits multiply by replica count.

**Gateway vs LB vs Ingress vs service mesh — the landscape map:**

| Component | Primary job |
|---|---|
| LB | distribute traffic (L4/L7) |
| Reverse proxy | HTTP entry point for backends |
| **API Gateway** | reverse proxy + **API policy** (auth, quotas, contracts) |
| Ingress | K8s-standard L7 entry (often *is* NGINX — a gateway in disguise) |
| Service mesh | east-west (service-to-service) mTLS/routing via sidecars |

South-north (edge) concerns → gateway; east-west (internal) → mesh. Real systems compose them: CDN → gateway → services ↔ mesh.

**The architectural caution — the "ESB fallacy" reincarnated:** putting *business logic* in the gateway (validation rules, orchestration, data enrichment) recreates the enterprise service bus antipattern: a smart middleman every team queues behind. Gateway = *policy*, not *logic*. Keep custom code out; when you need edge logic, prefer plugins with strict review.

## Real-World Example (DevOps flavored)

Kong in front of ShopEasy's API estate:

```yaml
routes:
  - paths: [/v1/payments]
    plugins:
      - jwt-verifier        # authN: OIDC tokens from the IdP
      - rate-limiting: { minute: 600, policy: redis }   # shared across replicas
      - correlation-id
  - paths: [/v1/public]
    plugins:
      - rate-limiting: { minute: 60, policy: local }
```

The ops wins you can measure: credential rotation in one place; the 429 wall that saved checkout during a partner's infinite retry loop; access logs with uniform request IDs feeding every trace (Observability phase).

## Common Mistakes

- Business logic creeping into gateway configs/plugins — the ESB trap
- Gateway as SPOF without HA/health-checked multi-replica deployment
- Per-node rate limits on a 12-replica gateway (12× the intended quota)
- Fine-grained authZ at the edge ("can user 42 edit order 9?") — that's service territory
- No versioning/deprecation policy — the API contract nobody dares change

## Mental Model

> The API gateway is the **main entrance desk of a large building**: one door, identity checked, quotas enforced, directions given, everyone logged — and deliberately *dumb about what happens on the floors*. The moment the desk starts making business decisions, the building has a bottleneck with a nametag.

## Remember This

1. Gateway = reverse proxy + API policy (authN/authZ, quotas, routing, metering)
2. Extracts cross-cutting concerns once instead of N teams × M services
3. Rate limiting: token bucket default; distributed counters when replicas share quotas
4. South-north (edge) = gateway; east-west (internal) = mesh — they compose
5. Policy at the edge, logic in services — the anti-ESB rule
6. It's a potential SPOF/bottleneck: HA it, monitor it, keep config reviewed

## One Sentence

An API gateway centralizes API policy — authentication, rate limits, routing, and metering — behind a single highly available entry point so services stay focused on logic instead of repeating edge concerns.

## Knowledge Check

1. Which concerns belong in a gateway and which must stay in services — and why does the split matter?
2. Your rate limit "works in staging, not in prod" — diagnose the replica/quota interaction.
3. How do gateway, Ingress, and mesh relate — compose them for a realistic system.
4. What is the ESB fallacy's modern form?

## Further Reading

- [Kong docs — rate limiting plugin](https://docs.konghq.com/hub/kong-inc/rate-limiting/)
- *Building Microservices* — Sam Newman, ch. on edge services
- [API Gateway pattern — Microsoft Azure architecture center](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/gateway)

---

**← Previous:** [CDN](cdn.md)
**Next:** [What Cloud Really Is](../cloud/cloud-fundamentals.md) →
**Related:** [Proxies & Load Balancers](proxies-load-balancers.md) · [HTTP & HTTPS](http-https.md)
