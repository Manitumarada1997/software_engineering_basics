# Internal Developer Platform & Backstage

## What Is It?

- **IDP (Internal Developer Platform)**: the assembled product — golden paths, self-service provisioning, runtime, and guardrails (previous pages) — the *capability* layer
- ****Internal Developer Portal****: the IDP's *front door* — the catalog, the docs, the software templates, the ownership map
- **Backstage** (Spotify, open-sourced 2020, CNCF): the dominant portal framework — plugins for catalog, scaffolder, docs, and everything custom

```text
IDP = the machine room (pipelines, clusters, policies — the course's phases)
Portal = the lobby (find it, understand it, order it, own it)
Backstage = the most popular lobby-kit
```

## Why Does It Exist?

Because scale creates the **discovery problem** the platform's machine room doesn't solve:

```text
"Who owns this service?"          (it's 3 AM, the dashboard says payments-7d9f)
"What does this API consume?"     (before you change its contract)
"Is this service production-critical?"  (before you break it)
"How do I get a Kafka topic?"     (the runbook archaeology)
```

Without a catalog, those answers live in tribal memory — and tribal memory doesn't survive scale or 3 AM. The portal makes the org's software *legible*: every component registered, owned, described, related — the **service metadata as data**.

## Layer 1 — Simple Explanation

The machine room (IDP) runs the building; the portal is the **lobby directory**: the floor plan (who's where), the services desk (order a topic/environment), the standards signs ("this corridor is PCI"), and each tenant's plaque (ownership, hours, contacts). Nobody fixes elevators in the lobby — but nobody navigates the building without it.

## Layer 2 — Engineer's View)

**Backstage's core concepts (the vocabulary):**

| Concept | What it does |
|---|---|
| **Catalog** | entities (services, APIs, resources, teams) from `catalog-info.yaml` *in each repo* — metadata lives with code (GitOps of org knowledge) |
| **Scaffolder** | golden-path templates → the one-command service creation (DX page) |
| **TechDocs** | docs-as-code per service, versioned with the service |
| **Plugins** | the extension ecosystem: cost, security posture, SLOs, on-call, search |
| **Software model** | relations: service → owns → API → consumed-by → service (the dependency graph) |

**The catalog discipline (what makes it real, not a directory of lies):**

```yaml
# catalog-info.yaml in every repo — the entity's plaque:
apiVersion: backstage.io/v1alpha1
kind: Component
spec:
  type: service · lifecycle: production · owner: team-checkout
  dependsOn: [resource:orders-db, component:payments-api]
```

- **Ownership mandatory, enforced** (unowned entities page the platform team)
- Metadata **generated from truth** where possible: ownership from CODEOWNERS, tiering from tags the pipelines verify (no self-declared "non-critical" prod services)
- **Location annotations** link entities to runbooks, dashboards, on-call — the portal as *launchpad*, not data silo #9

**The portal's product line (beyond the catalog):**

```text
1. Discover: search, ownership, dependency graphs ("blast radius before the change")
2. Order: scaffolder + self-service (topic, DB, environment — bounded, metered)
3. Understand: TechDocs, API specs (OpenAPI rendered), maturity/security scores
4. Act: plugin-deep-links → dashboards, traces, on-call, cost — one context
```

**Build vs adopt (the honest procurement take):** Backstage is a *framework* — plugins are written, UX is customized, ownership is ongoing (it's a product you operate — the platform team staffs it). The alternative stack: OpsLevel/Cortex/Port (managed portals) — same concepts, less building, less control. Either way: **a portal without the IDP machine room behind it is a pretty map of unpaved terrain** — the scaffolder must actually provision, the scores must reflect enforced reality.

## Real-World Example (DevOps flavored)

The 3 AM use case (the portal earning its keep):

```text
Alert: payments-7d9f failing → portal entity page in one search:
  owner: team-payments · on-call: (current rotation, linked)
  tier: 1 · SLO: 99.9% (burn-rate linked live)
  depends on: orders-db (green), 3ds-gateway (RED — the culprit's neighborhood)
  runbook: link · last deploy: 2h ago (linked to the PR)
Total: 90 seconds from page to "who, what, blast-radius, and first runbook step"
       — versus 15 minutes of chat archaeology. That delta, × every incident, is the ROI.
```

## Common Mistakes

- The catalog as wiki 2.0 — hand-maintained metadata rotting into lies
- Pretty portal over no IDP — ordering buttons that open tickets
- Optional ownership — the directory of mystery entities
- Scores not tied to enforced policy (self-reported maturity theater)
- Building Backstage customizations with no platform staff to own them (the product dies at v1)

## Mental Model

> The IDP is the **machine room**; the portal is the **lobby directory that actually works**: plaques synced to reality (Git-owned metadata), a services desk that actually provisions (scaffolder), and every door labeled with who to call at 3 AM. A lobby without the machine room is theater; a machine room without the lobby is a building nobody can navigate.

## Remember This

1. IDP = capability layer; portal = discovery/ordering front door; Backstage = the dominant kit
2. Catalog entities from repo-committed `catalog-info.yaml` — metadata as code
3. Ownership mandatory + verified; metadata generated from truth where possible
4. Portal jobs: discover, order, understand, act — deep-links into the observability stack
5. The 3 AM page→context seconds metric is the real ROI
6. Portal without IDP = pretty map of unpaved terrain; either needs product staffing

## One Sentence

The internal developer portal — with Backstage as the dominant framework — makes an organization's software legible and actionable: cataloged, owned, related, and orderable, turning tribal-memory questions like "who owns this and what does it affect" into 90-second lookups.

## Knowledge Check

1. Why must catalog metadata live in repos rather than the portal's database?
2. Trace the 3 AM flow on your org: what's missing from entity-page-to-resolution today?
3. Which catalog fields must be machine-verified rather than self-declared, and why?
4. Portal-first or path-first — what fails if you build them in the wrong order?

## Further Reading

- [backstage.io docs](https://backstage.io/docs/overview/what-is-backstage/) · CNCF landscape (portal tools)
- Next: [Platform as Product & Metrics](platform-as-product.md)

---

**← Previous:** [Developer Experience & Golden Paths](dx-golden-paths.md)
**Next:** [Platform as Product & Metrics](platform-as-product.md) →
**Related:** [Team Topologies](team-topologies.md)
