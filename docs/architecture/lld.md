# Low-Level Design (LLD)

## What Is It?

LLD is the **component-level design**: how one service/module is structured internally — its data models, interfaces, state machines, algorithms, and error paths — detailed enough that implementation becomes mostly transcription.

```text
HLD: "reorder-svc owns reorder logic, calls inventory (CP) and payments, p95 400ms"
LLD: the class/module diagram, the reorder state machine, the idempotency-key
     handling, the retry policy per dependency, the DB schema + indexes,
     the concurrency model — this component, executable on paper
```

## Why Does It Exist?

Because HLD's boxes hide exactly where bugs live: the state machine nobody drew, the retry that wasn't idempotent, the schema without the index the query needs. LLD forces the decisions that are *cheap on paper and expensive in production*:

- **State**: where it lives, every transition, every illegal transition *defended*
- **Interfaces**: signatures, error contracts, idempotency semantics (API page, internalized)
- **Data**: schemas, indexes, constraints — matching the query patterns
- **Concurrency**: what runs in parallel, what must not, what guards it

```text
The interview pairing (design an HLD → now design this class/parking-lot/elevator)
and the professional pairing (design doc → module spec) are, again, one discipline:
structure under constraints, zoomed in.
```

## Layer 1 — Simple Explanation

HLD is the **city master plan**; LLD is the **individual building's blueprint** — floor layouts, plumbing routes, fire doors, the elevator's state machine (idle→up→down→maintenance, and what happens when two buttons press during failure). Nobody pours concrete for the building from the master plan; nobody should code a service from the HLD alone.

## Layer 2 — Engineer's View

**The LLD artifact set (per component):**

| Artifact | Answers | The classic bug it prevents |
|---|---|---|
| Module/class diagram + responsibilities | who owns what logic | god-classes, feature envy |
| **State machine** (for anything stateful) | legal transitions + guards | order stuck in "processing" forever |
| Interface definitions | signatures, errors, idempotency | implicit contracts, retry corruption |
| Schema + indexes | storage shape *for the queries* | the full-scan "scalability" problem |
| Sequence diagrams for hard flows | ordering across async/timeouts | race conditions on paper |
| Error/exception taxonomy | which failure, handled where | swallowed exceptions, zombie states |

**The state machine discipline — LLD's highest-ROI habit:**

```text
Reorder: CREATED → RESERVED → PAID → FULFILLED
              ↘ CANCELLED           ↘ REFUNDING → CLOSED
Rules: every arrow a guarded transition; every non-arrow *impossible by construction*
       (single-writer row lock / optimistic version / idempotent transition API)
Timeout paths are arrows too: RESERVED --(payment timeout 15m)--> CANCELLED + release
The bug class this kills: "order in RESERVED for 3 days because the payment
       webhook was lost" — the timeout arrow nobody drew.
```

**SOLID and friends — the vocabulary for the class diagram** (one honest paragraph): single-responsibility, open/closed, dependency inversion — useful heuristics for module boundaries, cargo-culted into religion when applied to every method. The working rule: **apply at the boundary level you're designing** (modules for architects, classes for OOP, functions for the functional folks) and judge by the change-impact test: *one requirement change → how many modules touch?*

**Concurrency design (LLD's sharpest edge):** identify shared state (the bug's habitat); pick the guard per case — locks (short, named, ordered — deadlock prevention), optimistic versions (retry on conflict), actors/queues (share nothing) — and write the interleaving analysis for the two worst races (sequence diagrams over time). You did this mentally for the Job/CronJob page; LLD does it on paper.

**Testability as an LLD property:** seams for doubles (Testing page), deterministic clocks/IDs injectable, state machine transitions unit-testable in isolation — a design that can't be tested isn't designed yet.

## Real-World Example (DevOps flavored)

The reorder-svc LLD continuing the HLD page's example:

```text
Modules: api (transport) / reorder-core (state machine, pure) / adapters (inventory,
         payments, events) — core depends on nothing (dependency inversion: ports)
Schema: reorders(id, user_id, state, version, idem_key UNIQUE, created_at, ...)
        index (user_id, created_at desc) — the history query's index
State machine: as above, with the payment-timeout arrow + compensation via outbox
Races analyzed: webhook vs timeout (version-guarded: last writer checked state);
        double-tap (idem_key unique constraint answers)
Errors: taxonomy {retryable inventory, retryable payment, permanent user} → the
        adapter contracts state which is which (API page's failure fine print)
Tests: state machine transitions as pure unit table; adapters behind ports
```

Implementation after this is transcription — that's the deliverable.

## Common Mistakes

- Coding straight from HLD — the state machine and schema "designed" by accretion
- State machines without timeout/failure arrows — the zombie-state factory
- Schemas designed from entities, not from *queries* — the indexed-after-outage habit
- Concurrency "handled later" — shared state without named guards
- LLD as exhaustive UDD — the 40-page doc nobody reads; target: executable decisions

## Mental Model

> HLD zones the city; LLD draws the **building's blueprint** — and the state machine is the **fire-door plan**: every room's legal exits drawn, including the ones nobody takes ("payment timeout"), because the undocumented exit is where systems trap their users. When the blueprint is done, construction is typing.

## Remember This

1. LLD = executable-on-paper internals: modules, state machines, schemas, interfaces, races
2. State machines with timeout/failure arrows kill the zombie-state bug class
3. Schema from queries, not entities; indexes as part of design
4. Concurrency: name the shared state, name the guard, draw the two worst races
5. SOLID at your design level, judged by change-impact, not dogma
6. Testability is a design property (seams, injectable time/ids), not a phase

## One Sentence

Low-level design makes a single component executable on paper — modules, guarded state machines, query-shaped schemas, and named concurrency rules — so that implementation becomes transcription and the expensive bugs die in review instead of production.

## Knowledge Check

1. Draw the state machine for "subscription" including three failure arrows most teams forget.
2. A query filters by (tenant, status, created_at). Design the composite index and defend the column order.
3. The webhook-vs-timeout race: how does optimistic versioning resolve it?
4. Which LLD artifact would have caught your last zombie-record incident?

## Further Reading

- *Clean Architecture* — the ports/adapters core idea (read critically)
- HLD↔LLD interview pairing practice (the system-design → class-design sequence)
- Phase complete → [Why Platform Engineering](../platform-engineering/why-platform-engineering.md)

---

**← Previous:** [High-Level Design](hld.md)
**Next:** [Why Platform Engineering?](../platform-engineering/why-platform-engineering.md) →
**Related:** [HLD](hld.md) · [TDD & BDD](../development-practices/tdd-bdd.md)
