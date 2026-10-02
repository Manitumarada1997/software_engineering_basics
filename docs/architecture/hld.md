# High-Level Design (HLD)

## What Is It?

HLD is the **system-level design**: the whole solution's shape — components, their responsibilities, their interactions, technology choices, and how the NFRs are met — at the whiteboard altitude, before any class diagrams.

```text
The HLD document skeleton:
1. Requirements: functional (brief) + NFRs (the numbers — the NFR page)
2. Constraints & assumptions
3. Component diagram + responsibilities (bounded contexts as services/modules)
4. Key flows: the 3-5 journeys, end-to-end (sequence diagrams)
5. Data: stores per component, ownership, consistency model per datum (CAP sheet)
6. Technology choices with one-line ADR references
7. NFR satisfaction: latency budget, availability math, scale plan, security posture
8. Failure analysis: what breaks, what happens then (degradation ladder)
9. Open questions & risks
```

## Why Does It Exist?

Because expensive-to-change decisions (Thinking page) need a review *before* code: HLD is where the tradeoffs get argued, on paper, with the cheapest possible medium. The interview-skill version (system design) and the professional version (design doc) are the same discipline: **structured reasoning about structure**, under explicit constraints.

```text
The 45-minute interview and the 2-week design doc differ in depth, not in kind:
requirements → estimation → components → data → failure → tradeoffs → iterate
```

## Layer 1 — Simple Explanation

HLD is the **city master plan**: zoning (components), major roads (integrations), utilities (shared services), the flood plan (failure analysis) — drawn and argued *before* anyone pours concrete. LLD (next page) is the individual building's blueprint. Cities built blueprint-first produce commutes; cities built building-first produce Los Angeles.

## Layer 2 — Engineer's View

**The method (interview-grade and production-grade):**

```text
1. Clarify functional scope (the 3-5 journeys) + NFRs with numbers
2. Estimation (the back-of-envelope that anchors everything):
     users × requests/user → rps → read/write split → data size/day → growth
     → this arithmetic *chooses* the architecture (single DB vs sharded; sync vs async)
3. Sketch components (API, services, data stores, queues, caches)
     — start with the *journals* (journeys), place state where it's touched
4. Deep-dive the two hardest pieces (where the design lives or dies)
5. Failure-mode pass: kill each component in your head — the ladder answers
6. Bottleneck pass: at 10× the estimate, what breaks first? (the ceilings list)
7. Name the tradeoffs explicitly; ADR the two biggest choices
```

**The estimation arithmetic (the muscle to train — the interview's real subject):**

```text
1M DAU × 5 orders/day = ~58 orders/sec avg, ~6× peak = 350/s peak writes
500KB/order → 43GB/day → 15TB/year → retention policy required
Reads 10× writes → read-path caching early (Caching page)
→ these four lines just chose: async ingestion, cache-first reads, yearly archives
```

**Component vocabulary — your full course, as a palette:**

| Concern | Palette (all previously studied) |
|---|---|
| Entry | CDN, LB, API gateway, ZTNA |
| Compute | service/modules (spectrum page), functions |
| Data | SQL/NoSQL/cache/search (storage page), replication/sharding ladder |
| Integration | REST/gRPC/events+queues (API/Events pages) |
| Resilience | breaker, bulkhead, shedding, degradation ladder |
| Ops | observability substrate, IaC, GitOps, SLOs |

**What "good" looks like (the review checklist):**

- NFRs restated with numbers; the design visibly satisfies each (or negotiates it)
- Every component has a *reason* and an owner-shaped boundary (Conway)
- Data ownership explicit; consistency per datum (the CAP sheet)
- Failure analysis exists — you can answer "what happens if X dies" for every X
- 10× path named (the ceiling + the next ladder rung)
- Tradeoffs named with rejected options (mini-ADRs embedded)
- Open questions honest — the risks section isn't empty theater

## Real-World Example (DevOps flavored)

ShopEasy's "one-tap reorder" HLD — the skeleton, filled (condensed):

```text
Journeys: user taps reorder → history lookup → availability check → payment → confirmation
NFRs: p95 400ms · 99.9% · RPO 1m · 10M DAU
Estimation: 8k rps peak reorder-starts, 90% served from history-cache
Components: gateway → reorder-svc → (history cache [Redis], inventory [CP quorum],
            payments [existing]), events → orders topic
Deep-dive: availability-check consistency (CP — oversell economics) + cache key design
Failure: inventory down → block reorder (feature flag), not the catalog; payments degraded
         → queue + confirm-later path (async escape)
10×: history cache → cluster; inventory → shard by SKU; reorder-svc → stateless scale
Tradeoffs: sync inventory (correctness) vs async (latency) → CP chosen, ADR-033
```

Two pages; every line traces to a course page. That's HLD literacy.

## Common Mistakes

- Designing from requirements without NFRs — structure with no selection pressure
- Skipping estimation — architecture without arithmetic is decoration
- All components, no flows — a box diagram that explains nothing
- No failure pass — the sunny-day design; and no 10× pass — the 6-month rewrite
- Deep-diving everything (no depth anywhere) — pick the two hardest pieces

## Mental Model

> HLD is the **master plan argued on cheap paper**: zoning and roads sized by arithmetic (estimation), flood plans for every failure, and the 10-year road named. The interview version compresses it to 45 minutes; the professional version adds ADRs and owners — the discipline is identical: *structure, reasoned under constraints, before concrete*.

## Remember This

1. HLD = system shape under NFR pressure: components, flows, data, failure, tradeoffs
2. Estimation arithmetic chooses the architecture — train the muscle
3. Method: clarify → estimate → components → deep-dive two → failure → 10× → ADR the big ones
4. The palette is the whole course; every line of a good HLD traces to a pattern you know
5. Good = numbers restated, ownership explicit, every "what if X dies" answered
6. Interview HLD and design-doc HLD are one discipline at two depths

## One Sentence

High-level design shapes the whole system — components, data ownership, key flows, and failure behavior — as explicit tradeoffs under estimated load and numbered NFRs, before any low-level structure exists.

## Knowledge Check

1. Size it: 5M DAU, 20 reads/user/day, 2KB/read — rps, storage/year, and the architecture those numbers force.
2. Your HLD review: which missing section predicts the 3-month-later rewrite?
3. Why deep-dive exactly the hardest two components rather than all?
4. Where do the CAP sheet and degradation ladder appear in the HLD skeleton?

## Further Reading

- *Designing Data-Intensive Applications* ch. 1 (the mindset); System design primer (GitHub)
- Next: [Low-Level Design](lld.md)

---

**← Previous:** [Design Patterns & ADRs](patterns-adrs.md)
**Next:** [Low-Level Design](lld.md) →
**Related:** [NFRs](nfrs.md) · [Architecture Thinking](thinking-tradeoffs.md)
