# HTTP & HTTPS

## What Is It?

**HTTP** — the application-layer (L7) protocol of the web: a stateless request/response conversation between client and server. **HTTPS** — HTTP wrapped in **TLS**: encrypted, authenticated, integrity-protected transport (TLS details next page).

```http
POST /checkout HTTP/1.1
Host: api.shopeasy.io
Authorization: Bearer eyJ...
Content-Type: application/json

{"cart": "abc-123"}
```

```http
HTTP/1.1 201 Created
Content-Type: application/json
{"orderId": "7788"}
```

## Why Does It Exist?

The web needed one thing: a simple, uniform way to *name resources and act on them* (GET/PUT/POST/DELETE on URLs) that any client and any server could speak. Stateless = every request stands alone = servers need no conversation memory = trivial horizontal scaling. That single design choice is why load balancers can round-robin your fleet without sticky logic.

## Layer 1 — Simple Explanation

HTTP is a **restaurant order slip**: method (what to do), URL (which dish), headers (special instructions — "no cilantro, my card ends 4417"), body (the details). The kitchen responds with a status slip (200: here's the dish; 404: no such dish; 503: kitchen on fire).

HTTPS is that slip carried in an **armored briefcase** (TLS) — courier can't read it, can't swap it, and the restaurant's identity is verified by badge (certificate).

## Layer 2 — Engineer's View

**The status-code families (what your dashboards are made of):**

| Family | Meaning | The ops reading |
|---|---|---|
| 2xx | success | — |
| 3xx | redirect | 301/302 permanence matters for caching; 429-adjacent loops |
| 4xx | **client** error | 401/403 auth, 404 routing, 409 races, **429 rate limit** |
| 5xx | **server** error | 500 bugs, 502/503/504 = upstream/gateway problems — *your* pager |

**Headers that run production:**

| Header | Job |
|---|---|
| `Host` | name-based virtual hosting (which site on this IP) |
| `Content-Type` | body interpretation (JSON? form? multipart?) |
| `Authorization` | credentials (Bearer tokens/OIDC — Security phase) |
| `Cache-Control` | CDN/browser caching policy (CDN page) |
| `Connection: keep-alive` | reuse TCP connections — or death by handshake latency |
| `Retry-After` | the polite 429/503 companion |

**Version evolution — each version fixed a latency problem:**

| Version | Fix | Ops impact |
|---|---|---|
| 1.1 (1997) | keep-alive, Host | 6-connections-per-host limit → domain sharding era |
| 2 (2015) | one connection, multiplexed binary frames, header compression | less head-of-line at HTTP layer; still TCP-level HOL |
| 3 (QUIC, on UDP) | no TCP HOL blocking, 0-RTT resume, connection migration | LB/firewall support = the feature flag to check |

**The performance physics of HTTP ops:**

- Every request = network RTT chain (DNS + TCP + TLS + request) — *geography is latency* (CDN page exists for this)
- Keep-alive/connection reuse is the difference between 2ms and 200ms per call
- Timeouts are *contracts you must set* (connect/read/overall) — a missing timeout is a hanging thread pool

**Statelessness and its bill:** no conversation memory means state travels per-request (auth headers, cookies, tokens) — and horizontal scaling is free. Every "sticky session" you've configured is the cost of an app that regressed from this design.

**gRPC/REST/APIs (preview):** HTTP is the envelope; REST/JSON and gRPC (HTTP/2, binary, streaming) are letter formats — full treatment in the Architecture phase's API design page.

## Real-World Example (DevOps flavored)

Reading an incident from headers alone:

```bash
curl -v https://api.shopeasy.io/checkout
# < HTTP/1.1 502 Bad Gateway
# < Via: 1.1 google      ← the LB answered; upstream did not
# 502 from the LB = app crashed/refusing; 503 = app overloaded/no backends;
# 504 = upstream accepted but never answered (timeout or thread starvation)
```

That 502/503/504 trio — each from a different layer of your stack — is the single most valuable HTTP table for on-call.

## Common Mistakes

- No timeouts/retries (or retry storms on 5xx — add jittered backoff, retry only idempotent requests)
- 404s masked by SPA fallbacks (all paths return 200 + HTML) — monitoring goes blind
- Ignoring keep-alive (new connection per request through expensive middleware)
- Confusing 401 (unauthenticated) with 403 (unauthorized)
- Treating HTTP/3 support as default — LB/network policy must explicitly allow QUIC/UDP

## Mental Model

> HTTP is the **order slip**; HTTPS puts it in a **verified armored briefcase**. Stateless slips mean any cashier can serve any customer — which is exactly why you can scale a web tier to hundreds of replicas behind one load balancer.

## Remember This

1. Request = method + URL + headers + body; response = status + headers + body
2. Stateless design = free horizontal scaling; state rides in headers/tokens
3. Status families: 4xx client, 5xx server; 502/503/504 mean three different infrastructure failures
4. HTTP/2 multiplexes; HTTP/3 moves to QUIC/UDP — check LB support
5. Timeouts are contracts; retries need backoff + idempotency

## One Sentence

HTTP is the stateless request-response language of the web whose design makes web tiers horizontally scalable, and HTTPS wraps it in TLS so the courier can neither read nor tamper with the conversation.

## Knowledge Check

1. Why does statelessness make load balancing trivial but sessions expensive?
2. Distinguish 502, 503, and 504 by which component failed.
3. What latency did HTTP/2 and HTTP/3 each eliminate?
4. Why must retries be paired with idempotency and jitter?

## Further Reading

- [MDN HTTP guide](https://developer.mozilla.org/en-US/docs/Web/HTTP) — the best single reference
- *High Performance Browser Networking* — Grigorik (free), HTTP chapters

---

**← Previous:** [DNS, DHCP & NAT](dns-dhcp-nat.md)
**Next:** [TLS & Certificates](tls-certificates.md) →
**Related:** [Proxies & Load Balancers](proxies-load-balancers.md) · [API Gateway](api-gateway.md)
