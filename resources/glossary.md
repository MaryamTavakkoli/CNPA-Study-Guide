# CNPA Exam Glossary

Alphabetical reference of key terms across all CNPA exam domains.

---

## A

**Admission Controller** — A Kubernetes plugin that intercepts requests to the Kubernetes API server before objects are persisted. Admission controllers can validate (reject non-compliant resources), mutate (modify incoming resources), or do both. Examples: OPA/Gatekeeper (validating), the built-in NamespaceLifecycle controller.

**Argo CD** — A declarative, GitOps continuous delivery tool for Kubernetes. Argo CD pulls desired state from a Git repository and continuously reconciles it against the live cluster state, providing a UI and CLI for application management.

**Argo Rollouts** — A Kubernetes controller that provides advanced deployment strategies (canary, blue-green, progressive delivery) with automated analysis and rollback based on metrics.

**ArgoCD ApplicationSet** — A CRD that enables Argo CD to generate multiple Applications from a single template, using generators (Git path, cluster list, pull request, etc.) to support multi-cluster and mono-repo deployments at scale.

---

## B

**Backstage** — An open-source CNCF project (originally built by Spotify) that provides a framework for building Internal Developer Portals. It includes a software catalog, Software Templates (scaffolding), TechDocs, and an extensible plugin system.

**Blue-Green Deployment** — A deployment strategy where two identical production environments (blue = current, green = new) are maintained. Traffic is cut over instantly from blue to green, enabling zero-downtime releases and fast rollbacks.

---

## C

**Canary Deployment** — A progressive delivery strategy where a new version is released to a small subset of users or traffic first, with metrics monitored before gradually expanding the rollout to 100%.

**Change Failure Rate (CFR)** — One of the four DORA key metrics. The percentage of deployments to production that result in a degraded service or require a rollback or hotfix. A stability metric.

**Change Lead Time** — One of the four DORA key metrics. The time elapsed from code commit to running successfully in production. Measures delivery throughput.

**CI (Continuous Integration)** — The practice of automatically building and testing code changes on every commit, providing fast feedback to developers and maintaining a consistently releasable codebase.

**CNCF (Cloud Native Computing Foundation)** — A Linux Foundation project that hosts and maintains key open-source cloud native projects (Kubernetes, Prometheus, Envoy, Argo, etc.) and defines cloud native principles.

**Cognitive Load** — The mental effort required to understand and use a system. Platform engineering aims to reduce extraneous cognitive load on developers by abstracting infrastructure complexity behind self-service APIs.

**ConfigMap** — A Kubernetes object that stores non-sensitive key-value configuration data that can be mounted into Pods as environment variables or volumes, decoupling configuration from container images.

**Cosign** — A tool from the Sigstore project used to sign and verify container images and other software artifacts, enabling supply chain security by attesting the provenance and integrity of artifacts.

**Crossplane** — An open-source CNCF project that extends Kubernetes with the ability to provision and manage cloud infrastructure (databases, networks, storage) using the Kubernetes API and reconciliation loop. Enables infrastructure as Kubernetes resources.

**CRD (Custom Resource Definition)** — A Kubernetes extension mechanism that allows teams to define new API resource types. Once a CRD is registered, users can create, read, update, and delete instances of the new type using standard Kubernetes tooling.

---

## D

**Declarative Configuration** — A model where the user specifies the desired end state of the system (e.g., "3 replicas of this container image") and the system determines how to achieve and maintain that state, rather than specifying explicit steps.

**Deployment Frequency** — One of the four DORA key metrics. How often an organisation successfully releases to production. Elite teams deploy on demand or multiple times per day.

**DORA Metrics** — A set of four key metrics from the DevOps Research and Assessment program used to measure software delivery performance: Deployment Frequency, Change Lead Time, Change Failure Rate, and Mean Time to Recovery (MTTR).

**Drift** — A condition where the live state of a system diverges from its declared desired state in Git (or another source of truth). GitOps tools detect and correct drift automatically.

---

## E

**eBPF (Extended Berkeley Packet Filter)** — A Linux kernel technology that allows sandboxed programs to run in the kernel without changing kernel source code. Used by tools like Cilium (networking) and Falco (security) for high-performance, low-overhead observation and enforcement.

**Environment Parity** — The practice of keeping development, staging, and production environments as similar as possible to reduce environment-specific bugs and deployment surprises.

**Error Budget** — The permissible amount of unreliability for a service, calculated as `1 - SLO`. A 99.9% availability SLO has a 0.1% error budget (~43.8 minutes/month). When the budget is exhausted, reliability work takes priority over new features.

