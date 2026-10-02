# Non-Functional Requirements

## What Is It?

NFRs (the *-ilities*) specify **how well** the system must work, not what it does — and they are the architect's actual raw material (the Thinking page's forces, as requirements):

| NFR | Question | Measured as |
|---|---|---|
| Performance/latency | how fast? | p95/p99 per operation |
| Scalability | how big? | throughput/rps at SLO, growth headroom |
| Availability | how up? | SLO (99.9%), error budget |
| Durability | how safe from loss? | backup RPO, replication |
| Recoverability | how fast back? | RTO |
| Security | how protected? | controls/compliance posture |
| Maintainability | how changeable? | lead time, change failure rate (DORA!) |
| Observability | how diagnosable? | MTTD (time to detect) |
| Cost | at what price? | cost per transaction/request |

## Why Does It Exist?

Because **"what" has many valid implementations, and NFRs are what choose between them** — two systems can both "do checkout" and be wildly different products. And because unstated NFRs don't stay at zero — they default to *whatever the implementation accidentally delivers*, discovered in production (the Requirements page's warning, system-scale edition).

```text
The classic:  "The system should be fast."
The NFR:      "Checkout p95 < 800ms at 500 rps sustained, p99 < 2s, on the golden path,
               with 1 dependency degraded."
The first is a wish; the second is testable (load tests), alertable (SLOs),
and designable (architecture).
```

## Layer 1 — Simple Explanation

Functional requirements are the **menu**; NFRs are the **service standards**: both restaurants "serve pasta," but one arrives hot in 8 minutes with the wine you didn't run out of. Nobody chooses a restaurant for the menu alone — nobody should choose an architecture for features alone.

The measurable rule: **an NFR without a number and a condition is a vibe.** "Scalable" means nothing; "3× current peak with p95 held" means engineering.

## Layer 2 — Engineer's View

**NFRs → mechanisms — the mapping table you've been assembling all course:**

| NFR | Mechanisms (pages you know) |
|---|---|
| Latency | caching, async, connection reuse, CDN |
| Scalability | stateless tiers, queues, sharding, HPA |
| Availability | redundancy, multi-AZ, graceful degradation, circuit breakers |
| Durability | replication, backups (3-2-1), immutable storage |
| Recoverability | IaC rebuild, DR tiers, game days |
| Security | Zero Trust, IAM, PaC, supply chain |
| Maintainability | modularity, tests, CI/CD, DORA metrics |
| Observability | OTel, SLOs, alerting discipline |
| Cost | FinOps, right-sizing, reservations |

Architecture = choosing the minimal set of mechanisms that satisfies *all* the NFRs at least cost — and NFRs *conflict* (CAP is the formal statement — a page soon), which is why they can't be maximized independently.

**NFRs conflict, and that's the design conversation:**

```text
Strong consistency ←→ availability/partition tolerance (CAP)
Low latency       ←→ strong durability (sync replication waits)
High security     ←→ low friction/latency
Everything        ←→ cost
```

**Writing real NFRs — the template:**

```text
[metric] [comparator] [value] [under conditions] [measured at] [enforced how]
p95 checkout < 800ms at 500rps, with payments degraded, measured at gateway,
enforced by SLO + load test gate in pipeline
```

The "enforced how" column is what separates NFR-as-culture from NFR-as-documentation: load tests gate releases (Testing page), SLOs gate deploys (burn rates), PaC gates budgets, DORA gates maintainability.

**Per-journey NFRs, not system-wide:** checkout needs 99.95%/800ms; order-history tolerates 99%/2s. System-wide numbers force gold-plating everything to the strictest case — the over-plating cost trap.

## Real-World Example (DevOps flavored)

ShopEasy's NFR sheet per journey (the input to every architecture review):

```text
Journey: checkout — p95<800ms @ 500rps · 99.9% SLO · RPO 1m · RTO 30m · PCI scope
Journey: search   — p95<300ms @ 2krps · 99% · stale-allowed 10min (cache-friendly)
Journey: order-history — p95<2s · 99% · RPO 24h
Cross: audit log immutable 7y (compliance) · cost ceiling: 2% of GMV in infra
Each row: enforced-by column naming the gate (load test, SLO alert, backup drill, FinOps report)
```

When the search team asked for a dedicated Aurora cluster, the sheet answered: your NFRs are met by the shared one + cache — request denied *by the numbers*, not by hierarchy.

## Common Mistakes

- Unquantified NFRs ("fast", "scalable", "robust") — vibes driving architecture
- System-wide uniform targets — gold-plating to the strictest journey
- NFRs with no enforcement mechanism — documentation theater
- Maximizing instead of satisfying (CAP-confusion at the requirements level)
- Cost omitted as an NFR — the budget discovered by finance, quarterly

## Mental Model

> NFRs are the **restaurant's service standards** — the measurable promises (8 minutes, hot, never out of the wine) that distinguish products with identical menus. Unstated standards default to the kitchen's mood; unstated NFRs default to whatever the code accidentally achieves, discovered by users.

## Remember This

1. NFRs are the how-well requirements — and the actual input to architecture
2. A number, a condition, a measuring point, and an enforcement mechanism — or it's a vibe
3. NFRs conflict (CAP is the formal case); satisfying ≠ maximizing
4. Map NFR → mechanisms (the whole course is that table)
5. Per-journey NFRs prevent gold-plating
6. Cost is an NFR with the same standing as latency

## One Sentence

Non-functional requirements quantify the system qualities — latency, availability, durability, cost — with numbers, conditions, and enforcement mechanisms, and they are the actual raw material of architectural tradeoffs.

## Knowledge Check

1. Rewrite "the system must be highly available" as a real NFR (all five fields).
2. Which NFRs conflict for checkout, and which mechanism resolves each pair?
3. Why do system-wide availability targets waste money? Show with two journeys.
4. Your maintainability NFR — what metric and what gate enforce it?

## Further Reading

- ISO/IEC 25010 (quality model — the formal taxonomy)
- *Release It!* — Nygard (NFRs driving stability patterns)
- Next: [Monolith → Microservices](monolith-microservices.md)

---

**← Previous:** [Architecture Thinking & Tradeoffs](thinking-tradeoffs.md)
**Next:** [Monolith → Microservices](monolith-microservices.md) →
**Related:** [SLI/SLO](../sre/slo-error-budgets.md) · [Performance Testing](../development-practices/perf-security-testing.md)
