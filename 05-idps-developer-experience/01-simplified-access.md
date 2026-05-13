# Simplified Access to Platform Capabilities

## Overview

One of the primary goals of platform engineering is making powerful infrastructure and deployment capabilities accessible to developers without requiring them to understand every underlying technology. Simplified access is achieved through self-service portals, CLI tools, golden path templates, and platform APIs — all designed to reduce cognitive load and eliminate toil.

---

## The Developer Experience (DevEx) Concept

Developer Experience (DevEx) refers to the overall quality of a developer's interactions with the tools, systems, processes, and people in their work environment. High DevEx means developers can focus on writing valuable code; low DevEx means they spend significant time fighting infrastructure, waiting on approvals, or deciphering undocumented systems.

### Three Dimensions of DevEx (GitHub Research Framework)

| Dimension | Description | Example |
|-----------|-------------|---------|
| **Feedback Loops** | Speed and quality of signals developers receive | Fast CI, immediate test results |
| **Cognitive Load** | The mental effort required to do work | Clear documentation, sensible defaults |
  | **Flow State** | Uninterrupted, focused work | Minimal context switching, self-service tools |

### Why DevEx Matters to Platform Teams

- Developer satisfaction directly correlates with retention and productivity
- High cognitive load increases error rates and burnout
- Platform teams that invest in DevEx see faster feature delivery and lower support burden
- DORA research links developer experience to elite software delivery performance

---

## Cognitive Load Reduction

