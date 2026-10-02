# Multi-Region & Multi-Cloud

## What Is It?

- **Multi-region**: one provider, several regions — usually for latency (users everywhere), residency (data law), or DR beyond a region
- **Multi-cloud**: more than one provider — usually for consolidation after mergers, regulatory/procurement demands, avoiding single-vendor lock-in, or (rarely) true cross-provider HA

## Why Does It Exist?

Different forces pull toward each:

- Multi-region pull: **physics** (latency), **law** (GDPR), and **regional-outgrade risk** (rare but real — control-plane incidents have taken whole regions down)
- Multi-cloud pull: **organizational** (acquisitions, sovereignty rules, negotiating leverage) and **risk** ("what if AWS has a really bad day")

The honest framing: multi-region is an *architecture* decision with known patterns; multi-cloud is usually an *organizational* reality to be survived, occasionally a deliberate hedge.

## Layer 1 — Simple Explanation

- Multi-region: one supermarket chain, **stores in many cities** — same systems, localized stock, a playbook for moving customers if a city's store closes
- Multi-cloud: doing your shopping at **two different chains with different aisle layouts, coupon rules, and house brands** — possible, but you now do everything twice with different rules

## Layer 2 — Engineer's View

**Multi-region design — the three patterns:**

| Pattern | Shape | Best for |
|---|---|---|
| Active-passive | one region serves; other is DR (HA/DR ladder) | most workloads |
| Active-active, partitioned | each region owns its users/data (EU users in EU) | residency + latency |
| Active-active, replicated | all regions serve all users from replicated data | global low-latency, hardest |

The third pattern collides with physics and CAP: cross-region write replication means either conflict resolution (last-write-wins? CRDTs? per-user partitioning) or provider magic (DynamoDB/Cosmos multi-region). **Partition by user/region wherever possible** — it converts a distributed-systems problem into a routing problem.

**The four multi-region taxes:**

1. **Data gravity**: cross-region replication bandwidth and consistency lag
2. **Traffic steering**: health-checked DNS/anycast (Route 53/Front Door), low TTLs prepared in advance
3. **State everywhere**: caches, queues, secrets, registries — replicated or regional with a promotion story
4. **Testing**: failover drills across regions (HA/DR page's law)

**Multi-cloud — the honest engineering account:**

| Claim | Reality |
|---|---|
| "Avoid lock-in" | Abstractions leak; the *real* lock-in is data + IAM + operational muscle, not APIs |
| "Provider-outage insurance" | Only if workloads *actually fail over* — an idle GCP account saved you nothing |
| "Best-of-breed per service" | Integration cost usually exceeds the feature delta |
| Legit reasons | M&A estates, sovereignty rules, pricing leverage, distinct workload fit |

When it *is* your reality, the survival strategies (in order of sanity):

1. **Workload-level separation**: each workload lives fully in one cloud (per-team or per-domain) — no cross-cloud complexity, org-level hedging
2. **Portable layers**: containers + K8s as the common substrate; Terraform/OpenTofu for both; CI/CD abstracted
3. **Avoid** cross-cloud *distributed systems* (a request spanning two providers' latency and trust boundaries) at all costs

The cost multiplier is real: duplicated skills, duplicated pipelines, duplicated security programs, and an on-call rotation that now pages for two control planes. Choose deliberately or not at all.

**Sovereignty as the rising force:** data-residency laws increasingly mandate in-country regions (and even sovereign clouds) — often the *actual* reason "multi" appears in an architecture. Design region-pinned data models from day one if you might need them.

## Real-World Example (DevOps flavored)

ShopEasy goes global:

```text
Pattern: partitioned active-active
  eu-west: EU customers + their data (residency)     ┐ independent
  us-east: Americas                                   ├─ regions with
  ap-south: APAC + latency                            ┘  async analytics sync
Global: DNS latency routing + health checks; images/registry replicated per region;
       deployments: one pipeline, three regional stages (GitOps per cluster)
Failover: within-region multi-AZ first; cross-region only for regional loss (RPO 5m)
```

What they deliberately *didn't* do: cross-cloud. One estate, one skill surface, deliberate lock-in accepted in exchange for operational depth.

## Common Mistakes

- Multi-cloud "for resilience" with no tested failover — insurance that was never paid up
- Cross-cloud synchronous anything (a request touching two providers)
- Multi-region without the meta-dependencies (registry, DNS, secrets) — failover day surprise
- Believing a thin abstraction makes two clouds one platform
- Underestimating the *people* cost: two sets of quirks, limits, and 3 AM behaviors

## Mental Model

> Multi-region is a **chain with stores in many cities** — same playbook, regional stock, health-checked directions for customers. Multi-cloud is **shopping two rival chains at once** — sometimes the merger made you, sometimes the law made you, but never pretend the aisle layouts match. Partition your eggs; don't build one basket that spans two stores.

## Remember This

1. Multi-region for latency/residency/DR; patterns: passive, partitioned, replicated
2. Partition by user/region where possible — routing beats conflict resolution
3. Taxes: data gravity, steering, replicated state, drill obligations
4. Multi-cloud: usually organizational reality, not architecture virtue
5. If multi-cloud, separate workloads per cloud; portable layers (K8s, Terraform) — never cross-cloud requests
6. Sovereignty is the growing *legitimate* driver — design region-pinned data early

## One Sentence

Multi-region extends one architecture across geographies for latency, law, and disaster survival, while multi-cloud is an organizational condition best handled by partitioning workloads rather than pretending two providers are one platform.

## Knowledge Check

1. Why does user-partitioning turn a CAP problem into a routing problem?
2. An org claims multi-cloud resilience. What one question tests the claim?
3. List the four multi-region taxes and their mitigations.
4. When is multi-cloud the *right* call? Give three legitimate drivers.

## Further Reading

- Google SRE — multi-region service design chapters
- AWS/Azure global load-balancing (Route 53/Front Door) docs
- CNCF surveys — real-world multi-cloud adoption data

---

**← Previous:** [HA & Disaster Recovery](ha-dr.md)
**Next:** [Containers](../containers/containers.md) →
**Related:** [Regions & AZs](regions-az.md) · [IaC Fundamentals](../iac/iac-fundamentals.md)
