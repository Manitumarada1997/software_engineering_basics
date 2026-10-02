# Pipeline as Product

## What Is It?

Treating the CI/CD system not as a pile of YAML someone maintains, but as a **product with users (developers), a roadmap, requirements, SLAs, and success metrics** — owned by a team responsible for the *developer experience of shipping code*.

This page is the bridge between "I write pipelines" and the Platform Engineering phase: it's the same idea, before it got its modern name.

## Why Does It Exist?

The alternative — every team hand-rolls its own pipeline from scratch — fails in predictable ways (ShopEasy at 50 devs, no pipeline product):

- 6 teams, 6 pipeline styles; nobody can read each other's YAML
- Every new service = 2 weeks of copy-paste pipeline archaeology
- The same CVE-scanning gap replicated 6 times, discovered in 6 separate incidents
- Security updates to shared steps require nagging 6 teams ("please update your template")

And when the pipeline breaks? The "pipeline person" is on vacation; 50 developers are blocked. That's not a tooling problem — it's an **ownership and product** problem.

## Layer 1 — Simple Explanation

Think of the pipeline as the **company's electrical grid**:

- If every house wires itself into the grid ad hoc, fires follow
- A utility owns generation, distribution, and safety standards — invisibly
- Its "users" just flip switches and get light; they don't think about voltage
- And the utility has an SLA, maintenance windows, and a roadmap

Pipeline-as-product = your delivery system is infrastructure **with users who deserve an SLA**, not scripts the lucky ones figured out.

## Layer 2 — Engineer's View

**The product framing applied concretely:**

| Product question | Pipeline equivalent |
|---|---|
| Who are the users? | All developers; distinct personas (app dev, data, mobile) |
| What's the UX? | `template.yml` include, one-click service onboarding, clear failure messages |
| What's the SLA? | Pipeline availability, queue time p95, mean feedback duration |
| What's the roadmap? | Golden paths (later phase), new capabilities, security features |
| How do we measure success? | DORA metrics across the org; time-to-first-deploy for a new service |
| Support model? | Docs, office hours, chat channel — not "ping Kumar" |

**The architectural pattern that makes it scalable — shared templates + conforming repos:**

```text
 teams write:                      platform owns:
 ┌──────────────────┐              ┌───────────────────────────┐
 │ azure-pipelines: │──includes──► │ templates/build.yml       │
 │  - template:     │              │ templates/scan.yml        │
 │    build.yml@tpl │              │ templates/deploy-canary   │
 └──────────────────┘              │ + versioning + changelog  │
                                   └───────────────────────────┘
```

A team's pipeline is 10 lines of *intent*; the platform upgrades scanning for everyone by shipping `tpl@2.3`. This is "golden path" in embryo.

**The reliability discipline (the part most teams skip):**

- Pipelines are production systems — they need monitoring (queue depth, failure rates, flaky-test burn-down), on-call, and incident review
- **Downtime math:** pipeline down 2h × 200 developers = 400 engineering hours lost — treat outages accordingly
- Versioned templates with deprecation policy; no silent breaking changes to 40 repos
- Self-service reporting: every team sees their DORA metrics — transparency beats mandates

**The economics that justify the investment:** 10 minutes of pipeline time saved × 500 runs/day × 250 days ≈ 2 person-years annually. Platform leverage: one engineer optimizing the template multiplies across every team.

## Real-World Example (DevOps flavored)

The maturity ladder you've probably climbed rungs of:

```text
1. Hero pipelines     — one person's scripts; bus factor 1
2. Copy-paste era     — every repo for itself; drift everywhere
3. Templates era      — shared YAML; central upgrades propagate
4. Product era        — versioned templates + SLAs + metrics + support
5. Platform era       — golden paths, self-service portal (Phase 11)
```

Signals you're at rung 4: new service reaches production in a day (not weeks); template releases have changelogs; there's a dashboard of org-wide DORA metrics; a pipeline incident gets a postmortem like a production outage.

## Common Mistakes

- Building it as a side project — no ownership, no SLA, no roadmap = shared *scripts*, not a product
- Over-abstracting: templates so "flexible" they have 40 parameters and no happy path
- Breaking template consumers silently — version your templates
- No telemetry on the pipelines themselves (you monitor everything except the thing that ships everything)
- Measuring adoption by mandate ("all teams must...") instead of by being the easiest correct path
- Ignoring the support burden — product without support becomes an obstacle

## Mental Model

> The pipeline system is the **elevator in a skyscraper**. Nobody praises it when it works; everyone is trapped when it doesn't; and its quality is measured in uptime, speed, and how little riders must think about it. An engineering org that makes tenants climb stairs because "the elevator person left" has confused infrastructure with a hobby.

## Remember This

1. Pipelines are production systems with users, SLAs, and metrics — not scripts
2. Shared, versioned templates turn pipelines from copy-paste into a distributed product
3. Pipeline downtime is org-wide downtime — monitor and on-call it
4. Pipeline speed is org-wide leverage: minutes saved multiply by every developer every day
5. Adoption through ease (golden path), not mandate
6. This is Platform Engineering's seed form — the org design follows (Phase 11)

## One Sentence

Pipeline-as-product means operating your CI/CD system like an internal utility — with users, SLAs, telemetry, and a roadmap — because every property of the delivery system becomes a property of every team that ships through it.

## Knowledge Check

1. Compute the cost of 2 hours of pipeline downtime for a 150-developer org.
2. Why are shared templates superior to copy-pasted pipelines, organizationally and security-wise?
3. Which metrics would you put on a pipeline product's dashboard?
4. Why is "adoption by mandate" a failure signal for a pipeline product?

## Further Reading

- *Team Topologies* — Skelton & Pais (platform team interaction modes — Phase 11 preview)
- DORA —dora.dev (measuring delivery at org scale)
- *Accelerate* — Forsgren et al.

---

**← Previous:** [Deployment Strategies](deployment-strategies.md)
**Next:** [Processes & Threads](../linux/processes-threads.md) →
**Related:** [Platform Engineering](../platform-engineering/index.md) · [Roadmap](../roadmap.md)
