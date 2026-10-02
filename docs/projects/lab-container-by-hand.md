# Lab: Build a Container by Hand

**Level 1 · 30–60 minutes · Linux (root or a VM/WSL2 with cgroup v2)**

The capstone of the Linux phase: build a minimal "container runtime" with `unshare`, cgroups, and a chroot — then never again wonder what Docker does.

## What you'll prove

1. A container is a process with namespaces + cgroups + a filesystem view
2. Networking inside a container is veth/bridge/NAT plumbing
3. Resource limits are two file writes

## Step 0 — Setup

```bash
# Debian/Ubuntu VM or WSL2. Inspect first:
ls /sys/fs/cgroup/           # cgroup v2 present (unified: single hierarchy)?
unshare --version
```

## Step 1 — The isolation (namespaces)

```bash
sudo unshare --uts --pid --net --mount --fork bash
hostname box1                          # UTS: our own hostname
mount -t proc proc /proc               # PID: fresh process table
ps aux                                 # ← you are PID 1. That's a container's world.
# you just did what runc does at "create"
```

Feel the edges: `ls /` is still the *host's* root — namespaces ≠ filesystem isolation. Fix that next.

## Step 2 — The filesystem (chroot/pivot_root + the seed of images)

```bash
# in another shell, on the host:
mkdir -p /opt/minios/{bin,lib,lib64,etc}
cp /bin/busybox /opt/minios/bin/       # apt install busybox-static first
for lib in $(ldd /bin/busybox | grep -oE '/lib[^\s]*'); do
  cp --parents $lib /opt/minios/; done

# back inside the namespace shell:
pivot_root . oldroot 2>/dev/null || chroot /opt/minios /bin/busybox sh
ls /bin                                # busybox world — your "image layer"
```

You just assembled a tiny root filesystem — exactly what an OCI image **is**: a tarball of a rootfs plus a config file. Docker's `docker save` output is this, industrialized.

## Step 3 — The rations (cgroups)

```bash
# host shell:
mkdir /sys/fs/cgroup/box1
echo "max 50000 100000" > /sys/fs/cgroup/box1/cpu.max     # 0.5 CPU
echo 50M > /sys/fs/cgroup/box1/memory.max
echo 100 > /sys/fs/cgroup/box1/pids.max
echo <container-shell-pid> > /sys/fs/cgroup/box1/cgroup.procs

# inside: run busybox's yes > /dev/null, watch it get ~50% CPU.
# Then allocate 200MB with dd into tmpfs: the shell dies — OOMKilled by your own hand.
```

Exit 137. You've personally produced the Kubernetes `OOMKilled` event.

## Step 4 — The network (optional, the full experience)

```bash
ip netns add box1                                   # named netns
ip link add veth0 type veth peer name veth1
ip link set veth1 netns box1
ip addr add 172.20.0.1/24 dev veth0; ip link set veth0 up
ip netns exec box1 ip addr add 172.20.0.2/24 dev veth1
ip netns exec box1 ip link set veth1 up; ip netns exec box1 ip link set lo up
iptables -t nat -A POSTROUTING -s 172.20.0.0/24 -j MASQUERADE
ip netns exec box1 ping 8.8.8.8                     # container with internet
```

This sequence — veth pair, bridge side, address, NAT — is every CNI plugin's core.

## Cleanup

```bash
exit                                # namespace shell
rmdir /sys/fs/cgroup/box1; ip netns del box1; ip link del veth0
```

## What you learned (map to the real thing)

| You did | Docker/K8s name |
|---|---|
| unshare namespaces | `docker run` / runc create |
| pivot_root + rootfs | image layers + overlayfs |
| cgroup writes | `--cpus/--memory`, resources.limits |
| OOM by cgroup | OOMKilled (137) |
| veth/bridge/NAT | container networking, CNI |
| PID 1 in namespace | the PID-1/zombie problem (tini) |

**Debrief questions:**

1. Which single namespace would you drop to make this "container" see host processes — and what attack does that enable?
2. Your hand-container has no CPU throttling until you write `cpu.max`. What does Docker always write for you that you might forget?
3. Why did the OOM kill happen at *your* limit and not when host memory ran out?

---

**← Previous:** [cgroups](../linux/cgroups.md)
**Next:** [OSI & TCP/IP](../networking/osi-tcpip.md) →
**Related:** [Namespaces](../linux/namespaces.md) · [Containers](../containers/containers.md)