---

## F

**Falco** — An open-source CNCF runtime security tool that detects anomalous behaviour in containers and hosts by monitoring Linux kernel system calls using eBPF or a kernel module.

**Flux** — An open-source CNCF GitOps toolkit for Kubernetes. Flux is modular (separate controllers for source, kustomize, Helm, image automation) and uses a pull-based reconciliation model.

---

## G

**GitOps** — An operational model where Git is the single source of truth for declarative infrastructure and application configuration. Changes are made via pull requests; an automated agent continuously reconciles the live system to match the Git state.

**Golden Path** — A pre-built, opinionated, and well-supported way to perform common development tasks (create a service, set up CI, deploy to a cluster) that encodes platform best practices. Teams are encouraged — but not mandated — to use it.

**Gatekeeper** — An OPA-based Kubernetes admission controller that enforces custom policies written in Rego, distributed as ConstraintTemplates and Constraints.

---

## H

**Helm** — A Kubernetes package manager that uses Charts (templated YAML + values files) to define, install, and manage applications. Helm tracks release history and supports rollbacks.

**Helm Chart** — The package format used by Helm: a directory of YAML templates, a `Chart.yaml` metadata file, and default `values.yaml`. Charts can be stored in chart repositories and versioned.

---

## I

**IaC (Infrastructure as Code)** — The practice of defining and managing infrastructure resources using version-controlled, machine-readable configuration files rather than manual processes or GUIs.

**IDP (Internal Developer Platform)** — A self-service layer of tools, APIs, and documentation built by a platform team to help development teams build, deploy, and operate applications without direct platform team intervention.

**Image Digest** — A SHA256 hash that uniquely and immutably identifies a specific container image. Using digests instead of mutable tags (like `:latest`) in production ensures that the exact same image is deployed across environments.

---

## J

**Jaeger** — An open-source, CNCF distributed tracing system originally developed at Uber. Jaeger collects and visualises trace data from microservices to help diagnose latency and performance issues.

---

## K

**Kustomize** — A Kubernetes-native configuration management tool built into `kubectl`. It uses a base + overlays model to apply environment-specific patches to Kubernetes manifests without templating syntax.

**Kyverno** — A Kubernetes-native policy engine (CNCF project) that uses YAML-based policies (not Rego) to validate, mutate, and generate Kubernetes resources. Simpler to write policies compared to OPA/Gatekeeper for Kubernetes-specific use cases.

---

## L

**LimitRange** — A Kubernetes object that sets per-Pod or per-container default resource requests and limits within a namespace, preventing containers from running without defined resource bounds.

---

## M

**Managed Resource (MR)** — In Crossplane, a CRD that represents a single cloud infrastructure resource (e.g., an RDS instance, a GCS bucket). Each MR is reconciled by a provider controller.

**Mean Time to Recovery (MTTR)** — One of the four DORA key metrics. The average time to restore service after a production incident. Measures delivery stability and operational maturity.

**mTLS (Mutual TLS)** — An extension of TLS where both the client and server authenticate each other using certificates. Used in service meshes to provide zero-trust, encrypted, authenticated communication between services.

---

## N

**Namespace** — A Kubernetes object that provides a logical scope for resource names, access control (RBAC) boundaries, and resource quota enforcement. Namespaces are the primary multi-tenancy mechanism in Kubernetes.

**Network Policy** — A Kubernetes resource that controls ingress and egress traffic between Pods using label selectors and port specifications. Requires a CNI plugin that supports Network Policies (e.g., Calico, Cilium).

---

## O

**Observability** — The ability to understand the internal state of a system from its external outputs. In cloud native systems, observability is typically achieved through the three pillars: metrics, logs, and distributed traces.

**OPA (Open Policy Agent)** — A general-purpose, open-source policy engine that uses the Rego policy language to evaluate requests against rules. In Kubernetes, OPA is commonly deployed via Gatekeeper as an admission controller.

**Operator Pattern** — A Kubernetes pattern where a custom controller (the Operator) manages the lifecycle of a complex, stateful application by encoding operational knowledge into a CRD-based control loop.

---

## P

**Platform Engineering** — The discipline of designing and building toolchains and workflows that enable development teams to self-service their infrastructure and delivery needs, reducing cognitive load and improving developer productivity.

**Pod Security Admission (PSA)** — A Kubernetes built-in admission controller that enforces three security profiles (Privileged, Baseline, Restricted) at the namespace level, controlling what security settings Pods may use.

**Progressive Delivery** — An approach to releasing software that gradually shifts traffic to a new version while monitoring metrics, enabling automated rollback if quality signals degrade. Includes canary releases and feature flags.

