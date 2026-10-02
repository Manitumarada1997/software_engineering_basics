# Why Platform Engineering?

## What Is It?

**Platform engineering** (the discipline, ~2018+ name): building and operating an **Internal Developer Platform (IDP)** — the paved-road infrastructure and self-service layer that product teams consume to build, deploy, and run software — *as a product, with developers as customers*.

```text
It is not a new job title for what you already do. The difference:
DevOps (as practiced): every team does everything themselves, repeatedly
Platform engineering: a team builds the paved road once, as a product,
                      so every team doesn't rebuild the mud track forty times
```

## Why Does It Exist?

Because of the crisis this course has documented in slow motion — **the cognitive load explosion**:

```text
ShopEasy, year 1:  app + server.  Load on each team: tiny.
ShopEasy, year 6:  app + K8s + Helm + GitOps + service mesh + OTel + SLOs +
                   security policies + supply chain + cost + DR...
The 2023 DORA report's finding: cognitive load is THE dominant developer pain —
engineers spend more time understanding the platform than writing product code.
```

Every page since CI/CD has been adding capabilities *someone* must operate. The industry's answer to 40 teams × (K8s + security + observability + delivery) each done slightly differently: **a platform team productizes the doing.** Team Topologies named the org shape (next page); this page holds the why and the what.

You also predicted it: Pipeline-as-Product was platform engineering's seed. Zero Trust at scale demanded it. Golden-path instrumentation (OTel) required it. The course converged here because the industry did.

## Layer 1 — Simple Explanation

Every town could dig its own wells, lay its own pipes, and run its own power plant — some do, and they're called *poor*. The **utility model**: a competent few build water/power/roads once, to a standard, with meters and an SLA — everyone else builds *on top*.

Platform engineering is the **utility model for engineering capability**: the platform team runs the roads (delivery), water (infra), power (observability/security) — and product teams pour their energy into what customers actually pay for.

## Layer 2 — Engineer's View

**DevOps vs platform engineering — the honest comparison (they're allies, not rivals):**

| | DevOps (culture/method) | Platform Engineering (org/product answer) |
|---|---|---|
| Answers | how teams collaborate end-to-end | how to scale that to 30+ teams without drowning them |
| Unit | team-level ownership ("you build it, you run it") | a product serving many teams |
| Risk | everyone-owns-everything = everyone-expert-in-nothing | a platform nobody wants = shelfware |

Platform engineering is what DevOps *becomes* at scale — "you build it, you run it" survives only if running it is cheap, and making it cheap is the platform.

**The IDP's capabilities (the product's feature list — every one a course phase):**

```text
Delivery:      templates, pipelines, environments-on-demand, GitOps
Runtime:       K8s (managed), service scaffolding, mesh with defaults
Golden path:   new service → repo + CI/CD + dashboards + SLOs + security posture,
               in one command — the paved road
Self-service:  developer-provisioned (bounded) — DBs, topics, domains
Guardrails:    policy-as-code baked in (PaC page) — safety without gates
Observability: OTel + golden dashboards + SLO scaffolding by default
Docs/portal:   the catalog and the how-tos (Backstage — its own page)
```

**The failure mode to know (why "platform team" alone fails):** a platform built as an *internal project* — requirements by committee, adoption by mandate, no roadmap, no adoption metrics — becomes the shelfware enterprise tooling graveyard. The product discipline is the discipline: users researched, NPS tracked, roadmap driven by adoption data — the Pipeline-as-Product page's rules at org scale.

**The economics, stated once:** 40 teams × 1 platform-ish engineer each = 40 engineers doing it badly 20% of the time; 8 platform engineers doing it superbly 100% of the time, multiplied by every team's velocity. The platform is leverage, not overhead — *if* it's a product.

## Real-World Example (DevOps flavored)

ShopEasy, before/after the platform:

```text
Before: new service = 3 weeks (pipeline copy-paste, dashboards by hand, security
        review ping-pong, "how does X do it?" archaeology); 40 snowflake setups
After:  `shop new-service --template java-rest` → 1 day to production-grade:
        golden pipeline, GitOps repo, OTel+dashboards+SLOs, PaC-scanned,
        runbook scaffold. Platform team: 6 engineers, adoption 38/40 teams,
        tracked: time-to-first-deploy, ticket-to-self-service ratio, NPS +62
The metric that closed the debate: 3 weeks → 1 day, × 40 teams, forever.
```

## Common Mistakes

- Platform as internal project, not product — committee-designed shelfware
- Adoption by mandate — the platform nobody chose, nobody uses well
- Building everything (a platform is a thin layer over cloud primitives, not a rebuild)
- Ignoring the exit: no golden path for *leaving* the platform (lock-in fear kills adoption)
- Measuring the platform by features shipped instead of adoption + time-to-value

## Mental Model

> Platform engineering is the **utility model for engineering**: roads, water, and power built once by specialists with meters and SLAs, so forty teams build *their* things instead of their infrastructure. The product discipline is the whole trick — a utility people can choose to leave is the only utility built worth using.

## Remember This

1. The driver: cognitive load explosion — the documented dominant developer pain
2. Platform engineering = DevOps scaled: productize what 40 teams would each redo
3. IDP capabilities = the course's phases, productized (delivery, runtime, golden paths, guardrails, observability)
4. Product discipline is non-negotiable: users, adoption metrics, roadmap — or shelfware
5. Economics: 8 doing it superbly beats 40 doing it occasionally
6. Golden paths with exits — adoption is earned

## One Sentence

Platform engineering exists because modern delivery's cognitive load exceeds what product teams can carry — and it answers by productizing the road, the runtime, and the guardrails as an internal platform that teams consume instead of reconstruct.

## Knowledge Check

1. Quantify the cognitive load in your org: list every capability a team must master to ship one service safely.
2. Where does "you build it, you run it" break without a platform — and how does the IDP repair it?
3. Build the business case: platform team of N vs the distributed cost it replaces.
4. What distinguishes a platform product from an internal tooling project?

## Further Reading

- Team Topologies (next page — the org design that carries this)
- [platformengineering.org](https://platformengineering.org/) — the field's canon site
- DORA 2023 — cognitive load findings

---

**← Previous:** [Low-Level Design](../architecture/lld.md)
**Next:** [Team Topologies](team-topologies.md) →
**Related:** [Pipeline as Product](../cicd/pipeline-as-product.md)
