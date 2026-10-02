# High Availability & Disaster Recovery

## What Is It?

- **HA (High Availability)** — survive *component* failures automatically: redundancy + failover within your normal operating footprint (multi-AZ)
- **DR (Disaster Recovery)** — survive *catastrophic* failures (region loss): a plan, infrastructure, and data in another failure domain, plus the practiced motion to move there

The two numbers that define everything:

| Metric | Question | Set by |
|---|---|---|
| **RTO** (Recovery Time Objective) | How long until we serve again? | architecture + runbooks + practice |
| **RPO** (Recovery Point Objective) | How much data may we lose? | replication/backup design |

## Why Does It Exist?

Because everything fails at some rate — hardware, networks, regions, and *humans and their deployments* (most "disasters" are changes, not earthquakes). HA and DR are the engineered responses to two different failure classes:

```text
Failure scale:   one disk/VM/AZ   →   region/provider   →   logical (bad deploy, bad data)
Response:        HA (automatic)       DR (practiced)        backups + rollback + runbooks
```

Note the third column — the most common "disaster" is self-inflicted, and it's defeated by the CD page's rollback discipline plus the backup logic discussed here.

## Layer 1 — Simple Explanation

- **HA** is the **spare tire**: mounted, matching, and the swap is quick and rehearsed — driving continues (almost) seamlessly
- **DR** is the **second car in another garage**: for when the first car is truly gone. You must occasionally *drive it* to know it works, and you accept it's slower to reach for than the spare

RTO/RPO in car terms: how long you can be without wheels (RTO) and how much of today's driving you'd lose (RPO — how fresh was your map sync).

## Layer 2 — Engineer's View

**The DR strategy ladder — cost rises with continuity:**

| Strategy | Standby | RTO | RPO | Cost |
|---|---|---|---|---|
| **Backup & restore** | data in another region | hours–days | hours | 1× |
| **Pilot light** | minimal core running (replicated DB) | hours | minutes | ~1.2× |
| **Warm standby** | scaled-down full copy | ~tens of minutes | seconds | ~2× |
| **Active-active** | multi-region serving traffic | ~zero | ~zero | 2×+ (plus consistency design) |

RTO/RPO are *business decisions priced in architecture* — write them down per service tier, then buy exactly that much.

**The replication ladder and its RPO semantics:**

```text
sync replication   → RPO 0, latency cost per write (same-region)
async replication  → RPO seconds–minutes, distance-friendly (cross-region)
scheduled backup   → RPO hours, cheapest, survives logical corruption too
```

Cross-region sync = physics problem; that's why active-active RPO-0 multi-region writes need provider-specific magic (Cosmos/DynamoDB multi-region) or application-level conflict design (CAP — Architecture phase).

**Backups: the 3-2-1 discipline** — 3 copies, 2 media, 1 offsite (cross-region, and *immutable/WORM* so ransomware can't delete them). Plus: encryption, access separation (backup creds ≠ prod creds — attackers delete backups first), and **restore drills** (Storage page's law: backups you haven't restored are hypotheses).

**Failover is choreography, not a switch:**

```text
DNS/traffic steering (health-checked, low TTL prepared)
→ app tier rebuilt or promoted (IaC makes region rebuild feasible at all)
→ database promoted (and the un-replicated tail accepted: RPO realized)
→ queues/caches/state rebuilt → validation → announcement
```

Every step is a runbook that has been *executed*, not written. **Game days**: quarterly region-failover (or at least restore) drills are the difference between a DR *plan* and DR *capability*.

**Dependence audit — the question DR design lives or dies on:** "if this region vanished, what breaks that we haven't replicated?" — DNS? container registry? secrets? the IdP? Terraform state? CI (you rebuild with pipelines — is CI itself multi-region)?

## Real-World Example (DevOps flavored)

ShopEasy's tiered plan:

```text
Tier 1 (checkout):  multi-AZ HA + warm standby in region-2 (RTO 30m, RPO 1m async)
Tier 2 (catalog):   pilot light (RTO 4h, RPO 15m)
Tier 3 (internal):  backup-restore (RTO 24h, RPO 24h)
Backups:            PITR + daily cross-region immutable snapshots; quarterly restore drill
Drill finding 2025: registry + Terraform state weren't replicated — fixed before it mattered
```

## Common Mistakes

- DR documented but never drilled — capability assumed, never proven
- RPO/RTO unstated → architecture can't be right or wrong
- Backups same-region/same-credentials as prod (ransomware deletes both)
- Forgetting the *meta*-dependencies: CI, DNS, registries, secrets, IdP
- Active-active without conflict-resolution design — multi-region split-brain writes
- Treating HA as DR (an AZ-level pattern survives nothing regional)

## Mental Model

> HA is the **spare tire** — instant, automatic, for the everyday puncture. DR is the **second car in another garage** — for the day the garage burns. RTO is how fast you need wheels; RPO is how much of the day's driving you'll lose; and the drill is the occasional drive of car #2, because a second car that's never started isn't a car — it's furniture.

## Remember This

1. HA = automatic component survival (multi-AZ); DR = practiced region survival
2. RTO/RPO are business decisions that price the architecture — define per tier
3. Strategy ladder: backup → pilot light → warm standby → active-active
4. Replication ≠ backup: RPO ladder vs logical-corruption protection — need both
5. 3-2-1 + immutable + credential separation for backups
6. Drills convert plans into capability; audit meta-dependencies (CI, DNS, secrets, registry)

## One Sentence

High availability survives component failures automatically through redundancy, disaster recovery survives catastrophic losses through priced RTO/RPO objectives, replicated data, and — most of all — rehearsed execution.

## Knowledge Check

1. Why does async replication give you RPO>0, and what business conversation does that number force?
2. A ransomware event encrypts your primary and replicas. Which protection saves you, and which design detail made it possible?
3. Tier your services and assign strategy + RTO/RPO — defend checkout vs. reporting.
4. List the four meta-dependencies whose regional loss would prevent your DR from executing.

## Further Reading

- AWS Resilience Hub / Azure resilience docs — RTO/RPO formalized as tooling
- Google SRE book — ch. 3 (embracing risk), "Preparing for disasters"
- Backblaze + DHS CISA guidance on 3-2-1 + immutability

---

**← Previous:** [Load Balancing & Autoscaling](lb-autoscaling.md)
**Next:** [Multi-Region & Multi-Cloud](multi-region-cloud.md) →
**Related:** [Regions & AZs](regions-az.md) · [Storage & Databases](storage-databases.md)
