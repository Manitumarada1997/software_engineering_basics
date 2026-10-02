# Glossary

Alphabetical quick reference. Entries are added as concepts are completed.

| Term | One-line meaning |
|------|------------------|
| API Gateway | A single entry point that routes, secures, and manages API traffic. |
| Artifact | A versioned, immutable output of a build (JAR, container image, package). |
| Blue/Green Deployment | Run two identical environments; switch traffic instantly between them. |
| Canary Deployment | Release to a small percentage of traffic first, then widen. |
| CAP Theorem | Under a network partition, a distributed system must choose consistency or availability. |
| CI/CD | Continuous Integration / Continuous Delivery — automating build, test, and release. |
| cgroups | Linux kernel feature that limits and accounts resource usage (CPU, memory) per process group. |
| ConfigMap | Kubernetes object for injecting non-sensitive configuration into Pods. |
| Container | An isolated process built from Linux namespaces + cgroups + a layered filesystem. |
| Deployment (K8s) | Manages ReplicaSets to roll out and roll back Pod versions safely. |
| DevSecOps | DevOps with security embedded across the lifecycle, not appended at the end. |
| Error Budget | Allowed unreliability (1 − SLO); spending it wisely enables faster change. |
| etcd | The distributed key-value store that holds all Kubernetes cluster state. |
| Feature Flag | A switch to enable/disable functionality at runtime without redeploying. |
| GitOps | Declarative desired state in Git, synced to the cluster by an agent (pull model). |
| Golden Signals | Latency, traffic, errors, saturation — the four core service health metrics. |
| Golden Path | A supported, paved route for developers to build, deploy, and operate a service. |
| IaC | Infrastructure as Code — declare infrastructure declaratively in versioned files. |
| IDP | Internal Developer Platform — the self-service platform product of platform engineering. |
| Ingress | Kubernetes object managing external HTTP(S) access to Services. |
| Namespaces (Linux) | Kernel feature isolating what a process can see: network, mounts, PIDs, users. |
| Namespaces (K8s) | Virtual clusters within one cluster for isolation of resources and policy. |
| OCI | Open Container Initiative — the standard for image and runtime formats. |
| OpenTelemetry | Vendor-neutral standard for emitting traces, metrics, and logs. |
| Pod | The smallest Kubernetes unit: one or more containers sharing network and storage. |
| Postmortem | A blameless written analysis of an incident: causes, timeline, actions. |
| Reconciliation Loop | Continuously compare desired vs actual state and correct drift — the K8s/GitOps pattern. |
| RBAC | Role-Based Access Control — permissions assigned through roles, not users directly. |
| ReplicaSet | Keeps a stable set of replica Pods running. |
| SBOM | Software Bill of Materials — an inventory of every component inside an artifact. |
| SCA | Software Composition Analysis — scanning dependencies for known vulnerabilities. |
| SAST / DAST | Static (code-level) / Dynamic (running-app) security testing. |
| Secret | Kubernetes object for sensitive data (with caveats — covered in the security phase). |
| Service (K8s) | A stable virtual IP/DNS in front of a changing set of Pods. |
| SLI / SLO / SLA | Indicator (measured signal) / Objective (target) / Agreement (contract with consequences). |
| StatefulSet | K8s workload for apps needing stable identity and storage (databases). |
| TLS | Transport Layer Security — encryption + identity for traffic in transit via certificates. |
| Zero Trust | Never trust, always verify — no implicit trust based on network location. |
