# Developer Experience & Golden Paths

## What Is It?

- **DX (developer experience)**: how it *feels* and *flows* to get work done in your org — from repo-clone to production, including the friction, the waiting, and the mystery. The platform's *product quality* (the Why-Platform page's customer experience).
- **Golden path** (aka paved road): the **supported, opinionated, default way** to build, deploy, and run a service — the fastest route *and* the compliant one, by design:

```text
shop create service --template java-rest
→ repo + CI/CD + GitOps registration + OTel + dashboards + SLO scaffolding +
  security posture + runbook stub + portal entry — in minutes, compliant by default
```

## Why Does It Exist?

Because the alternative is the default path being *cobblestone*: copy-paste from the last service, three weeks of archaeology, security review ping-pong, and 40 snowflake variants of every concern (Why-Platform page's before/after). The golden path's insight:

> **The path of least resistance and the path of best practice must be the same path** — or best practice loses every time it's under deadline.

Golden paths are paved by *taking the boring, standard route and making it one command* — guardrails (PaC) embedded invisibly: the template's pipeline scans, signs, and enforces because that's what the template *is*.

## Layer 1 — Simple Explanation

The **airport vs the wilderness**: both get you to a city. The airport path — check-in kiosk, security line, boarding scan — is *paved*: every step designed, standardized, fast because millions use it. Some travelers need private jets (custom, expensive, their problem); most should never hack through the woods with a compass (the hand-rolled pipeline).

The golden path is the airport: opinionated (one security process, not your own), fast (volume-optimized), safe by design — and you can still charter a jet when you truly must.

## Layer 2 — Engineer's View)

**DX metrics — measure the experience like a product (DORA + SPACE + flow):**

| Metric | Captures |
|---|---|
| Time-to-first-deploy (new joiner) | onboarding friction — the canary metric |
| Lead time (commit→prod) | end-to-end flow (DORA) |
| DORA four (lead, freq, failure rate, MTTR) | delivery health (CI page's table) |
| SPACE (satisfaction, perf, activity, comm/collab, efficiency) | human sustainability |
| Cognitive load surveys | the 2023 DORA finding — measure it directly |
| Platform NPS / adoption % | product health (pipeline-as-product rules) |

The discipline: instrument *friction*, not activity — commits/day measures noise; time-from-idea-to-prod measures experience.

**Golden path anatomy (what the one command assembles — every phase, one product):**

```text
Template engine:      language/framework scaffolds, versioned
Delivery:             golden pipeline (CI page's staged gates), GitOps registration
Runtime:              namespace, mesh-on, resource defaults, HPA policy
Observability:        OTel auto-instrumented, golden dashboards, SLO scaffold (Metrics page)
Security:             PaC posture, SBOM+signing, secret defaults (SBOM page)
Docs & discover:      portal entry, team API (Team Topologies), runbook stub
Ownership:            CODEOWNERS, on-call scaffold — "you build it" made runnable
```

**The governance of paths (how they stay paved):**

- Versioned templates with deprecation policy (the pipeline-product rules again — no silent breaking)
- Platform upgrades propagate: the template *is* the upgrade mechanism (fix once, 40 teams inherit)
- Escape hatches explicit: go off-path knowingly, own the consequences (the jet charter)
- Path telemetry: which steps of onboarding take longest → the roadmap *is* the friction list

**The anti-pattern: the false golden path** — documentation describing what you *should* do (a wiki path) instead of executable tooling doing it. If it's a doc, it's cobblestone with a map; paths are code.

## Real-World Example (DevOps flavored)

ShopEasy's template metrics dashboard (the platform's product analytics):

```text
Golden path v3: create→prod in 4h median (v1: 2 weeks)
Adoption: 38/40 services; the 2 exceptions: legacy ERP adapter (justified) + one
          team "waiting for the right moment" (the roadmap's next conversation)
Friction telemetry: biggest time sink = DNS/ingress request (3-day wait) →
          self-service ingress provisioning shipped in v4 → 20 minutes
DX survey trend: "I can deploy without asking anyone" — 22% → 81% in two quarters
```

The platform roadmap emerged from friction data, not opinion — that's the product operating.

## Common Mistakes

- Golden paths as documentation, not executable tooling
- Forcing adoption of v1 paths (unpaved "paved roads" — adoption is earned by speed)
- No path telemetry — the platform ships features blind
- Treating escape hatches as betrayal (they're pressure valves — the PaC page's exception law)
- Measuring activity (commits, tickets) instead of experience (lead time, friction)
- Paths without upgrades: 40 services pinned to template v1 forever (the drift disease, platform edition)

## Mental Model

> The golden path is the **airport**: paved, opinionated, safe by design, fast because standard — with jet charters for the genuine exceptions. DX is the **traveler's experience measured as a product** — friction telemetry as the roadmap, adoption as the revenue. When least resistance equals best practice, compliance stops being a conversation.

## Remember This

1. Golden path = least resistance == best practice, as executable tooling
2. It assembles the whole course: delivery, runtime, observability, security, ownership
3. Templates are the upgrade propagation mechanism — fix once, inherited ×40
4. DX measured as product: time-to-first-deploy, lead time, friction telemetry, NPS
5. Escape hatches are pressure valves, not betrayal
6. Wiki paths aren't paths — if it's not code, it's cobblestone

## One Sentence

Developer experience is the platform's product quality measured through flow and friction metrics, and golden paths are its mechanism — executable, opinionated scaffolding where the fastest route to production is automatically the compliant, observable, secure one.

## Knowledge Check

1. Map your onboarding's every waiting-point; which one golden-path tooling deletes first?
2. Why is time-to-first-deploy the canary metric for the whole platform?
3. The template IS the upgrade mechanism — explain how a security fix reaches 40 services.
4. Distinguish friction telemetry from activity metrics with two examples each.

## Further Reading

- [goldenpath/paved-road writings — Spotify Backstage docs](https://backstage.io/docs/overview/what-is-backstage/)
- SPACE framework paper (Microsoft/GitHub); DORA reports
- Next: [IDP & Backstage](idp-backstage.md)

---

**← Previous:** [Team Topologies](team-topologies.md)
**Next:** [IDP & Backstage](idp-backstage.md) →
**Related:** [Pipeline as Product](../cicd/pipeline-as-product.md) · [Policy as Code](../security/policy-as-code.md)
