# The Capstone: ShopEasy at 500 Engineers

**Level 3 · Enterprise project — the whole course in one design**

## The scenario

It's 2029. ShopEasy is 500 engineers across 40 teams, 3 regions, ~200 services, a 24/7
platform on-call rotation, and a board asking why cloud spend grew faster than revenue.
You are the principal engineer asked to produce the full engineering design.

## Part 1 — The architecture (HLD)

Produce a High-Level Design document containing:

1. **NFR sheet** per journey (checkout, search, order-history, support): latency, SLO,
   RPO/RTO, consistency model (CAP sheet), cost ceiling
2. **Component map**: bounded contexts → the service spectrum choice per context
   (monolith module / modular / extracted service — each justified by a numbered NFR)
3. **Data architecture**: ownership per store, the replication/sharding ladder position
   for orders (with the trigger measurements), event backbone topics + schemas
4. **Estimation**: DAU → rps → storage → the arithmetic that sizes the choices
5. **Failure analysis**: degradation ladder per journey; kill-each-component table
6. **10× path**: which ceiling breaks first, which ladder rung is next

## Part 2 — The delivery system

Design the complete path from `git commit` to production:

- Branching (trunk-based) + review + the staged test pyramid gates
- CI: build, scan (SAST/SCA/secrets), sign, SBOM — the supply chain, end to end
- Artifact promotion: immutable, digested, signed; admission-enforced in the cluster
- GitOps: repo structure (app-of-apps), environments-as-directories, promotion-as-PR
- Deployment: canary with SLO-gated analysis per service tier
- Rollback: git-revert seconds; feature-flag instantaneous — when each applies

## Part 3 — The platform

- The **IDP**: golden paths (what the `create-service` command assembles), self-service
  catalog (what teams may provision vs. request), guardrails embedded (the PaC set)
- **Team Topologies**: the org chart — stream-aligned/platform/enabling teams, team APIs
- **Platform metrics**: the full stack (outcomes ← adoption ← experience ← health)
- **Portal**: catalog entity model, the 3 AM user journey through it

## Part 4 — Operations

- **Observability**: OTel everywhere, the golden dashboards, SLO scaffolding defaults
- **SRE program**: error budgets per journey, the exhaustion policy, on-call design
- **Incident management**: roles, runbooks, postmortem cadence and tracking
- **Capacity**: N+1 per region, the ceilings audit, pre-scaling for known events
- **Chaos**: the experiment ladder, game-day calendar

## Part 5 — Security & compliance

- Zero-Trust staged plan (identity/device/segmentation) — where each pillar stands
- The supply chain per the SLSA ladder; signing + admission enforcement
- Secrets architecture (federated identity > dynamic > centralized)
- Compliance-as-code: PCI scope as Kyverno/SCP policies; continuous evidence
- Threat model of the delivery system itself

## Part 6 — The business of it

- **FinOps**: tagging/allocation, the lever ranking with estimated yields, unit economics
- The platform's business case (the one-slide leadership narrative)
- **ADR appendix**: the twelve biggest decisions, options-and-consequences form

## Grading yourself (the rubric)

| Standard | Check |
|---|---|
| Traceability | every design line maps to a course page; every page shows up somewhere |
| Numbers | NFRs, estimates, costs — no vibes survive review |
| Tradeoffs | rejected options recorded for every major choice |
| Failure-first | every component's death designed, not hoped away |
| Product thinking | platform adoption, DX metrics, and FinOps as first-class |

If you can produce this design — and defend every line — you have achieved the course's
goal: the thinking of a Senior/Staff/Principal-level Platform, Cloud, and DevOps engineer.

---

**← Previous:** [AI for DevOps & LLMOps](../advanced/ai-devops-llmops.md)
**Next:** [Roadmap](../roadmap.md) · **Related:** every page of this course
