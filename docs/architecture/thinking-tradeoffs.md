# Architecture Thinking & Tradeoffs

## What Is It?

**Architecture** = the set of decisions that are *expensive to change later* (structure, boundaries, technology, data flows). **Architecture thinking** = making those decisions as explicit tradeoffs between competing forces, instead of defaults and fashion.

The architect's first law (via Mark Richards, essence): **everything is a tradeoff** — and the job is naming them:

```text
"There is no solution, only trade-offs." — Thomas Sowell, adopted by every architect
```

## Why Does It Exist?

Because late-changing decisions are where systems die:

- Boundaries drawn wrong = every feature crosses teams (Conway coupling)
- The wrong data store = a rewrite
- The "microservices because Netflix" default = distributed-systems costs with monolithic scale needs

The whole course has been tradeoff literacy in disguise — every "Why does it exist" section was one. This phase assembles the general skill: **requirements → forces → options → explicit choice → record it**.

## Layer 1 — Simple Explanation

Architecture is **city planning**: zoning (boundaries), roads (integration), utilities (shared platforms), and building codes (standards) — decisions that outlast any building. The planner who says "we'll just use whatever the trendiest city uses" gets a city that doesn't fit its geography.

Tradeoffs are the **budget**: every quality you buy (scale, resilience, flexibility) is paid in another currency (complexity, latency, cost, focus). No free qualities exist — architects who don't name the price leave the invoice for engineers.

## Layer 2 — Engineer's View

**The forces (what you're trading between) — all of them familiar by now:**

| Force | Bought with | The bill |
|---|---|---|
| Scalability | statelessness, distribution | complexity, latency, consistency |
| Consistency | coordination, quorums | availability, latency (CAP page next) |
| Availability | redundancy, replication | cost, complexity |
| Security | layers, verification | latency, friction |
| Speed of delivery | simpler patterns | scale ceilings |
| Evolvability | modularity, abstraction | upfront design cost |

**The decision method:**

```text
1. Constraints (non-negotiable): compliance, deadline, existing estate, budget
2. NFRs (the real requirements): the -ilities with numbers (next page)
3. Options ≥ 3, each with: gains, costs, failure modes, reversibility
4. Choose — optimizing for the constraint you can least afford to break
5. Record (ADR — patterns page) with the rejected options and why
6. Revisit triggers: "we re-decide if X" (make it a plan, not a religion)
```

**Reversibility as the master heuristic** (Bezos's one-way/two-way doors):

```text
One-way doors (irreversible-ish): data model, public API contract, org boundaries
    → analyze deeply, choose conservatively
Two-way doors (reversible): frameworks, infra choices with facades, deploy patterns
    → decide fast, learn by trying
Senior judgment = spending deliberation where reversibility demands it.
```

**The three failure modes of non-thinking:**

1. **Resume-driven design** — the tech du jour for problems you don't have
2. **Cargo cult** — Netflix's solution for Netflix's constraints at your scale
3. **Default inheritance** — "we've always used X" with no recorded reason (and no revision trigger)

**Architecture as ongoing process (the modern view):** not a Big Design Upfront phase — a continuous discipline: evolve with evidence (observability!), boundary decisions made just-in-time, fitness functions (automated checks that the -ilities still hold — PaC for architecture). "Architecture is the decisions you keep making, recorded."

## Real-World Example (DevOps flavored)

ShopEasy's "should checkout be a separate service?" — the method applied:

```text
Constraints: team of 6 owns checkout; PCI scope must be contained; 18-month horizon
NFRs: p95<800ms, 99.9%, deploy independently of catalog
Options:
  a) modular monolith module — simplest; PCI containment via deployment scope; deploy
     coupling with catalog (violates NFR #3 partially)
  b) separate service + own DB — meets all NFRs; costs: distributed tx (saga), ops surface
  c) service sharing the monolith DB — worst of both (distributed + coupled)
Choice: (b) — NFR #3 and PCI outweigh the saga complexity
Recorded: ADR-014; revisit trigger: "if team drops below 4, consider merging back"
```

That ADR paragraph — options, choice, trigger — is architecture thinking on one page.

## Common Mistakes

- Presenting choices without the rejected options (no tradeoff = no reasoning shown)
- Deliberating two-way doors for months and strolling through one-way ones
- Optimizing the -ility you're already best at (the scale-obsessed 200-user startup)
- No revisit triggers — decisions harden into religion
- Architects deciding alone; the team inherding decisions they don't understand

## Mental Model

> Architecture is **city planning under a budget**: zoning decisions outlast buildings; every quality is bought in another quality's currency; and the professional's signature is the *ledger* — options weighed, choice made, price named, revisit condition set. Cities without planners don't lack architecture — they have accidental, expensive architecture.

## Remember This

1. Architecture = the expensive-to-change decisions; thinking = explicit tradeoffs
2. Forces trade pairwise: scalability↔complexity, consistency↔availability, speed↔ceilings
3. Method: constraints → NFRs → 3+ options with costs → choose → record (ADR) → revisit triggers
4. Reversibility decides deliberation depth: one-way doors slow, two-way fast
5. Failure modes: resume-driven, cargo cult, default inheritance
6. It's continuous: evolve with evidence, keep fitness functions

## One Sentence

Architecture thinking makes the expensive-to-change decisions as explicit, recorded tradeoffs between competing quality forces — choosing deliberately what defaults and fashion would otherwise choose for you.

## Knowledge Check

1. Classify as one/two-way doors: database engine, message queue brand, public API shape, service mesh.
2. Name three forces you'd sacrifice for a 3-month deadline — and the invoice each runs up.
3. Write the ADR skeleton for "monorepo vs polyrepo" with revisit triggers.
4. What's the failure mode of optimizing an -ility you're already strong on?

## Further Reading

- *Fundamentals of Software Architecture* — Richards & Ford (forces, tradeoffs)
- *Building Evolutionary Architectures* — Ford, Parsons, Kua

---

**← Previous:** [Chaos Engineering](../sre/chaos-engineering.md)
**Next:** [Non-Functional Requirements](nfrs.md) →
**Related:** [Requirements & Work Items](../software-engineering/requirements-work-items.md)
