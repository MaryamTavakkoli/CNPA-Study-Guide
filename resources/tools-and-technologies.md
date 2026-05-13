# Tools and Technologies Reference

A capability-oriented reference of tools relevant to the CNPA exam. For each tool, the CNCF project status is noted where applicable.

> **CNCF Status Key:** Graduated | Incubating | Sandbox | Not a CNCF project

---

## CI (Continuous Integration)

| Tool | Category | License | Description | CNCF Status |
|------|----------|---------|-------------|-------------|
| **Tekton** | CI pipeline | Apache 2.0 | Kubernetes-native CI/CD pipeline framework built from Tasks and Pipelines as CRDs; each pipeline step runs as a Kubernetes Pod | Graduated |
| **Jenkins** | CI server | MIT | Widely-adopted, extensible CI automation server with a large plugin ecosystem; Jenkinsfile defines pipelines as code | Not CNCF |
| **GitHub Actions** | CI/CD SaaS | Proprietary (free tier) | Event-driven workflow automation tightly integrated with GitHub; YAML-defined workflows run on GitHub-hosted or self-hosted runners | Not CNCF |
| **GitLab CI/CD** | CI/CD platform | MIT (CE) / Commercial (EE) | Built-in CI/CD in GitLab; pipelines defined in `.gitlab-ci.yml`; supports runners, environments, and artifacts natively | Not CNCF |
| **Dagger** | CI engine | Apache 2.0 | Portable CI/CD pipeline engine that runs pipelines in containers using a programmable API (Go, Python, TypeScript SDKs) | Not CNCF |
| **Drone** | CI server | Apache 2.0 (community) | Container-native CI system where each pipeline step runs in an isolated container; YAML-configured | Not CNCF |

---

## CD / GitOps

| Tool | Category | License | Description | CNCF Status |
|------|----------|---------|-------------|-------------|
| **Argo CD** | GitOps CD | Apache 2.0 | Declarative GitOps continuous delivery tool for Kubernetes; provides a web UI, CLI, and ApplicationSet for multi-cluster/app deployments | Graduated |
| **Flux** | GitOps CD | Apache 2.0 | Modular GitOps toolkit for Kubernetes; composed of separate controllers (source, kustomize, helm, image automation); CLI/API-first design | Graduated |
| **Argo Rollouts** | Progressive delivery | Apache 2.0 | Kubernetes controller for advanced deployment strategies (canary, blue-green) with automated metric-based analysis and rollback | Incubating |
| **Spinnaker** | CD orchestrator | Apache 2.0 | Multi-cloud CD platform supporting complex deployment pipelines, manual approvals, and canary analysis across multiple cloud providers | Not CNCF |
| **Helm** | Package manager / CD | Apache 2.0 | Kubernetes package manager using Charts; manages installation, upgrades, and rollbacks of application releases | Graduated |
| **Flagger** | Progressive delivery | Apache 2.0 | Kubernetes Operator for automating canary, A/B testing, and blue/green deployments using service mesh traffic management | Incubating |

---

## Policy Engines

| Tool | Category | License | Description | CNCF Status |
|------|----------|---------|-------------|-------------|
| **OPA (Open Policy Agent)** | Policy engine | Apache 2.0 | General-purpose policy engine using the Rego language; used for Kubernetes admission control, API authorisation, and infrastructure policy | Graduated |
| **Gatekeeper** | Kubernetes admission controller | Apache 2.0 | OPA-based Kubernetes admission controller; enforces policies as ConstraintTemplates (Rego) and Constraints applied to the cluster | Graduated |
| **Kyverno** | Kubernetes policy engine | Apache 2.0 | Kubernetes-native policy engine using YAML policies; supports validation, mutation, and generation of Kubernetes resources without Rego | Graduated |
| **Kubewarden** | Policy engine | Apache 2.0 | Kubernetes policy engine that runs policies compiled to WebAssembly (Wasm), enabling policies written in any Wasm-compatible language | Sandbox |
| **Conftest** | Policy testing | Apache 2.0 | CLI tool for testing configuration files (YAML, JSON, Terraform) against OPA Rego policies; useful in CI pipelines for pre-cluster validation | Not CNCF |

---

## Observability

