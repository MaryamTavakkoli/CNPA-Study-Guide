# Platform Architecture and Capabilities

## What Is a Platform?

In platform engineering, a **platform** is a foundation of self-service APIs, tools, services, knowledge, and support. It is arranged as a **compelling internal product**.

The CNCF defines a platform as:
> "A platform for cloud-native computing is an integrated collection of capabilities defined and presented according to the needs of the platform's users."

Key characteristics:
- **Self-service**: Developers provision what they need without creating tickets
- **Opinionated**: Provides "golden paths" that encode best practices
- **Product-oriented**: Built for internal users with UX in mind
- **Composable**: Built from building blocks (open source tools, cloud services)

---

## The Platform Maturity Model

Platforms evolve over time. The CNCF Platform Engineering Maturity Model defines four levels:

| Level | Name | Characteristics |
|---|---|---|
| 1 | **Provisional** | Ad hoc, individual heroics, manual processes |
| 2 | **Operationalized** | Consistent processes, some automation, team dedicated to platform |
| 3 | **Scalable** | Self-service capabilities, documented golden paths, metrics |
| 4 | **Optimizing** | Platform as a product, continuous improvement, developer NPS tracked |

---

## Platform Capabilities

A well-built platform addresses these capability areas:

### 1. Developer Control Planes / Portals

- Self-service portal (e.g., Backstage) for discovering and provisioning services
- Service catalog: list of available services, APIs, documentation
- Software templates: scaffold new services with pre-baked CI/CD, monitoring, etc.

### 2. Compute and Orchestration

- Kubernetes clusters (managed or self-hosted)
- Autoscaling (Horizontal Pod Autoscaler, Vertical Pod Autoscaler, Cluster Autoscaler)
- Workload placement policies (node selectors, affinity/anti-affinity, taints/tolerations)

### 3. Delivery Pipelines

- CI pipelines: build, test, scan on every commit
- CD pipelines or GitOps controllers: deploy to environments
- Promotion workflows: move from dev → staging → prod

### 4. Observability

- Metrics collection (Prometheus)
- Log aggregation (Loki, Elasticsearch)
- Distributed tracing (Jaeger, Tempo)
- Dashboards (Grafana)
- Alerting (Alertmanager, PagerDuty)

### 5. Networking and Service Mesh

- Ingress management
- Service-to-service communication (optionally via service mesh: Istio, Linkerd)
- TLS/mTLS termination
- DNS management

### 6. Security

- Secret management (Vault, External Secrets Operator, AWS Secrets Manager)
- Policy enforcement (OPA/Gatekeeper, Kyverno)
- Image scanning (Trivy, Snyk)
- RBAC and identity

### 7. Storage and Data

- PersistentVolume provisioning
- Database services (managed or operator-managed)
- Backup and restore

### 8. Infrastructure Provisioning

- IaC tooling (Terraform, Crossplane)
- Self-service environment creation
- Cost management

---

## Internal Developer Platform (IDP) Architecture

An IDP is the technical implementation of the platform. It glues together tools and provides a unified interface.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Developer Interface Layer                      │
│            (Portal, CLI, API, Slack bot, etc.)                   │
└─────────────────────────────┬───────────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────────┐
│                    Platform Orchestration                         │
│   (Backstage, Port, Cortex, Humanitec, custom API gateway)       │
└──────────┬──────────────────┬───────────────────────────────────┘
           │                  │
     ┌─────▼──────┐    ┌──────▼──────┐
     │  CI System │    │ CD / GitOps │
     │ (Jenkins,  │    │ (ArgoCD,    │
     │ GitHub Act,│    │ Flux)       │
     │ Tekton)    │    └──────┬──────┘
     └────────────┘          │
                        ┌────▼────────────────────┐
                        │   Kubernetes Clusters    │
                        │   (dev, staging, prod)   │
                        └─────────────────────────┘
```

### Backstage

Backstage is the most popular open-source developer portal, created by Spotify and donated to CNCF.

Core features:
- **Software Catalog**: Tracks all services, libraries, websites, pipelines
- **Software Templates**: Self-service scaffolding for new services
- **TechDocs**: Documentation-as-code, rendered in the portal
- **Plugins**: Hundreds of plugins for integrating with every tool in the ecosystem

```yaml
# Backstage catalog entity (catalog-info.yaml)
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: my-service
  description: Payment processing service
  tags:
    - payments
    - java
spec:
  type: service
  lifecycle: production
  owner: payments-team
  system: payment-platform
```

---

## Platform as a Product

Platform engineers must treat the platform as a product with internal customers (developers).

This means:
- **Discovery**: Understand developer pain points through interviews, surveys, support ticket analysis
- **Prioritization**: Build what reduces the most cognitive load or toil
- **User research**: Observe how developers actually use the platform
- **Metrics**: Track adoption, time-to-first-deployment, developer NPS
- **Roadmap**: Communicate what's coming to build trust
- **Support SLAs**: Define how quickly platform issues are addressed

The platform team should **not** be a gatekeeper or bottleneck. If developers need approval for every action, the platform has failed its self-service goal.

---

## Thinnest Viable Platform (TVP)

Avoid over-engineering. The **Thinnest Viable Platform** is the smallest platform that:
- Reduces cognitive load on developers
- Enables teams to deliver faster
- Is sustainable for the platform team to maintain

Start with the highest-pain areas, not the most technically interesting ones.

---

## Key Takeaways

- A platform is a self-service internal product for developers
- Platform maturity progresses from provisional → operationalized → scalable → optimizing
- Key platform capabilities: compute, delivery, observability, networking, security, storage, provisioning
- Backstage is the leading open-source developer portal (software catalog + templates + plugins)
- Treat the platform as a product: interview users, measure adoption, iterate
- Thinnest Viable Platform: build what helps most, not what's most complex
