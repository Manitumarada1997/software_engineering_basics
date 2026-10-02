# Storage & Databases

## What Is It?

Two families of *keeping bytes*:

- **Storage** — objects, files, blocks (the "how do I keep files" menu)
- **Databases** — structured data with query, transaction, and consistency semantics (relational, NoSQL, managed variants)

## Why Does It Exist?

Because "save this" has irreconcilable different requirements — and each storage type is a different answer to *how the bytes are accessed*:

| Type | Products | Access pattern | Latency | Analogy |
|---|---|---|---|---|
| **Object** | S3, Blob, GCS | whole object via HTTP API | ~10–100ms | self-storage warehouse |
| **Block** | EBS, Managed Disks | raw disk blocks to ONE machine | <1–10ms | a private hard drive |
| **File** | EFS, Files | NFS/SMB shared filesystem | ms | shared office drive |

Pick by access: object = write-once/read-many at any scale (cheapest per GB); block = the disk an OS/database needs (exclusive); file = many machines, POSIX semantics.

## Layer 1 — Simple Explanation

- **Object storage**: a **warehouse of sealed boxes** — you store/retrieve whole boxes by label; can't edit "half a box"; infinitely many shelves; dirt cheap
- **Block storage**: a **private desk drawer** — instant access, only you, sized per drawer
- **File storage**: the **shared office drive** — everyone mounts it, files and folders
- **Databases**: the **filing clerk** — not just storage but *lookups, rules, and promises* (indexes, transactions, consistency)

## Layer 2 — Engineer's View

**Object storage engineering details that bite:**

- Buckets are flat keyspaces (no real folders), names are globally unique in some clouds
- Durability is 11-nines *of the objects you successfully wrote* — availability ≠ durability (a "temporarily unavailable" bucket is an outage in your app)
- **Storage classes + lifecycle**: hot → infrequent → glacier automations are real money (FinOps)
- **Consistency**: strong-on-write now (S3 since 2020) — but *listing* lags; read-after-write by key ✓, by prefix-listing ~eventually
- Security: bucket policies, encryption defaults, *public access blocked at org level* — the most famous misconfiguration in cloud history

**Managed databases — what "managed" buys and what it doesn't:**

| They manage | You still own |
|---|---|
| Hardware, patching (engine), backups, HA replication | schema design, indexes, query performance |
| Point-in-time restore | RPO/RPO choices, testing restores |
| Some scaling knobs | capacity planning, connection pooling |

The trade: control (no superuser, limited extensions, engine-locked) + egress gravity (data has mass; leaving is expensive).

**The database taxonomy — match to workload:**

| Family | Use | Examples |
|---|---|---|
| **Relational** | transactions, joins, strong consistency | Postgres, MySQL, Aurora |
| **Document** | flexible/nested documents by key | MongoDB, Cosmos, DynamoDB (key-value) |
| **Wide-column/KV** | massive scale, predictable keys | Cassandra, Bigtable |
| **Cache** | speed layer | Redis, Memcached |
| **Search** | inverted index / full-text | Elastic, OpenSearch |
| **Time-series / graph** | metrics / relationships | Timestream, Neptune |

The deep concepts — replication, sharding, CAP, consistency — get the full Architecture phase. Cloud-page takeaway: *choose managed by default; choose engine by access pattern, not fashion*.

**Backups vs replication — the distinction that saves careers:**

```text
Replication protects availability  (a replica survives node loss)
Backups protect against logic      (DROP TABLE replicates instantly to all replicas!)
```

You need both, and you need restore *tests* — an untested backup is a hypothesis.

## Real-World Example (DevOps flavored)

ShopEasy's storage map:

```text
Product images:     S3/Blob + CDN in front, lifecycle to cool after 90d
VM/K8s disks:       managed block (Premium for DB)
Build artifacts:    object storage (immutable, versioned — Artifact Repos page)
Orders:             Postgres (managed, PITR, cross-AZ replica)
Sessions:           Redis (managed, evictable — cache semantics)
Logs:               object storage → lifecycle → warehouse (Observability phase)
```

The DR drill that matters: quarterly PITR restore into a scratch database + row-count reconciliation — because "backups exist" and "backups restore" are different claims.

## Common Mistakes

- Using block/file storage where object + CDN fits (paying 10× for ms you don't need)
- Public buckets "for the demo" — and the data breach that follows
- Replication mistaken for backup (logical corruption replicates)
- Never testing restores (restore speed = RTO reality; often *hours* nobody planned for)
- Choosing the database of the month over the access pattern
- Forgetting egress: cross-region replication and API reads are bandwidth billed

## Mental Model

> Storage tiers answer *how you reach your stuff*: sealed boxes in a warehouse (object), your private drawer (block), the shared drive (file). Databases add a **clerk with rules** — find, sort, transact, promise consistency. Managed versions rent you the clerk and the warehouse staff, but *you* still decide what goes in which drawer and whether the fire drill (restore test) ever ran.

## Remember This

1. Object / block / file — pick by access pattern, not habit
2. Managed DBs: hardware/HA/backups managed; schema, indexes, capacity, restores still yours
3. Replication ≠ backup — logic errors replicate; test restores quarterly
4. Lifecycle classes + egress are the two hidden cost dimensions
5. Public bucket blocking at org level; buckets are the #1 cloud misconfiguration
6. Engine choice = access pattern (transactions vs documents vs scale-keys vs search)

## One Sentence

Storage options trade access granularity against scale and cost, databases add query and consistency semantics on top, and the managed cloud versions remove operations but never remove your responsibility for schema, capacity, and restore.

## Knowledge Check

1. Why is an 11-nines durability object store still "down" for your app sometimes?
2. Your replica is healthy; last night's migration script corrupted orders. Which protection applies?
3. Place: product images, DB disk, shared configs, orders, sessions — which storage/database and why?
4. What exactly does a quarterly restore drill prove that backup dashboards don't?

## Further Reading

- AWS S3 docs / Azure Blob tiers (concepts portable)
- *Designing Data-Intensive Applications* — Kleppmann ch. 1–3 (preview of Architecture phase)
- Backblaze/ACM stories on backup testing culture

---

**← Previous:** [Compute Options](compute.md)
**Next:** [Load Balancing & Autoscaling](lb-autoscaling.md) →
**Related:** [HA & Disaster Recovery](ha-dr.md) · [Caching](../architecture/caching.md)
