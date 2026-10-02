# FinOps & Cost Optimization

## What Is It?

**FinOps** — cloud financial operations: the practice of making cloud cost **visible, attributable, and optimize-able**, as an engineering discipline rather than a finance-afterthought. The operating loop:

```text
Inform   → allocate (tags/labels → per-team, per-service, per-env cost)
Optimize → right-size, commitment-manage, architect cheaper
Operate  → policies, budgets, anomaly alerts, showback/chargeback — continuously
```

## Why Does It Exist?

Because the cloud's pricing model inverted the economics of infrastructure: from *predictable CapEx* (budget once, buy once) to **continuous consumption pricing** — where cost is the *integral of engineering decisions*:

```text
Every architecture choice is now a price: idle dev environment = $4k/month;
the unindexed query = 30% of your DB bill; the log-everything setting = TB/month.
Nobody in finance can optimize this — only engineers can, and only if they can SEE it.
```

FinOps exists to close that loop: engineers seeing cost as a first-class NFR (the NFR page listed it — this is that page's expansion).

## Layer 1 — Simple Explanation

The shared apartment where **every roommate's usage is metered**: who left the AC on (idle environments), who runs the kiln daily (that batch job) — with the bill split by meter, discussed monthly, and the kiln-schedule optimized *by the kiln-user once they see the number*. Without meters: one bill, mutual distrust, and the AC stays on forever.

## Layer 2 — Engineer's View)

**Attribution first (nothing works without it):**

```text
Tagging standard: owner, service, env, cost-center — enforced (PaC page:
org policies deny untagged resources) — tags are the meters
Allocation hierarchy: showback (teams see their spend) → chargeback (budgets
attached) — showback changes behavior alone; chargeback changes org structure too
Unit economics: cost per order / per request / per DAU — the only number that
scales conversationally ("cost up 20% — but orders up 30%: efficiency improved")
```

**The optimization levers, ranked by effort-to-yield (your existing knowledge, monetized):**

| Lever | Mechanism | Typical yield |
|---|---|---|
| **Right-sizing** | requests/limits vs actual (cgroups data!) | 20-40% |
| **Commitments** | reserved/savings plans for steady baseline (Compute page) | 30-60% of baseline |
| **Spot** | evictable workloads (CI, batch) | 60-90% of those |
| **Scheduling** | dev envs off nights/weekends | 100% of off-hours |
| **Storage lifecycle** | hot→cool→archive automation (Storage page) | varies, large |
| **Architecture** | caching, serverless for spiky, killing zombies (Drift page's audit) | step-change |

The zombie audit deserves its headline: **most estates' first FinOps win is deleting things nobody uses** — the Drift page's quarterly ritual, now with a dollar figure.

**The engineering culture shift (the real product):**

```text
Cost alerts like SLO alerts: budget-burn anomalies page the owner (unit-cost
   spike = often an incident symptom — the cost signal IS an observability signal)
Design reviews include cost: the HLD page's estimation included $/month — that's FinOps
The anti-pattern to retire: "cost is finance's problem" — finance cannot right-size
   your requests; only the team seeing the meter can
```

**Showback's honest politics:** unit-cost dashboards per team, monthly, in engineering all-hands — names attached. The two failure modes: weaponized showback (budget theater replacing engineering judgment — teams optimize the metric, e.g., deleting observability to save line-items) and vanity-green dashboards nobody actions. Guard: optimize *unit* cost and *waste*, never absolute spend against a growing business.

## Real-World Example (DevOps flavored)

ShopEasy's first FinOps quarter (the typical arc):

```text
Month 1 (Inform):    tagging enforced; showback live; headline: $412k/month, 41% unattributed→0
Month 2 (Optimize):  zombie audit: 19 idle envs, 3 orphaned data warehouses (-$61k);
                    right-size 200 over-requested deployments (-$38k);
                    dev envs scheduled (-$22k)
Month 3 (Operate):   reserved plans on the steady baseline (-35% of it); anomaly alert
                    catches a retry-storm's egress spike 2 days before the invoice;
                    unit economics live: $0.021/order → the number every review now cites
Result: 29% lower bill, zero performance regressions, and cost reviews that take
         30 minutes because the meters exist
```

## Common Mistakes

- Cost reviewed quarterly-by-finance (lagging, blame-shaped) instead of continuously-by-engineers
- No tagging enforcement — allocation by archaeology
- Optimizing absolute spend against growth; optimizing the metric by deleting observability/security
- Ignoring commitments (paying on-demand for the known baseline — the ignorance tax)
- Missing the anomaly signal: unit-cost spikes as incident symptoms

## Mental Model

> FinOps puts **meters on every apartment** (tags → allocation), posts the **split bill monthly** (showback), and turns cost into an **engineering signal like latency** — paged when it burns abnormally, designed-in at the HLD, and optimized by the people whose choices create it. The first month's win is always the same: discovering what was left running.

## Remember This

1. Loop: inform (tag→allocate) → optimize → operate (budgets, anomalies) — continuously
2. Attribution is the precondition; tags enforced by policy, not requests
3. Levers ranked: zombies, right-sizing, scheduling, commitments, spot, lifecycle
4. Unit economics ($/order) is the scalable conversation; absolute spend isn't
5. Cost anomalies are observability signals (retry storms bill first)
6. Design reviews include $/month — cost as NFR, finally

## One Sentence

FinOps makes cloud cost an engineering signal — allocated by enforced tagging, optimized by ranked levers, and alerted on like latency — because in consumption-priced infrastructure, only engineers seeing their meters can spend wisely.

## Knowledge Check

1. Build the tagging standard and enforcement mechanism for your estate.
2. Rank the levers for your current bill with estimated yields — what's the zombie audit likely to find?
3. Why is $/order the right review number and absolute monthly spend the wrong one?
4. A unit-cost spike pages you at 2 AM — three plausible engineering causes?

## Further Reading

- [FinOps Foundation framework](https://www.finops.org/framework/) — the phases, formally
- Next: [AI for DevOps & LLMOps](ai-devops-llmops.md)

---

**← Previous:** [CNCF & Cloud-Native](cncf-cloud-native.md)
**Next:** [AI for DevOps & LLMOps](ai-devops-llmops.md) →
**Related:** [Compute Options](../cloud/compute.md) · [NFRs](../architecture/nfrs.md)
