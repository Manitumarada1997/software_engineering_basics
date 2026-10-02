# Linux Networking

## What Is It?

The kernel's networking stack: how packets actually move through a Linux box — sockets, interfaces, routing tables, netfilter/nftables — **before** any cloud VNet or Kubernetes CNI dresses it up.

*(Protocol theory — TCP/IP, ports, DNS — gets its own phase next. This page is the Linux machine's view.)*

## Why Does It Matter to a DevOps Engineer?

Because "network problem" on a server, in a container, or in Kubernetes **bottoms out here**:

- Container networking = a veth pair + a bridge + netfilter rules (this page + namespaces)
- NodePort/iptables, service meshes, hostNetwork pods — all netfilter under the hood
- When connectivity is broken and dashboards shrug, these tools are the ground truth

## Layer 1 — Simple Explanation

The kernel's network stack is the **building's postal system**:

- **Sockets** are mail slots apps open (IP + port = apartment + slot number)
- **Interfaces** (eth0, lo, docker0) are the loading docks
- **Routing table** decides which dock each letter leaves from
- **netfilter/iptables** is the mailroom security desk: forwards, blocks, rewrites (NAT) envelopes

## Layer 2 — Engineer's View

**The toolbox — the six commands that answer 90% of "is it the network?":**

| Command | Answers |
|---|---|
| `ss -tulpn` | What's listening, on which port, by which PID? (replaced netstat) |
| `ip addr` / `ip route` | What interfaces and routes exist? (replaced ifconfig/route) |
| `ip -s link` | Interface errors/drops — the "bad cable" check |
| `conntrack -L` | The kernel's live connection table |
| `tcpdump -i any port 443` | What's actually on the wire |
| `dig`/`resolvectl` | Names → IPs (DNS phase) |

**The packet's path through the box (simplified):**

```text
NIC → driver → netfilter (PREROUTING: DNAT) → routing decision
  → local socket (INPUT chain) → app
  → app → socket → netfilter (FORWARD/OUTPUT: SNAT/MASQUERADE, filter) → NIC
```

- **iptables = tables of chains of rules**; you mostly live in `filter` (ACCEPT/DROP) and `nat` (rewrite addresses)
- **Docker's magic, decoded:** container IP 172.17.0.2 → outside world sees node IP because of the `MASQUERADE` rule on the NAT table — that's all "container with outbound internet" ever was
- Kubernetes kube-proxy in iptables mode: the *Service VIP* is hundreds of NAT rules doing random load balancing — this is why huge Services historically hurt iptables performance (IPVS mode exists for this)

**Nameserver resolution is the other half:** `/etc/resolv.conf`, `/etc/hosts`, NSS. "The network is down" is very often "DNS is down" — the discipline is always: check both by IP (`curl 10.0.4.12`) and by name (`curl payments.internal`).

**Virtual interfaces you operate:**

| Interface | What it is |
|---|---|
| `lo` | Loopback — 127.0.0.1, never leaves the kernel |
| `eth0` | The real NIC |
| `docker0` / `cni0` | Linux bridge connecting container veths |
| `veth` pair | A virtual cable: one end in the container's namespace, one on the bridge |

## Real-World Example (DevOps fluent)

"The service can't reach the database":

```bash
ss -tlnp | grep 5432          # is postgres even listening? on 127.0.0.1 or 0.0.0.0?
ip route get 10.0.4.12        # which interface/route would it take?
curl -m 2 telnet://10.0.4.12:5432   # TCP reachable at all?
dig db.internal +short        # does the name resolve — to what?
sudo tcpdump -i any -n port 5432 & # what comes back: SYN with no SYN-ACK? RST? ICMP?
```

The pattern SYN-with-no-reply = firewall/drop; connection-refused (RST) = reachable, nothing listening; works-by-IP-fails-by-name = DNS. Each has a different owner and fix — this is the triage tree.

## Common Mistakes

- Debugging DNS problems as network problems (or vice versa) without testing both IP and name
- App binding to `127.0.0.1` in a container — unreachable from outside its own netns
- Deep iptables edits by hand on a kube node — fight the CNI/kube-proxy and lose
- Assuming iptables rules persist across reboots without netfilter-persistence
- Ignoring `ip -s link` errors (dropped packets at the NIC = hardware/VM issue, not app)

## Mental Model

> The kernel network stack is a **postal sorting facility**: sockets are mail slots, interfaces are loading docks, routing picks the dock, and netfilter is the security desk that forwards, rejects, or *rewrites addresses* (NAT) — Docker and Kubernetes services are just pre-written rulebooks for that desk.

## Remember This

1. Sockets, interfaces, routes, netfilter — the four layers of the box's network truth
2. iptables/nftables: NAT (rewrite) + filter (allow/deny) — Docker/K8s are rule generators for it
3. veth + bridge = container networking's physical layer
4. Always test IP and name separately — DNS failures masquerade as network failures
5. SYN-no-reply vs RST vs name-resolution — the connectivity triage tree
6. `ss`, `ip`, `conntrack`, `tcpdump` are the ground truth instruments

## One Sentence

Linux networking moves packets through interfaces, routing tables, and netfilter rules — the machinery that container bridges, Docker NAT, and Kubernetes Services are all configured on top of.

## Knowledge Check

1. Decode what Docker's MASQUERADE rule does for container outbound traffic.
2. curl by IP works, by name fails. Which subsystem, which files/tools?
3. What's the difference (packet-wise) between "connection timed out" and "connection refused"?
4. How does a Kubernetes Service VIP become real packets on a node (iptables mode)?

## Further Reading

- `man 8 ip`, `man 8 iptables-extensions`
- [WireGuard-esque intro: "Linux networking explained"](https://github.com/goldshtn/linux-tracing-workshop) / Julia Evans' networking zines
- iptables-tutorial (frozentux) — the classic

---

**← Previous:** [Filesystems & Permissions](filesystems-permissions.md)
**Next:** [systemd](systemd.md) →
**Related:** [Namespaces](namespaces.md) · [Networking (protocols)](../networking/osi-tcpip.md)
