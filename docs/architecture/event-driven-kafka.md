# Event-Driven Architecture, Queues & Kafka

## What Is It?

**EDA**: systems communicating by *publishing facts* ("OrderPlaced") rather than *requesting actions* ("create invoice") — via brokers between producers and consumers.

The vocabulary:

| Pattern | Mechanism | Guarantee | Use |
|---|---|---|---|
| **Queue** (point-to-point) | one consumer gets each message | work distribution | task offloading |
| **Pub/Sub** (topics) | all subscribers see each event | broadcast | integration, fan-out |
| **Event log/stream** (Kafka) | ordered, append-only, *replayed* log | retention + replay | the source of truth pattern |

Kafka = the industrial event log: partitioned (parallelism), replicated (durability), offset-based consumption (consumers track position; the log retains).

## Why Does It Exist?

Two chronic diseases of synchronous architecture (Fundamentals page's physics):

```text
1. Temporal coupling: caller and callee must both be up and fast
   → queue decouples: work buffered, consumer catches up later
2. Integration sprawl: every new consumer needs a change in every producer
   → events decouple: publish once, subscribe independently — new consumers
     without touching producers
```

Plus the property no request/response system has: **the log as replayable truth** — rebuild a consumer, a cache, even a whole database by replaying history (the Git-history idea applied to runtime data).

## Layer 1 — Simple Explanation

Request-driven is a **phone call**: both parties needed *now*, busy signals, cascades. Event-driven is the **company newsletter + bulletin board**: departments publish facts ("order placed"); anyone interested reads and reacts — *this quarter or next* — and a new subscriber can go read the archives (the log).

The newsletter's tradeoffs: no instant confirmation ("did they handle it?"), reading order questions (partitions), and a mailroom to run (the broker — now critical infrastructure).

## Layer 2 — Engineer's View

**The guarantees you must design explicitly (no defaults exist):**

| Concern | The decision |
|---|---|
| Delivery | at-most-once (loss ok) / **at-least-once + idempotent consumers** / exactly-once (bounded scope) |
| Ordering | per-partition only — partition by key (order_id) for per-entity order |
| Retention | hours → forever (the replay decision = the architecture) |
| Poison messages | DLQ (dead letter queue) + alerting — stuck messages must not block partitions |
| Schema | schema registry + compatibility rules (the API versioning problem, async) |

**Kafka mechanics — the mental model:**

```text
topic: orders, 12 partitions, replication factor 3
producer → partition by key hash (same order → same partition → ordered)
consumer groups: each partition consumed by exactly one member per group
                → group parallelism = partition count; rebalance on membership change
offset: consumer's bookmark — committed after processing; rewind = replay
```

The consumer-group rebalance is the operational sharp edge (the pause that looks like an outage); static membership + cooperative rebalancing blunt it.

**The architecture-level patterns events enable:**

```text
Event sourcing:  the log IS the database (state = fold(events)) — audit + rebuild for free;
                 complexity: queries need projections
CQRS:            separate write model (commands) from read models (projections fed by events)
Outbox pattern:  DB write + event publish atomically (write events to an outbox table,
                 a relay publishes) — kills the dual-write inconsistency
Saga:            distributed transactions as event-choreographed steps + compensations —
                 the microservices answer to "no 2PC"
```

**The honest costs (why not everything is events):** eventual consistency everywhere (CAP page — you chose it), debugging across async boundaries (traces must propagate through the log — Tracing page's queue-boundary warning), versioned event schemas are *contracts* as hard as APIs, and **the broker is crown-jewel infrastructure** (multi-AZ, retention cost, rebalances).

**Mixing rule (the mature pattern):** synchronous where you need an answer now (payment authorization), events where you need decoupling and durability (order fulfillment fan-out), the outbox where both worlds meet.

## Real-World Example (DevOps flavored)

ShopEasy's order flow — the canonical mix:

```text
checkout API (sync): validate + authorize payment → return order confirmation  ← needs answer now
  └─ outbox row in same DB transaction
relay → Kafka topic orders.placed
  ├─ inventory consumer  (group: inventory — key: order_id partition → ordered)
  ├─ notification        (independent group — its own offsets)
  ├─ analytics projection (CQRS read model)
  └─ fraud (new subscriber added 6 months later — zero producer changes)
DLQ + lag alerts (consumer lag = the SLO of async systems!)
New warehouse rebuild 2027: replay orders.placed from offset 0 — history as backup
```

Note the monitoring translation: *consumer lag* is the async golden signal (queues filling = saturation).

## Common Mistakes

- Events as commands in disguise ("CreateInvoice" — coupled, and now also async: worst of both)
- Non-idempotent consumers under at-least-once — duplicates as data corruption
- Dual writes (DB + publish, no outbox) — the inconsistency generator
- Unkeyed producers → no ordering where the domain needs it
- No DLQ/limits — one poison message stalls a partition forever
- Treating the broker as plumbing — it's the most critical system you run (or buy managed)

## Mental Model

> Events are the **company bulletin board with perfect archives**: departments publish facts, anyone subscribes anytime, and the archive (the log) lets you rebuild the company from its history. The phone call (sync) remains right when you need an answer *now* — the maturity is knowing which conversations are which.

## Remember This

1. Events = published facts; queues distribute work; the log adds ordered replay
2. Buys temporal decoupling + integration decoupling; pays eventual consistency + broker ops
3. Decide per concern: delivery (idempotent at-least-once default), per-key ordering, retention, DLQ, schemas
4. Outbox kills dual-write; sagas replace 2PC; event sourcing/CQRS ride the log
5. Consumer lag = async saturation signal; the broker is crown-jewel infra
6. Mix: sync for answers-now, events for decoupled durability

## One Sentence

Event-driven architecture replaces temporal and integration coupling with publishable facts on durable logs — buying decoupling, replay, and fan-out at the price of eventual consistency and a broker that becomes your most critical infrastructure.

## Knowledge Check

1. Why does "exactly-once" need qualification — what's actually guaranteed?
2. Design ordering for per-order processing: key choice, consumer group implications.
3. What does the outbox pattern prevent, mechanically?
4. Your consumer lag is growing at 2 AM. What are the three suspects?

## Further Reading

- *Designing Data-Intensive Applications* ch. 11 (streams — the canonical chapter)
- Kafka docs — consumer groups, delivery semantics
- Next: [Caching](caching.md)

---

**← Previous:** [CAP & Consistency](cap-consistency.md)
**Next:** [Caching](caching.md) →
**Related:** [CAP](cap-consistency.md) · [Tracing across queues](../sre/tracing-otel.md)
