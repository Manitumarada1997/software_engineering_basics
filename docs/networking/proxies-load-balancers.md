# Proxies & Load Balancers

## What Is It?

- A **proxy** is a middleman that speaks both directions of a connection on your behalf.
  - *Forward proxy* — middleman for the **client** (egress control, filtering)
  - *Reverse proxy* — middleman for the **server** (entry point for many backends: NGINX, Envoy, Traefik, an Ingress)
- A **load balancer** is a reverse proxy whose main job is **distributing traffic across backends** — at layer 4 (TCP/UDP) or layer 7 (HTTP-aware).

## Why Does It Exist?

Both solve the "single point" problem from opposite ends:

- Without reverse proxy/LB: one server = one bottleneck + one point of failure; TLS/config/routing logic smeared across every app
- Without forward proxy: no control or visibility over what leaves your network

The reverse proxy is also the *natural policy point*: TLS termination, routing, auth offload, rate limiting, observability — one place instead of every service.

## Layer 1 — Simple Explanation

- **Forward proxy**: the **assistant who makes calls for you** — the bank never hears your voice; the assistant can also refuse to call certain numbers (egress filtering)
- **Reverse proxy**: the **restaurant's front desk** — one phone number, many kitchens; reception routes, holds, balances, and refuses if the kitchens burn
- **LB**: the receptionist's *seating chart strategy*: round-robin tables, least-busy assignment, or remembering which customer sat where (sticky)

## Layer 2 — Engineer's View

**L4 vs L7 balancing — the fundamental choice:**

| | L4 (TCP) | L7 (HTTP) |
|---|---|---|
| Sees | IP + port | full HTTP: paths, headers, cookies |
| Decisions | connections | per-request: route /api→svcA, canary by header, session by cookie |
| Cost | cheap, opaque | terminate TLS, parse HTTP — higher |
| Examples | cloud NLB, IPVS | ALB, NGINX/Envoy/Ingress |
| Blindness | can't see errors inside | can cache, retry, rewrite |

**Load-balancing algorithms:**

| Algorithm | Trades | Notes |
|---|---|---|
| Round robin | fairness | ignores actual load |
| Least connections | load awareness | better for variable request cost |
| Consistent hashing | stickiness | cache locality; adds/removing backends moves ~1/n keys |
| Weighted | capacity differences | asymmetric fleets |

**Health checks — where LBs earn their keep:** active probes (`/healthz` per interval) vs passive (count 5xx/timeouts). The two failure modes to fear: no health check (dead backend keeps receiving traffic) and a **health check everyone believes but tests nothing** (checks the process, not the dependency — returns 200 while the DB is down).

**Connection-level physics:** L7 proxies terminate client TCP and open backend TCP separately → **connection pooling + keep-alive to backends** matter enormously (the HTTP page's lesson, mechanized). Slow-client protection (buffering) is another reverse-proxy superpower: clients on bad WiFi can't tie up backend threads.

**The modern extensions of the pattern:**

| Thing | Actually is |
|---|---|
| Kubernetes Ingress | a reverse proxy config contract (implemented by NGINX/Envoy/Traefik controllers) |
| Service mesh sidecar | a reverse proxy per pod (mTLS, retries, metrics) — L4+L7 |
| API Gateway | a reverse proxy with opinions (auth, quotas) — next page |
| Cloud ALB/NLB | managed L7/L4 LB as a service |

One concept — middleman-at-the-entry-point — repeated at every layer of the stack.

## Real-World Example (DevOps flavored)

The NGINX config you've probably written, annotated:

```nginx
upstream payments {                 # the "backend pool" + health checks
    server 10.0.4.11:8080 max_fails=3 fail_timeout=10s;
    server 10.0.4.12:8080;
    keepalive 32;                   # pooled backend connections (HTTP page)
}
server {
    listen 443 ssl;                 # TLS termination here, plain HTTP inside
    location /pay/ { proxy_pass http://payments; }
    location /healthz { return 200; }
}
```

And the incident: after a deploy, 50% of requests fail though pods are healthy — the LB still had *stale* endpoints because health-check/deregistration delay exceeded rollout speed. LBs and deployment strategies must be designed together (CD page).

## Common Mistakes

- Sticky sessions everywhere (kills the stateless-scaling HTTP gives you)
- Health checks that test liveness, not readiness
- No backend keep-alive — proxy re-handshakes per request
- One giant central LB as SPOF — front it with DNS/anycast or cross-zone HA
- Forgetting idle timeouts < backend timeouts (mysterious mid-stream resets)

## Mental Model

> The reverse proxy/LB is the **front desk of a hospital**: one entrance (IP/DNS), triage by symptom (L7 path routing), distribution to available doctors (least-connections), health checks on staff (probes), and visitors on bad connections wait in the lobby (buffering) instead of blocking the ER.

## Remember This

1. Forward proxy serves the client; reverse proxy/LB serves the server
2. L4 balances connections cheaply; L7 sees HTTP and makes smart per-request decisions
3. Algorithms: round-robin / least-conn / consistent-hash (cache locality + stickiness)
4. Health checks gate correctness — test what makes the service *usable*, not just alive
5. Ingress, meshes, gateways are all this one pattern at different layers
6. Backend keep-alive and timeout alignment are the hidden performance/correctness levers

## One Sentence

Proxies interpose a controlled middleman on connections — forward for client egress, reverse for server entry — and load balancers extend that to distribute traffic by connection or request, with health-gated backend pools.

## Knowledge Check

1. Why can an L7 LB implement canary releases and an L4 one cannot?
2. Design a health check that would have caught your last "200-but-broken" incident.
3. Why does consistent hashing matter for cache-backed fleets?
4. What breaks when LB idle timeout exceeds backend keep-alive?

## Further Reading

- [NGINX basics](https://nginx.org/en/docs/beginners_guide.html), [Envoy architecture overview](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/arch_overview)
- *Designing Data-Intensive Applications* ch. 1 (load balancing in context)

---

**← Previous:** [TLS & Certificates](tls-certificates.md)
**Next:** [Firewalls & VPNs](firewalls-vpns.md) →
**Related:** [API Gateway](api-gateway.md) · [Services & Ingress](../kubernetes/services-ingress.md)
