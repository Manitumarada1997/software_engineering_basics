# Docker & Images

## What Is It?

- **Docker** (2013): the tooling that made containers usable — image format, `docker build/run`, and a distribution model (registries). The product that turned a kernel feature into an industry.
- An **image** = ordered, read-only **filesystem layers** + metadata (entrypoint, env, exposed ports) — the executable artifact of the container world.

## Why Does It Exist?

Kernel features don't ship software. What Docker added was the **artifact story**:

```text
Dockerfile (recipe) → build (deterministic-ish) → image (artifact) → registry (warehouse) → any host runs it
```

Same pipeline shape as Maven → JAR → repository (Build Automation page). Docker made the *application environment itself* a buildable, versionable, promotable artifact.

## Layer 1 — Simple Explanation

An image is an **onion of layers**: base OS bits, then dependencies, then your app — each layer only the *diff* from the one below. Containers get a thin writable layer on top (your scratch paper); everything below is frozen and shared across all containers using it.

```text
Image:      [base ubuntu] + [JDK] + [deps] + [app]     ← read-only, shared
Container:  same + [writable scratch layer]             ← dies with the container
```

Build a thousand times: the base layers download/build **once** (content-addressed — Git Internals' hashing idea again).

## Layer 2 — Engineer's View

**The Dockerfile, read as a cache contract:**

```dockerfile
FROM eclipse-temurin:21-jre           # base layer — pinned (Versioning page!)
WORKDIR /app
COPY pom.xml .                          # deps first: layer cache survives app changes
RUN mvn dependency:go-offline          # ↓ cache breakpoint
COPY src ./src
RUN mvn -q package                      # rebuilt only when src changes
USER app                                # never root (Processes/capabilities pages)
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Two disciplines this encodes:

1. **Layer-order caching**: most-frequently-changed last. `COPY . .` before dependency resolution = rebuilding deps every commit = your 12-minute pipeline
2. **Multi-stage builds**: compile in a fat builder, copy only the JAR into a slim runtime — 1.4 GB → 180 MB. Image size *is* deploy latency (nodes pull images; pods wait)

**The layer mechanics under the hood (overlayfs — Filesystems page):** layers are tarballs mounted read-only; container's writes trigger **copy-up** (file from a lower layer gets copied to the writable layer first). That's why "deleting" a big file in a container doesn't shrink the image — and why secrets `COPY`'d then deleted in a later layer are still *in* the image (forensic scanners find them).

**The commands map to the process model:**

| Command | Actually does |
|---|---|
| `docker run` | create namespaces/cgroups + overlay mount + exec entrypoint (the Linux lab, automated) |
| `docker stop` | SIGTERM → grace period → SIGKILL (the Signals choreography) |
| `docker logs` | the process's stdout/stderr, persisted |
| `docker exec` | another process *joined into the same namespaces* |
| `docker stats` | reading the cgroup accounting files |

**Image tags are labels, not identities:** `:2.1.0` is a movable pointer (mutable!) — the true identity is the **digest** `@sha256:abc...`. Pin by digest for reproducibility-critical places (base images, GitOps), tag for humans. (Artifact Repositories page's immutability law, now for images.)

**Docker vs the ecosystem (2015 schism → present):** Docker the *company* lost the runtime war to open standards: images and runtimes are now **OCI** (next page); containerd runs your containers whether Docker, K8s, or Podman fronts it. Docker remains the beloved developer UX (`docker build`) — its pipeline role: build tool, nothing more sacred.

## Real-World Example (DevOps flavored)

The production-grade build stage you own:

```yaml
# CI
- docker build --cache-from type=registry,ref=reg/shop/payments:cache
  -t reg/shop/payments:${GIT_SHA} -t reg/shop/payments:cache .
- docker push reg/shop/payments:${GIT_SHA}        # immutable-by-convention: the SHA tag
- trivy image reg/shop/payments:${GIT_SHA}         # scan before promotion (Security phase)
```

And the incident you prevent by knowing layers: the "deleted" API key in layer 4 of a public image — it was never deleted, only hidden; the fix is a *new image built without it* plus **rotation** (the secret was public the moment it was pushed).

## Common Mistakes

- `latest` tags anywhere near deployment
- `COPY . .` before deps → cache busting → slow CI (and fat layers)
- Secrets in build args/layers (visible in history, recoverable forever)
- Running as root; no HEALTHCHECK; unpinned base images
- Rebuilding deps inside the image instead of multi-stage + CI cache

## Mental Model

> An image is an **onion of frozen diffs**; a container is the onion with a sticky note layer on top (wiped clean each run). Building is assembling the onion — so order your layers by churn (stable core, volatile skin), and never write secrets on an onion: you can peel it back forever.

## Remember This

1. Image = ordered read-only layers + metadata; containers add a writable scratch layer
2. Layer caching: order Dockerfile least→most volatile; multi-stage for size
3. Copy-up explains image-size mysteries and why deleted secrets aren't deleted
4. Tags are mutable labels; digests are identity — pin critical references by digest
5. Docker = developer UX + build; OCI + containerd = the standard underneath
6. Image size and pull time are deployment latency — treat it as a KPI

## One Sentence

Docker turned containers into a pipeline artifact: images are content-addressed stacks of filesystem layers defined by a Dockerfile whose layer ordering, size, and tagging discipline determine how fast, reproducibly, and safely your software ships.

## Knowledge Check

1. Why does `COPY . .` early in a Dockerfile destroy your build cache?
2. A secret was COPY'd and `rm`'d in the next layer. Is it safe? Explain via layers.
3. When must you reference an image by digest rather than tag?
4. Explain `docker stop`'s behavior using only the Signals page.

## Further Reading

- Dockerfile best practices — docs.docker.com
- Next: [OCI & Runtimes](oci-runtimes.md) · [Registries](registries.md)

---

**← Previous:** [Containers](containers.md)
**Next:** [OCI & Runtimes](oci-runtimes.md) →
**Related:** [Build Automation](../cicd/build-automation.md) · [Artifact Repositories](../cicd/artifact-repositories.md)