| Tool | Category | License | Description | CNCF Status |
|------|----------|---------|-------------|-------------|
| **Prometheus** | Metrics / Alerting | Apache 2.0 | Time-series metrics database with a pull-based scraping model; PromQL query language; paired with Alertmanager for alerts | Graduated |
| **Grafana** | Visualisation | AGPL 3.0 (OSS) | Dashboard and visualisation platform; supports Prometheus, Loki, Tempo, and many other data sources via plugins | Not CNCF |
| **Grafana Loki** | Log aggregation | AGPL 3.0 (OSS) | Log aggregation system inspired by Prometheus; indexes log metadata (labels) rather than full-text, making it cost-efficient | Not CNCF |
| **Jaeger** | Distributed tracing | Apache 2.0 | End-to-end distributed tracing system for microservices; supports OpenTelemetry for instrumentation; provides trace visualisation UI | Graduated |
| **OpenTelemetry** | Observability framework | Apache 2.0 | Vendor-neutral APIs, SDKs, and tools for generating and collecting metrics, logs, and traces; the CNCF standard for instrumentation | Graduated |
| **Thanos** | Long-term metrics storage | Apache 2.0 | Extends Prometheus with global query view, long-term object storage, and high-availability multi-cluster metrics federation | Incubating |
| **Falco** | Runtime security | Apache 2.0 | Cloud native runtime security tool that detects anomalous behaviour using eBPF-based system call monitoring and customisable rule sets | Graduated |
| **Kube-state-metrics** | Kubernetes metrics | Apache 2.0 | Exposes Kubernetes object state as Prometheus metrics (replica counts, pod phases, resource limits); essential for Kubernetes monitoring | Not CNCF |

---

## Service Mesh

| Tool | Category | License | Description | CNCF Status |
|------|----------|---------|-------------|-------------|
| **Istio** | Service mesh | Apache 2.0 | Feature-rich service mesh providing mTLS, traffic management (VirtualServices, DestinationRules), observability, and Wasm-based extensibility | Graduated |
| **Linkerd** | Service mesh | Apache 2.0 | Lightweight, Rust-based service mesh focused on simplicity and low resource overhead; provides mTLS, observability, and traffic management | Graduated |
| **Cilium** | eBPF networking + mesh | Apache 2.0 | eBPF-based networking, observability, and security for Kubernetes; includes a service mesh mode (Cilium Service Mesh) without sidecars | Graduated |
| **Envoy** | Proxy | Apache 2.0 | High-performance L7 proxy and communication bus; the sidecar proxy used by Istio and many other service meshes and API gateways | Graduated |
| **Consul** | Service mesh / discovery | MPL 2.0 | HashiCorp's service mesh and service discovery solution; supports Kubernetes and non-Kubernetes workloads in hybrid environments | Not CNCF |

---

## Secret Management

| Tool | Category | License | Description | CNCF Status |
|------|----------|---------|-------------|-------------|
| **HashiCorp Vault** | Secrets manager | BSL (community) | Centralised secrets management with dynamic secrets, leases, audit logging, and integration with Kubernetes via the Vault Agent or CSI driver | Not CNCF |
| **External Secrets Operator** | Secrets sync | Apache 2.0 | Kubernetes Operator that syncs secrets from external providers (Vault, AWS Secrets Manager, GCP Secret Manager) into Kubernetes Secrets | Not CNCF |
| **Sealed Secrets** | GitOps-safe secrets | Apache 2.0 | Bitnami tool that encrypts Kubernetes Secrets into SealedSecret resources safe to store in Git; decrypted by a controller in the cluster | Not CNCF |
| **SOPS** | Secret encryption | MPL 2.0 | Mozilla tool for encrypting YAML/JSON/ENV files using AWS KMS, GCP KMS, Azure Key Vault, age, or PGP; used in GitOps for secret-in-Git workflows | Not CNCF |
| **cert-manager** | Certificate management | Apache 2.0 | Kubernetes-native certificate controller for automating TLS certificate issuance and renewal from Let's Encrypt, Vault, or internal CAs | Graduated |

---

## IaC / Infrastructure Provisioning

