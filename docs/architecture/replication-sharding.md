# Replication & Sharding

## What Is It?

The two orthogonal answers to "one database can't hold/serve it all":

```text
Replication — same data, multiple nodes        (availability + read scale)
Sharding    — different data, multiple nodes   (write scale + size beyond one machine)
        ┌── shards: orders-0 (users A-M), orders-1 (users N-Z)
        │      each: primary + 2 replicas
        └── replication factor 3 per shard
```

## Why Does It Exist?

Two different walls, two different ladders:

- **Replication** answers failure and read-load: a primary's death (failover), geography (read replicas near users), analytics isolation (offsloading heavy reads). You already know its tradeoffs: **sync = latency (PACELC), async = failover data loss (RPO!)**
- **Sharding** answers the ceiling no replication touches: *one* primary's write throughput and *one* machine's storage. When the single writer is the bottleneck, splitting the data is the only scale-out.

## Layer 1 — Simple Explanation

- **Replication**: the **same book in every branch library** — any branch can let you read it; damage one copy, the others survive. Getting the *newest edition* everywhere at once is exactly the CAP problem
- **Sharding**: the **encyclopedia split across volumes** — nobody has the whole set; the index card (router) tells you which volume your topic lives in. Reorganizing the volumes (resharding) is the nightmare that keeps librarians honest about *where they cut*

## Layer 2 — Engineer's View

**Replication topologies:**

| Topology | How | Trade |
|---|---|---|
| Single-primary | writes→primary; replicas serve reads | simple; failover; replica lag = staleness |
| Multi-primary | any node accepts writes | availability; **conflict resolution becomes yours** |
| Quorum (Raft/Paxos) | majority vote per write | CP; latency of majorities — etcd's choice |

The failover choreography you operate: health detection → promotion → client/routing update → (async tail: the lost writes — your RPO number, from the HA/DR page, now mechanically explained).

**Sharding decisions — the three that matter:**

```text
1. Shard KEY:    what splits the data — user_id, tenant_id, order_id?
   → query affinity: queries must carry the key, or they FAN OUT to all shards
2. Strategy:     range (user A-M) — cheap, hot-spot-prone
                hash — even, key-only queries
                directory/consistent-hash — flexible, lookup service
3. Rebalancing:  when shard 0 outgrows: split + move — online vs downtime;
   the operational complexity that makes "don't shard yet" a valid answer
```

**The honest ladder (shard last — the strongest advice on this page):**

```text
1. Index + query tuning       (most "we need sharding" is missing indexes)
2. Caching                    (read scale — previous page)
3. Read replicas              (read scale, replication only)
4. Vertical scaling           (bigger primary — buys years)
5. Functional partitioning    (separate DBs by domain — the monolith page's boundaries!)
6. Shard                      (the last resort with real operational weight)
```

**Cross-shard operations — the costs the design buys:** transactions spanning shards (2PC — avoid; sagas — the Events page), joins (no longer possible — moves to app or denormalization), unique constraints across shards (the index card can't check what it doesn't have). Sharded systems trade query flexibility for write scale — deliberately, per domain.

**Where the ecosystem sits:** managed Postgres/MySQL (replication native, sharding yours), Vitra (MySQL sharding), Citus (Postgres extension), MongoDB/ Cassandra (sharding native, hash/range built-in), Spanner/CockroachDB (sharding + quorum replication — the full machine). "Buy the machinery or be the machinery."

## Real-World Example (DevOps flavored)

ShopEasy's orders-store evolution — the ladder, walked honestly:

```text
Year 2: missing index = "the DB is slow" → 40min query → 4ms (step 1, no architecture needed)
Year 3: read replicas for analytics + geo reads (step 3); replica lag SLO: p95 < 2s
Year 4: orders split from the monolith DB (step 5 — functional partition by bounded context)
Year 6: writes at 3× single-primary ceiling → shard by user_id, hash, 8 shards × 3 replicas
        + directory service; cross-shard orders-reporting moved to an event-fed warehouse
        (the Events page's projections — joins replaced by subscription)
Each step's trigger: a measured ceiling, not a milestone. Each step's cost: written down.
```

## Common Mistakes

- Sharding at step 2 (missing-index "scalability problems")
- Shard key without query affinity — every query fans out (a distributed JOIN per request)
- Hot shards: celebrity tenant / monotonic keys (timestamps as shard keys)
- No rebalancing plan — the year-3 split attempted live, badly
- Believing replicas solve write bottlenecks (they replicate the bottleneck)
- Quorum everywhere for data that needed eventual (latency paid for nothing — CAP page)

## Mental Model

> Replication puts the **same book in every branch** (survives fires, serves readers, newest-edition lag is the fine print); sharding splits the **encyclopedia into volumes** (no single shelf holds it all, the index card routes you). The ladder rule: index, cache, replicate, grow, partition — and shard only when the *write* wall is measured, because rebalancing volumes is the librarian's nightmare.

## Remember This

1. Replication = availability/read scale; sharding = write scale/size — orthogonal, combined at scale
2. Sync/async replication = latency/RPO trade; failover loses the async tail
3. Shard key decides query affinity — wrong key = fan-out on every query
4. The ladder: index → cache → replicas → vertical → functional → shard (last)
5. Sharding trades joins, cross-shard transactions, and rebalancing peace for write scale
6. Buy the machinery (Vitess/Citus/native-SaaS) or become it

## One Sentence

Replication copies data for availability and read scale at the price of consistency latency or failover loss, while sharding partitions data for write scale at the price of cross-shard query flexibility — two orthogonal ladders to climb only as measured ceilings demand.

## Knowledge Check

1. Why don't read replicas help a write-bottlenecked primary — what does?
2. Pick the shard key: orders queried by user, by date-range reports, and by order_id lookup — reconcile all three.
3. What exactly is lost in async failover, and which NFR number bounds it?
4. Walk ShopEasy's ladder; name the trigger measurement at each step.

## Further Reading

- *DDIA* ch. 5-6 (replication/partitioning — the definitive chapters)
- Vitess/Citus architecture docs (real sharding machinery)
- Next: [API Design](api-design.md)

---

**← Previous:** [Caching](caching.md)
**Next:** [API Design](api-design.md) →
**Related:** [CAP & Consistency](cap-consistency.md) · [HA & DR](../cloud/ha-dr.md)
