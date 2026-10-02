# ConfigMaps, Secrets & Volumes

## What Is It?

The configuration and state subsystem of K8s:

- **ConfigMap** — non-sensitive configuration (key-values, files) injected into pods
- **Secret** — the same mechanism, nominally for sensitive data (base64, more handling rules)
- **Volumes / PV / PVC / StorageClass** — durable storage decoupled from pod lifecycles

## Why Does It Exist?

The twelve-factor discipline (Images page): *the same image runs everywhere; configuration differs*. K8s' answer separates three concerns:

```text
image     = code (immutable — Artifact pages)
config    = environment-specific non-secrets (ConfigMap)
secrets   = environment-specific credentials (Secret + a real vault)
storage   = state that outlives pods (Volumes/PV)
```

## Layer 1 — Simple Explanation

- **ConfigMap** = the **office's settings memo** pinned to the wall ("printer is on floor 2") — same employees (image), different memo per office (environment)
- **Secret** = the **safe-deposit box** with the office keys — nominally locked
- **PVC/PV** = the **filing cabinet in the basement**: you request one (claim), facilities provides it (persistent volume), and it stays even when the office is rebuilt

## Layer 2 — Engineer's View

**ConfigMap's three injection modes (and their reload semantics):**

| Mode | How | Update behavior |
|---|---|---|
| env vars | `envFrom` | **never** updates (env is set at process start) |
| command args | rendered at container start | never |
| mounted files | volume mount | updates (eventually) — if the app re-reads |

The classic surprise: config change → nothing happens → because it's an env var. File-mount + reloader (Reloader/stakater) or rollout-restart-on-change is the standard pattern.

**Secrets — the honest security accounting:**

- Base64 ≠ encryption (it's *encoding*); at rest, etcd encryption must be enabled (it often isn't by default)
- Anyone with pod-exec or etcd read can read them; RBAC on secrets is *the* access control
- They still beat ConfigMaps + (subsup the leak surface: no accidental logs of full objects, volume-exposure control, rotation tooling hooks) — but for real secrets posture: **external secret operators** (Vault/ESO) — secrets synced on demand, never in Git, audit trails — the Secrets Management page (Security phase) owns this

**The volume taxonomy (the ladder from Containers page, K8s edition):**

| Volume type | Lifecycle | For |
|---|---|---|
| emptyDir | pod lifetime (scratch, empty at start) | tmp space, inter-container sharing |
| configMap/secret mounts | projected into pod | config-as-files |
| PV (via PVC) | independent of pods | databases, anything durable |
| ephemeral (generic ephemeral volumes) | pod lifetime but provisioned | scratch on real storage |

**PV/PVC/StorageClass — the separation of concerns:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
spec:
  accessModes: [ReadWriteOnce]
  resources: { requests: { storage: 50Gi } }
  storageClassName: fast-ssd      # → StorageClass provisioner (cloud disk) auto-creates PV
```

- **PVC** = the developer's request ("50Gi RWO"); **PV** = the concrete volume; **StorageClass** = the menu (fast/slow/replicated) + provisioner
- `ReadWriteOnce/Many` semantics bind scheduling (RWO volumes pin pods to a node!)
- **Reclaim policy**: Delete vs Retain — the "test cluster ate the prod data" guard
- StatefulSets' `volumeClaimTemplates` give per-replica stable storage (next page)

**The statelessness gradient to internalize:** emptyDir (scratch) → ephemeral PV → PVC → StatefulSet PV → external managed DB (the Storage & Databases page — often the *right* answer). Each step left is more self-managed operational weight.

## Real-World Example (DevOps flavored)

The two incidents that teach this page:

```text
1. "We rotated the DB password but pods still fail": secret file-mounted, app caches at
   boot → env-var-style staleness. Fix: reload pattern or restart-on-secret-hash
   annotation (Reloader), then external-secrets for real rotation.

2. "Rescheduled pod stuck ContainerCreating for 6 min": RWO PV still attached to the
   dead node (volume attach lag). Fix: wait for detach timeout / force-detach;
   long-term: regional RWO options or storage topology awareness.
```

And the anti-pattern audit: `kubectl get configmap giant -o yaml` with 40 keys including a DB URL with credentials inside — split config vs secret, then secret → vault-synced.

## Common Mistakes

- Secrets in ConfigMaps (or in Git at all) — the base64 fallacy
- Expecting env-var config to hot-reload
- `reclaimPolicy: Delete` on prod data volumes "because the test cluster did it"
- Giant kitchen-sink ConfigMaps (every deploy restarts everything for one flag)
- emptyDir used for "data we'd like to keep" — it's scratch, by definition
- RWO volume topology fights ignored until pods hang ContainerCreating

## Mental Model

> ConfigMaps are **settings memos**, Secrets their **locked-box cousins** (locked only as well as your RBAC and etcd config), and PVCs are **basement filing cabinets you request by form** — the office above may burn nightly, but the cabinet, and its contents, persist. The gradient of cabinets (emptyDir → PVC → external DB) is really a gradient of *how much storage you're willing to operate yourself*.

## Remember This

1. Image/config/secrets/storage separated by concern; env vars freeze at start, file-mounts reload
2. Secrets = base64 + RBAC + (maybe) etcd encryption — real posture needs vault integration
3. PVC requests, PV provides, StorageClass menus; RWO pins scheduling
4. Reclaim policy is the prod-data guard; StatefulSet templates per-replica storage
5. Statelessness gradient: scratch → PVC → StatefulSet → external managed DB
6. Config-as-files + restart-on-change is the standard dynamic-config pattern

## One Sentence

ConfigMaps and Secrets inject environment-specific configuration and credentials into otherwise-identical images, while the PVC/StorageClass system decouples durable storage from disposable pods — together completing the twelve-factor contract inside Kubernetes.

## Knowledge Check

1. Why didn't the ConfigMap update reach your app? Two mechanisms, two answers.
2. What actual protections does a Secret have that a ConfigMap lacks — and what does it lack?
3. Why does an RWO volume "pin" a pod, and what symptom does the pinning cause?
4. Place on the statelessness gradient: Redis cache, PostgreSQL, tmp scratch, Kafka — and defend each.

## Further Reading

- [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/) / [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) / [Storage](https://kubernetes.io/docs/concepts/storage/) — kubernetes.io
- External Secrets Operator docs (preview of the Security phase)

---

**← Previous:** [Services & Ingress](services-ingress.md)
**Next:** [StatefulSets, DaemonSets, Jobs](workloads.md) →
**Related:** [Container Networking & Storage](../containers/networking-storage.md) · [Secrets Management](../security/secrets-vault.md)
