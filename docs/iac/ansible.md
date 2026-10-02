# Ansible & Configuration Management

## What Is It?

**Configuration management (CM)**: keeping machines *configured correctly* — packages, files, services, users. **Ansible** (2012, Red Hat) is its dominant tool: YAML **playbooks** of **idempotent** tasks, pushed over **SSH** (no agents).

```yaml
- name: payments host setup
  hosts: payments
  become: true
  tasks:
  - apt: { name: openjdk-21-jre, state: present }
  - template: { src: app.j2, dest: /etc/shopeasy/app.conf }
    notify: restart shopeasy-api
handlers:
  - name: restart shopeasy-api
    service: { name: shopeasy-api, state: restarted }
```

## Why Does It Exist?

IaC creates machines; **CM makes them correct on the inside** — historically the bigger half of ops. The lineage: cfengine → Puppet/Chef (agents, pull, 2000s) → Ansible (agentless push, YAML, 2012) → then two things shrank its territory:

1. **Immutable infrastructure**: golden images (Packer) + replace-don't-patch (IaC page) moved config *into the build*
2. **Containers/K8s**: the app's "inside" became an image

What remains native CM ground today: VM fleets that must live long (legacy/enterprise), the *host under the platform* (node bootstrap/gardeners — kubespray is Ansible!), network appliances, and brownfield estates.

## Layer 1 — Simple Explanation

CM is the **hotel housekeeping standard**: every room gets the same checklist — towels folded *exactly so*, minibar restocked, lights set — and the checklist is **idempotent**: run it twice, the room stays correct (no double towels).

Ansible is housekeeping **walking room to room with a master key (SSH)** carrying the checklist — no resident manager (agent) needed in each room.

## Layer 2 — Engineer's View

**Idempotency — the concept that IS configuration management:**

```text
A task is idempotent if running it N times has the same effect as running it once.
"ensure package X present" ✓        "install package X" ✗ (fails or duplicates)
```

Why it's sacred: CM runs *repeatedly, forever* (scheduled convergence — the reconcile loop in yet another costume). Non-idempotent tasks make the second run a disaster. (Same discipline as K8s manifests, applied to `apt` and files.)

**The architecture (what "agentless" means):**

```text
control node (your laptop/CI) --SSH--> targets: python executes modules, returns JSON
inventory (INI/YAML/dynamic-from-cloud) + playbooks + roles + variables (per-env)
```

- **Roles** = Ansible's modules (reuse unit): tasks/handlers/templates/vars, versioned, shared (galaxy)
- **Handlers**: run *only if notified* (config changed → restart) — the converge-with-minimum-churn pattern
- **Dynamic inventories**: query the cloud for current hosts — fleet membership as data (IaC creates, CM consumes)
- **check mode** (`--check --diff`): the plan-equivalent — dry-run preview

**Where Ansible still wins in a cloud-native world:**

| Job | Why Ansible |
|---|---|
| Node bootstrap (pre-cloud-init/GPU drivers) | machines must be *made ready* to join the fleet |
| Legacy/enterprise VM estates | can't rebuild nightly; must manage in place |
| Network appliances (switches/firewalls) | SSH+vendor modules; no containers there |
| kubespray/edge/bare-metal | the installer of immutable-platform kitchens |

**The honest split for a modern estate:**

```text
Cloud resources → Terraform/OpenTofu
App runtime     → images + K8s (immutable — replacement as config mgmt)
The in-between  → Ansible: node prep, legacy, appliances, one-off orchestration
```

Also: Ansible Tower/AWX for the UI/RBAC/scheduling layer — the pipeline-product pattern applied to CM.

## Real-World Example (DevOps flavored)

The two surviving Ansible jobs at ShopEasy:

```yaml
# 1. GPU node onboarding (drivers can't be a container):
- name: NVIDIA driver + container toolkit
  hosts: gpu_nodes
  tasks: [{ apt: ...}, { shell: nvidia-ctk runtime configure }, ...]
  # triggered by instance tag via dynamic inventory; idempotent — safe on re-runs

# 2. Quarterly fleet compliance sweep:
- name: CIS baseline
  hosts: all
  tasks: [ssh ciphers, sudo config, auditd...]
  # check-mode first (diff = drift report), then apply; scheduled nightly = converge
```

## Common Mistakes

- Non-idempotent tasks (shell without `creates:` guards) — the second run detonates
- Playbooks as 800-line scripts instead of roles — the monolith anti-pattern, YAML edition
- Managing app deployments with Ansible in a container estate (using the wrong tool for "the inside" that no longer exists)
- No dynamic inventory — hand-maintained host lists aging into fiction
- Secrets in plaintext vars (ansible-vault, or better, external lookup to vaults — Security phase)

## Mental Model

> Configuration management is **hotel housekeeping with a magic checklist**: any room, any time, run the list — the room ends correct. Idempotency is the magic (run twice = run once). Immutable infrastructure shrunk the hotel to its foundations and utility rooms — but someone still has to set up the floors the container-kitchens sit on, and that's still housekeeping.

## Remember This

1. CM = idempotent convergence of machine interiors; Ansible = agentless SSH-push YAML
2. Idempotency is the core discipline — repeated execution must be safe by design
3. Immutable infra + containers moved app config into builds; CM survives at the host/legacy/appliance layer
4. Roles = reuse; dynamic inventories = fleet as data; check-mode = the plan equivalent
5. The estate split: Terraform for cloud, images+K8s for apps, Ansible for the in-between
6. kubespray exists — the immutable platform itself is installed by the mutable tool

## One Sentence

Configuration management converges the inside of machines to declared state through idempotent task execution — a role largely absorbed by immutable images, surviving where hosts must be prepared, nurtured, or lived with.

## Knowledge Check

1. Why does idempotency matter more in CM than in one-off scripts?
2. Split responsibility for: VPC, VM image, GPU driver, legacy WAR app, firewall rule.
3. What replaced CM for application configuration, and what specifically replaced "restart on change"?
4. What does `--check --diff` correspond to in Terraform? In K8s?

## Further Reading

- [docs.ansible.com](https://docs.ansible.com/) — idempotency & roles chapters
- Next: [Drift & Infrastructure Lifecycle](drift-lifecycle.md)

---

**← Previous:** [OpenTofu & Pulumi](opentofu-pulumi.md)
**Next:** [Drift & Infrastructure Lifecycle](drift-lifecycle.md) →
**Related:** [systemd](../linux/systemd.md) · [IaC Fundamentals](iac-fundamentals.md)
