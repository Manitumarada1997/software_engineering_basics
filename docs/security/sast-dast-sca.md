# SAST, DAST & SCA

## What Is It?

The three automated security scanners, each looking at a different artifact through a different lens:

| | **SAST** (static) | **SCA** (composition) | **DAST** (dynamic) |
|---|---|---|---|
| Looks at | source code | dependencies | running app |
| Finds | injection patterns, crypto misuse, secrets | known CVEs in libs, licenses | runtime vulns: XSS, misconfig, auth bypass |
| Runs | per commit (CI) | per commit (CI) | against deployed env |
| Blind to | runtime behavior | your own code | internals/origin |
| Tools | SonarQube, Semgrep, CodeQL | Dependabot, Trivy, Snyk | OWASP ZAP, Burp |

You've operated SAST+SCA via SonarQube and image scans — this page makes the *why* and the *limits* explicit.

## Why Does It Exist?

Because security review doesn't scale by humans, and each scanner covers a defect class the others structurally cannot:

```text
Your code is dangerous       → SAST (patterns in source)
Your dependencies are dangerous → SCA (known CVEs — you didn't write them, someone must match)
Your system-as-assembled is dangerous → DAST (only the running thing reveals config/auth/assembly bugs)
```

The overlap is near zero — which is why "we have SonarQube" is never a full answer.

## Layer 1 — Simple Explanation

- **SAST** = proofreading the **manuscript** before printing: finds dangerous sentences, but can't know how the book reads in a dark room
- **SCA** = checking the **ingredients list** against recall notices: your recipe is fine; the flour was recalled
- **DAST** = a **burglar walking the finished house**: doors, locks, windows — the assembled reality, not the blueprints

## Layer 2 — Engineer's View

**SAST — mechanics and the false-positive bargain:**

- Pattern/data-flow analysis over source; catches injection, hardcoded secrets, unsafe crypto
- The known pain: false positives → alert fatigue → ignored findings (the flaky-test death, security edition)
- The mature practice: **baseline + ratchet** — old findings tracked, *new* findings block the PR (SonarQube quality gates' "new code" model — shift-left with survivable friction)
- Semgrep's insight: *writable* rules — teams encode their own vuln patterns (policy-as-code for source)

**SCA — the dependency problem, honestly:**

- Modern apps: 70–90% of code is dependencies (why: the Build Automation page's resolver hands you hundreds of transitive packages)
- SCA matches manifests/lockfiles/images against CVE databases; the *hard* part is **reachability**: a CVE in an unused code path is noise — tools (and SBOM analysis, next page) increasingly score by "actually callable?"
- **Version conflict danger** (Build Automation page warned): conflict resolution can silently downgrade a security fix — SCA is the detector
- License compliance rides along: copyleft in a commercial product is a legal CVE

**DAST — why it runs per-deployment:**

- Crawls/attacks the running app (staging, ephemeral envs): finds what assembly created — exposed admin paths, TLS misconfig, auth bypasses, header issues
- Finds *classes* SAST can't see (the config, the deployment, the combination) — and it's black-box: language-agnostic, catches third-party appliances too
- Pipeline placement: post-deploy against staging per merge (the Perf/Security Testing page's staged filter)

**The pipeline assembly (staged economics, revisited):**

```yaml
PR:      semgrep + gitleaks + SCA (fail on new criticals)
CI:      sonar quality gate + image scan (trivy) → block promotion
Deploy:  staging → DAST (ZAP baseline) + smoke
Quarter: pentest (humans — the class nothing automated replaces)
```

**The metrics that keep scanners alive:** mean-time-to-remediate per severity, false-positive rate (tracked!), % builds scanned. A scanner nobody believes in is worse than none — trust maintenance is part of the engineering.

## Real-World Example (DevOps flavored)

The SonarQube upgrade you've probably wished for, made concrete:

```text
Before: gate = "no criticals" on 200k-line legacy → always failing → bypassed by admins
After:  baseline snapshot; gate = no NEW issues on changed code (ratchet);
        weekly burndown of legacy criticals by severity;
        SCA added with auto-PRs (Dependabot) for patchable CVEs;
        ZAP baseline against staging nightly, diffs only
Result: devs trust the gates (they fail for reasons they control) — the entire point
```

## Common Mistakes

- One scanner as the complete answer (the classes don't overlap)
- Gates on absolute counts over legacy code → bypass culture (the anti-ratchet)
- Untriaged CVE floods — severity without reachability context = ignored
- DAST in prod only (or never) — needs a disposable deployed target
- Secrets scanning skipped (SAST's most valuable sub-feature — cheap, catches the permanent Git sin)

## Mental Model

> SAST reads the **manuscript**, SCA checks the **ingredients against recall lists**, DAST sends a **burglar through the finished house**. Three inspectors, three blind spots — and the gate architecture matters as much as the scanners: ratchets keep trust, absolute gates breed bypasses.

## Remember This

1. SAST (source patterns) / SCA (dependency CVEs+licenses) / DAST (running system) — near-zero overlap
2. Baseline-and-ratchet gates: block *new* issues; burndown the legacy separately
3. Reachability > raw severity in SCA triage — noisy CVSS kills trust
4. Staged economics: static per-commit, dynamic per-deploy, humans quarterly
5. Secrets scanning is the cheapest high-value control in the set
6. Scanner trust is an engineering asset — measure FP rate and MTTR

## One Sentence

SAST, SCA, and DAST automate security review of your code, your dependencies, and your running system respectively — complementary lenses whose findings must gate incrementally (ratchets) or they get ignored.

## Knowledge Check

1. Why can't SAST find what DAST finds? Give one concrete example each way.
2. Design the ratchet policy for a legacy monrepo with 400 existing criticals.
3. A critical CVE in an unimported dependency: what does reachability analysis change?
4. Why do absolute-count quality gates produce *less* security over time?

## Further Reading

- OWASP SAMM / DevSecOps guidance; Semgrep docs (rule writing)
- Next: [SBOM & Supply Chain](sbom-supply-chain.md)

---

**← Previous:** [Kubernetes & Container Security](k8s-container-security.md)
**Next:** [SBOM & Supply Chain Security](sbom-supply-chain.md) →
**Related:** [Performance & Security Testing](../development-practices/perf-security-testing.md) · [Shift Left](../development-practices/shift-left-right.md)
