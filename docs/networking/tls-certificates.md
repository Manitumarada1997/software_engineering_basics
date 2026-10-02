# TLS & Certificates

## What Is It?

**TLS** (Transport Layer Security) gives a TCP conversation three guarantees:

1. **Encryption** — eavesdroppers see noise
2. **Integrity** — tampering is detected
3. **Authentication** — the server (and optionally client) is who it claims to be

**Certificates** are the identity documents that make #3 work: a public key plus a name, **signed by someone both parties trust** (a Certificate Authority, CA).

## Why Does It Exist?

TCP delivers bytes reliably — but to *anyone on the path*. On the open internet, plaintext HTTP means your ISP, the coffee-shop WiFi, and every interposed box reads and can rewrite everything. TLS's problem was never secrecy alone; it was **trusting a stranger's key across an untrusted network**. Certificates + CAs solve the key-trust bootstrapping: you can't verify a stranger, but you can verify a chain back to an anchor your OS/browser already trusts.

## Layer 1 — Simple Explanation

The handshake, simplified:

```text
You → "hello, I speak these cipher suites"
Server → "here's my certificate (identity card signed by a CA)"
You → verify the signature chain to a CA you trust ✓
You → "here's a session key, encrypted with your public key"
Both → symmetric encryption for the rest of the conversation
```

Asymmetric crypto (slow, for identity + key exchange) bootstraps symmetric crypto (fast, for the data). Like exchanging house keys via a notarized introduction, then talking freely.

## Layer 2 — Engineer's View

**What a certificate actually contains:**

```bash
openssl x509 -in cert.pem -noout -text
# Subject: CN=api.shopeasy.io           ← the identity (name it's valid FOR)
# Issuer:  CN=R11, Let's Encrypt        ← who vouched
# Validity: notBefore / notAfter        ← the expiry that pages you
# SAN: DNS:api.shopeasy.io, DNS:*.shopeasy.io   ← modern identity field
# Key usage, extensions (EKU), signature algo
```

**The chain of trust:**

```text
leaf (api.shopeasy.io) → intermediate (R11) → root CA (in your OS trust store)
```

Servers must serve leaf **+ intermediates** — the #1 "works in browser, fails in curl/Java" bug (browsers cache/AIA-chase intermediates; strict clients don't).

**The operational certificate lifecycle:**

```text
generate key + CSR → CA issues (DNS-01/HTTP-01 challenge) → install (leaf+chain+key)
        → monitor expiry → rotate before expiry → repeat
```

The discipline you own: **automate everything** (cert-manager in K8s; ACME/Let's Encrypt for public; internal CA — Vault, step-ca — for private). Certificate *expiry* is the classic self-inflicted outage; manual renewal is how it happens.

**Handshake details worth knowing:**

- **TLS 1.3**: 1-RTT handshake, 0-RTT resumption, obsolete crypto removed — prefer it everywhere
- **SNI**: the client sends the *hostname* in the clear so servers can pick the right cert — why one IP can serve many certs, and why SNI is metadata leaking (ECH exists to fix)
- **mTLS**: *both* sides present certificates — the foundation of service meshes and zero-trust internal networks (Security phase)
- **Cipher suites / policies**: modern practice = TLS 1.2+ with sane defaults; old cipher fiddling is mostly obsolete — prefer policy presets (Mozilla SSL config generator)

**Debugging toolkit (memorize these three):**

```bash
openssl s_client -connect api.shopeasy.io:443 -servername api.shopeasy.io
#   → chain presented? verify return code? expiry?
curl -vI https://...                      # TLS layer in the trace
echo | openssl s_client -connect ... 2>/dev/null | openssl x509 -noout -dates
```

**Trust stores are local facts:** a CA is trusted because *this machine's* store says so (`/etc/ssl/certs`, the JVM cacerts — the source of "works in curl, fails in Java" #2). Private/internal TLS = distributing your internal CA to every client, and rotating it.

## Real-World Example (DevOps flavored)

Saturday 02:00, all internal calls failing: `x509: certificate has expired or is not yet valid`. The internal CA — installed by hand 3 years ago on every node — expired. Every leaf it signed fails validation even though *their* dates are fine. Recovery: new CA, distribute to trust stores everywhere (images, nodes, JVMs), re-issue leaves. Prevention: cert-manager + expiry alerts + *automated rotation of the CA's own lifecycle* — trust infrastructure is still infrastructure.

## Common Mistakes

- Manual renewal and no expiry monitoring (the outage you scheduled yourself)
- Missing intermediate in the served chain
- Internal CA distributed by hand — no story for rotation
- Clock skew on nodes breaking `notBefore`/`notAfter` validation
- Assuming HTTPS = secure app — TLS secures the pipe, not the payload's logic (injection still rides inside TLS)

## Mental Model

> TLS is the **armored briefcase** (encryption + integrity) with a **notarized passport** (certificate + CA chain) checked at the door. The passport expires, the notary can be compromised, and you must re-verify everyone periodically — automation isn't a luxury, it's the lifecycle.

## Remember This

1. TLS = encryption + integrity + authentication; certificates solve key-trust
2. Serve leaf + intermediates; SANs are the identity; chains anchor in local trust stores
3. Asymmetric handshake → symmetric session; TLS 1.3 preferred; SNI enables multi-cert IPs
4. mTLS = both directions — meshes and zero-trust are built on it
5. Automate issuance/rotation (ACME, cert-manager) — expiry is a self-inflicted outage
6. Trust is local: OS/JVM stores decide, which explains half of all "TLS works here, not there"

## One Sentence

TLS encrypts and authenticates connections using certificates — identity keys vouched for by a chain of trust — and running it reliably is a lifecycle automation problem more than a cryptography problem.

## Knowledge Check

1. Why does a cert chain to an expired intermediate fail even with a valid leaf?
2. "Works in the browser, fails in curl/Java" — give both TLS explanations.
3. What does SNI leak, and what replaced hand-shaking per-cipher fiddling as best practice?
4. Design the certificate story for a private K8s cluster: issuance, rotation, trust distribution.

## Further Reading

- [Let's Encrypt — how it works](https://letsencrypt.org/how-it-works/)
- [The Illustrated TLS Connection](https://tls12.xargs.org/) / [TLS 1.3](https://tls13.xargs.org/) — byte-by-byte
- Mozilla SSL Configuration Generator

---

**← Previous:** [HTTP & HTTPS](http-https.md)
**Next:** [Proxies & Load Balancers](proxies-load-balancers.md) →
**Related:** [PKI & Encryption](../security/pki-encryption.md) · [Zero Trust](../security/zero-trust.md)
