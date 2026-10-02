# Regions & Availability Zones

## What Is It?

The cloud's **physical topology hierarchy** — the geography that all HA/DR design (next pages) is built from:

```text
Region (e.g., eu-west-1, West Europe)        — a geography, independent failure domain
 └── Availability Zone (AZ, e.g., eu-west-1a) — 1+ isolated datacenters
      ├── separate power, cooling, network
      └── low-latency fiber links to sibling AZs (< ~2ms)
```

Plus **edge locations** (CDN PoPs — hundreds, no compute) — don't confuse them with regions.

## Why Does It Exist?

Because availability is a **choice of independent failure domains**. The physics of failure:

- One server dies (disk, RAM) → AZs solve nothing; instance-level redundancy does
- One DC dies (power, fire, network) → spread across AZs
- One region dies (regional outage, natural disaster, control-plane bug) → spread across regions

Each level is rarer but *more* catastrophic. Cloud providers built the hierarchy so you can *buy* exactly the level of survival you're willing to pay for — cross-AZ traffic and multi-region complexity are the price.

## Layer 1 — Simple Explanation

- **AZs** are **buildings on the same campus**: far enough apart that one fire doesn't take both, close enough to walk between (fast links)
- **Regions** are **campuses in different cities**: one flood/earthquake/blackout can't touch both — but moving between them takes real time

Your HA tier ladder:

```text
Single instance   → survives: nothing
AZ-spread         → survives: DC failure        (+ ~zero latency cost)
Region-spread     → survives: region failure    (+ latency, cost, data-sync design)
Multi-cloud       → survives: provider failure  (+ all previous costs, squared)
```

## Layer 2 — Engineer's View

**The default design rule:** *multi-AZ always, multi-region deliberately.*

- Multi-AZ is nearly free operationally: managed LBs, ASGs, managed DBs handle it natively; cross-AZ latency (~1–2ms) is invisible to most apps
- Multi-region demands real architecture: data replication (consistency! CAP page looms), failover runbooks, traffic steering — never accidental

**Consistency across AZs — the hidden cost:** everything spanning AZs is effectively a *distributed system*. Managed services abstract this (MySQL/Postgres HA replicas, Cosmos/ DynamoDB multi-region writes) but the trade-offs (async replication = potential data loss on failover, RPO>0) don't disappear — the service just chose defaults for you. Know your service's RPO/RPO promises (HA/DR page).

**Region selection criteria (the checklist):**

1. Data residency/legal (GDPR: EU data in EU regions)
2. Latency to users (physics — CDN page)
3. Service availability (not every service is in every region — check before designing)
4. Cost (prices differ per region; egress compounds)

**Zonal vs regional resources — the distinction that bites:**

| Zonal | Regional |
|---|---|
| lives in one AZ (a VM, a disk) | spans/is reachable across AZs (LB, VPC) |
| dies with its AZ | survives AZ failure |
| IP-in-one-zone: some services (older managed DBs) are zonal → your "multi-AZ app" has a SPOF |

The audit question for any "highly available" architecture: *name the zonal resource*. If there's one zonal component in the critical path, you're single-AZ with extra steps.

## Real-World Example (DevOps flavored)

```text
# The audit of "we're highly available":
App: ASG across 3 AZs ✓
LB: regional ✓
DB: primary az-a, sync replica az-b, automated failover ✓
Cache: single node in az-a ✗  ← found it. Session cache → cache stamps/latency on az-a loss
K8s: 3 node-pools spread, pod topology spread constraints ✓
```

That single cache line is the difference between surviving an AZ failure and a 2 AM page. Audits find these; incidents find them more loudly.

## Common Mistakes

- "Multi-AZ" claimed while a zonal disk/instance/IP sits in the critical path
- Multi-region without data-replication design — failover to an empty database
- Cross-AZ chatter in chatty architectures (microservice A in az-a calling B in az-b per request) — the 2ms tax multiplied by hop count on the p99
- Ignoring data residency until legal asks pointed questions
- Cross-region DR that has never been failed over — DR you haven't tested is a wish

## Mental Model

> AZs are **campus buildings** (survive a fire, walkable); regions are **cities** (survive a flood, real distance). HA design is deciding *which disasters you rent immunity to* — each tier up the ladder costs latency, money, and design complexity. And the audit is always the same question: *which single building does everything still secretly live in?*

## Remember This

1. Region = geography; AZ = isolated DCs with fast links; edge ≠ region
2. Failure-domain ladder: instance → AZ → region → provider; each rung rarer, costlier
3. Multi-AZ always (nearly free); multi-region deliberately (architecture, not checkbox)
4. Services spanning AZs are distributed systems — know their replication/RPO defaults
5. Audit for zonal resources in every "HA" design
6. Region choice: residency, latency, service availability, cost

## One Sentence

Regions and availability zones are the cloud's hierarchy of independent failure domains, letting you buy exactly the level of disaster survival you design — and pay for — through multi-AZ by default and multi-region by decision.

## Knowledge Check

1. Why is cross-AZ latency "invisible" to one service call but ruinous to a chatty chain?
2. Find the failure: app spread across AZs, sync-DB replica, single-AZ cache — what happens when that AZ dies?
3. What does a managed DB's async replication promise about data loss on failover?
4. Which region-selection criterion is non-negotiable in an EU-GDPR workload?

## Further Reading

- AWS/Azure/GCP reliability/regions docs (concepts identical)
- Google SRE book ch. 26 — data processing (failure-domain thinking)

---

**← Previous:** [IaaS, PaaS, SaaS, Serverless](service-models.md)
**Next:** [VPC / VNet](vpc-vnet.md) →
**Related:** [HA & Disaster Recovery](ha-dr.md) · [Linux Networking](../linux/networking.md)