**Prometheus** — An open-source CNCF monitoring system and time-series database. It scrapes metrics from instrumented applications and infrastructure, stores them, and supports alerting via Alertmanager.

**Provider (Crossplane)** — A Crossplane plugin that contains CRDs and controllers for managing resources on a specific cloud platform (e.g., `provider-aws`, `provider-gcp`). Providers extend Crossplane's ability to manage external infrastructure.

---

## R

**RBAC (Role-Based Access Control)** — A Kubernetes authorisation mechanism where permissions are defined in Roles (namespace-scoped) or ClusterRoles (cluster-scoped) and granted to users, groups, or ServiceAccounts via RoleBindings or ClusterRoleBindings.

**Reconciliation Loop** — The core mechanism of Kubernetes controllers: observe the current state, compare it to the desired state, and act to close any gap. This loop runs continuously, providing self-healing behaviour.

**Rego** — The policy language used by Open Policy Agent (OPA). Rego is a declarative query language designed for expressing policy rules over structured data (JSON/YAML).

**ResourceQuota** — A Kubernetes object that limits the aggregate amount of compute resources (CPU, memory) and the number of objects (Pods, Services, etc.) that can exist in a namespace.

**Rollback** — The act of reverting a deployment to a previous known-good version. In GitOps, this is a Git revert; in Helm, it is `helm rollback`; in Argo Rollouts, it can be triggered automatically by metric analysis.

---

## S

**SBOM (Software Bill of Materials)** — A formal, structured inventory of all components, libraries, and dependencies in a software artifact. SBOMs enable rapid vulnerability assessment (e.g., determining if a CVE affects any deployed software) and licence compliance.

**Secret** — A Kubernetes object that stores sensitive data such as passwords, tokens, and keys. Secrets are base64-encoded (not encrypted by default) and should be protected by etcd encryption at rest and strict RBAC.

**Service Catalog** — A component of an Internal Developer Portal (e.g., Backstage) that provides a searchable, organised inventory of all services, APIs, libraries, and other software components in an organisation, including ownership and documentation.

**Service Mesh** — A dedicated infrastructure layer for managing service-to-service communication in a microservices architecture. Provides mTLS, traffic management, observability, and policy enforcement — typically via sidecar proxies. Examples: Istio, Linkerd, Cilium Service Mesh.

**SLA (Service Level Agreement)** — A contractual commitment, typically with an external customer, specifying the expected level of service (availability, latency) and consequences for violations.

**SLI (Service Level Indicator)** — A specific, measurable metric used to track a service's behaviour relative to an SLO. For example, the percentage of requests completing in under 200ms.

**SLO (Service Level Objective)** — An internal reliability target defined in terms of an SLI over a time window. For example, "99.9% of requests return HTTP 200 over a rolling 30-day window."

**SLSA (Supply-chain Levels for Software Artifacts)** — A security framework developed by Google that defines levels of supply chain integrity for software. Higher SLSA levels impose stricter build process requirements, provenance attestation, and tamper-resistance.

**Software Template (Backstage)** — A Backstage feature that provides a self-service form for creating new software components. Templates execute automated scaffolding actions (repo creation, CI setup, catalog registration) to implement Golden Paths.

---

## T

**Team Topologies** — A book and organisational framework by Matthew Skelton and Manuel Pais that defines four team types (stream-aligned, platform, enabling, complicated-subsystem) and three interaction modes (collaboration, facilitating, X-as-a-service) for effective software delivery.

**Tekton** — An open-source CNCF framework for building CI/CD pipelines as Kubernetes-native resources (Tasks, Pipelines, Triggers). Pipelines run as Kubernetes Pods, enabling native integration with cluster capabilities.

**Trivy** — An open-source CNCF vulnerability scanner (by Aqua Security) that detects CVEs in container images, filesystems, Git repositories, and Kubernetes cluster configurations. Supports SBOM generation.

**Trunk-Based Development** — A source control branching model where developers commit frequently to a shared trunk (main branch) using short-lived feature branches, minimising merge conflicts and enabling continuous integration.

---

## V

**Validating Admission Webhook** — A Kubernetes extension point where an external service (webhook) is called to validate API requests before resources are persisted. OPA Gatekeeper and Kyverno both operate as validating admission webhooks.

---

## X

**XRC (Composite Resource Claim)** — In Crossplane, a namespace-scoped resource that developers use to request a composite infrastructure resource. The claim triggers a Composition, which assembles the underlying managed resources on the developer's behalf.
