# Firewalls & VPNs

## What Is It?

- A **firewall** filters traffic by policy: 5-tuple (source IP, source port, destination IP, destination port, protocol) plus state — *allow/deny at the network boundary*. Linux-native form: netfilter/iptables/nftables (Linux Networking page); network form: Security Groups, cloud ACLs, enterprise appliances.
- A **VPN** builds an encrypted tunnel across an untrusted network so distant networks/hosts behave as if directly connected — * confidentiality + reachability*.

## Why Does They Exist?

Complementary answers to an untrusted middle:

- **Firewall** = the perimeter rule: everything not explicitly allowed is denied (default-deny). The network's access-control layer, complementing file permissions and IAM.
- **VPN** = the private wire: encrypts and encapsulates so the public internet becomes plumbing — site-to-site (offices↔datacenter) or client-to-site (you↔the VNet).

## Layer 1 — Simple Explanation

- **Firewall**: the **doorman with a guest list** — checks who (source), going where (destination/port), with what (protocol), and remembers who's already inside (stateful: reply traffic allowed automatically)
- **VPN**: a **guarded underground tunnel between buildings** — nobody on the street sees or joins what travels through; both buildings feel adjacent

## Layer 2 — Engineer's View

**Stateful vs stateless — the distinction that explains cloud networking:**

| | Stateless | Stateful |
|---|---|---|
| Decides | each packet alone, by rule | track connections; allow established/related replies |
| Rules needed | allow BOTH directions explicitly | allow initiator direction only |
| Example | cloud NACLs, classic ACLs | iptables, cloud Security Groups |

The classic trap: stateless NACL with allow-inbound-443 but no ephemeral outbound return rule — SYNs arrive, replies die.

**Security Groups (cloud firewalls), operationally:**

- Instance/subnet-level, stateful, default-deny inbound
- Applied to *instances* not addresses — rules travel with the workload
- Anti-patterns: `0.0.0.0/0` on SSH/RDP, SG sprawl where nobody can prove the minimum surface — SG audits are a recurring compliance task (Security phase)

**Firewall design principles:**

1. Default deny; allow by exception with justification (tied to service owner)
2. Least privilege in *network* form: app subnet → DB subnet on 5432 only, not "any"
3. Zones/tiers: public / app / data — trust flows inward through gates
4. Log and review — a firewall without logging is a bouncer without a guest list to audit

**Kubernetes NetworkPolicies are firewalls for pods:** default-deny per namespace, allow by label selectors — same 5-tuple logic, enforced by CNI. (Full treatment in the K8s phase.)

**VPN taxonomy:**

| Type | Use | You've met |
|---|---|---|
| Site-to-site (IPsec, WireGuard) | VNet ↔ on-prem / VNet ↔ VNet | hybrid cloud |
| Client-to-site | workstation ↔ VNet | OpenVPN/WireGuard clients |
| "VPN" for privacy | you ↔ commercial exit | consumer product — different threat model |

**Tunnel mechanics you operate:**

- Encapsulation overhead shrinks effective **MTU** — the "ping works, TLS hangs" pathology from the OSI page, *caused here*
- **Encryption is CPU** and a single tunnel endpoint is often the throughput ceiling — measure before blaming the app
- **IPsec vs WireGuard**: IPsec = the enterprise standard (complex, NAT-traversal quirks); WireGuard = tiny codebase, simple keys, fast — increasingly the internal choice
- Cloud-native successors: peering/Private Link/Private Endpoint often *replace* VPNs for cloud-to-cloud — fewer moving parts, no tunnel to babysit

**Zero Trust's critique (bridge to Security phase):** the traditional VPN model grants *network-level* trust — connect to the flat corporate network, reach everything. Zero Trust removes network locality as a trust signal: per-request authentication regardless of location. VPNs don't die; their *trust meaning* changes from "inside = trusted" to "encrypted reachability only."

## Real-World Example (DevOps flavored)

```bash
# "pod can't reach RDS" triage
kubectl exec pod -- nc -zv -w 2 db.x.us-east-1.rds.amazonaws.com 5432
# timeout → check: NetworkPolicy (pod egress), SG on RDS (allows VPC CIDR? the pod SNAT range?),
# NACL statelessness on the subnet, and route tables
# refused → SG/NACL passed; RDS itself says no (or wrong port)
aws ec2 describe-security-groups ... # the audit that finds 0.0.0.0/0 on 22 from 2019
```

## Common Mistakes

- Stateless NACLs missing return-path rules (the silent half-broken subnet)
- Security Groups as archaeology: inherited rules nobody dares delete — schedule audits
- VPN as implicit security — encryption ≠ authorization; the flat network behind it is the vulnerability
- Ignoring tunnel MTU until "big requests fail mysteriously"
- Firewall rules without owner/justification metadata — untouchable rules accumulate

## Mental Model

> Firewalls are **doormen with guest lists at every door** (stateful ones remember who came in). VPNs are **underground tunnels between buildings** — private passage, but everyone in the tunnel can still only enter rooms they hold keys to. Zero Trust is the new policy: *tunnels for privacy, keys for every door, location means nothing.*

## Remember This

1. Firewalls = default-deny 5-tuple filtering; stateful ones allow reply traffic automatically
2. Cloud: Security Groups (stateful, attach to workloads) vs NACLs (stateless, subnet-wide)
3. NetworkPolicies are pod-level firewalls — same concept, CNI-enforced
4. VPNs encrypt and bridge networks; MTU overhead and endpoint throughput are real
5. WireGuard > IPsec for simplicity; peering/Private Link often replaces tunnels
6. "Inside the VPN" must never mean "trusted" — that's Zero Trust's whole point

## One Sentence

Firewalls enforce default-deny network access with stateful 5-tuple rules, and VPNs provide encrypted tunnels that make separated networks adjacent — together the perimeter layer that Zero Trust redefines but does not eliminate.

## Knowledge Check

1. Why do SYNs arrive but no handshake complete under a misconfigured stateless NACL?
2. Your pod-to-RDS connection times out. List the four network layers to check in order.
3. What did Zero Trust change about what a VPN *means*?
4. Why does a VPN tunnel break large requests but not pings?

## Further Reading

- `man 8 nft`, AWS/Azure SG vs NACL docs (concept-identical across clouds)
- [WireGuard paper](https://www.wireguard.com/papers/wireguard.pdf) — readable and short
- NIST 800-207 (Zero Trust) — preview of the Security phase

---

**← Previous:** [Proxies & Load Balancers](proxies-load-balancers.md)
**Next:** [CDN](cdn.md) →
**Related:** [Network Security](../security/network-security.md) · [Zero Trust](../security/zero-trust.md)
