# PKI & Encryption

## What Is It?

- **Encryption** — the primitives: **symmetric** (one shared key: AES — fast, key-distribution problem) and **asymmetric** (keypair: RSA/ECC — slow, solves distribution and signing)
- **PKI** (Public Key Infrastructure) — the *system* that makes public keys trustworthy: CAs, certificates, revocation, lifecycle (the TLS page's trust chains, generalized)

## Why Does It Exist?

Two hard problems the primitives alone don't solve:

1. **Key distribution**: how do two strangers agree on a secret over a hostile wire? Asymmetric crypto *is* the answer (TLS handshake — you know it), but it spawns the follow-up: **whose public key is real?** → PKI (certificates = public keys with vouched identity)
2. **Key lifecycle at org scale**: generation, distribution, rotation, revocation, recovery — for humans, services, pipelines, clusters. Done manually, it rots (the TLS page's 3 AM expiry incident) → automation is the actual product (cert-manager, Vault, step-ca)

## Layer 1 — Simple Explanation

- **Symmetric** = the **house door key**: one key, fast, but you must get a copy to each family member securely
- **Asymmetric** = the **mailbox**: anyone drops mail through the slot (public key); only you have the key to open it (private key)
- **PKI** = the **notary system** proving the mailbox on Main Street really is yours — plus the county office for revoking notarized fakes

## Layer 2 — Engineer's View

**The four jobs of crypto (know which you need):**

| Job | Mechanism | Where you meet it |
|---|---|---|
| Confidentiality | encryption | TLS, disk, envelope encryption |
| **Integrity** | hash (SHA-256) | checksums, image digests, Git objects |
| **Authenticity** | signature (private key signs, public verifies) | JWTs, cosign, git signed commits |
| Non-repudiation | signature + identity binding | audit-grade signing (Sigstore) |

Note how much of your stack is *signatures*: Git's Merkle tree, image digests, OIDC JWTs — hashes and signing are the course's invisible connective tissue.

**Envelope encryption — how data at rest actually works:**

```text
data --data key (AES)--> ciphertext
data key --KEK (key-encryption-key, in KMS/HSM)--> wrapped
why: re-wrapping one KEK rotates everything without re-encrypting terabytes
```

Cloud KMS = the KEK custodian; rotating the KEK ≠ re-encrypting data. (Answer to the classic interview question.)

**PKI operations — the checklist you own:**

- **Hierarchy**: root CA (offline, ceremony-guarded) → intermediates → leaf certs — compromise containment by design
- **Short-lived leaves**: 90 days → hours; rotation stops being an event and becomes background (cert-manager, SPIFFE/SVIDs)
- **Revocation** reality: CRLs/OCSP are weak in practice — *short lifetimes* are the real revocation
- **Trust stores are local facts** (TLS page): OS/JVM/node — internal CAs need distribution machinery
- **Your PKI estate today**: web PKI (Let's Encrypt), cloud PKIs (managed DB TLS), cluster PKIs (K8s CA, mesh SPIFFE), pipeline signing (Sigstore) — *know how many CAs you actually run*

**Key management hygiene (the failures that make news):**

- Private keys never in repos/images (the Layers page's forensics), never world-readable
- HSM/KMS-backed generation where it matters; export controls on KEKs
- Rotation as routine (canary your own expiries: `step certificate inspect`); break-glass documented

## Real-World Example (DevOps flavored)

The internal-PKI build (ShopEasy's, and probably your future):

```text
step-ca (or Vault PKI) as intermediate CA, offline root
cert-manager ClusterIssuers:      ACME (public) + internal CA (private workloads)
SPIFFE/SPIRE:                     workload identity certs, hours-lifetime, auto-rotated
cosign:                           image + SBOM signing at CI; admission verifies (Registries page)
KMS envelope:                     app data keys wrapped by cloud KEK, rotation yearly
Expiry monitoring:                metrics on cert_not_after across the fleet (never again 3 AM)
```

One infrastructure, four consumers (TLS, workload identity, artifact signing, data encryption) — that's PKI as a *platform service*.

## Common Mistakes

- Treating rotation as an annual human event — the 3 AM outage you scheduled
- Long-lived certificates "to reduce churn" — revocation in practice *is* expiry
- Keys in CI variables / baked into images (found by scanners, eventually by attackers)
- Encrypting but not authenticating (or vice versa) — both properties required, separately
- Not knowing your CA inventory — shadow PKIs discovered during outages

## Mental Model

> Symmetric crypto is the **fast house key**; asymmetric the **mailbox pair**; PKI the **notary system** making mailboxes provable — and modern practice makes notarization *short-lived and auto-renewed*, because a badge that expires hourly needs no revocation ceremony.

## Remember This

1. Symmetric = fast bulk; asymmetric = distribution + signatures; TLS combines both (handshake)
2. Four crypto jobs: confidentiality, integrity (hash), authenticity (signature), non-repudiation
3. Envelope encryption: rotate the KEK, not the terabytes
4. Short-lived certs > revocation machinery; automation (cert-manager/SPIFFE) is the product
5. Trust stores are local; know your CA inventory
6. Signatures already saturate your stack (Git, images, JWTs) — you're operating PKI knowingly now

## One Sentence

Encryption provides the symmetric and asymmetric primitives, and PKI is the trust system binding public keys to identities at organizational scale — with short-lived certificates and automated rotation as its only operationally sane form.

## Knowledge Check

1. Why does envelope encryption make rotation cheap? Trace the two layers.
2. Why are short certificate lifetimes better than revocation lists?
3. Which of the four crypto jobs does a Git commit hash serve? An image digest? A cosign signature?
4. List every CA your current estate trusts, on any one machine.

## Further Reading

- [Cryptographie right-sized: "Serious Cryptography" — Jean-Philippe Aumasson](https://nostarch.com/serious-cryptography)
- smallstep blog — practical PKI/cert-management articles (best in class)

---

**← Previous:** [Zero Trust](zero-trust.md)
**Next:** [Secrets Management & Vault](secrets-vault.md) →
**Related:** [TLS & Certificates](../networking/tls-certificates.md) · [SBOM & Supply Chain](sbom-supply-chain.md)
