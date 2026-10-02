# Distributed Systems Fundamentals

## What Is It?

The physics you accept the moment two components talk over a network:

```text
1. The network is reliable            — FALSE (drops, partitions, DNS, middleboxes)
2. Latency is zero                    — FALSE (ms that become your p99)
3. Bandwidth is infinite              — FALSE (payloads and fan-out cost)
4. The network is secure              — FALSE (TLS/mTLS — you know this chapter)
5. Topology doesn't change            — FALSE (pods die, routes shift)
6. There is one administrator         — FALSE (blast-radius thinking)
7. Transport cost is zero             — FALSE (serialization taxes every call)
8. The network is homogeneous         — FALSE (mixed protocols, versions)
```

The **Fallacies of Distributed Computing** — every one is a production incident you've debugged wearing a costume.

## Why Does It Exist?

You were promised this page since the Monolith one: microservices buy autonomy and *pay in distributed physics*. The payment comes in specific, learnable failure modes — the ones every SRE interview and every 3 AM has in common:

| Failure mode | What it looks like | The defense |
|---|---|---|
| **Partial failure** | 1 of 40 replicas hangs — averages hide it | timeouts, health gates, per-instance isolation |
| **Cascading failure** | slow dependency → exhausted pools → total collapse | circuit breakers, bulkheads, backpressure |
| **Timeout misconfiguration** | client waits longer than its own callers | timeout budget hierarchy |
| **Retry storm** | failure × retries = self-DDoS | jittered exponential backoff, retry budgets |
| **Split brain** | two leaders after a partition | quorum protocols (CAP page) |
| **Clock skew** | TTLs expiring wrong, ordering lies | logical clocks, NTP humility |

## Layer 1 — Simple Explanation

A distributed system is a **team working across distant offices with only email**: messages get lost (network), arrive late (latency), someone's always offline (partial failure), and nobody shares a wall clock (skew). The defenses — read receipts (ack), office assistants (queues), "if no reply in 2 days escalate" (timeouts), "stop emailing the closed office" (circuit breakers) — are what this page formalizes.

## Layer 2 — Engineer's View

**Timeout budgets — the discipline that orders all of it:**

```text
user request budget: 1000ms
  gateway→checkout: 950 (reserve overhead)
    checkout→payments: 700 (leave room for its own work)
      payments→db: 500
Rule: every hop's timeout < its caller's remaining budget.
Inverted budgets = hung requests upstream of "working" services — the
"everything is slow but nothing is failing" mystery.
```

**The resilience patterns (each a Release It! classic, and each already met in this course):**

| Pattern | Mechanism | Where you met it |
|---|---|---|
| **Circuit breaker** | stop calling the failing dep; fail fast; probe to close | Alerting/canary pages |
| **Bulkhead** | isolate pools per dependency — one sink doesn't flood the ship | capacity page's ceilings |
| **Backpressure/load shedding** | refuse work beyond capacity (503 > hang) | capacity ladder |
| **Retry + jittered backoff** | retry idempotent ops with exponential random delay | HTTP page |
| **Fallback** | degrade to cache/default rather than error | degradation ladder |
| **Idempotency** | safe retries: idempotency keys on writes | Jobs/CD pages |

The combinatorial insight: patterns *interact*. Retries without circuit breakers = storms; shedding without prioritization = shedding the wrong users; fallbacks to stale data without flags = subtle wrongness. Design them as a system, chaos-test them (Chaos page — level 4, dependency blackhole).

**Coordination cost — the Amdahl of distribution:** every synchronous coordination point (distributed locks, quorum writes, chain-of-calls) serializes the system. The scaling architecture move is *removing coordination*: ownership partitions (no shared writes), eventual consistency where tolerable, async everywhere latency permits — the CAP and Event-driven pages complete this thought.

**Observability as a distributed-systems requirement** (why SRE preceded architecture here): you cannot debug these failure modes with logs on one box — traces (causality), cohort metrics (partial failure), and SLOs (user truth) are the *instruments this physics demands*. The phase ordering wasn't accidental.

## Real-World Example (DevOps flavored)

The classic cascade, anatomized (it happens to everyone once):

```text
14:00 payments DB slow (10% of queries 5s)
14:03 payments pool exhausts waiting (no bulkhead: all threads on slow queries)
14:05 checkout threads exhaust calling payments (no breaker; timeouts 30s > budget)
14:07 gateway 503s; LB health checks time out; autoscaler adds pods (more retry load)
14:10 full outage — from 10% slowness in one dependency
Postmortem fixes: timeout hierarchy (500ms payments), breaker (50% error → open 30s),
        bulkheads per dependency, jittered retries, shed-to-queue for non-urgent
Chaos game day re-runs the scenario quarterly: cascade now = 2-min blip
```

## Common Mistakes

- Default (or infinite) timeouts — hung threads as architecture
- Retries without jitter/idempotency — engineering the storm
- One shared connection pool — the un-bulkheaded ship
- Synchronous call chains 6 deep — latency addition + cascade fuel
- Treating partial failure as someone else's problem — averages hiding the 1%
- No chaos verification — resilience as untested claims (the Chaos page's law)

## Mental Model

> A distributed system is **offices communicating by email across a unreliable postal network**: letters lost, late, offices dark, no shared clocks. The professional's kit — timeouts (escalation deadlines), breakers (stop mailing the closed office), bulkheads (per-office mailrooms), backpressure (the mailroom that refuses), idempotency (reference numbers on every letter) — turns the postal apocalypse into an occasional delay.

## Remember This

1. The 8 fallacies: every one is a debugging session you've had
2. Core failure modes: partial failure, cascades, timeout inversion, retry storms, split brain
3. Timeout budgets: every hop < caller's remaining budget
4. Pattern system: breakers, bulkheads, backpressure, jittered retries, fallbacks, idempotency — designed together, chaos-tested together
5. Coordination is the scaling tax — remove it (ownership, eventual consistency, async)
6. Traces/SLOs are the instruments this physics requires

## One Sentence

Distributed systems impose network physics — partial failure, latency, and lost messages — answered by a designed system of timeouts, circuit breakers, bulkheads, backpressure, and idempotency whose interaction, not whose parts, determines whether a dependency hiccup stays a blip.

## Knowledge Check

1. Map all 8 fallacies to incidents you've personally debugged (the exercise worth an hour).
2. Why does a 30s timeout on a dependency whose callers budget 2s guarantee hanging?
3. Design the pattern set for "payments is occasionally slow": which three patterns, configured how?
4. Why does removing coordination scale better than tuning it?

## Further Reading

- *Release It!* — Nygard (circuit breakers' origin; the cascade chapter is this page)
- *Designing Data-Intensive Applications* — Kleppmann, ch. 8 (the physics deep-dive)
- Next: [CAP & Consistency](cap-consistency.md)

---

**← Previous:** [Monolith → Microservices](monolith-microservices.md)
**Next:** [CAP & Consistency](cap-consistency.md) →
**Related:** [Chaos Engineering](../sre/chaos-engineering.md) · [Event-Driven & Kafka](event-driven-kafka.md)
