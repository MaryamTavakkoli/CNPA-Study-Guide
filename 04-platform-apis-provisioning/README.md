# Domain 4: Platform APIs & Provisioning

**Exam Weight: 12%**

This domain covers the APIs and provisioning mechanisms that underpin cloud native platforms — from how Kubernetes reconciles desired state to how platform teams expose self-service infrastructure through CRDs, Operators, and tools like Crossplane.

---

## Topics in This Domain

| # | File | Topic |
|---|------|-------|
| 1 | [01-kubernetes-reconciliation-loop.md](./01-kubernetes-reconciliation-loop.md) | Kubernetes Reconciliation Loop |
| 2 | [02-apis-self-service-crds.md](./02-apis-self-service-crds.md) | APIs, Self-Service & CRDs |
| 3 | [03-infrastructure-provisioning.md](./03-infrastructure-provisioning.md) | Infrastructure Provisioning with Kubernetes |
| 4 | [04-kubernetes-operator-pattern.md](./04-kubernetes-operator-pattern.md) | The Kubernetes Operator Pattern |

---

## Key Concepts

### Reconciliation & Control Loops
- Kubernetes is a **declarative** system: you declare desired state, controllers reconcile actual state toward it
- The control loop pattern: **Observe → Diff → Act**
- Controllers use **Informers** to watch resources efficiently without polling the API server directly
- **Work queues** decouple event detection from reconciliation logic
- **Eventual consistency**: the system will converge to desired state, but not instantaneously
- Reconciliation functions must be **idempotent** — safe to run multiple times with the same result

### CRDs and API Extension
- **Custom Resource Definitions (CRDs)** extend the Kubernetes API without modifying core code
- CRDs enable platform teams to define domain-specific abstractions (databases, queues, certificates)
- **Schema validation** (OpenAPI v3) enforces structure and defaults on custom resources
- The **aggregated API server** pattern allows full API server extension with independent backing storage
- Popular CRDs: Crossplane XRDs, Argo `Application`, cert-manager `Certificate`, External Secrets `ExternalSecret`

### Infrastructure Provisioning
- **Crossplane** provisions cloud infrastructure through the Kubernetes API using providers and managed resources
- **Compositions** and **XRDs** let platform teams build opinionated, self-service infrastructure APIs
- Claims are namespace-scoped; Composite Resources (XRs) are cluster-scoped
- **Terraform Operator** and **Pulumi Kubernetes Operator** bring IaC workflows into Kubernetes
- Platform teams choose between Crossplane (Kubernetes-native) and Terraform (ecosystem maturity) based on team skills and operational model

### Operator Pattern
- Operators encode **operational knowledge** about a specific application as code
- An Operator = CRD + Controller(s) that know how to manage the full lifecycle of that application
- **Operator SDK** provides scaffolding for Go, Ansible, and Helm-based operators
- **Finalizers** ensure cleanup happens before resource deletion; **owner references** cascade deletion
- **OperatorHub.io** is the community registry for discovering and installing operators
- Maturity levels progress from basic Install through Autopilot (full self-management)

---

## Exam Tips

- Know the difference between a **CRD** (schema definition) and a **Custom Resource** (instance of the schema)
- Understand why reconciliation must be **idempotent** and what **level-triggered** means vs event-triggered
- Be able to explain how Crossplane **Compositions** abstract cloud resources from application teams
- Know the **five operator maturity levels** and what capabilities each introduces
- Understand **finalizers**: a resource with a finalizer will not be deleted until the finalizer is removed
- Know the difference between **cluster-scoped** and **namespace-scoped** custom resources and why it matters for multi-tenancy

---

## Related Domains

- Domain 3 (Continuous Delivery) — GitOps controllers like Argo CD and Flux use the reconciliation loop pattern
- Domain 5 (IDPs & Developer Experience) — Self-service platforms are built on top of the CRD/Operator/Crossplane foundations covered here
- Domain 2 (Observability) — Understanding controller metrics and logs is key to platform operations
