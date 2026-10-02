# Cloud IAM

## What Is It?

**Identity and Access Management** — the cloud's security kernel: *who* (identity) can do *what* (actions) to *which resources* (scope). Every API call in a cloud is evaluated against IAM **before** anything happens.

```text
Principal (who: user / service / workload) 
   → is allowed Action (s3:GetObject)
   → on Resource (arn:...:bucket/payments-logs)
   → if Condition (source IP, MFA, time)        ← a policy
```

AWS IAM, Azure RBAC + Entra ID, GCP IAM — same model, different dialects.

## Why Does It Exist?

Recall the cloud mental shift: security went from **perimeter** (network location = trust) to **identity** (every API call authenticated and authorized). IAM is the system that makes that real. In a world where infrastructure is API calls, *controlling the API calls IS controlling the infrastructure*:

- Network controls (VPC page) gate *reachability*
- IAM gates *capability* — even inside the network, and especially from the internet-facing control plane

Which is why cloud breaches are almost always IAM failures: leaked admin keys, over-permissive roles, public-by-default misconfigurations.

## Layer 1 — Simple Explanation

IAM is the **building's credential system**: badges (identity), clearance levels (roles), per-door rules (policies), badge scanners everywhere (every API call checked), and a guest-sign-in desk for external visitors (federation).

Key vocabulary in building terms:

| Term | Building equivalent |
|---|---|
| **User** | an employee's badge |
| **Role** (assume-able) | a clearance vest anyone authorized can *put on temporarily* |
| **Policy** | the written rulebook: vest X opens doors Y if condition Z |
| **Group** | department with shared clearances |
| **Federation** | accepting another company's badges (SSO/OIDC — Security phase) |

## Layer 2 — Engineer's View

**Roles vs users — the principle that changes everything:**

```text
Bad:  long-lived AWS access key in pipeline variables (a permanent employee badge
      taped inside the elevator — anyone in the elevator owns it)
Good: pipeline assumes a role via OIDC trust (a temporary vest, issued per run,
      expires in 1h, audited by who assumed it)
```

**Workload identity federation** (OIDC between your CI and the cloud — GitHub Actions/Azure DevOps → role assumption) is the modern default: *no stored cloud credentials anywhere in the pipeline*. This is the single highest-leverage IAM improvement most orgs can make.

**Least privilege, practically:**

1. Start deny-all; grant per observed access (IAM Access Analyzer / policy insights)
2. Scope by resource ARN, not `resource: "*"`
3. Separate read from write; separate data-plane from admin-plane
4. Conditions add precision: `aws:SourceIp`, `aws:RequestedRegion`, MFA-required
5. Audit continuously — permissions drift is inevitable; unused permissions are inventory for attackers

**The blast-radius hierarchy — protect the control plane above everything:**

```text
Tier 0: org/account-level IAM admin     ← compromise = total loss
Tier 1: resource admins (VPC/K8s admin)
Tier 2: workload roles (app needs S3 read)
Tier 3: read-only/auditors
```

Human admins: SSO + MFA + just-in-time elevation (PIM) — standing Tier-0 access for daily work is a finding, not a convenience.

**Separation of duties & accounts structure:** prod vs non-prod accounts/subscriptions (hard boundary, not just naming), billing/payer isolation — the account boundary is IAM's strongest wall.

**Audit as first-class:** CloudTrail/Activity Logs record every API call — identity, source IP, action. Incident forensics and compliance both *are* these logs (Observability phase: they're telemetry too).

## Real-World Example (DevOps flavored)

Your pipeline, before and after:

```yaml
# Before: variables with AZURE_CLIENT_SECRET / AWS_ACCESS_KEY_ID  ❌
# After: service connection via OIDC federation                   ✅
- task: AzureCLI@2
  inputs:
    addSpnToEnvironment: true   # token issued per-run via federated credential
# AWS equivalent: aws-actions/assume-role with oidc token — no stored secrets
```

And the audit you should run quarterly:

```bash
aws iam get-credential-report | jq '.[] | select(.access_key_1_active=="true")'
# keys older than 90d? users with passwords but no MFA? policies with Action: "*"?
```

## Common Mistakes

- Long-lived keys in pipelines, `.env` files, and laptops — the #1 breach origin
- Wildcard actions/resources "to unblock the demo" — policies only ever grow unless audited
- Humans with standing admin instead of JIT elevation
- Trusting the VPC: "it's in our network" — IAM is the control plane; network is one layer
- One account for everything — no hard boundary between prod and experiments

## Mental Model

> Cloud IAM is the **badge system for a building where every doorknob is an API**: every action requires a scanned badge checked against a per-door rulebook. Roles are *temporary vests* rather than permanent badges — and a leaked master key isn't a stolen badge, it's handing someone the badge-printing machine (Tier 0).

## Remember This

1. Every cloud API call is IAM-evaluated: principal → action → resource → condition
2. Identity replaced perimeter as the trust model; IAM failures = most cloud breaches
3. Federated workload identity (OIDC) > stored credentials, always
4. Least privilege is a process (deny-all + observed-access + audit), not a one-time setup
5. Tier 0 (IAM admin) is the crown jewel: SSO, MFA, JIT elevation
6. Account/subscription boundaries are the strongest walls — use them

## One Sentence

Cloud IAM authorizes every API call by matching principals to actions on resources under conditions — making identity, not network location, the security boundary of the cloud.

## Knowledge Check

1. Why is OIDC federation strictly better than an access key in pipeline variables?
2. What does Tier-0 compromise mean that Tier-2 doesn't?
3. Design the least-privilege policy for a pipeline that builds images and pushes to one registry.
4. Why are unused permissions a security problem, not just hygiene?

## Further Reading

- AWS IAM docs / Azure RBAC fundamentals (model is portable)
- [IAM Access Analyzer policy generation](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-policy-generation.html)
- Preview: [OAuth/OIDC deep-dive](../security/oauth-oidc-saml.md) in the Security phase

---

**← Previous:** [VPC / VNet](vpc-vnet.md)
**Next:** [Compute Options](compute.md) →
**Related:** [RBAC & ABAC](../security/rbac-abac.md) · [Zero Trust](../security/zero-trust.md)
