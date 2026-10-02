# Secrets Management & Vault

## What Is It?

**Secrets management** is the discipline of storing, delivering, rotating, and revoking credentials — passwords, API keys, tokens, certs — such that:

1. They're **never** in Git, images, or plain config
2. Access is authenticated, authorized, and **audited**
3. Rotation is routine, not an archaeology project

**Vault** (HashiCorp) is the category-defining tool: a centralized secrets broker with dynamic generation, leasing, and revocation. Alternatives: cloud secret managers (AWS SM/Parameter Store, Azure Key Vault, GCP SM), and the K8s-native bridge: External Secrets Operator.

## Why Does It Exist?

Because credentials are the *operational* form of identity, and every place we naively put them turned into a breach amplifier:

```text
Git        → in history forever (Git Internals page — objects never die)
Images     → in layers forever (Layers page — deletions are cosmetic)
Config     → sprawled, unrotated, unowned
Env vars   → leak into logs, crash dumps, child processes
```

The deeper design insight: **static secrets are doomed** — shared forever, rotated never, revocation theoretical. Vault's move: **dynamic secrets** — generated on demand, short-lived, automatically expiring. A database credential valid for 1 hour needs no rotation program; it needs *nothing*.

## Layer 1 — Simple Explanation

A vault is the **building's key-card system**: cards are issued *per person, per door, per shift* (dynamic), every swipe is logged (audit), a lost card is deactivated instantly (revocation), and the master keys live in a safe (HSM/KMS).

Storing secrets in Git, by contrast, is **taping a copy of the key under the welcome mat — in a neighborhood with a permanent camera** (history is forever).

## Layer 2 — Engineer's View

**Vault's model (the concepts transfer to every tool):**

```text
AuthN (who):  tokens, K8s serviceaccounts, OIDC, cloud workload identity
AuthZ (what): path-based policies ("read payments/db/creds, nothing else")
Secrets:      KV (static, at least centralized+audited) · DYNAMIC engines:
              db/creds → per-client, TTL'd login; PKI → cert issuer;
              cloud engines → short-lived cloud creds
Sealing:      vault locked until unsealed (Shamir/cloud KMS auto-unseal)
Audit:        every read/write logged (the forensic trail)
```

**The delivery problem (the half most teams get wrong):** *how does the workload get the secret?* Ranked:

| Method | Verdict |
|---|---|
| Baked into image/env at build | ❌ permanent, leaky |
| K8s Secret objects | ⚠️ better (RBAC'd) but base64 + etcd; fine for low-sensitivity |
| **External Secrets Operator** | ✅ vault/cloud manager → synced into K8s Secrets; references in Git, values never |
| Vault agent / CSI driver | ✅ direct injection, dynamic renewal in-pod |
| Short-lived identity instead | ✅✅ the real answer where possible: OIDC federation (no secret at all) |

The hierarchy to internalize: **federated identity (no secret) > dynamic short-lived secret > centralized static secret > scattered secrets > Git.**

**Rotation economics:** static secret rotation is a mini-project every N days across consumers (why it never happens). Dynamic TTL'd secrets make rotation *continuous by construction*. When static is unavoidable (third-party API keys), automate rotation *and* break-glass, and treat non-rotatable secrets as accepted risk with compensating controls.

**Vault operations — the honest costs:** it's critical infrastructure itself: HA + unseal strategy, upgrades, and the bootstrap paradox (who authenticates to vault? — solved by trusted platform identity: K8s SA / cloud workload identity). Cloud secret managers trade power for zero-ops — often the right default.

## Real-World Example (DevOps flavored)

ShopEasy's secrets architecture, tiered:

```text
CI→cloud/cloud→cloud:    OIDC federation — no secrets exist (IAM page)
Database creds:          Vault dynamic engine — 1h TTL per pod, per-service policies
Third-party API keys:    cloud secret manager + ESO sync + quarterly automated rotation
K8s-native config:       ConfigMaps (non-secret) + minimal Secrets
Audit:                   vault audit log + cloud access logs → SIEM alerts on anomalous reads
Git:                     contains REFERENCES (ESO ExternalSecret specs) only
```

The drill that proves it: quarterly "rotate everything" — with this stack it's a no-op for 80% of secrets (they expire hourly by themselves).

## Common Mistakes

- Secrets in Git "for now" — history is forever; rotation + history-scrub (BFG) is the only recovery
- Long-lived static secrets everywhere because dynamic engines "seemed complex"
- Vault as a fancy static KV (no dynamic engines, no audit) — expensive envelope for the same old problem
- Over-broad policies (`read secret/*`) — the vault reproduces the flat-network sin
- No break-glass/access-recovery runbook — sealed vault + absent admins = self-DoS

## Mental Model

> The vault is the **key-card office**: per-shift cards (dynamic secrets), every swipe logged, instant deactivation — versus the industry's old habit of **taping master keys under mats and photocopying them into the archive (Git)**. The endgame isn't better card storage: it's **doors that recognize faces** (federated identity) and never issue cards at all.

## Remember This

1. Secrets never in Git/images/env-var sprawl; centralize → audit → dynamize
2. Dynamic, TTL'd secrets eliminate rotation as a human program
3. Delivery: ESO/Vault-agent/CSI — Git holds references, never values
4. Best secret is none: OIDC/workload federation where possible
5. Vault ops costs (HA, unseal, bootstrap) — cloud managers as sane defaults
6. Audit logs are the point: who read what, when

## One Sentence

Secrets management centralizes credentials into audited, policy-controlled storage and replaces static long-lived secrets with dynamically generated, automatically expiring ones — ideally eliminating secrets entirely via federated workload identity.

## Knowledge Check

1. Rank the delivery methods and defend the top choice for a K8s workload.
2. Why does a 1-hour dynamic DB credential need no rotation program — and what does it still need?
3. What's the bootstrap problem, and how does platform identity solve it?
4. A secret leaked into Git a year ago. Full remediation list?

## Further Reading

- [Vault docs — dynamic secrets](https://developer.hashicorp.com/vault/docs/concepts) · External Secrets Operator
- Next: [Network Security](network-security.md)

---

**← Previous:** [PKI & Encryption](pki-encryption.md)
**Next:** [Network Security](network-security.md) →
**Related:** [ConfigMaps, Secrets & Volumes](../kubernetes/config-secrets.md) · [Cloud IAM](../cloud/iam.md)
