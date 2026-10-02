# Chaos Engineering

## What Is It?

The discipline of **running controlled experiments on production** — injecting realistic failures (killed pods, network latency, AZ loss) to verify the system's resilience *claims* — because distributed systems fail in ways dashboards never show until users do.

**The method (the scientific method, applied to prod):**

```text
1. Hypothesis:  "checkout stays within SLO when a payments replica dies"
2. Baseline:    measure steady-state (SLIs)
3. Inject:      kill one replica (the experiment — smallest blast radius first)
4. Observe:     did steady-state hold? for how long? what recovered it?
5. Learn:       document; fix what failed; widen scope next time
```

Tools: Chaos Mesh / Litmus (K8s-native fault injection), AWS FIS, Gremlin.

## Why Does It Exist?

Because of the gap between **claimed and actual resilience**:

```text
Claimed:  "multi-AZ, auto-failover, retries everywhere"
Actual:   the failover was never executed → DNS TTL still 300s → "self-healing"
          took 22 minutes of user pain to self-discover
```

Every mechanism you've met — Deployment replicas, HPA, retries, circuit breakers, multi-AZ — is a *hypothesis* until exercised. Chaos engineering is the test suite for resilience: **failures will happen; you choose whether to schedule them** (Netflix's Chaos Monkey insight, 2011).

It's shift-right's logical completion: production is the only honest test environment for production failures (Testing page's law, final form).

## Layer 1 — Simple Explanation

A **fire drill**: setting off the alarm on purpose, on a scheduled Tuesday, with the exits known — versus waiting for the real fire to learn that door 3 was chained shut since 2019. The drill's value: the *discovery* happens when it's cheap and staffed, not at 3 AM in smoke.

## Layer 2 — Engineer's View

**The safety rails (what separates experiments from incidents):**

| Rail | Practice |
|---|---|
| Blast radius | smallest first: 1 pod → 1% traffic → 1 node → 1 AZ |
| Auto-halt | SLI breach or guard-metric trips → injection stops immediately (experiment aborted, not outage) |
| Business hours only (initially) | staffed, caffeinated, watching |
| Rollback rehearsed | you know how to stop before you start |
| Game days | scheduled, scenario-driven drills (the DR page's law, continuous) |

**The experiment ladder (maturity progression):**

```text
L1 kill a pod                  → does anything notice? (readiness, retries)
L2 inject latency/errors       → do timeouts trip before cascades? (circuit breakers real?)
L3 node drain/AZ loss          → multi-AZ claims (Regions page) executed
L4 dependency blackhole        → does checkout survive payments degraded?
L5 steady-state-verification   → continuous, automated background chaos ("game days" → culture)
```

Level 4 is where architecture debt surfaces: dependency failures reveal the couplings no dashboard admits (circuit breakers that don't exist, retries that amplify — the thundering herd).

**What chaos uniquely finds (the findings catalog):**

- Timeout hierarchies inverted (client times out *after* its dependency — hung, not failed)
- Retries without jitter amplifying load (the storm)
- Cache assumptions (cold cache = capacity cliff)
- Singletons you forgot (the "HA" service with one Kafka consumer group leader)
- Alert gaps: the failure dashboards never showed — until the experiment

**Relation to the rest of the course:** chaos = postmortem action items run *proactively*; game days = DR drills (HA/DR page) generalized; the guard rails = SLOs (this phase); the injection targets = every mechanism the CD/Deployment pages built. Chaos engineering is the *verification layer* over the entire reliability story.

## Real-World Example (DevOps flavored)

Game day at ShopEasy — "AZ failure during Friday peak":

```text
Scenario: cordoned all nodes in AZ-b (traffic ~35%), 17:45, team assembled
T+0:    injection; steady-state dashboard live (checkout SLI, latency, errors)
T+20s:  HPA scales in AZ-a/c; queue depth rises (expected)
T+95s:  payments latency p95 1.4s — ABOVE hypothesis (expected < 1s)
T+2m:   found: retry amplification without jitter — retries tripled the AZ's load
T+5m:   abort (guard metric), AZ restored
Outcome: jittered-backoff fix shipped next week; THEN the experiment passed
         One config line — found on a Tuesday, not a holiday
```

## Common Mistakes

- "Chaos" without hypothesis or steady-state definition — just breaking things (an outage with paperwork)
- Skipping the abort guard — the experiment that becomes the incident it studied
- Testing only infra failure (pods/AZs), never application-level (degraded deps, bad data)
- One-off stunts instead of the maturity ladder toward continuous background chaos
- Running experiments where you can't observe outcomes (no SLIs = no experiment, just vandalism)
- No follow-through — findings not tracked like postmortem items (the museum, again)

## Mental Model

> Chaos engineering is the **scheduled fire drill** — hypothesis, baseline, injection, auto-halt, learning — discovering the chained exits (inverted timeouts, missing breakers, retry storms) on staffed Tuesdays instead of smoke-filled nights. Resilience claims become test results; the drill is the only honest test of a door.

## Remember This

1. Controlled experiments: hypothesis → steady-state → inject (small first) → observe → learn
2. Every resilience mechanism is a claim until chaos exercises it
3. Rails: blast-radius laddering, auto-halt guards, business hours, rehearsed rollback
4. Uniquely finds: timeout inversions, retry storms, hidden singletons, cold-cache cliffs
5. Game days = continuous DR drills; the ladder ends in automated steady-state chaos
6. No SLIs = no experiment — observation is a precondition, not an accessory

## One Sentence

Chaos engineering tests resilience claims by injecting realistic failures under controlled, auto-halting experiments — converting "it should survive" into "it did, measurably" before production schedules the same test without supervision.

## Knowledge Check

1. Frame "kill a pod in prod" as an experiment: hypothesis, steady-state, guard, blast radius.
2. Why do retry storms appear only under chaos, never in dashboards?
3. Design the AZ-loss game day: scenario, guards, expected findings, abort criteria.
4. Why is "no observability, no chaos experiment" a hard rule?

## Further Reading

- *Chaos Engineering* — Basiri et al. (O'Reilly, the short canon); PrinciplesOfChaos.org
- Chaos Mesh / LitmusChaos docs
- Phase complete → [Architecture Thinking](../architecture/thinking-tradeoffs.md)

---

**← Previous:** [Capacity Planning](capacity-planning.md)
**Next:** [Architecture Thinking & Tradeoffs](../architecture/thinking-tradeoffs.md) →
**Related:** [Chaos ↔ Game Days](../cloud/ha-dr.md) · [SLOs & Error Budgets](slo-error-budgets.md)
