# Platform Engineering Goals, Objectives, and Approaches

> **Podcast episodes for this topic:**
> - [Episode 1: The CNCF Platform Engineering Maturity Model](https://github.com/MaryamTavakkoli/cnpa-study-guide/releases/download/v1.0/The_CNCF_Platform_Engineering_Maturity_Model.m4a)
> - [Episode 2: CNCF Platforms White Paper](https://github.com/MaryamTavakkoli/cnpa-study-guide/releases/download/v1.0/CNCF.Platforms.White.Paper.m4a)

## Why Platform Engineering Exists

As organizations scale their cloud-native adoption, developers face increasing complexity:
- Multiple Kubernetes clusters and namespaces to manage
- CI/CD pipeline configuration to maintain
- Secrets, certificates, and IAM policies to manage
- Observability stacks to configure
- Security policies to understand and comply with

Each team reinventing the wheel creates **duplicated effort**, **inconsistency**, and **cognitive overload**.

Platform engineering solves this by centralizing shared concerns and exposing them as self-service capabilities.

---

## Primary Goals

### 1. Reduce Cognitive Load

Cognitive load is the mental effort required to do a job. There are two types:

- **Intrinsic cognitive load**: Complexity inherent to the work itself (business logic)
- **Extraneous cognitive load**: Complexity imposed by tools, processes, and infrastructure

Platform engineering eliminates extraneous cognitive load. A developer building a payment service should think about payment logic — not Kubernetes networking or certificate rotation.

### 2. Enable Developer Self-Service

Developers should be able to:
- Provision a new service (with all dependencies) in minutes, not days
- Create a new environment for a feature branch
- Deploy a change to production without creating a ticket
- Access logs, metrics, and traces for their service

Self-service removes the platform team as a bottleneck.

### 3. Standardize and Enforce Best Practices

The platform encodes organizational standards:
- Security policies (no root containers, required resource limits, image scanning)
- Logging and metrics format
- Deployment patterns (blue-green, canary)
- Secret management approach

Standards become defaults, not documentation.

### 4. Accelerate Delivery

By removing friction from the path to production, platforms increase **deployment frequency** and reduce **lead time for changes** — two of the four key DORA metrics.

### 5. Improve Reliability

Consistent infrastructure, automated deployments, and built-in observability reduce:
- Configuration drift
- Human error in deployments
- Time to detect and respond to incidents

---

## Platform Engineering Approaches

### Approach 1: Build vs. Buy vs. Compose

Platform teams rarely build everything from scratch. The decision framework:

| Option | When to Use |
|---|---|
| **Buy** | Commodity capability (log aggregation, secret management) where SaaS/managed service fits |
| **Build** | Core differentiator specific to your organization's context |
| **Compose** | Integrate open-source tools (Kubernetes + ArgoCD + Prometheus + Backstage) into a unified experience |

Most platform engineering is **compose** — integrating CNCF ecosystem tools and providing a thin opinionated layer on top.

### Approach 2: Top-Down vs. Bottom-Up

**Top-down**: Executive mandate to use the platform. Fast adoption but risks building the wrong thing.

**Bottom-up**: Platform earns adoption by being genuinely useful. Slower but more sustainable. Platform team must market internally.

Best practice: hybrid. Executive support for the platform team's existence + bottom-up adoption through genuine value.

### Approach 3: Centralized vs. Federated Platform

| Model | Characteristics |
|---|---|
| **Centralized** | One platform team serves all dev teams; maximum consistency |
| **Federated** | Multiple platform teams (per business unit); more autonomy but inconsistency risk |
| **Hybrid** | Core platform team provides foundations; embedded platform engineers in stream-aligned teams |

### Approach 4: Evolutionary Architecture

Platforms must evolve. Avoid big-bang rewrites. Instead:
- Start with the highest-pain areas
- Add capabilities incrementally
- Deprecate old capabilities with migration paths
- Measure impact before adding complexity

---

## Platform Team Responsibilities

| Responsibility | Description |
|---|---|
| **Build and operate the platform** | Keep CI/CD, Kubernetes, monitoring running reliably |
| **Enable developers** | Provide documentation, examples, and support |
| **Define golden paths** | Opinionated templates and workflows |
| **Enforce standards** | Policy-as-code, required CI checks |
| **Capacity planning** | Ensure clusters have enough resources |
| **Incident response** | Own SLOs for the platform itself |
| **Cost management** | Track and optimize infrastructure spend |

---

## What Platform Engineering Is NOT

- **Not a gatekeeper**: The platform should reduce approvals, not add them
- **Not an ivory tower**: Platform decisions must be grounded in developer needs
- **Not purely infrastructure**: It's as much UX and product work as it is ops
- **Not a one-time project**: Platforms require ongoing investment

---

## Cognitive Load and Team Topologies

The *Team Topologies* book introduced the concept of cognitive load limits for teams. A team can only manage so much cognitive load before quality and velocity suffer.

Platform engineering's job is to keep stream-aligned teams' cognitive load within bounds by:
- Providing reliable platform services (so teams don't think about the platform)
- Handling cross-cutting concerns (security, networking, storage)
- Exposing simple, well-documented APIs

A stream-aligned team should interact with the platform via a **thin API** — not by understanding the internals of the platform.

---

## Key Takeaways

- Platform engineering exists to reduce cognitive load and eliminate duplicated effort
- Primary goals: self-service, standardization, faster delivery, improved reliability
- Most platforms are composed from open-source tools, not built from scratch
- Earn adoption by being genuinely useful; avoid becoming a mandatory bottleneck
- Platform team should own the platform's SLO — it is a product with reliability requirements
- Cognitive load limits teams; platform reduces the load stream-aligned teams carry
