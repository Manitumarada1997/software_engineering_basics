# Platform as Product & Platform Metrics

## What Is It?

The operating model that determines whether everything in this phase becomes leverage or shelfware: run the platform **as a product** — customers (developers), a roadmap, adoption metrics, support, versioned releases — with explicit **platform metrics** proving (or disproving) value.

```text
Internal project:  built to requirements, "delivered", adoption mandated, measured by features
Product:           built to user problems, adopted voluntarily, measured by outcomes
The platform's P&L: value = (developer-hours saved + risk reduced) − platform cost
```

## Why Does It Exist?

Because internal platforms fail in a specific, repeatable way: built as projects (requirements gathered once, shipped, "done"), then maintained as afterthoughts — while stream teams route around them (the mud tracks reappear). The product model is the *only* sustained answer because:

- Voluntary adoption forces the platform to actually be the fastest path (Golden path page)
- Continuous user research keeps the roadmap tied to real friction (DX telemetry)
- Metrics keep the platform honest — and funded (the business case must be shown, not asserted)

## Layer 1 — Simple Explanation

Two **company cafeterias**: the mandated one (requirements-driven menu from a 2-year-old survey — everyone orders delivery) and the one run as a business (menu evolves with what people actually choose, prices tracked, queue times measured). The second stays full *because it must earn every diner daily*. The platform must earn every team daily — mandate is the confession of a bad product.

## Layer 2 — Engineer's View)

**The metric stack (lead with outcomes, support with drivers):**

| Tier | Metrics | What they prove |
|---|---|---|
| **Outcome** | time-to-first-deploy, lead time (commit→prod), change failure rate, MTTR — *aggregated across consumer teams* (DORA) | the platform is moving org-wide delivery |
| **Adoption** | % services on golden paths, self-service ratio (tickets vs automated), portal usage | teams choose it |
| **Experience** | DX surveys (cognitive load, "can deploy without asking anyone"), NPS | it feels good |
| **Health** | platform uptime/SLA (it's production — CI page's law), template version currency (drift!), cost per developer | it's operable |

The north-star most orgs choose: **time-to-tenth-service** (the marginal cost of the next service) — the platform's whole promise in one number.

**The product mechanics (imported wholesale from product management):**

```text
Roadmap:      driven by friction telemetry + user interviews (DX page) — not committee
Releases:     versioned golden-path templates with changelogs + deprecation windows
Support:      docs, chat channel, office hours — the SLA a product owes its users
Discovery:    quarterly DX surveys; win/loss interviews with teams that left the path
Pricing:      showback per team (FinOps page's lever) — usage-based conversations
Lifecycle:    features retire (the zombie-feature audit — Drift page, product edition)
```

**The team's guard (the platform-as-product's own failure modes):**

| Failure | The guard |
|---|---|
| Feature factory (shipping, unadopted) | adoption as the release criterion |
| Captive roadmap (one big customer's needs = everyone's) | enterprise-vs-default tiering |
| Infrastructure envy (polishing the fun parts) | friction telemetry picks the work |
| No maintenance budget (build-team disbanded post-v1) | ops staffing is the budget line |

**The economic narrative for leadership (the one-slide version):**

```text
Before: 40 teams × ~15% time on undifferentiated platform work = 6 FTE-equivalents,
        badly, with snowflake risk
Platform: 6 engineers + tooling = the same capability, professionally, once
Outcome: time-to-first-deploy 3wk→1d; lead time -70%; audit findings -90%
The platform pays for itself in flow, risk, and hiring (engineers prefer paved orgs)
```

## Real-World Example (DevOps flavored)

ShopEasy's platform QBR (the product operating, visibly):

```text
Metrics: lead time p50 2.1d (-64% YoY) · adoption 38/40 · self-service ratio 87% ·
         DX "would recommend" 71 · platform SLA 99.7% · template currency: 34/38 on v3
Roadmap (from telemetry): v5 = ephemeral environments (biggest remaining wait),
         DB self-service tiering, mesh mTLS default for new services
Retirement: the legacy Jenkins paths — sunset announced, 2 teams migrated, 1 left
Risk shown honestly: template currency 89% — the drift metric watched in public
```

## Common Mistakes

- Measuring the platform by features shipped (the feature-factory trap)
- Adoption by mandate — hiding the product problem, not fixing it
- No support model — the product users can't get help with
- Skipping outcome metrics — leadership sees cost, not flow
- Treating the platform itself as non-production (no SLA, no on-call — the CI page's original sin, org-scale)

## Mental Model

> The platform is the **cafeteria that must earn its diners daily**: menu by measured appetite (friction telemetry), prices posted (showback), queue times on the wall (metrics), and the cooks staffed for the whole operation (ops budget) — because the alternative delivery apps (shadow IT) are always one bad week away from winning.

## Remember This

1. Product model: voluntary adoption, friction-driven roadmap, versioned releases, support SLA
2. Metric stack: outcomes (DORA aggregated) ← adoption ← experience ← health
3. North star: marginal cost of the next service (time-to-tenth)
4. Template currency = the platform's drift metric — watch it in public
5. The platform is production: uptime, on-call, maintenance budget
6. Leadership narrative: flow, risk, hiring — numbers per slide

## One Sentence

Platform-as-product runs the internal platform like a business earning its users daily — adoption-driven roadmap, versioned releases, support, and an outcome metric stack proving that developer flow improves by more than the platform costs.

## Knowledge Check

1. Your platform ships 2 features/quarter; adoption is flat. Which metric tier is lying to you, and which one tells the truth?
2. Build the one-slide business case for your platform with real numbers.
3. Why is "time-to-tenth-service" a better north star than "time-to-first"?
4. What does template currency < 100% predict, and by which earlier page's mechanism?

## Further Reading

- *Team Topologies* ch. on platform team operating model; [platformengineering.org](https://platformengineering.org/)
- DORA reports — the outcome metrics' source
- Next: [CNCF & Cloud-Native](../advanced/cncf-cloud-native.md)

---

**← Previous:** [IDP & Backstage](idp-backstage.md)
**Next:** [CNCF & Cloud-Native](../advanced/cncf-cloud-native.md) →
**Related:** [Pipeline as Product](../cicd/pipeline-as-product.md) · [FinOps](../advanced/finops.md)
