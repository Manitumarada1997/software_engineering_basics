# SLI, SLO, SLA & Error Budgets

## What Is It?

The SRE measurement contract (Google SRE book's core innovation):

| Term | Definition | Example |
|---|---|---|
| **SLI** | the *indicator*: a measured ratio of good/total events | "99.2% of checkouts < 800ms" |
| **SLO** | the *objective*: the SLI target, internally | "≥ 99.9% over 30 days" |
| **SLA** | the *contract*: external promise with consequences | "below 99.5% → credits" |
| **Error budget** | 100% − SLO: the allowed imperfection | 0.1% = 43 min/month of permitted badness |

```text
SLO 99.9% → budget 0.1% → 43.2 min unavailability OR 0.1% failed checkouts per month
Everything users experience as "bad" spends this budget — bugs, deploys, restarts, incidents
```

## Why Does It Exist?

To resolve the fundamental Ops-vs-Dev conflict:

```text
Dev:  "ship features!"              Ops: "stability!"
Both right, both unmeasured → war: change freezes vs shadow changes

SLO resolution: 99.9% = 43 min of budget/month.
Budget intact? → ship fast, experiments, big refactors (that's what the budget is FOR)
Budget burned? → feature freeze, hardening, reliability work — by AGREED POLICY, not opinion
```

Reliability becomes a *product decision with a price tag* — quantified in minutes of change velocity. The second gift: **honest measurement of user pain** (SLIs on user journeys, not infra trivia — the Alerting page's symptom doctrine, formalized).

## Layer 1 — Simple Explanation

The SLI is the **speedometer**, the SLO the **speed limit you choose**, the error budget the **points on your license** — and the deal with yourself: points remaining, drive freely; points spent, take the train until they reset.

The license metaphor's key insight: a *perfect driver* (100% uptime) never goes anywhere interesting — perfection is the wrong goal; **the right amount of reliability, explicitly budgeted** is.

## Layer 2 — Engineer's View

**SLI design — the craft (garbage in, everything out):**

```text
Good event / total events, measured at the USER'S edge:
  availability:   successful responses / all responses
  latency:        requests < 800ms / all requests        (p95 thresholds as ratios)
  correctness:    fresh-enough reads / all reads
  journey-based:  end-to-end (checkout worked), not per-service

Anti-patterns: infra SLIs (CPU, uptime-of-pod) — the pod can be up and the user sad
Window: rolling 30d (or multi-window burn alerts) — short windows punish noise,
        long ones punish slowly
```

**Burn-rate alerting — the SLO-native paging strategy:**

```promql
# fast burn: 14.4x rate → 2% of 30d budget in 1h → PAGE
error_ratio > 14.4 * (1 - 0.999)
# slow burn: 6x rate for 6h → ticket
```

Two-tier burn rates replace dozens of raw thresholds — paging exactly when *the budget* (the agreed business resource) is threatened (Alerting page's upgrade, delivered).

**The governance loop that makes SLOs real:**

```text
1. SLIs per user journey (product + SRE agree)      2. SLOs priced (product decides)
3. dashboards: budget remaining, burn rate          4. policy: exhausted → reliability freeze
5. review monthly: incidents + releases vs budget   6. adjust SLOs (up costs money, down costs users)
```

Step 4 is where most orgs fake it: without the exhaustion policy, the error budget is a dashboard, not a contract.

**SLA vs SLO discipline:** SLO is your internal bar (aim above the SLA); SLA consequences (credits) hit when you've already *long* blown the SLO. Never alert on SLAs — that's legal territory, not operational.

**Error budget math for canaries/releases (the CD page's unspoken dependency):** release rollback criteria = "this deploy burns X% of budget" — measurable, pre-agreed, no heroics.

## Real-World Example (DevOps flavored)

ShopEasy's SLO sheet — one page per journey:

```text
Journey: checkout
SLI: checkouts succeeded AND < 1s / total checkout attempts (gateway-measured)
SLO: 99.9% / 30d  →  budget: 43 min or ~0.1% failures
Policy: budget < 25% → releases need SRE co-sign; < 10% → freeze + hardening sprint
Alerts: fast-burn page / slow-burn ticket
Monthly review: Nov: 99.93% — 3 releases, 1 incident, 21 of 43 min spent — healthy
```

The org change: the December "freeze vs ship" war became a 10-minute budget review — data instead of arm wrestling.

## Common Mistakes

- SLOs on infra metrics (pod uptime ≠ user happiness)
- No exhaustion policy — budgets as decoration
- 100%/99.99% vanity targets — unpriced perfection (budget → zero, dev stops)
- One global SLO for everything — journeys differ (checkout ≠ order-history)
- Alerting on SLA boundaries
- Never re-negotiating: an SLO is a hypothesis about users, reviewed with data

## Mental Model

> SLI = speedometer; SLO = the limit you *choose*; error budget = license points that **buy speed**. The genius is the spent-points policy: reliability and velocity stop being warring camps and become one metered resource — 43 minutes a month, spent deliberately.

## Remember This

1. SLI (measured good/total) → SLO (internal target) → SLA (contract with consequences)
2. Error budget = 100%−SLO: the resource that *purchases* change velocity
3. SLIs on user journeys at the edge — never infra trivia
4. Burn-rate alerts (fast=page, slow=ticket) replace threshold sprawl
5. The exhaustion policy is what makes it a contract, not a dashboard
6. SLOs are priced product decisions, reviewed and re-negotiated with data

## One Sentence

SLOs set an explicit reliability target measured by user-facing SLIs, and the resulting error budget becomes a shared resource that deliberately purchases change velocity — replacing the stability-versus-speed war with metered policy.

## Knowledge Check

1. Compute: 99.95% monthly SLO — budget in minutes? What does a 22-minute incident leave?
2. Design SLIs for "search results" — what's the good event? Where measured?
3. Your SLO is 99.9% and you've been at 100% for 4 months. Diagnose the mispricing.
4. Write the fast-burn alert condition and its rationale in budget terms.

## Further Reading

- Google SRE book ch. 3-4 (the source); [sre.google/workbook](https://sre.google/workbook/implementing-slos/)
- Next: [Incident Management](incident-management.md)

---

**← Previous:** [Alerting](alerting.md)
**Next:** [Incident Management](incident-management.md) →
**Related:** [Deployment Strategies](../cicd/deployment-strategies.md) · [Postmortems](postmortems.md)
