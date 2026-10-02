# VPC / VNet

## What Is It?

A **Virtual Private Cloud** (AWS VPC / Azure VNet / GCP VPC) is *your private network inside one cloud region*: an address space (CIDR) you control, subdivided into subnets, with routing, firewalls, and gates to the outside — implemented in **software** (SDN) on the provider's fabric.

It's the IP/subnet/routing knowledge from the Networking phase, productized:

```text
VPC 10.0.0.0/16 (region: West Europe)
 ├── subnet 10.0.1.0/24 (AZ-a, public)  — LB entry points
 ├── subnet 10.0.2.0/24 (AZ-b, public)
 ├── subnet 10.0.11.0/24 (AZ-a, private) — app tier
 └── subnet 10.0.21.0/24 (AZ-a, data)    — DB tier, no egress
Route tables · Security Groups · NAT gateway · Internet gateway
```

## Why Does It Exist?

Multitenancy question: how do 10,000 customers share one physical network safely? Answer: give each a **software-defined slice** — isolated at the routing layer, configurable by API. The VPC is why two companies' workloads can sit on the same switch and never see each other's packets.

For you, it exists because **network architecture becomes versionable design**: tiers, zones, and trust boundaries drawn in config (Terraform), not in patch panels.

## Layer 1 — Simple Explanation

The VPC is your **company campus inside a rented city**: your fence line (CIDR), internal roads (route tables), buildings by function with separate locks (subnets + security groups), one guarded main gate (internet gateway/NAT), and private tunnels to your other campuses (peering/VPN).

"Public subnet" = building with a street entrance. "Private subnet" = building reachable only from inside the campus.

## Layer 2 — Engineer's View

**Subnet design — the tiering pattern (defense in depth):**

```text
public tier:   LBs, bastions        ← internet-reachable, hardened, nothing else
app tier:      services              ← reachable from LB tier only
data tier:     DBs, caches           ← reachable from app tier, on precise ports
                                             (+ often: no route to internet at all)
```

Each boundary = Security Group rules (stateful firewalls). Compromise of the LB tier should still face two locked doors before reaching the database.

**The components map (memorize the five):**

| Component | Job |
|---|---|
| **Route table** | subnet's routing decisions (longest-prefix — IP page) |
| **Internet gateway** | region's door to the public internet |
| **NAT gateway** | private subnets' *outbound-only* door (patching etc.) — billed per GB, a classic surprise |
| **Security group** | stateful firewall attached to *instances/NICs* |
| **NACL** | stateless firewall at subnet level (order matters; return-path rules!) |

**Connectivity patterns between networks:**

| Pattern | For | Notes |
|---|---|---|
| Peering | VPC↔VPC, no overlap | non-transitive! |
| Transit gateway/hub | many VPCs | the hub-and-spoke enterprise standard |
| Private Link/Private Endpoint | service-specific private access | DBs/object stores without public hop |
| VPN / DirectConnect | on-prem hybrid | Firewalls/VPNs page |

**The non-negotiable planning rule:** address space is the *hardest thing to change* in a cloud estate (overlaps can never peer). Plan CIDRs centrally, generously, with room for a future region/VPC (e.g., 10.<env>.<region>.0/16 scheme) — the "we'll fix it later" of overlapping CIDRs is unfixable later.

**Egress control as an underrated security layer:** default VPCs allow all egress. Restricting it (allowlist via NAT + NACLs/proxy, or endpoint policies) is how exfiltration paths die — a Zero Trust-adjacent move most estates skip.

## Real-World Example (DevOps flavored)

Terraform as the medium (IaC preview):

```hcl
resource "aws_subnet" "data" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.21.0/24"
  availability_zone = "eu-west-1a"
  tags = { tier = "data" }
}
resource "aws_security_group" "db" {
  ingress { from_port = 5432  protocol = "tcp"
            security_groups = [aws_security_group.app.id] }  # only app tier
  egress  []                                              # data tier needs none
}
```

The code review question that matters: *which rule is broader than its tag implies?* ("app tier can reach everything on 8080" — why?)

## Common Mistakes

- Everything in one public subnet "temporarily" (2019) — still there
- Overlapping CIDRs blocking peering — permanent
- Security groups as archaeology (`0.0.0.0/0` rules from the PoC era)
- Forgetting NACL statelessness (the return-path trap from the Firewalls page)
- NAT gateway costs discovered by finance, not engineering

## Mental Model

> The VPC is your **walled campus in the provider's city**: subnets are purpose-built buildings with separate locks, route tables are the internal road signs, the internet gateway is the main gate, NAT the outbound-only delivery door. Because it's all software, your campus blueprint lives in Git — and a network redesign is a code review, not a cable pull.

## Remember This

1. VPC = your software-defined private network per region; subnets are its tiers
2. Design tiers public/app/data with least-privilege SG chains between them
3. Five components: route tables, IGW, NAT, SGs, NACLs — stateful vs stateless!
4. Peering is non-transitive; transit gateways scale; private endpoints beat public hops
5. CIDR planning is permanent — centralize it before it hurts
6. Egress restriction is the underused security layer

## One Sentence

A VPC is your private, software-defined network slice in a cloud region — where subnets, routing, and stateful firewalls become versionable design rather than physical infrastructure.

## Knowledge Check

1. Why can't a database in a private subnet fetch OS patches, and what are the two standard fixes?
2. Explain the tiering pattern's security logic — what does an attacker face after compromising a load balancer?
3. Why is CIDR planning considered irreversible in practice?
4. What breaks if a NACL allows inbound 443 but no ephemeral outbound return rule?

## Further Reading

- AWS VPC docs / Azure VNet concepts (identical mental model)
- Terraform registry VPC modules — read a reference implementation

---

**← Previous:** [Regions & Availability Zones](regions-az.md)
**Next:** [Cloud IAM](iam.md) →
**Related:** [IP, Subnets & Routing](../networking/ip-routing.md) · [Firewalls & VPNs](../networking/firewalls-vpns.md)
