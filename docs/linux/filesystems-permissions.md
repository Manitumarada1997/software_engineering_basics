# Filesystems & Permissions

## What Is It?

- A **filesystem** is how Linux organizes persistent bytes: the virtual filesystem switch (**VFS**) layer lets `ext4`, `xfs`, `nfs`, `overlayfs`, and pseudo-filesystems (`/proc`, `/sys`) all appear as one tree starting at `/`.
- **Permissions** are the kernel's access-control decision on every file operation: user/group/other × read/write/execute, plus the deeper layers (SUID, capabilities, mount options).

## Why Does It Matter to a DevOps Engineer?

Because **everything in Linux is a file**, and the two biggest abstraction stacks you operate are filesystem tricks:

- **OverlayFS** is what Docker image layers are (Containers phase)
- **/proc and /sys** are where live kernel/process state lives — your best debugging surface
- Permissions decide: "permission denied" in a pipeline, container breakout hardening, the entire Unix security model that K8s SecurityContext inherits

## Layer 1 — Simple Explanation

The filesystem is a **library system**:

- The tree (`/`) is the catalog structure — one entrance, uniform shelves
- Different *branches* can be different physical libraries (mounts): local disk, NFS, a fake library that writes books as you ask about them (`/proc`)
- Permissions are the **lending rules per shelf**: owner reads/writes, group reads, strangers get nothing

## Layer 2 — Engineer's View

**Everything is a file descriptor** — the uniform interface:

| File type | Meaning | Example |
|---|---|---|
| `-` regular | bytes on disk | `/var/log/app.log` |
| `d` directory | a list of names | `/etc` |
| `l` symlink | a pointer to another name | `/usr/bin/python` |
| `c/b` device | kernel driver as a file | `/dev/null`, `/dev/sda` |
| `s` socket | network endpoint as a file | `/var/run/docker.sock` ← how the Docker CLI talks to the daemon |
| `p` pipe | kernel buffer connecting processes | shell `\|` |

The last two are operationally huge: the Docker socket *is* root-equivalent access to the host — a mounted socket is a security decision, not a convenience.

**The permission bits (the 60-second refresher, then the real model):**

```text
  -rwxr-x---  1 app  appgrp  file
  │┠─┴─┴─┘
  │ │  │  └── others: ---
  │ │  └───── group: r-x
  │ └──────── owner: rwx
  └────────── type

Execution requires +x; directory traversal requires x on every path component.
```

Beyond rwx — the layers that matter in production:

| Mechanism | What it does | Where you meet it |
|---|---|---|
| **SUID** | Run binary with *owner's* rights | `passwd` (writes /etc/shadow as root) |
| **SGID** | Files in dir inherit dir's group | team share dirs |
| **Sticky bit** | Only owner deletes in dir | `/tmp` |
| **Capabilities** | Split "root" into granular powers | `CAP_NET_BIND_SERVICE`, container `--cap-add` |
| **Mount options** | Kernel-enforced policy | `noexec`, `nosuid`, `ro` — hardening |
| **umask** | Default bits *removed* from new files | pipeline-written artifacts' perms |

**Capabilities deserve emphasis:** the old model is binary (root or not); capabilities let you grant *just* binding port 443 or *just* packet capture. Containers run with a *dropped default set* — `capsh --print`, `--cap-add=NET_BIND_SERVICE` instead of `--privileged`. This is the K8s SecurityContext conversation (Security phase).

**Filesystems you actually operate:**

| FS | Nature | Ops notes |
|---|---|---|
| ext4/xfs | Local, journaling | the default; xfs for large files/parallel IO |
| overlayfs | Stacked layers | **Docker images**: read-only layers + writable top; "copy-up" on write of a lower-layer file |
| tmpfs | RAM-backed | `/dev/shm`; fast, volatile — sizes matter for some apps |
| nfs/efs | Network | latency + locking semantics; Classic source of mysterious hangs (`D` state!) |
| proc/sysfs | Kernel state as files | `/proc/<pid>/*`, `/sys/fs/cgroup/*` — read/write the kernel |

**inodes & "disk full" that isn't:** a filesystem runs out of *inodes* (file count) with space to spare — `df -i` when `df -h` looks clean; classic container-host death by millions of tiny session files.

## Real-World Example (DevOps flavored)

```bash
# "Permission denied" in pipeline step writing to /var/app
ls -ld /var/app                 # drwxr-x--- root root
id                              # pipeline user: uid 1001, no group access
# fix: chown app:app /var/app (or setfacl -m u:1001:rwx) — not chmod 777

# host disk "full":
df -h /   # 41% used?!
df -i /   # IUse 100% — inode exhaustion; find the tiny-file fountain:
find / -xdev -type f | wc -l; du --inodes -sx /var | sort -rn | head
```

And the security connection you already know: `ls -l /var/run/docker.sock` → `srw-rw---- root docker` — anyone in group `docker` can spawn a root-mounted container on the host. Filesystem permissions *were* your first security boundary, and still are the host's.

## Common Mistakes

- `chmod 777` as a debugging tool — the permission equivalent of disabling TLS
- Missing +x when copying scripts (git preserves modes; some CI copy steps don't)
- Confusing "deleted but growing log": a process holds an open fd to a deleted file — space not freed until process restarts (`lsof +L1` — the classic "df says full, du disagrees")
- Running containers `--privileged` where one capability would do
- Forgetting `noexec,nosuid,nodev` on data mounts — cheap hardening left on the table

## Mental Model

> VFS is a **universal adapter**: every source of bytes — disk, network, kernel state, other processes — speaks the same file API, so tools compose (`cat`, `grep`, redirection work on everything). Permissions are the **locks on each shelf**, and capabilities are the **skeleton-key set split into individual keys** so you can hand out only the one needed.

## Remember This

1. One tree; mounts graft different filesystems (including fake ones: /proc, /sys)
2. Everything is a file — sockets and pipes included; docker.sock is root access
3. rwx × u/g/o, plus SUID/SGID/sticky, capabilities, mount flags — layered control
4. Capabilities = fine-grained root; containers drop them by default
5. `df -i` for inode exhaustion; `lsof +L1` for deleted-but-open files
6. OverlayFS layering is Docker images — copy-up on write (next phase uses this)

## One Sentence

The Linux filesystem presents every resource — disk, kernel state, even other processes — as one uniform tree of files guarded by layered permissions, which is why your tools compose and why socket/group membership is a security decision.

## Knowledge Check

1. Why is membership in the `docker` group effectively root access?
2. `df -h` shows 40% free but writes fail. Two possible causes and their checks?
3. What do `nosuid,noexec` mount options actually prevent?
4. Explain copy-on-write/copy-up in overlayfs in one sentence — and why `rm` on a file in an image layer doesn't shrink the image.

## Further Reading

- `man 5 capabilities`, `man mount`, `man inode`
- *The Linux Command Line* — William Shotts (ch. 14–15)
- [Docker overlayfs driver docs](https://docs.docker.com/storage/storagedriver/overlayfs-driver/) — bridge to the Containers phase

---

**← Previous:** [Memory](memory.md)
**Next:** [Linux Networking](networking.md) →
**Related:** [Namespaces](namespaces.md) · [K8s Security](../security/k8s-container-security.md)
