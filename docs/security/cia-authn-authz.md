# CIA Triad & AuthN/AuthZ

## What Is It?

- **CIA triad** — security's three goals: **Confidentiality** (only authorized see), **Integrity** (data isn't altered improperly), **Availability** (systems work when needed)
- **AuthN vs AuthZ** — authentication (*who are you?*) vs authorization (*what may you do?*) — the most-confused pair in security

## Why Does It Exist?

Every security control ever built maps to one of three questions — CIA is the checklist that prevents one-dimensional security:

```text
Encrypt everything (Confidentiality ✓) but no backups (Availability ✗)
Five-9s uptime (A ✓) but anyone can edit records (Integrity ✗)
```

And every access decision decomposes into two independent steps:

```text
AuthN: prove identity       (password, key, token, certificate)
AuthZ: check permissions    (roles, policies, scopes)
AuthN without AuthZ = identified strangers rummaging everywhere
AuthZ without AuthN = anyone claiming to be "admin" is believed
```

## Layer 1 — Simple Explanation

- **Confidentiality** = **sealed envelopes** (only the recipient opens)
- **Integrity** = **tamper-evident tape** (any alteration is detectable)
- **Availability** = the **shop being open** (guarded against mobs and closures — DoS is a security problem)
- **AuthN** = the **passport check**; **AuthZ** = the **ticket/seat check** — different officers, different questions, both required

## Layer 2 — Engineer's View

**Mapping your existing world onto CIA (you know more than you think):**

| Control you operate | Protects |
|---|---|
| TLS everywhere (Networking phase) | C in transit + I |
| Encryption at rest / immutable tags | C at rest, I of artifacts |
| Backups / multi-AZ / rate limits / LBs | **A** — remember: availability is security |
| Checksums/signatures (digests!) | I |
| IAM/RBAC (cloud + K8s pages) | the AuthN/AuthZ machinery |

**AuthN factors — the taxonomy:**

```text
something you know (password) · have (key/token) · are (biometric)
MFA = two different factors (password + SMS ≠ 2 factors if SMS phishable — NIST demoted it)
```

**The DevOps-authN reality you must internalize:**

- Humans → SSO + MFA (federated identity — next pages)
- Workloads → short-lived cryptographic identity (OIDC tokens, SPIFFE, K8s SAs) — *never* long-lived passwords/keys (the IAM page's OIDC pattern is *the* answer)
- The anti-pattern to hunt: shared accounts, stored secrets as "identity", `connection strings with passwords` in config

**AuthZ models (one line each, deep dives next):**

- **RBAC**: permissions via roles (K8s/cloud IAM pages)
- **ABAC**: permissions via attributes (labels, tags, context)
- **ACLs/direct**: per-resource lists (file permissions — Linux phase)
- The engineering rule: **deny by default; grant minimum; audit everything** (logs of "who did what" are the forensic substrate — CloudTrail/audit logs from earlier pages)

**Confused deputy & privilege escalation — the two attack shapes to know:** a deputy with more power than its caller (SSRF hitting cloud metadata = the classic cloud exploit chain) — mitigations: least privilege *for the deputy*, IMDSv2/locked metadata, no ambient credentials. Most real breaches are AuthZ failures wearing AuthN costumes.

## Real-World Example (DevOps flavored)

The SSRF chain you defend against (the cloud era's signature exploit):

```text
1. App has an SSRF bug (fetches user-supplied URLs)
2. App pod has a cloud role attached (IAM page) with broad S3 read
3. Attacker: curl http://169.254.169.254/latest/meta-data/iam/... → steals role creds
4. Now reads the bucket — AuthN "succeeded" (stolen, but valid); AuthZ far too broad
Defense: least-privilege the role (damage = nothing), IMDSv2 hop-limit, egress control
```

Each mitigation maps to a page you've already read. Security is the course, assembled.

## Common Mistakes

- Confusing AuthN/AuthZ in design docs ("we have authentication, we're secure")
- Availability treated as ops, not security — until the DDoS page becomes an outage
- Long-lived credentials as workload identity
- Broad service roles ("it needed S3:* to work") — the confused-deputy fuel
- No audit logging — incidents become unsolvable mysteries

## Mental Model

> CIA is the **three locks on the vault door**: opaque (confidential), tamper-evident (integrity), and reliably openable (availability). AuthN is the **passport check at the door; AuthZ the room-by-room keycard** — and most real heists are stolen keycards used on over-permissive doors, not broken passports.

## Remember This

1. CIA: confidentiality, integrity, availability — availability *is* security
2. AuthN (identity) and AuthZ (permission) are separate, both required
3. Humans: SSO+MFA; workloads: short-lived federated identity — never stored secrets
4. Deny by default, least privilege, audit everything
5. The classic cloud exploit is an AuthZ failure (over-privileged deputy) reached via SSRF
6. Your existing controls already map to CIA — security is architecture, not a product

## One Sentence

Security rests on the CIA triad — keeping data hidden, unaltered, and available — while access control splits into proving identity (authentication) and checking permissions (authorization), with most real breaches being over-permissioned rather than broken authentication.

## Knowledge Check

1. Map five controls from earlier phases onto C/I/A.
2. Explain the SSRF-to-bucket chain as an AuthN/AuthZ failure pair.
3. Why is a password+SMS not two proper factors, cryptographically?
4. Why is a DDoS a *security* incident under the CIA model?

## Further Reading

- NIST SP 800-63 (identity assurance) — the AuthN bible
- Next: [RBAC & ABAC](rbac-abac.md)

---

**← Previous:** [GitOps](../iac/gitops.md)
**Next:** [RBAC & ABAC](rbac-abac.md) →
**Related:** [Cloud IAM](../cloud/iam.md) · [Namespaces, RBAC & Policies](../kubernetes/rbac-policies.md)
