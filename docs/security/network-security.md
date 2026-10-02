# Network Security

## What Is It?

The defensive layer you've already studied piece-by-piece, now assembled as a *system*: firewalls (default-deny, stateful), segmentation (zones/tiers/micro), encryption in transit (TLS/mTLS), ingress/egress control, and DDoS absorption — protecting **availability and reachability** as security properties.

*(Mechanics live in the Networking phase pages; this page is the doctrine.)*

## Why Does It Exist?

Because identity-based controls (IAM, AuthN/Z) answer *whether* a connection is allowed — network controls shape *what can even attempt*, absorb volumetric attacks identity can't see, and contain the blast radius of everything else failing:

```text
Layered defense:
  DDoS absorbed at the edge (CDN/LB)      — no identity involved, just volume
  Reachability minimized by segmentation   — SSRF lands in a lobby (Zero Trust page)
  mTLS authenticates what firewalls pass   — defense in depth, not either/or
```

## Layer 1 — Simple Explanation

The doctrine in one image — a **castle built by a pessimist**:

```text
Moat + edge walls       (edge: CDN/WAF/DDoS, cloud edge firewalls)
Inner walls per district (segmentation: VPC tiers, subnets, NetworkPolicies)
Every courier escorted   (encryption in transit: TLS/mTLS)
Guards at the gates      (ingress control: LBs, API gateways, ZTNA)
Searched on the way out (egress control: NAT allowlists, proxies)
```

Each layer assumes the previous one eventually fails.

## Layer 2 — Engineer's View

**The doctrine's five pillars, with your existing vocabulary:**

| Pillar | Doctrine | Implementation pieces |
|---|---|---|
| **Default-deny** | allow by exception, owned + justified | SGs/NACLs, NetworkPolicies, egress allowlists |
| **Segmentation** | smallest blast radius affordable | VPC tiers (Cloud page), namespaces + policies (K8s page), microsegmentation |
| **Encryption in transit** | assume the wire is hostile | TLS everywhere (internal too!), mTLS mesh, no cleartext protocols |
| **Ingress control** | one door per exposed service | LB → WAF → gateway; private endpoints for internal consumers |
| **Egress control** | data walks out the same guarded doors | NAT allowlists, egress proxies, DNS filtering |

**Egress — the neglected half:** ingress gets the attention; exfiltration leaves via *outbound*. Controls: default-deny egress + allowlists per workload (NetworkPolicy egress rules), forced through inspecting proxies, and DNS as a channel worth filtering (tunneling over DNS is a classic). The Zero-Trust-adjacent move most estates skip (VPC page flagged it).

**WAF — where app-layer meets network:** rule sets (OWASP CRS) at the edge/gateway filtering injection, traversal, anomalous inputs — complementary to (never replacing) secure code: WAF is a seatbelt, not a driving license.

**DDoS — availability as a security war:** volumetric (absorb at CDN/edge — never at your origin), protocol (SYN floods — LB/syncookies), application (slowloris — timeouts, rate limits at the gateway page). Doctrine: **capacity you rent** (CDN/cloud scrubbing) + **rate limiting you configure** (gateway page's token buckets) + **origin locked to the edge** (CDN page).

**Internal traffic — the modern heresy that's now doctrine:** mTLS for *all* east-west traffic (mesh/SPIFFE — Zero Trust page), because "the internal network" is a fiction cloud dissolved. Cleartext internal HTTP = the finding every audit returns.

## Real-World Example (DevOps flavored)

The containment proof, staged (this is the measurable value of doctrine):

```text
Compromised pod, before doctrine: flat network → port-scans the VPC → finds DB:5432
                        open to the app tier → credentials in old config → full breach
Same pod, after:        egress default-deny (can't scan); DNS filtered; DB reachable
                        only via mTLS'd service identity; no ambient creds (Vault page)
                        → attacker holds a container and nothing else
```

Network security didn't *prevent* the compromise — it turned a breach into an incident.

## Common Mistakes

- Fortress-front, trampoline-back: perfect ingress, unrestricted egress
- Flat internal networks because "it's private" — the dissolved-perimeter fiction
- Security groups as unmaintained archaeology (Firewalls page) — sprawl *is* misconfiguration
- WAF as a substitute for fixing code — the seatbelt fallacy
- Segmentation everywhere (cost, complexity) or nowhere (blast radius) — size it to asset value

## Mental Model

> Network security is **castle-building for pessimists**: edge walls (DDoS/WAF), district walls (segmentation), escorted couriers (TLS), guarded gates (ingress), searched exits (egress) — each layer designed on the assumption the previous one eventually fails, because it will.

## Remember This

1. Doctrine: default-deny, segmentation, encrypted transit, ingress + *egress* control
2. Network controls contain blast radius and absorb volume; identity controls decide access — you need both
3. Egress is the neglected half — exfiltration leaves outbound
4. Internal ≠ private: mTLS east-west is modern doctrine
5. DDoS: rent capacity at the edge + rate-limit at the gate + hide the origin
6. Segmentation sized to asset value — not maximal, never absent

## One Sentence

Network security shapes what can even attempt contact with your systems — default-deny boundaries, segmentation, encrypted transit, and guarded ingress and egress — so that when other controls fail, the attacker lands in a lobby instead of the vault.

## Knowledge Check

1. Why do identity controls and network controls not substitute for each other?
2. Design egress for a multi-tenant cluster: defaults, exceptions, DNS.
3. Which DDoS class does each counter address (CDN, LB, gateway rate limits)?
4. Why is cleartext internal HTTP an audit finding even "inside the VPC"?

## Further Reading

- OWASP CRS / WAF docs; the Networking phase (mechanics live there)
- Next: [Cloud Security](cloud-security.md)

---

**← Previous:** [Secrets Management & Vault](secrets-vault.md)
**Next:** [Cloud Security](cloud-security.md) →
**Related:** [Firewalls & VPNs](../networking/firewalls-vpns.md) · [Zero Trust](zero-trust.md)
