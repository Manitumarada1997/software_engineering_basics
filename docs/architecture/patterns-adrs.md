# Design Patterns & ADRs

## What Is It?

Two complementary tools of the working architect:

- **Design patterns** — named, reusable solutions to recurring problems (vocabulary for structure)
- **ADR (Architecture Decision Record)** — a short document capturing *one* decision: context, options, choice, consequences (vocabulary for decisions)

```markdown
# ADR-014: Extract payments as a separate service

## Status: Accepted (2026-03-12)      ## Context
PCI scope containment requires isolation; deploy-independence NFR (checkout SLO sheet);
team of 6 owns payments. Current: modular-monolith module, shared deploy train.

## Options considered
1. Keep as module — simplest; fails PCI isolation + deploy-independence
2. Extract service + own DB (CHOSEN) — meets NFRs; cost: saga for checkout flow
3. Service sharing monolith DB — distributed + coupled: worst of both

## Consequences
+ independent deploys, PCI boundary, own scaling
− distributed transaction via saga; +1 service's ops surface
Revisit trigger: team size < 4 → consider merging back
```

## Why Does It Exist?

**Patterns** exist because naming solutions makes them discussable ("use a circuit breaker here" = a paragraph of design in two words — and its failure modes known). You already speak several from this course: retry+jitter, circuit breaker, saga, outbox, strangler fig, CQRS — the Fundamentals/Events pages were patterns wearing context.

**ADRs** exist because **decisions outlive the deciders and the reasoning**: the new engineer asking "why does payments have its own database?" gets "ADR-014" instead of folklore or silence. And the reverse failure: architecture wikis describe the *current state* endlessly while the *why* dies with the team — making change irrational (nobody knows which constraints still bind).

## Layer 1 — Simple Explanation

Patterns are the **professional trades' vocabulary**: a plumber saying "pressure-reducing valve" communicates a component, its purpose, and its failure modes in three words. ADRs are the **decision logbook**: every consequential choice with its reasoning on one page — so the building's next renovator knows which walls are load-bearing and *why the weird pipe routing exists* before demolishing it.

## Layer 2 — Engineer's View

**The pattern vocabulary worth fluency (you've met them; now they're named):**

| Category | Patterns (this course's chapters) |
|---|---|
| Resilience | circuit breaker, bulkhead, retry+jitter, timeout budget, fallback, load shedding |
| Data | outbox, saga, event sourcing, CQRS, read-your-writes sessions |
| Evolution | strangler fig, expand/contract, anti-corruption layer, facade |
| Scaling | cache-aside, consistent hashing, shard key, shared-nothing |
| Delivery | canary, feature flag, blue/green, pipeline-as-product |

The judgment patterns don't replace: *"add a pattern when a measured problem demands it"* — pattern-driven architecture (everything gets a saga) is resume-driven design's cousin. Each pattern has a *cost* row in its table; the Thinking page's rule applies.

**ADR discipline — what makes them work:**

| Rule | Why |
|---|---|
| One decision per ADR | reviewable, linkable, supersede-able |
| Context + options + consequences | the tradeoff IS the content (Thinking page's method, archived) |
| Rejected options included | prevents re-litigation and reveals forgotten constraints |
| Status lifecycle: proposed→accepted→superseded | never deleted — superseded by ADR-021, history preserved |
| In the repo, next to code | reviewed in PRs; found where the code lives |
| Short (1 page) | an ADR nobody reads preserves nothing |

**The anti-pattern: decision archaeology without ADRs** — the alternatives: commit-message folklore, the one architect's memory, or the Slack search from hell. Teams without ADRs make the *same decisions repeatedly* because the previous reasoning is unrecoverable (drift, for decisions).

**Patterns + ADRs = the architecture communication stack:** patterns compress *solutions*; ADRs persist *choices*. The review comment "this needs an outbox" and the ADR "we chose outbox over dual-write (ADR-022)" together let a 100-engineer org share one architectural brain.

## Real-World Example (DevOps flavored)

ShopEasy's `docs/adr/` directory — the architecture's real documentation:

```text
ADR-003 modular monolith (rejected microservices at 15 devs)
ADR-009 event backbone: Kafka (options: RabbitMQ, cloud bus) 
ADR-011 API gateway (options: Kong, cloud-native, none)
ADR-014 payments extraction — superseded constraints revisited 2027 → ADR-031
ADR-022 outbox over dual-write
Onboarding effect: new senior dev reads 31 ADRs in an afternoon and debates
                  like a five-year veteran — the org's decisions, transferable.
```

## Common Mistakes

- Pattern-driven architecture — solutions hunting for problems (the saga nobody needed)
- ADRs as essays (10 pages, unread) or as state-descriptions (no options/consequences)
- Deleting/refusing to supersede — history erased, re-litigation enabled
- ADRs in a wiki far from code — findability death
- Decisions without revisit triggers hardening into religion

## Mental Model

> Patterns are the **trade's shared vocabulary** — compressed solutions with known failure modes; ADRs are the **site's logbook** — each consequential choice with its reasoning on one page. Together they form the organization's architectural memory: vocabulary to discuss, logbook to remember — and demolition crews who know the load-bearing walls.

## Remember This

1. Patterns = named reusable solutions; you already speak a dozen from this course
2. Every pattern has a cost row — adopt on measured demand, not fashion
3. ADR: one decision, context/options/consequences, rejected options, status lifecycle
4. Supersede, never delete; keep them short and in the repo
5. ADRs prevent decision re-litigation and archaeology — the org's memory
6. Patterns compress solutions; ADRs persist choices — the communication stack

## One Sentence

Design patterns give architecture a compressible solution vocabulary and ADRs give it a durable decision memory — together letting a large organization discuss, choose, and remember its structure without folklore.

## Knowledge Check

1. Name the pattern each course chapter quietly taught (one per phase).
2. Write ADR-001 for your current project's biggest unexplained decision.
3. Why are rejected options the most valuable section when read 3 years later?
4. When does adopting a pattern become anti-pattern behavior?

## Further Reading

- Michael Nygard — "Documenting Architecture Decisions" (the founding essay)
- *Release It!*/DDIA — the pattern sources you've been reading all along
- Next: [High-Level Design](hld.md)

---

**← Previous:** [API Design](api-design.md)
**Next:** [High-Level Design](hld.md) →
**Related:** [Architecture Thinking](thinking-tradeoffs.md)
