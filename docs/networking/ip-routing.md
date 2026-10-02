# IP, Subnets & Routing

## What Is It?

- **IP addresses** identify a network interface: IPv4 (32-bit: `10.0.4.12`) and IPv6 (128-bit: `fd00::4`).
- **Subnetting** (CIDR) divides address space into networks: `10.0.4.0/24` = 256 addresses in one network.
- **Routing** moves packets between networks, hop by hop, each router picking the longest matching prefix.

## Why Does It Exist?

IP's only job: **globally identify hosts and get a packet one hop closer to its destination** — without any global coordination per packet. Subnetting makes that scalable: routing decisions match *prefixes*, not individual hosts. Every router on earth doesn't need to know every machine; it needs a default direction plus specifics for its neighborhood.

## Layer 1 — Simple Explanation

- An IP address is a **postal address**: street (network) + house number (host)
- The subnet mask (`/24`) is the **city boundary**: how much of the address is "street"
- A route is a **delivery instruction**: "for ZIP codes starting 10.0.4, use that road"
- The **default route** (`0.0.0.0/0`) is the local post office: "everything else goes there, they'll know"

## Layer 2 — Engineer's View

**Reading CIDR fluently (the skill to internalize):**

```text
10.0.4.0/24      network portion: 24 bits → hosts 10.0.4.1 – 10.0.4.254
10.0.0.0/16      bigger street: 65,534 hosts
10.0.4.12/32     one specific machine
0.0.0.0/0        everything (default route)

Special blocks (RFC 1918):
10.0.0.0/8 · 172.16.0.0/12 · 192.168.0.0/16   → private, not routable on internet
169.254.0.0/16  link-local (failed DHCP)
127.0.0.0/8     loopback
```

**The routing decision, exactly as the kernel makes it:**

```bash
ip route get 8.8.8.8
# 8.8.8.8 via 10.0.0.1 dev eth0  src 10.0.0.5
```

1. Longest-prefix match across the route table
2. Found → send via that interface to that gateway (or directly if on-link)
3. Nothing matches → default route
4. No default → "network unreachable"

**Everything you operate decodes into this table:**

| Environment | The routes |
|---|---|
| Docker | `172.17.0.0/16 dev docker0` + MASQUERADE for default |
| K8s pod network | per-node `10.244.0.0/16` routes (or overlay/VXLAN) |
| VNet/subnet | the CIDR you typed creating it; route table attached |
| Peering/VPC links | routes to the other VPC's CIDR |
| "No internet from the pod" | trace which route won the longest-prefix race |

**The traceroute method (know what you're seeing):** packets with increasing TTLs; each router decrements, hits 0, returns ICMP Time Exceeded — you literally watch the hop list. The asterisks (filtered ICMP) are why "traceroute shows gaps" ≠ dead hops.

**IPv6 in one paragraph:** v4 exhaustion solved by 128-bit addresses; no NAT-by-design, SLAAC autoconfiguration, mandatory for modern clusters (dual-stack K8s). Operationally: same longest-prefix logic, `fd00::/8` for private, check `ip -6 route`.

## Real-World Example (DevOps flavored)

"Pod can't reach the database, VM can":

```bash
kubectl exec pod -- ip route          # default via 10.244.0.1
kubectl exec pod -- ip route get 10.5.2.9   # which interface really wins?
# → via 10.244.0.1 dev eth0 — the CNI/SNAT path
# VM: ip route get 10.5.2.9 → dev eth1, direct route to db subnet
# Diagnosis: the pod's SNAT or the node's route misses 10.5.0.0/16;
# fix the route table / CNI config — not the app, not DNS
```

## Common Mistakes

- Overlapping CIDRs between VNet and on-prem (or between two peered VNets) — the permanent classic; plan address space like the scarce resource it is
- Confusing "ping works" (ICMP allowed) with "route + port open"
- Adding more-specific routes without realizing they now win the longest-prefix race
- Public IPs on private workloads because subnet planning was skipped

## Mental Model

> Routing is the **postal system's sorting hierarchy**: your mailbox (interface), the local branch (subnet), regional hubs (aggregated prefixes), and the international exchange (default route). Each hub reads only the ZIP prefix and forwards — no hub knows every house, and that's exactly why it scales.

## Remember This

1. Address = network prefix + host; `/n` says where the split is
2. Routing = longest-prefix match per hop; default route is the fallback
3. RFC 1918 blocks are private; overlapping CIDRs cause the worst outages
4. `ip route get X` is the kernel's actual answer — use it in every triage
5. Docker/K8s/cloud networking = managed routes over this exact logic

## One Sentence

IP addressing and CIDR subnets give every interface a location, and routing moves packets hop by hop using longest-prefix matching — the mechanism underlying every VNet, CNI, and Docker bridge you configure.

## Knowledge Check

1. Which route wins for `10.5.2.9`: `10.0.0.0/8`, `10.5.0.0/16`, or default — and why?
2. Why do overlapping VNet CIDRs cause outages that even a fix can't cleanly undo?
3. What does `ip route get` tell you that `ping` cannot?
4. Why is `169.254.x.x` on your NIC a diagnostic finding?

## Further Reading

- `man 8 ip-route`
- *Computer Networking: A Top-Down Approach* — ch. 4
- RFC 1918, RFC 4632 (CIDR)

---

**← Previous:** [OSI & TCP/IP](osi-tcpip.md)
**Next:** [DNS, DHCP & NAT](dns-dhcp-nat.md) →
**Related:** [Linux Networking](../linux/networking.md) · [VPC / VNet](../cloud/vpc-vnet.md)
