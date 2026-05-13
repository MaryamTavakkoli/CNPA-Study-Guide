# Domain 5: Internal Developer Platforms (IDPs) & Developer Experience

**Exam Weight: 8%**

This domain covers how platform engineering teams design and deliver internal developer platforms (IDPs) that reduce cognitive load, improve developer experience, and accelerate software delivery. It spans self-service patterns, service catalogs, developer portals, and the role of AI/ML in platform automation.

---

## Topic List

| # | Topic | File |
|---|-------|------|
| 1 | Simplified Access to Platform Capabilities | [01-simplified-access.md](./01-simplified-access.md) |
| 2 | API-Driven Service Catalogs | [02-api-driven-service-catalogs.md](./02-api-driven-service-catalogs.md) |
| 3 | Developer Portals | [03-developer-portals.md](./03-developer-portals.md) |
| 4 | AI/ML in Platform Automation | [04-ai-ml-platform-automation.md](./04-ai-ml-platform-automation.md) |

---

## Key Concepts

### What Is an Internal Developer Platform (IDP)?

An IDP is the totality of technology and tooling that a platform team builds and maintains to enable application developers to self-serve infrastructure, deployment pipelines, environments, and operational capabilities without requiring deep platform expertise.

IDPs are **not** a single product — they are a curated, opinionated layer of tools, automation, and documentation stitched together to form a coherent developer experience.

> "A platform is a product built on top of infrastructure to serve developers." — Team Topologies

### The Developer Experience (DevEx) Problem

Without an IDP, developers face:

- **Cognitive load overload** — understanding Kubernetes, Terraform, CI/CD, secrets management, observability, and security compliance all at once
- **Toil accumulation** — repetitive, manual work that does not produce lasting value
- **Inconsistency** — each team inventing its own deployment approach, leading to security drift and high support burden
- **Slow onboarding** — new developers can take weeks or months to become productive

IDPs solve this by abstracting infrastructure complexity behind self-service workflows, golden path templates, and well-documented APIs.

### Core IDP Components

| Component | Purpose | Example Tools |
|-----------|---------|---------------|
| Service Catalog | Discover, track, and understand all software components | Backstage, Port.io |
| Developer Portal | Unified UI for self-service, documentation, and tooling | Backstage, Cortex |
| Golden Path Templates | Opinionated scaffolding for new services | Backstage Scaffolder, Cookiecutter |
| Self-Service Provisioning | Create environments/infrastructure on demand | Crossplane, Terraform Cloud, Radius |
| Internal Documentation | Up-to-date runbooks, ADRs, onboarding guides | TechDocs (Backstage), Confluence |
| Observability Integration | Surface metrics, logs, traces to developers | Grafana, Datadog plugins |

### IDP Maturity Levels

| Level | Description |
|-------|-------------|
| Level 0 | No IDP — manual tickets, wiki pages, tribal knowledge |
| Level 1 | Basic automation — CI/CD pipelines, some Terraform modules |
| Level 2 | Self-service — service catalog, golden paths, environment provisioning |
| Level 3 | Fully integrated IDP — unified portal, AI-assisted workflows, full self-service |

### The CNCF IDP Landscape

The Cloud Native Computing Foundation (CNCF) hosts and incubates many tools relevant to IDP construction:

- **Backstage** (CNCF Incubating) — open-source developer portal framework by Spotify
- **Crossplane** (CNCF Graduated) — Kubernetes-based infrastructure control plane
- **Argo CD** (CNCF Graduated) — GitOps continuous delivery
- **Tekton** (CNCF Incubating) — cloud-native CI/CD pipelines
- **KubeVela** (CNCF Incubating) — application-centric delivery platform

---

## Exam Focus Areas

- Understand the **difference between a platform and an IDP**
- Know the **components of a service catalog** (especially Backstage entities)
- Be familiar with **golden path templates** and why they reduce cognitive load
- Understand how **Backstage plugins** extend portal functionality
- Know the role of **AI/ML** in modern platform engineering workflows
- Understand **self-service provisioning** patterns and the role of platform APIs

---

## Quick Reference

### Key Terms

| Term | Definition |
|------|-----------|
| IDP | Internal Developer Platform — the full set of tooling, processes, and automation available to developers |
| DevEx | Developer Experience — how productive, satisfied, and effective developers feel using platform tooling |
| Golden Path | The recommended, opinionated way to accomplish a task (build, deploy, provision) |
| Cognitive Load | The mental effort required to understand and operate a system |
| Toil | Manual, repetitive operational work that provides no lasting value |
| Scaffolder | A tool that generates project skeletons from templates |
| Service Catalog | A registry of all software components, APIs, and resources in an organization |
| Platform API | A programmatic interface to platform capabilities (provisioning, deployment, secrets, etc.) |

---

## Further Reading

- [CNCF Platforms White Paper](https://tag-app-delivery.cncf.io/whitepapers/platforms/)
- [Backstage Documentation](https://backstage.io/docs)
- [Team Topologies — Manuel Pais & Matthew Skelton](https://teamtopologies.com/)
- [Google DORA State of DevOps Report](https://dora.dev/)
- [Port.io Documentation](https://docs.getport.io/)
