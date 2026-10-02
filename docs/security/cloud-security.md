# Cloud Security

## What Is It?

Applying the security doctrine (previous pages) to the cloud's specific attack surface:

1. **The control plane is the crown jewel** — cloud APIs manage everything
2. **Configuration is the perimeter** — there's no box to harden, only settings to get right
3. **Shared responsibility draws the line** — provider secures *of* the cloud; you secure *in* it (Cloud page)

You already own the components: IAM (identity), VPC/SGs (network), KMS (keys), logging (forensics). This page assembles them into the cloud threat model.

## Why Does It Exist?

Because the cloud inverted the anatomy of a breach:

```text
On-prem breach:  exploit an app → pivot the network → find the DB
Cloud breach:    find a leaked key / over-broad role / public bucket → done
```

Data: most cloud incidents are **misconfigurations**, not exploits — public storage, over-permissive IAM, exposed management endpoints. The attack surface is *your YAML* — which is why the defense is also your YAML (IaC + policy-as-code, later page).

## Layer 1 — Simple Explanation

The cloud provider runs a **guarded building with excellent locks** — and hands *you* the lock-setting panel. The panel is powerful, defaults are dangerous (public-by-default buckets, admin-by-convenience roles), and nobody patrols your settings for you.

Cloud security = **the discipline of setting ten thousand switches correctly, continuously** — because attackers scan for the one you missed, automatically, within minutes.

## Layer 2 — Engineer's View

**The cloud threat model — top failure families:**

| Family | The classic | The defense (page) |
|---|---|---|
| Credential exposure | keys in Git/images/CI variables | OIDC federation (IAM, Secrets pages) |
| Over-privilege | `*` policies, standing admin | least privilege, JIT (RBAC page) |
| Public exposure | public buckets/DB snapshots/ports | org-level public-block, CSPM scanning |
| Metadata/SSRF | pod → 169.254.169.254 → role theft | IMDSv2, least-priv roles (CIA page) |
| Control-plane compromise | org-level admin takeover | Tier-0 hygiene, MFA, phishing-resistant |

**The four security services every cloud ships (learn once, portable forever):**

| Service | Job | AWS / Azure examples |
|---|---|---|
| **IAM** | identity + policy | IAM/Entra |
| **KMS** | keys + envelope encryption | KMS/Key Vault |
| **Audit logging** | every API call recorded | CloudTrail/Activity Log — *turn it on everywhere, centralize* |
| **Config/CSPM** | continuous conformance scanning | Config/Defender for Cloud |

**CSPM — the category that operationalizes the doctrine:** continuously evaluates your estate against benchmarks (**CIS**, vendor benchmarks) and alerts on drift: public buckets, unencrypted volumes, ancient credentials, exposure changes. Tools: cloud-native + third-party (Wiz, etc.). The workflow: findings → owner → fix (via IaC PR, not console — the drift lesson inverted into security).

**Guardrails > gates (the platform-scale pattern):** instead of reviewing every account's settings, enforce org-level **service control policies / Azure Policy**: "deny public buckets", "require tags", "restrict regions" — misconfiguration *becomes impossible*, not merely detectable. This is security-as-platform (Phase 11 foreshadow again — guardrails are the security chapter of golden paths).

**Incident response in the cloud** — the runbook skeleton: detect (guard-duty/alarm logs) → contain (revoke sessions/keys, isolate network) → scope (audit logs: what did that role touch?) → eradicate/recover → postmortem (SRE phase). Everything depends on **logs existing and being centralized** — an org without CloudTrail has incidents without investigations.

## Real-World Example (DevOps flavored)

ShopEasy's cloud-security baseline, as Terraform modules (the only way it scales):

```hcl
module "org-guardrails" {          # applied org-wide
  deny_public_buckets = true; require_encryption = true; allowed_regions = [...]
}
module "audit" { central_cloudtrail + org-wide, KMS-encrypted, replicated }  
module "iam-baseline" { no root keys, MFA-enforced, OIDC-only for CI, break-glass alarmed }
# plus: scheduled CSPM scan → findings as GitHub issues, auto-assigned to resource owner tags
```

The finding that proves the design works: a public-bucket attempt by a contractor team — *rejected by policy* before creation. Detection without prevention is a slower headline.

## Common Mistakes

- Security as a checkpoint project (audit once) instead of continuous posture (CSPM + guardrails)
- Detection without centralized logs — alerts pointing at darkness
- Long-lived keys because "federation is complex" — the single most common root cause
- Ignoring Tier-0 (org admin) hygiene while hardening workloads
- Console fixes for CSPM findings — recreating the drift disease under security pressure

## Mental Model

> Cloud security is **ten thousand switches and no patrol**: the provider's building is fine; your settings are the walls. The mature posture stops *finding* bad switches (CSPM) and starts *removing the switch* (guardrails) — so the question shifts from "did we miss one?" to "was it even possible?"

## Remember This

1. Cloud breaches are mostly misconfiguration, not exploitation
2. Attack surface = your config; so is the defense (IaC + org-level guardrails)
3. The portable four: IAM, KMS, audit logging, CSPM/config scanning
4. Prevention by policy (SCP/Azure Policy) beats detection — make bad states impossible
5. Centralized audit logs are the substrate of all incident response
6. Tier-0 (control-plane admins) is the crown jewel — MFA, JIT, monitored

## One Sentence

Cloud security is the continuous discipline of correct configuration — identity, exposure, encryption, and logging — enforced at scale through guardrails and posture management rather than periodic audits.

## Knowledge Check

1. Why did the breach anatomy change from "exploit and pivot" to "find the misconfiguration"?
2. Design an org-level guardrail set: five policies you'd deny outright.
3. What can't you investigate without centralized CloudTrail — walk an incident.
4. Compare CSPM detection vs SCP prevention with one concrete control.

## Further Reading

- CIS Benchmarks (cloud providers) — the conformance dictionaries
- AWS SCPs / Azure Policy docs (same concept, both clouds)
- Next: [Kubernetes & Container Security](k8s-container-security.md)

---

**← Previous:** [Network Security](network-security.md)
**Next:** [Kubernetes & Container Security](k8s-container-security.md) →
**Related:** [Cloud IAM](../cloud/iam.md) · [Policy as Code](policy-as-code.md)
