# Monolith, Modular Monolith & Microservices

## What Is It?

The service-decomposition spectrum:

```text
Monolith          — one deployable, one database, one process
Modular monolith  — one deployable, strictly-enforced internal modules (bounded contexts)
Microservices     — many deployables, each owning its data, communicating over the network
```

## Why Does It Exist?

Because **organizational scale and domain complexity** pressure the single deployable:

- One deployable = one release train: every team's change waits on everyone (batching — the CD page's enemy)
- One database = the schema nobody may change (integration database antipattern)
- One process = one scaling profile, one failure blast radius, one tech stack

Microservices answer those — at the price of *everything distributed* (next pages): network failure, consistency, observability, deployment ×40. The tradeoff is organizational: **you buy team autonomy with system complexity**.

Conway's law (1968, the deep reason): *system structure mirrors communication structure*. Microservices work when they match team boundaries; drawn against them, they produce the worst of both worlds.

## Layer 1 — Simple Explanation

- **Monolith**: a **single big shop** — everyone shares one entrance, one storeroom, one renovation schedule. Efficient at small scale; any renovation closes the whole shop
- **Modular monolith**: the **shop with strict departments** — one building, but menswear can't touch the grocery storeroom. Renovations still close the building, but messes stay in departments
- **Microservices**: the **shopping mall** — every store renovates, stocks, and fails independently; the cost: halls, signage, security contracts between stores (the network)

## Layer 2 — Engineer's View

**The honest comparison:**

| | Monolith | Modular monolith | Microservices |
|---|---|---|---|
| Deploy cadence | all together | all together | per-service |
| Data ownership | shared soup | modules' schemas | per-service DBs |
| Failure isolation | process-wide | process-wide | per-service |
| Scaling | whole app | whole app | per-service |
| Transactions | ACID, local | ACID, local | sagas, eventual |
| Observability | trivial | trivial | the whole SRE phase |
| Team autonomy | low | medium | high |
| Ops cost | 1× | ~1× | 3-10× |

Read that table with the NFR page's lens: microservices *buy* autonomy, isolation, scale — *paying* in consistency, latency, complexity, cost. The 20-dev org buying microservices is paying rent for a mall with two stalls.

**The modular monolith — the underrated middle** (Shopify, Basecamp, Stack Overflow): enforced module boundaries (separate schemas, no cross-module imports — via linters/arch-unit *enforced as code*, PaC-style) capture 70% of microservices' design value at monolith cost. The discipline it requires — clean bounded contexts — is *identical* to what microservices need; you're doing the hard part (domain modeling) either way.

**The migration law (Strangler Fig):** monolith → services *incrementally*: extract the boundary that hurts most (deploy friction, scaling need, PCI scope), behind an interface; route traffic gradually; repeat. Never big-bang. And the precondition: the modular-monolith boundaries must exist *first* — you can't extract what isn't bounded.

**The microservice prerequisites (the checklist before the first split):**

```text
✓ CI/CD per service (CD phase)     ✓ observability across services (SRE phase)
✓ deployment automation (K8s)      ✓ schema-change discipline (expand/contract)
✓ domain boundaries identified     ✓ teams organized to own services (Conway)
```

Missing any, and you've built a **distributed monolith**: network costs *and* coupling — the worst quadrant, where most failed adoptions live.

## Real-World Example (DevOps flavored)

ShopEasy's evolution (the whole course's company, structurally):

```text
Year 1: monolith (correct: 5 devs, shipping fast, one deployable)
Year 3: modular monolith — modules: catalog, checkout, payments, notifications;
        separate schemas, enforced boundaries; deploy friction begins to hurt at 25 devs
Year 5: extracted payments (PCI isolation NFR) + search (scaling NFR) as services;
        the rest stays modular — extracted only where an NFR demanded it
Rule they followed: every extraction justified by a numbered NFR (previous page),
        never by fashion; every non-extraction justified too.
```

## Common Mistakes

- Microservices at monolith scale — paying distributed costs with no autonomy to show
- The distributed monolith: services + shared database + release coordination = worst of both
- Skipping the modular stage: extracting unbounded goo produces unbounded services
- Big-bang rewrites instead of strangler figs
- Boundaries by technical layers (a "UI team", a "DB team") instead of domain boundaries
- Believing microservices fix a broken dev process — they multiply it by the service count

## Mental Model

> One shop, one renovation schedule (monolith); departments with locked storerooms (modular monolith); the mall where every store runs itself but you maintain halls and contracts (microservices). Conway's law is the **zoning commission**: systems mirror org charts whether you plan it or not — so plan the org chart you want, then build the building to match.

## Remember This

1. Spectrum: monolith → modular (enforced boundaries) → microservices (network + data ownership)
2. Microservices buy team autonomy and isolation; they pay in consistency, ops, cost
3. Modular monolith = the domain-modeling discipline without the network bill
4. Prerequisites (CI/CD, observability, IaC, Conway-aligned teams) or distributed monolith
5. Migrate by strangler fig, one NFR-justified boundary at a time
6. Boundaries follow the domain, never technical layers

## One Sentence

Service decomposition trades the coordination costs of one deployable for the network costs of many — a trade worth making only at organizational scale, after modular boundaries exist, and only where numbered NFRs demand it.

## Knowledge Check

1. Your 15-dev company is "adopting microservices." Which prerequisite gaps predict the distributed monolith?
2. Why must modular boundaries precede extraction — what does extracting unbounded code produce?
3. Justify extracting search but not order-history, in NFR terms.
4. How does Conway's law explain your current system's worst boundary?

## Further Reading

- *Monolith to Microservices* — Sam Newman (the balanced book)
- Martin Fowler — monolith-first, strangler fig articles
- Team Topologies (next phase) — the org side of boundaries

---

**← Previous:** [Non-Functional Requirements](nfrs.md)
**Next:** [Distributed Systems Fundamentals](distributed-fundamentals.md) →
**Related:** [Architecture Thinking](thinking-tradeoffs.md) · [Team Topologies](../platform-engineering/team-topologies.md)
