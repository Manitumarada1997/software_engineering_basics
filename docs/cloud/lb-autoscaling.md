# Load Balancing & Autoscaling in the Cloud

## What Is It?

The cloud's productized version of the LB + scaling pair (earlier pages), unified by one new property: **elasticity by API** — capacity becomes a parameter, not a procurement.

- Cloud LBs: regional L4/L7 managed services (ALB/NLB, Application/Load Balancer, Cloud Load Balancing) with health checks, TLS, targets
- Autoscaling: **VM Scale Sets / Auto Scaling Groups** (VM tier) — and request-driven scaling in higher tiers (Compute page)

## Why Does It Exist?

Elasticity was cloud's founding promise (NIST's "rapid elasticity"). LB + autoscaling is how the promise cashes out: traffic arrives → LB distributes → scaling policy adds capacity → LB health-gates it into rotation. The loop that replaces capacity planning with control theory.

## Layer 1 — Simple Explanation

A **restaurant that hires and fires staff by queue length**: the host (LB) seats guests; when the line exceeds a threshold, a Cook is called in (scale-out) — takes 5 minutes to wash up and start (boot + health check); when idle, extra staff go home (scale-in). Guests never see the staffing chaos — just service.

## Layer 2 — Engineer's View

**The autoscaling control loop — a thermostat, with the same pathologies:**

```text
metric (CPU/queue/requests) → policy (target 60%) → action (±N instances)
                    ↑____________ feedback ____________↓
```

Thermostat failures map exactly:

| Pathology | Autoscaling equivalent |
|---|---|
| Overshoot/oscillation | scale up → metric drops → scale down → repeat (flapping fleets) |
| Lag (heating takes minutes) | instances take 2–5 min to boot + warm — *reactive* scaling always late for spikes |
| Wrong sensor placement | CPU 90% but bottleneck is DB connections — scaling makes it *worse* |

Antidotes: cooldown periods, step/scheduled policies for known peaks (pre-scale), scaling on **queue depth / requests-per-target** (closer to actual load than CPU), and *predictive scaling* for diurnal patterns.

**Health checks as the admission mechanism:** an instance joins the LB only when passing checks — so boot time + app warmup (JIT, caches) determines real scale-out latency. This couples autoscaling with your deployment story: bake images (Packer) to shorten boot; warm-on-start hooks to shorten readiness.

**Scaling limits are architecture facts:**

- Max instances per group (quota — request increases *before* Black Friday)
- **Downstream ceilings**: 10× app instances behind a 1× database = autoscaling your outage
- Scale-in protection: instances serving long requests need drain time (connection termination)

**The multi-AZ contract:** ASGs spread across AZs; on AZ failure the group *automatically rebalances* into survivors — this is the cheapest HA in the cloud (Regions page) and the reason "LB + ASG across AZs" is the default pattern everywhere.

**L4 vs L7 product choice (LB page's table, productized):** NLB for raw TCP/UDP throughput and static IPs; ALB for HTTP routing (paths, headers — canary-friendly). Managed K8s services hide this behind LoadBalancer/Ingress — but the cloud LB is still underneath.

## Real-World Example (DevOps flavored)

ShopEasy flash-sale prep — the complete pattern:

```text
1. Baseline: ASG 6–24 instances, 3 AZs, target-tracking 60% CPU
2. Pre-scale: scheduled action 09:00 sale day → min 20 (beat the lag)
3. Queue-based scaling for order workers: target = 2 msgs/instance
4. DB: read replicas + connection pool ceiling documented (the real limit)
5. Quotas checked, scale-in protection during checkout peak
6. Postmortem metric: scale-out lead time vs. first latency spike
```

The honest metric teams discover: autoscaling is *reactive* — it limits the depth of sustained overload; it cannot make boot+warm instant. Pre-scaling and request-shedding (LB 503s with Retry-After) cover the spike gap.

## Common Mistakes

- Scaling on CPU when the bottleneck is elsewhere (I/O, connections, locks)
- No cooldowns → flapping fleets paying boot-time churn
- Forgetting instance quotas and per-AZ capacity reservations before events
- Scale-in killing long-running work (no drain/protection)
- Assuming autoscaling fixes a too-slow app — it multiplies whatever you have, good or bad

## Mental Model

> Autoscaling is a **thermostat for capacity**, the LB its ventilation system distributing load across whatever rooms exist. The thermostat reacts in minutes and measures only its one sensor — so the engineer's job is choosing the right sensor, pre-heating before known cold snaps, and making sure the furnace (boot+warm) and the chimney (downstream capacity) can actually keep up.

## Remember This

1. Metric → policy → action feedback loop; thermostat pathologies = flapping, lag, wrong sensor
2. Scale-out latency = boot + warmup + health check — bake images, warm on start
3. Scale on load-representative signals (queue depth, req/target) over CPU
4. Downstream ceilings (DB!) bound every autoscaling win — document them
5. Scheduled/predictive pre-scaling covers the reactive gap for known peaks
6. LB + multi-AZ ASG is the cloud's default HA pattern — use it by default

## One Sentence

Cloud load balancing and autoscaling turn capacity into a control loop — distributing traffic across health-gated instances while policies add and remove them against metrics — bounded always by boot latency, sensor quality, and the downstream systems that don't scale with you.

## Knowledge Check

1. Why is CPU-based scaling a trap for an I/O-bound service? What's the better signal?
2. Your fleet flaps every 20 minutes. Which three knobs fix it?
3. Autoscaling to 40 instances made the outage worse. What didn't scale?
4. Design the scaling story for a 10 AM flash sale from a 6-instance baseline.

## Further Reading

- AWS Auto Scaling / Azure Scale Sets docs (target-tracking policies)
- Google SRE book — autocomplete of "cascading failures" chapter (load shedding)

---

**← Previous:** [Storage & Databases](storage-databases.md)
**Next:** [HA & Disaster Recovery](ha-dr.md) →
**Related:** [Proxies & Load Balancers](../networking/proxies-load-balancers.md) · [Autoscaling (K8s)](../kubernetes/autoscaling.md)
