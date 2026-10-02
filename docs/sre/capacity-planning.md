# Capacity Planning & Scalability

## What Is It?

- **Capacity planning**: ensuring the system has enough resources (compute, memory, network, licenses, *people*) for expected demand — ahead of time, with headroom policy
- **Scalability**: the system's *architectural* ability to absorb load (more users) — vertical (bigger boxes) vs horizontal (more boxes)

```text
Headroom formula (the SRE rule):
  peak utilization ≤ N-1 rule — survive your busiest day with one node/replica/AZ dead
utilization data (USE metrics) + growth trend + event calendar + lead time = the plan
```

## Why Does It Exist?

Because resources have **lead times** and failures have **bad timing**:

```text
Lead times: nodes provision 2-5 min (autoscaler) — but quotas/capacity reservations take days;
            enterprise licenses take weeks; hiring takes months
Bad timing:  traffic spikes and node failures correlate (everyone shops during the outage)
```

Autoscaling (Cloud/K8s pages) made *routine* elasticity automatic — but didn't eliminate planning: quotas, downstream ceilings (the database that doesn't autoscale!), cost budgets, and *sustained* growth curves all still need forethought. Planning's object shifted from instances to *constraints*.

## Layer 1 — Simple Explanation

Capacity planning is the **restaurant's Friday-night staffing**: you know Fridays double, New Year's Eve triples, and staff can't be hired the hour guests arrive — so you schedule ahead with one extra "in case someone calls in sick" (the N+1 rule). Scalability is the kitchen's *design*: one giant stove (vertical) vs. modular stations that replicate (horizontal) — only one of those survives a stove fire.

## Layer 2 — Engineer's View

**The planning loop:**

```text
1. Measure: utilization at current load (USE metrics — the cgroups numbers)
2. Model: units of capacity per unit of demand ("1 replica per 200 rps at p95<800ms")
3. Forecast: growth trend × seasonality × event calendar (Black Friday known in advance)
4. Constrain check: quotas, DB ceilings, cache slot limits, network egress, COST
5. Decide: pre-provision vs trust autoscale; reservations for the baseline (FinOps!)
```

Step 2 is the engineering heart: **load factors per service** — measured (load tests — Testing page), not guessed. Step 4 is where plans die: everything autoscales into the one component that doesn't.

**Scalability — the architectural vocabulary (Architecture phase previews):**

| Axis | Meaning | Ceiling |
|---|---|---|
| Vertical | bigger machine | hardware limits, cost curves, restart to resize |
| Horizontal | more machines | state: sessions, caches, DB — the real work |
| Elastic | automatic both ways | cold-start lag (the Cloud page's thermostat) |

The stateless/horizontal sermon you know (HTTP page, Pods page) — here's its planning corollary: **every stateful dependency is a capacity ceiling that autoscaling elsewhere will find**. Scale the fleet 10× and you meet your connection-pool limit, your Redis single-thread, your DB disk — in order of pain.

**Overload handling — planning's last line of defense:**

```text
You WILL be overloaded someday (forecast wrong, spike, partial outage).
Graceful degradation ladder:
  load shedding (503 + Retry-After — fail fast, survive)
  > feature degradation (disable recommendations, keep checkout)
  > queueing/batching (async the non-urgent)
  > collapse (static "we're busy" beats a hung site)
Systems without this ladder don't degrade — they fall.
```

(Couple shedding with *prioritized* capacity: keep checkout alive while search dies first.)

## Real-World Example (DevOps flavored)

ShopEasy's Black Friday plan (the annual exam):

```text
Forecast: 6× average peak (last year × growth), concentrated 19:00-22:00
Pre-scale: HPA minReplicas raised 3→20 at 08:00 (beat cold-start lag — Autoscaling page)
Quotas: node/quota increase requested in October (lead time!)
Ceilings audited: DB max_connections → pgbouncer; Redis → cluster mode; egress doubled
N+1: peak sized to survive one AZ loss (Regions page)
Load test at 6.5× in October (125% headroom — Testing page)
Degradation ladder armed: search → recommendations → full, by flag
Cost: reserved baseline + spot for the surge delta (Compute page economics)
```

Result: peak p95 = 740ms vs 800ms SLO; zero pages. The night was boring — *which is the deliverable*.

## Common Mistakes

- Planning compute but not the ceilings: connection pools, licenses, quotas, egress
- Trusting reactive autoscaling for known events (lag + image pulls + warmup)
- No N+1: peak capacity that an AZ failure converts into an outage
- Guessing load factors instead of load-testing them
- No degradation ladder — overload becomes total collapse instead of partial discomfort
- Ignoring cost as a constraint (FinOps) — capacity plans that finance vetoes silently

## Mental Model

> Capacity planning is **Friday-night staffing with a calendar and one spare**: measured load factors, known events pre-staffed, quotas requested before they're ceilings. Scalability is the **kitchen's replicable-station design** — and the degradation ladder is the **fire plan**: what you burn first (search) so the crown jewels (checkout) keep serving.

## Remember This

1. Headroom = N+1 at peak; the plan covers constraints with lead times, not just instances
2. Load factors measured, not guessed; forecast = trend × season × events
3. Every stateful dependency is a ceiling your horizontal scale will eventually meet
4. Known events get pre-scaling; autoscaling handles the unknown
5. Graceful degradation ladder turns overload from collapse into discomfort
6. Cost is a first-class constraint; reservations for baseline, elastic for surge

## One Sentence

Capacity planning ensures resources with lead times — headroom, quotas, and the ceilings that don't autoscale — are ready before demand arrives, while scalability is the architecture that lets load be absorbed by replication rather than heroics.

## Knowledge Check

1. Scale your web tier 10×. List, in pain order, the next three ceilings you meet.
2. Why does N+1 beat "we have autoscaling" for peak days?
3. Design the degradation ladder for a travel-booking site.
4. What has lead time in your estate besides nodes? (At least four answers.)

## Further Reading

- Google SRE book ch. 7 (capacity planning); *Release It!* — Nygard (stability patterns)
- Next: [Chaos Engineering](chaos-engineering.md)

---

**← Previous:** [Postmortems](postmortems.md)
**Next:** [Chaos Engineering](chaos-engineering.md) →
**Related:** [Load Balancing & Autoscaling](../cloud/lb-autoscaling.md) · [Performance Testing](../development-practices/perf-security-testing.md)