Cognitive load in software development falls into three categories (based on John Sweller's Cognitive Load Theory):

| Type | Description | Platform Solution |
|------|-------------|-------------------|
| **Intrinsic** | Complexity inherent to the task (e.g., business logic) | Accepted — developers should focus here |
| **Extraneous** | Unnecessary complexity from poor tooling or documentation | Eliminated via opinionated golden paths and clear docs |
| **Germane** | Effort spent building useful mental models | Supported via consistent abstractions and learning resources |

Platform teams should aggressively eliminate **extraneous** cognitive load. A developer should not need to understand the internals of Kubernetes admission webhooks to deploy a microservice.

---

## Self-Service Portals

A self-service portal is a web application that gives developers direct access to platform capabilities without raising tickets or requesting help from the platform team.

### Capabilities Typically Exposed via a Portal

- Provision new environments (dev, staging, production)
- Create new services from templates
- View deployment status and history
- Access logs, metrics, and traces
- Manage secrets and configuration
- Browse the service catalog
- View on-call schedules and incident status
- Request access to resources (with automated approval workflows)

### Key Design Principles for Self-Service Portals

1. **Progressive disclosure** — show simple options first, advanced options on demand
2. **Sensible defaults** — pre-fill forms with production-ready values
3. **Immediate feedback** — show status of provisioning requests in real time
4. **Auditability** — log all actions for compliance and debugging
5. **Idempotency** — allow users to safely retry operations
6. **Discoverability** — make it easy to find what you need without documentation

---

## CLI Tools

Not all developers prefer web UIs. CLI tools are essential for developers who live in the terminal, need scriptability, or work in CI/CD automation.

### Design Principles for Platform CLIs

```bash
# Good CLI design: clear verbs, predictable flags, useful help
platform create service --name my-api --language go --template grpc-service

# Bad CLI design: opaque flags, no defaults, cryptic errors
plat svc --n my-api -l 1 -t 3
```

### Characteristics of a Good Platform CLI

- **Verb-noun command structure** (`platform create environment`, `platform list services`)
- **Shell completion** (bash/zsh/fish autocomplete)
- **Consistent output formats** (`--output json|yaml|table`)
- **`--dry-run` flag** to preview changes before applying
- **Interactive and non-interactive modes** (for both humans and automation)
- **Context management** (switch between clusters, namespaces, projects)
- **Versioned and distributed via package managers** (brew, apt, winget, asdf)

### Example: Internal Platform CLI Usage

```bash
# Create a new service from a golden path template
plat service create \
  --name payments-api \
  --team payments \
  --template go-grpc-service \
  --region eu-west-1

# Provision a dev environment
plat env create \
  --name payments-dev-maryam \
  --service payments-api \
  --ttl 7d

# Check deployment status
plat deploy status --service payments-api --env production

# Rotate a secret
plat secret rotate --service payments-api --key DB_PASSWORD
```

---

## Golden Path Templates

A **golden path** (also called a paved road or golden road) is the recommended, pre-configured, opinionated way to accomplish a common task. It is built and maintained by the platform team and represents a production-ready starting point that embodies organizational best practices.

### What a Golden Path Template Includes

```
my-go-service-template/
├── .github/
│   └── workflows/
│       ├── ci.yaml          # Pre-configured CI pipeline
│       └── release.yaml     # Semantic release workflow
├── Dockerfile               # Multi-stage, minimal base image
├── helm/
│   └── values.yaml          # Sane Kubernetes defaults
├── src/
│   └── main.go              # Service entrypoint with health checks
├── Makefile                 # Standard build/test/lint targets
├── catalog-info.yaml        # Backstage catalog registration
├── OWNERS                   # Code ownership declaration
└── README.md                # Auto-populated from template vars
```

### Example: Backstage Software Template (scaffolder-template)

```yaml
# template.yaml — Backstage scaffolder template
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: go-grpc-service
  title: Go gRPC Microservice
  description: Creates a production-ready Go gRPC service with CI/CD, observability, and security defaults
  tags:
    - go
    - grpc
    - recommended
spec:
  owner: platform-team
  type: service
  parameters:
    - title: Service Information
      required:
        - name
        - owner
      properties:
        name:
          title: Service Name
          type: string
          description: Unique name for the service (lowercase, hyphens only)
          pattern: '^[a-z][a-z0-9-]*$'
        owner:
          title: Owning Team
          type: string
          ui:field: OwnerPicker
          ui:options:
            catalogFilter:
              kind: Group
        description:
          title: Description
          type: string
    - title: Infrastructure
      properties:
        region:
          title: Primary Region
          type: string
          default: eu-west-1
          enum:
            - eu-west-1
            - us-east-1
            - ap-southeast-1
  steps:
    - id: fetch-template
      name: Fetch Base Template
      action: fetch:template
      input:
        url: ./skeleton
        values:
          name: ${{ parameters.name }}
          owner: ${{ parameters.owner }}
          region: ${{ parameters.region }}

    - id: publish-github
      name: Publish to GitHub
      action: publish:github
      input:
        repoUrl: github.com?owner=my-org&repo=${{ parameters.name }}
        defaultBranch: main

    - id: register-catalog
      name: Register in Catalog
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps['publish-github'].output.repoContentsUrl }}
        catalogInfoPath: /catalog-info.yaml

  output:
    links:
      - title: Repository
        url: ${{ steps['publish-github'].output.remoteUrl }}
      - title: Open in Catalog
        url: ${{ steps['register-catalog'].output.entityRef }}
```

### Benefits of Golden Paths

| Benefit | Description |
|---------|-------------|
| **Security by default** | Network policies, RBAC, non-root containers baked in |
| **Observability out of the box** | Prometheus metrics, structured logging, tracing configured |
| **Compliance** | License headers, SBOM generation, vulnerability scanning in CI |
| **Faster onboarding** | New developers productive in hours, not weeks |
| **Reduced support burden** | Platform team supports fewer one-off configurations |
| **Consistency** | All services have the same operational surface area |

> Golden paths are recommended, not mandated. Developers can diverge when necessary, but the path of least resistance should be the correct one.

---

## Toil Reduction

Toil is defined by Google SRE as work that is:
- **Manual** — requires human action
- **Repetitive** — the same work done over and over
- **Automatable** — could be done by a machine
- **Tactical** — reactive, not strategic
- **Devoid of lasting value** — does not improve the system
- **Scales linearly with service growth** — more services = more toil

### Measuring Toil

Platform teams should track the percentage of time spent on toil versus project work. Google SRE recommends keeping toil below 50% of any engineer's time.

```
Toil % = (Hours on toil per week / Total working hours per week) × 100
```

### Common Platform Toil and Automation Solutions

| Toil Activity | Automation Solution |
|---------------|---------------------|
| Manually creating Kubernetes namespaces | Namespace-as-a-Service via platform API |
| Manually rotating secrets | Automated secret rotation (Vault agent, ESO) |
| Approving environment access requests | Policy-based auto-approval (OPA, Cedar) |
| Updating base Docker images | Renovate Bot / Dependabot |
| Running security scans | Integrated into CI/CD (Trivy, Snyk) |
| Creating DNS entries | External DNS operator |
| Provisioning SSL certificates | cert-manager |
| Onboarding new teams | Golden path template + automated provisioning |

---

## Self-Service Provisioning Patterns

### Pattern 1: Request-Based (Ticket → Automation)

```
Developer → Creates request in portal → Platform API validates → 
Terraform/Crossplane applies → Resource created → Developer notified
```

Suitable for: Infrequently created, high-cost, or compliance-sensitive resources.

### Pattern 2: GitOps-Based (PR → Apply)

```
Developer → Opens PR with resource definition → 
Policy check (OPA) → Merge → ArgoCD/Flux applies → Resource created
```

```yaml
# Developer submits this file in a PR to the environments repo
# environments/payments/dev.yaml
apiVersion: platform.myorg.io/v1alpha1
kind: Environment
metadata:
  name: payments-dev
  namespace: platform-system
spec:
  service: payments-api
  tier: development
  region: eu-west-1
  ttl: 14d
  resources:
    cpu: "2"
    memory: "4Gi"
  database:
    engine: postgres
    version: "15"
    size: db.t3.small
```

### Pattern 3: Self-Service API (Programmatic)

```bash
# Developer calls platform API directly (e.g., from CI/CD)
curl -X POST https://platform.myorg.io/api/v1/environments \
  -H "Authorization: Bearer ${PLATFORM_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "payments-dev-pr-123",
    "service": "payments-api",
    "tier": "ephemeral",
    "ttl": "4h"
  }'
```

---

## Platform APIs

A platform API is the programmatic interface to all platform capabilities. It is the foundation on which portals, CLIs, and automation are built.

### Characteristics of a Good Platform API

- **RESTful or GraphQL** with OpenAPI/Swagger documentation
- **Idempotent operations** — safe to call multiple times
- **Consistent authentication** (OIDC/JWT, service accounts)
- **Rate limiting and quotas** — prevent abuse
- **Audit logging** — every mutation logged with actor identity
- **Versioned** — `/api/v1/`, `/api/v2/` for backward compatibility
- **Event-driven** — webhooks or event streams for async operations

### Platform API Capability Areas

| Domain | Example Endpoints |
|--------|-------------------|
| Services | `POST /services`, `GET /services/{name}`, `DELETE /services/{name}` |
| Environments | `POST /environments`, `GET /environments/{name}` |
| Deployments | `POST /deployments`, `GET /deployments/{id}/status` |
| Secrets | `POST /secrets/{name}/rotate`, `GET /secrets/{name}/versions` |
| Access | `POST /access-requests`, `PUT /access-requests/{id}/approve` |
| Observability | `GET /services/{name}/metrics`, `GET /services/{name}/logs` |

---

## Exam Tips

- **Golden paths are recommended, not required** — know this distinction
- **Toil must scale linearly with growth** to qualify as true toil
- **Self-service reduces the platform team as a bottleneck** — this is the core value proposition
- **Platform APIs enable CLI, portal, and automation** — they are the foundation layer
- **DevEx is measured** — SPACE framework, developer NPS, time-to-first-deployment
- Backstage Scaffolder is the most commonly tested tool for golden path templates
