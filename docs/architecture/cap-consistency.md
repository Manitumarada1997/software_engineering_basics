# CAP Theorem & Consistency Models

## What Is It?

**CAP** (Brewer, 2000): a distributed data store, during a **network partition**, can guarantee only one of:

```text
C (linearizable consistency): every read sees the latest write — or fails
A (availability):             every request gets a (non-error) response
P (partition tolerance):      the system survives the network splitting

P is not optional — networks DO partition (Fundamentals page).
So the real choice: during a partition, do you refuse answers (CP) or serve possibly-stale ones (AP)?
```

## Why Does It Exist?

Because replication (multiple copies for availability/performance — Storage page) creates the **split-brain question**: two replicas, network split, writes arrive on both sides. Both answer? (AP — they may *disagree* later.) Refuse until healed? (CP — available to no one.) There is no third answer — that's the theorem.

**The practical subtlety everyone gets wrong:** CAP is about *partitions*, which are rare. The everyday tradeoff is actually **latency-vs-consistency** (PACELC): even partition-free, sync replication costs latency (wait for both copies) — so:

```text
PACELC: if Partition → Availability vs Consistency
        Else          → Latency vs Consistency
You pay in latency what you don't pay in staleness — always.
```

## Layer 1 — Simple Explanation

Two branch offices keeping the same ledger, connected by one phone line:

- The line breaks (partition). A customer walks into each office wanting the *latest* balance
- **CP**: both offices refuse until the line heals — "we can't be sure" (correct, closed)
- **AP**: both answer from their local ledger — customers served, ledgers *may disagree* (open, eventually reconciled... maybe)

There is no management technique that serves both customers *and* guarantees identical answers on a broken line. The theorem is just this office story, proven.

## Layer 2 — Engineer's View

**The consistency spectrum (the real menu — "strong vs weak" is too coarse):**

| Model | Guarantee | You know it from |
|---|---|---|
| **Linearizable** | read = latest write, global order | single-node systems, CP stores (ZooKeeper, etcd) |
| **Sequential** | some global order, all see it | slightly weaker, cheaper |
| **Causal** | causally-related ops ordered | CRDTs, collaborative edits |
| **Read-your-writes** | *you* see your writes | session consistency — most user-facing needs! |
| **Eventual** | replicas converge... eventually | DNS, AP stores (Dynamo lineage), caches |

The design insight: **most user-facing features need read-your-writes, not linearizability** — "I see my own order confirmation" ≠ "every support agent sees it this instant". Matching the cheapest sufficient model per feature is the architecture work; demanding linearizability everywhere is the distributed tax paid in latency (PACELC).

**Where your stack sits (the map):**

| System | Class | Because |
|---|---|---|
| etcd/ZooKeeper (K8s state!) | CP | the cluster must never disagree about reality |
| Cassandra/Dynamo-style | AP | always writable, tunable read staleness |
| Postgres primary | linearizable (single node) | replication = async (failover loses tail — RPO!) |
| Cache layers | eventual, TTL-bounded | your own CAP decisions, per key |
| DNS | eventual | the TTL page, explained formally |

**Consistency as a product decision (the SLO-echo):** "may a support agent see an order 2 seconds stale?" is a *business* question with an architecture price. Write the answer per feature — same discipline as NFRs, one level down.

**Conflict resolution mechanics (when you choose AP):** last-write-wins (loses data by clock — humble about NTP!), CRDTs (merge without loss — counters, sets), app-level reconciliation (sagas), or version vectors (detect, don't resolve). Choosing AP *is* choosing one of these — "eventual" doesn't mean "magic".

## Real-World Example (DevOps flavored)

ShopEasy's per-feature consistency sheet:

```text
Inventory decrement:     CP — overselling is a real cost (sync quorum write)
Order confirmation view: read-your-writes — session pin to the written replica
Reviews/counts:          eventual — CRDT counters, minute-stale fine
Cart:                    AP + LWW on merge — a lost "add to cart" is recoverable
Support agent view:      5s-stale allowed — replicate async, page says "as of X"
Each row = a product-signed tradeoff with a mechanism — not a default.
```

The K8s footnote worth remembering: the control plane is **CP on purpose** (etcd) — a cluster that disagrees about what's running is worse than one you can't update during a partition. Your workloads may be AP; the orchestrator's brain cannot.

## Common Mistakes

- "We're CP/AP" system-wide — consistency is per-feature/per-data, not per-company
- Choosing AP with no conflict mechanism — "eventual" without the reconciliation plan
- Demanding linearizability for features needing read-your-writes (latency paid for nothing)
- Forgetting async replication's failover data loss — AP in a CP costume (the RPO page again)
- LWW with trusting clocks — NTP drift as data-loss mechanism

## Mental Model

> Two branch offices, one broken phone line, one customer at each counter: serve both (AP — ledgers may disagree) or close one (CP — correct but unavailable) — *no third branch manager exists*. PACELC adds the fine print: even with the phone working, agreeing costs a phone call (latency). The architecture job: deciding, *per ledger*, which offices may disagree.

## Remember This

1. CAP: during partitions, refuse (CP) or risk staleness (AP); P isn't optional
2. PACELC: partition-free, the same trade bills you in latency
3. The menu is a spectrum: linearizable → causal → read-your-writes → eventual
4. Match per feature: most UX needs read-your-writes, not linearizability
5. AP requires a conflict mechanism: LWW/CRDTs/sagas — eventual ≠ magic
6. K8s's etcd is CP by design — the orchestrator's brain doesn't do eventual

## One Sentence

CAP states that under network partitions a replicated store must choose between refusing requests and risking stale answers — and PACELC adds that even healthy networks bill the same choice as latency, making consistency a per-feature product decision with a named conflict-resolution mechanism.

## Knowledge Check

1. Why is P not a choice? What does choosing "C or A" actually mean operationally?
2. Classify: shopping cart, inventory decrement, view counts, cluster state store — model + mechanism.
3. Why is async-replicated Postgres "linearizable" until it isn't? Which NFR number describes the gap?
4. What does LWW lose, and to which fallacy?

## Further Reading

- Brewer's CAP follow-up ("CAP twelve years later"), Kleppmann's DDIA ch. 5+9
- Jepsen analyses — consistency failures of real systems, empirical
- Next: [Event-Driven, Queues & Kafka](event-driven-kafka.md)

---

**← Previous:** [Distributed Systems Fundamentals](distributed-fundamentals.md)
**Next:** [Event-Driven, Queues & Kafka](event-driven-kafka.md) →
**Related:** [Storage & Databases](../cloud/storage-databases.md) · [CAP ↔ HA/DR](../cloud/ha-dr.md)
