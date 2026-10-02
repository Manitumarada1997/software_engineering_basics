# OSI Model & TCP/IP

## What Is It?

Two layered models of "how bytes become a conversation between machines":

```text
OSI (7 layers, the vocabulary)     TCP/IP (4 layers, the reality)
7 Application   ─ HTTP, DNS, SSH   ── Application
6 Presentation  ─ TLS, encoding
5 Session       ─ sockets
4 Transport     ─ TCP / UDP        ── Transport   (ports, reliability)
3 Network       ─ IP, ICMP         ── Internet    (routing, addresses)
2 Data link     ─ Ethernet, ARP    ── Link        (MAC, local hop)
1 Physical      ─ cables, radio
```

OSI never shipped as designed, but its **numbers remain the industry's shared vocabulary** — "a layer 7 load balancer" or "that's a layer 4 problem" presumes this table.

## Why Does It Exist?

Layering is **division of labor with stable contracts**. Each layer solves one problem and exposes a guaranteed interface to the one above:

- Link: "I can move bytes to the next physical box"
- Network (IP): "I can move bytes to any box on earth — unreliably"
- Transport (TCP): "I can make that a reliable, ordered stream between programs"
- Application: "we agree on meaning (HTTP)"

Without layering, every app would re-solve routing, reliability, and medium access. With it, WiFi swapped for Ethernet never touches your HTTP code. This is the same reason your whole career composes: platform layers with contracts.

## Layer 1 — Simple Explanation

**Sending a package via a courier company:**

- Physical/Link: the truck and local roads (one hop)
- Network (IP): the nationwide address system — each depot forwards the package closer to the ZIP code (best effort; may lose it!)
- Transport (TCP): registered mail with tracking — numbered, receipted, re-sent if lost, delivered in order
- Application: the *letter inside* — its language and meaning (HTTP)

## Layer 2 — Engineer's View

**TCP vs UDP — the fundamental fork:**

| | TCP | UDP |
|---|---|---|
| Guarantee | ordered, reliable, byte-stream | none — datagrams |
| Cost | handshake, state, retransmission, congestion control | near-zero |
| Used by | HTTP/1-2, SSH, databases | DNS queries, video, games, QUIC's foundation |

Why both: reliability isn't free — for a DNS lookup (fits in one datagram), a handshake is waste; for a file transfer, it's essential. (QUIC/HTTP3 = UDP carrying a userspace TCP-replacement: layering evolution in action.)

**The TCP handshake and its operational fingerprints:**

```text
SYN → SYN-ACK → ACK        (connection established)
FIN/FIN-ACK or RST         (close / abort)
```

- "Connection refused" = RST — *reachable, nothing listening on that port*
- "Timed out" = no SYN-ACK — dropped packet, firewall, or wrong route (Linux Networking page's triage tree)
- SYN flood = exhausting the handshake queue — classic DoS

**Sockets — the address of a conversation:**

```text
(IP address, port) pair at layer 4
10.0.4.12:5432  ←→  10.0.7.3:49152
```

Everything listening (`ss -tlnp`), every NAT translation, every K8s Service target is expressed in these tuples.

**MTU and fragmentation:** 1500 bytes typical; VPNs/overlays (IPsec, WireGuard, VXLAN) shrink effective MTU — the cause of the classic "small pings work, HTTPS handshakes hang" pathology (big packets die, keepalives live). `ping -M do -s 1472` diagnoses.

**Where your abstractions live on the stack:**

| You operate | Layer(s) |
|---|---|
| K8s Service (ClusterIP) / kube-proxy | 3–4 (NAT on L4) |
| Ingress / NGINX | 7 (reverse proxy on L7) |
| Cloud NLB vs ALB | 4 vs 7 — the naming tells you |
| NetworkPolicy / firewalls | 3–4 (IP/port) |
| Service mesh sidecars | 4–7 (L4 proxy + L7 routing) |

## Real-World Example (DevOps flavored)

"Payments can't reach the DB over the VPN — ping works!": ICMP (layer 3) passes the tunnel; TCP 1521 packets exceed the tunnel's effective MTU; handshake bytes fit, data packets die. Fix: MSS clamping / MTU 1400. A pure layer-1-vs-4 diagnosis that costs days if you think "network works, app is broken."

## Common Mistakes

- Memorizing OSI as trivia instead of using it as a *localization tool* ("which layer fails?")
- Assuming reliability — UDP-based protocols need their own (or QUIC's)
- Blaming the app when small-packet protocols succeed but data flows fail — think MTU
- Forgetting that "load balancer" is ambiguous until you name the layer

## Mental Model

> The stack is a **courier relay**: trucks (link), the address system (IP), registered-mail service (TCP), and the letter's language (HTTP). Debugging = asking, with evidence, *which relay stage lost the package* — never guessing.

## Remember This

1. OSI = vocabulary; TCP/IP = implementation; know both tables
2. IP is best-effort; TCP adds ordered-reliable streams; UDP stays raw
3. Socket = (IP, port); refused (RST) vs timeout (silent drop) localizes faults
4. MTU shrinkage by tunnels breaks big packets only — the "ping works, TLS hangs" trap
5. Name the layer of every device you operate (NLB=L4, ALB/Ingress=L7)

## One Sentence

The layered network models split "bytes between machines" into stable contracts — link, network, transport, application — letting you localize any connectivity failure to exactly one layer.

## Knowledge Check

1. Distinguish "connection refused" from "timeout" at the packet level.
2. Why does ping succeed where HTTPS fails over a VPN?
3. What layer is a Kubernetes Service? An Ingress? Why the difference?
4. Why would DNS use UDP but your database refuse to?

## Further Reading

- *Computer Networking: A Top-Down Approach* — Kurose & Ross, ch. 3
- High Performance Browser Networking — Ilya Grigorik (free online), ch. 1–2

---

**← Previous:** [Lab: Container by Hand](../projects/lab-container-by-hand.md)
**Next:** [IP, Subnets & Routing](ip-routing.md) →
**Related:** [Linux Networking](../linux/networking.md) · [DNS, DHCP & NAT](dns-dhcp-nat.md)
