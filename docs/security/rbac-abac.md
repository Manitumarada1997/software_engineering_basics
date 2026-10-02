# RBAC & ABAC

## What Is It?

Two models of **authorization** (the "what may you do?" half of the previous page):

- **RBAC** (Role-Based): permissions attach to **roles**; users/workloads get roles
- **ABAC** (Attribute-Based): decisions evaluate **attributes** of subject, resource, and context (time, IP, labels, tags)

You already know RBAC twice over — cloud IAM (`roleRef`) and K8s (`Role`/`RoleBinding`). This page makes the model itself — and its scaling limits — explicit.

## Why Does They Exist?

Direct per-user permission lists (ACLs) die at scale: 500 users × 5,000 resources = millions of entries, each a review burden. Both models solve it by **grouping the decision inputs**:

```text
ACL:  grant(user=alice, resource=db7, action=read)          ← per-pair entries
RBAC: grant(role=db-reader, resource=dbs, action=read)       ← grouped by function
       alice → db-reader
ABAC: grant(subject.department=payments → resources tagged payments-*, hours 9-17)
                                                              ← grouped by properties
```

## Layer 1 — Simple Explanation

- **RBAC** = **job badges**: "Welder" opens the welding bays — whoever holds the badge, wherever the bays are. Adding a welder = handing over one badge
- **ABAC** = **context-aware doors**: "opens for employees of the payments department, during work hours, from office IPs" — no badges; doors evaluate *properties of the moment*

## Layer 2 — Engineer's View

**RBAC — strengths and the wall you hit:**

| Strength | The wall |
|---|---|
| Auditable: enumerate role → permissions | Role explosion: per-team × per-env permutations multiply |
| Simple mental model | Coarse: same role = same power everywhere it's bound |
| Maps to org structure | Org changes lag role redesign |

The role-explosion cure is *hierarchies and composition* (K8s' aggregation, Azure's PIM JIT elevation) — not more roles.

**ABAC — strengths and the wall:**

| Strength | The wall |
|---|---|
| Expressive: context-sensitive (IP, time, labels) | Opaque: "can Alice read X?" requires evaluating the whole policy engine |
| Scales with attributes, not role count | Debugging: why was access denied? (policy simulators exist for a reason) |

**Reality: hybrid everywhere.** Cloud IAM *is* RBAC-with-ABAC-conditions (`aws:SourceIp`, tags) — the IAM page's `Condition` field was ABAC all along. K8p RBAC + admission (namespace labels) = same hybrid.

**The design rules (model-independent):**

1. **Deny by default**; permissions are added with justification and owner
2. **Bind to groups/identities, not users** — people leave, groups persist
3. **Separate the power tiers**: who grants roles is higher-stakes than any role (IAM page's Tier-0)
4. **JIT over standing**: elevation with expiry beats permanent privilege
5. **Test authorization**: `kubectl auth can-i`, IAM policy simulator — authZ is testable code (policy-as-code, later page)

**ReBAC** (relationship-based, one line): Google Zanzibar-style — "the owner of a doc's *parent* folder can..." — drives shared-drive style systems; the third model, for when relationships themselves are the permission source.

## Real-World Example (DevOps flavored)

The audit that pays for itself (run it quarterly):

```bash
# cloud: unused permissions = attacker inventory
aws iam get-account-authorization-details   # full role graph → diff vs access logs
# who can do Tier-0 actions?
aws iam simulate-principal-policy --policy-source-arn ... --action-names iam:*
# k8s:
kubectl auth can-i --as=system:serviceaccount:payments:ci create pods -n payments
kubectl get rolebindings -A | grep -v namespace | wc -l   # binding sprawl?
```

Finding: the CI role that `create pods` in all namespaces (it needed one) — the pod-creation escalation from the RBAC page, now auditable as data.

## Common Mistakes

- Roles as考古 layers — nobody knows what `legacy-admin-full` does but everyone holds it
- Role explosion from copy-paste ("db-reader-prod-except-friday")
- ABAC policies without simulator access — denials nobody can explain
- User-direct bindings — leaving employees as permanent ghosts in policies
- Forgetting: **who administers the model** outranks everything the model grants

## Mental Model

> RBAC hands out **job badges** (simple, auditable, prone to badge-cabinet sprawl); ABAC builds **context-aware doors** (flexible, opaque). Real systems bolt both: badges whose power depends on where and when they're swiped. The admin office that prints badges is the true crown jewel — guard it above every door.

## Remember This

1. RBAC groups permissions by role; ABAC decides on attributes/context — hybrids dominate
2. RBAC wall: role explosion; ABAC wall: explainability — know both failure modes
3. Deny-default, group bindings, JIT elevation, separated admin tier
4. Authorization is testable code — simulate/verify, don't assume
5. Unused permissions are attacker inventory — audit quarterly
6. ReBAC exists for relationship-driven permissions (Zanzibar lineage)

## One Sentence

RBAC authorizes through roles and ABAC through contextual attributes, and production systems blend both — with the universal disciplines of deny-by-default, least privilege, and guarding whoever administers the model.

## Knowledge Check

1. Why do role hierarchies/JIT elevation cure role explosion where more roles don't?
2. Which ABAC fields have you already used in cloud IAM conditions?
3. Enumerate your access-debugging tools per system (K8s, AWS/Azure) — can you answer "why was this denied?"
4. Why is "who can grant roles" the highest-value audit question?

## Further Reading

- NIST RBAC standard; [Google Zanzibar paper](https://research.google/pubs/pub48990/) (ReBAC)
- Next: [OAuth 2.0, OIDC & SAML](oauth-oidc-saml.md)

---

**← Previous:** [CIA Triad & AuthN/AuthZ](cia-authn-authz.md)
**Next:** [OAuth 2.0, OIDC & SAML](oauth-oidc-saml.md) →
**Related:** [Cloud IAM](../cloud/iam.md) · [Namespaces, RBAC & Policies](../kubernetes/rbac-policies.md)