| Tool | Category | License | Description | CNCF Status |
|------|----------|---------|-------------|-------------|
| **Crossplane** | Universal control plane | Apache 2.0 | Extends Kubernetes to manage cloud infrastructure (databases, networks, storage) using CRDs, Compositions, and provider plugins | Graduated |
| **Terraform** | IaC | BSL (community) | Declarative infrastructure provisioning tool using HCL; manages resources across cloud providers via providers; widely adopted | Not CNCF |
| **OpenTofu** | IaC | MPL 2.0 | Open-source fork of Terraform (Linux Foundation) created after HashiCorp's BSL license change; maintains compatibility with Terraform | Not CNCF |
| **Pulumi** | IaC | Apache 2.0 | Infrastructure as Code using general-purpose programming languages (TypeScript, Python, Go, C#) instead of DSLs | Not CNCF |
| **Cluster API (CAPI)** | Cluster lifecycle | Apache 2.0 | Kubernetes sub-project for declarative provisioning, upgrading, and operating Kubernetes clusters using Kubernetes-style APIs | Not CNCF (SIG) |
| **Karpenter** | Node provisioning | Apache 2.0 | Kubernetes node autoscaler that provisions right-sized nodes just-in-time based on Pod scheduling constraints | Not CNCF |

---

## Developer Portals

| Tool | Category | License | Description | CNCF Status |
|------|----------|---------|-------------|-------------|
| **Backstage** | Developer portal framework | Apache 2.0 | CNCF project (originally by Spotify) providing a software catalog, Software Templates (Golden Paths), TechDocs, and a plugin ecosystem for IDPs | Graduated |
| **Port** | Developer portal | Commercial (free tier) | SaaS Internal Developer Portal with a flexible catalog, self-service actions, and RBAC; integrates with Kubernetes, GitHub, and cloud providers | Not CNCF |
| **Cortex** | Service catalog | Commercial (free tier) | Developer portal focused on service maturity scorecards, catalog, and on-call integration; SaaS-based | Not CNCF |
| **OpsLevel** | Developer portal | Commercial | IDP platform with a service catalog, maturity levels, self-service runbooks, and integrations with CI/CD and monitoring tools | Not CNCF |

---

## Image Security / Supply Chain

| Tool | Category | License | Description | CNCF Status |
|------|----------|---------|-------------|-------------|
| **Trivy** | Vulnerability scanner | Apache 2.0 | Comprehensive scanner for CVEs in container images, filesystems, repos, and Kubernetes; also generates SBOMs (CycloneDX, SPDX) | Graduated |
| **Cosign** | Image signing | Apache 2.0 | Part of the Sigstore project; signs container images and attestations with keyless (OIDC-based) or key-based signatures | Not CNCF (Sigstore) |
| **Syft** | SBOM generation | Apache 2.0 | CLI tool by Anchore for generating SBOMs from container images, filesystems, and source repositories in SPDX, CycloneDX, and Syft formats | Not CNCF |
| **Grype** | Vulnerability scanner | Apache 2.0 | Vulnerability scanner by Anchore that works with Syft-generated SBOMs or directly against container images to detect CVEs | Not CNCF |
| **Notary / Notation** | Image signing | Apache 2.0 | CNCF project for signing and verifying container images and other OCI artifacts; v2 (Notation) integrates with OCI registries natively | Graduated |

---

## Container Registries

| Tool | Category | License | Description | CNCF Status |
|------|----------|---------|-------------|-------------|
| **Harbor** | Container registry | Apache 2.0 | CNCF-graduated private container and Helm chart registry with built-in vulnerability scanning (Trivy), RBAC, content trust, and replication | Graduated |
| **Zot** | OCI registry | Apache 2.0 | Minimal, OCI-native container registry focused on spec compliance and simplicity; CNCF sandbox project | Sandbox |
| **Amazon ECR** | Managed registry | Commercial | AWS managed container registry with built-in IAM integration, vulnerability scanning, and lifecycle policies | Not CNCF |
| **Google Artifact Registry** | Managed registry | Commercial | GCP managed registry for containers, Helm charts, Maven, npm, and other artifact types with IAM-based access control | Not CNCF |
| **GitHub Container Registry (GHCR)** | Managed registry | Commercial (free for public) | GitHub-hosted OCI container registry integrated with GitHub Actions and GitHub's permission model | Not CNCF |
| **Docker Hub** | Public/private registry | Commercial (free tier) | The default public container registry; widely used for public images; rate limits apply to unauthenticated pulls | Not CNCF |
