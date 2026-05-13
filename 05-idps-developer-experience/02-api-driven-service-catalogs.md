# API-Driven Service Catalogs

## Overview

A service catalog is the authoritative registry of all software components, APIs, infrastructure resources, and teams in an organization. It answers foundational questions: What services exist? Who owns them? What do they depend on? Where are they deployed? An API-driven service catalog makes this information programmatically accessible, enabling automated governance, dependency analysis, and developer self-service.

---

## What Is a Service Catalog?

A service catalog is a structured inventory of:

- **Services and applications** — microservices, monoliths, front-ends
- **APIs** — internal and external HTTP, gRPC, GraphQL, event-based APIs
- **Infrastructure resources** — databases, queues, storage buckets, clusters
- **Teams** — who owns what, on-call contacts, communication channels
- **Documentation** — runbooks, architecture decisions, onboarding guides
- **Dependencies** — which services call which other services

### Problems a Service Catalog Solves

| Problem | Without Catalog | With Catalog |
|---------|----------------|--------------|
| Finding service owners | Asking Slack, digging through GitHub | Catalog lookup returns owner team instantly |
| Understanding dependencies | Reading code, asking engineers | Dependency graph visualization |
| Onboarding new developers | Weeks of exploration | Self-service catalog browsing |
| Impact analysis | Manual investigation | Automated dependency traversal |
| Security audits | Spreadsheets | API-driven querying of all components |
| Compliance reporting | Manual data collection | Automated catalog exports |

---

## Backstage Software Catalog In Depth

Backstage is an open-source developer portal framework created by Spotify and donated to the CNCF (now at Incubating maturity). Its **Software Catalog** is the most widely adopted service catalog implementation in cloud-native organizations.

### Backstage Entity Model

Everything in the Backstage catalog is an **entity** — a structured YAML object stored in source control and ingested by Backstage. Entities are described by `catalog-info.yaml` files.

#### Entity Structure

```yaml
# General structure of any Backstage entity
apiVersion: backstage.io/v1alpha1   # or v1beta1
kind: <EntityKind>                  # Component, API, System, Domain, Resource, Group, User
metadata:
  name: <unique-name>
  namespace: default                # optional, defaults to "default"
  title: Human Readable Title
  description: What this thing does
  tags:
    - go
    - payments
    - pci-dss
  annotations:
    # Annotations link entities to external systems
    github.com/project-slug: my-org/my-repo
    backstage.io/techdocs-ref: dir:.
    pagerduty.com/service-id: P1234XY
spec:
  # Kind-specific fields
```

### The Five Core Entity Kinds

#### 1. Component

A deployable software artifact — a microservice, frontend app, library, or data pipeline.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: payments-api
  description: Handles payment processing for checkout flows
  tags:
    - go
    - grpc
    - payments
    - pci-dss
  annotations:
    github.com/project-slug: my-org/payments-api
    backstage.io/techdocs-ref: dir:.
    grafana/dashboard-selector: "payments"
    pagerduty.com/service-id: P7K2MN1
spec:
  type: service          # service | website | library | documentation | ml-model
  lifecycle: production  # experimental | production | deprecated
  owner: group:payments-team
  system: checkout-system
  dependsOn:
    - component:user-service
    - resource:payments-db
    - resource:payment-events-topic
  providesApis:
    - payments-grpc-api
  consumesApis:
    - fraud-detection-api
```

**Component `type` values:**
- `service` — backend service (API, worker, daemon)
- `website` — frontend application
- `library` — shared code package (npm, Go module, pip package)
- `documentation` — standalone documentation site
- `ml-model` — machine learning model artifact

#### 2. API

Describes an interface that a component exposes or consumes.

```yaml
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: payments-grpc-api
  description: gRPC API for payment operations
  tags:
    - grpc
    - payments
spec:
  type: grpc             # openapi | asyncapi | grpc | graphql | trpc
  lifecycle: production
  owner: group:payments-team
  system: checkout-system
  definition:
    $text: https://github.com/my-org/payments-api/blob/main/api/payments.proto
```

**API `type` values and their definition formats:**

| Type | Definition Format | Use Case |
|------|------------------|----------|
| `openapi` | OpenAPI 3.x / Swagger YAML/JSON | REST APIs |
| `asyncapi` | AsyncAPI YAML | Event-driven / message-based APIs |
| `grpc` | Protocol Buffers (.proto) | gRPC services |
| `graphql` | GraphQL SDL | GraphQL APIs |

#### 3. System

A collection of related components and APIs that together deliver a capability.

```yaml
apiVersion: backstage.io/v1alpha1
kind: System
metadata:
  name: checkout-system
  description: All components involved in the e-commerce checkout flow
  tags:
    - payments
    - checkout
spec:
  owner: group:payments-team
  domain: ecommerce
```

#### 4. Domain

A high-level grouping of systems aligned to a business domain (maps well to Domain-Driven Design bounded contexts).

```yaml
apiVersion: backstage.io/v1alpha1
kind: Domain
metadata:
  name: ecommerce
  description: All systems related to the e-commerce product line
spec:
  owner: group:ecommerce-leadership
```

#### 5. Resource

An infrastructure dependency that components rely on — databases, queues, storage, clusters.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Resource
metadata:
  name: payments-db
  description: PostgreSQL database for payment records
  tags:
    - postgres
    - pci-dss
  annotations:
    aws.amazon.com/rds-instance-id: payments-prod-db
spec:
  type: database        # database | cache | message-broker | storage | kubernetes-cluster
  owner: group:platform-team
  system: checkout-system
  dependsOn:
    - resource:payments-vpc
```

### Additional Entity Kinds

| Kind | Purpose |
|------|---------|
| `Group` | Represents a team or organizational unit |
| `User` | An individual contributor (usually synced from identity provider) |
| `Location` | Tells Backstage where to find more catalog files |
| `Template` | A Scaffolder template for creating new components |

---

## catalog-info.yaml Placement and Discovery

### Standard Placement

Each repository should have a `catalog-info.yaml` at its root:

```
my-service/
├── catalog-info.yaml    ← Backstage reads this
├── src/
├── helm/
└── README.md
```

### Location Entity for Multi-Component Repos

For monorepos or repos with multiple components:

```yaml
# catalog-info.yaml at repo root
apiVersion: backstage.io/v1alpha1
kind: Location
metadata:
  name: payments-monorepo
  description: All payment-related services
spec:
  targets:
    - ./services/payments-api/catalog-info.yaml
    - ./services/refund-service/catalog-info.yaml
    - ./libraries/payments-sdk/catalog-info.yaml
```

### Catalog Discovery Methods

Backstage discovers catalog files through **integrations** and **location providers**:

1. **GitHub/GitLab integration** — Backstage scans all repos in an organization for `catalog-info.yaml`
2. **Static locations** — Manually listed URLs in `app-config.yaml`
3. **AWS S3 / GCS** — Read catalog files from object storage
4. **Custom providers** — Sync from CMDB, ServiceNow, or other authoritative sources

```yaml
# app-config.yaml — catalog discovery configuration
catalog:
  rules:
    - allow: [Component, API, Resource, System, Domain, Group, User, Location, Template]
  locations:
    # Static location
    - type: url
      target: https://github.com/my-org/platform/blob/main/catalog/all-components.yaml
  providers:
    github:
      my-org:
        organization: my-org
        catalogPath: /catalog-info.yaml
        filters:
          branch: main
          repository: '.*'   # all repos
        schedule:
          frequency: { minutes: 30 }
          timeout: { minutes: 3 }
```

---

## Entity Relations

Relations express how entities connect to each other. They are bidirectional and automatically resolved by Backstage.

| Relation | Description | Example |
|----------|-------------|---------|
| `ownedBy` / `ownerOf` | Team ownership | `payments-api` ownedBy `payments-team` |
| `partOf` / `hasPart` | System membership | `payments-api` partOf `checkout-system` |
| `dependsOn` / `dependencyOf` | Runtime dependency | `payments-api` dependsOn `payments-db` |
| `providesApi` / `apiProvidedBy` | API exposure | `payments-api` providesApi `payments-grpc-api` |
| `consumesApi` / `apiConsumedBy` | API consumption | `payments-api` consumesApi `fraud-detection-api` |
| `memberOf` / `hasMember` | Team membership | User `maryam` memberOf `payments-team` |

Relations are automatically computed from `spec.dependsOn`, `spec.providesApis`, etc. and displayed as a visual dependency graph in the Backstage UI.

---

## TechDocs

TechDocs is Backstage's built-in documentation system. It follows the **docs-as-code** approach — documentation lives in the source repository alongside the code and is rendered in Backstage.

### How TechDocs Works

```
Source Repo (mkdocs.yml + docs/*.md) 
  → TechDocs CLI generates static site (MkDocs + Material theme)
  → Published to object storage (S3, GCS, Azure Blob)
  → Backstage serves docs inline in the portal
```

### Configuration

```yaml
# catalog-info.yaml — link to TechDocs
metadata:
  annotations:
    backstage.io/techdocs-ref: dir:.  # docs/ folder at repo root
```

```yaml
# mkdocs.yml — documentation site configuration
site_name: Payments API
docs_dir: docs
nav:
  - Home: index.md
  - Architecture: architecture.md
  - API Reference: api-reference.md
  - Runbooks:
    - Deployment: runbooks/deployment.md
    - Incident Response: runbooks/incident-response.md
plugins:
  - techdocs-core
```

---

## Dependency Mapping and Discoverability

### Dependency Graph Use Cases

| Use Case | How Catalog Helps |
|----------|------------------|
| **Impact analysis** | Find all components that depend on a service before making a breaking change |
| **Incident response** | Quickly identify downstream systems affected by an outage |
| **Capacity planning** | Understand infrastructure resource consumption across all services |
| **Security audits** | Find all components using a vulnerable library version |
| **Migration planning** | Identify all consumers of a legacy API |

### Backstage Catalog API

Backstage exposes a REST API for programmatic catalog access:

```bash
# Get all components owned by a team
GET /api/catalog/entities?filter=kind=Component,spec.owner=group:payments-team

# Get a specific entity
GET /api/catalog/entities/by-name/component/default/payments-api

# Get all entities with a specific tag
GET /api/catalog/entities?filter=metadata.tags=pci-dss

# Search catalog
GET /api/catalog/entities/by-query?fullTextFilter=payments
```

---

## Port.io as an Alternative

Port is a commercial developer portal and service catalog platform that competes with and complements Backstage.

### Port.io Key Concepts

| Concept | Description | Backstage Equivalent |
|---------|-------------|---------------------|
| **Blueprint** | Schema definition for a catalog entity type | Entity kind (Component, API) |
| **Entity** | Instance of a blueprint | Entity instance |
| **Relation** | Typed connection between entities | Entity relation |
| **Action** | Self-service operation a developer can trigger | Scaffolder template step |
| **Scorecard** | Automated quality/maturity scoring | No direct equivalent |

### Port.io Blueprint Example

```json
{
  "identifier": "microservice",
  "title": "Microservice",
  "schema": {
    "properties": {
      "language": {
        "type": "string",
        "enum": ["go", "java", "python", "typescript"]
      },
      "tier": {
        "type": "string",
        "enum": ["tier-1", "tier-2", "tier-3"]
      },
      "pciScope": {
        "type": "boolean",
        "title": "PCI DSS in scope"
      }
    }
  },
  "relations": {
    "team": {
      "target": "team",
      "required": true,
      "many": false
    }
  }
}
```

### Backstage vs. Port.io Comparison

| Feature | Backstage | Port.io |
|---------|-----------|---------|
| Licensing | Open source (Apache 2.0) | Commercial SaaS |
| Hosting | Self-hosted (your infrastructure) | SaaS (managed) |
| Customization | High — plugin architecture | Medium — configuration-driven |
| Setup effort | High — requires engineering investment | Low — no-code/low-code |
| Scorecards | Via plugins | Built-in |
| API-first | Yes (REST API) | Yes (REST API) |
| CNCF affiliation | Incubating project | Commercial product |

---

## Exam Tips

- **Know the five Backstage entity kinds** — Component, API, System, Domain, Resource
- **`catalog-info.yaml`** is the registration file that lives in each repository
- **Relations are declared in `spec`** fields like `dependsOn`, `providesApis`, `consumesApis`
- **TechDocs** follows the docs-as-code approach using MkDocs
- **Backstage is CNCF Incubating** — know this for questions about CNCF project maturity
- **Port.io uses Blueprints** as the schema definition concept (vs. Backstage's entity kinds)
- A **Location entity** is used to reference multiple catalog files from a single file
- The **Catalog API** makes catalog data available to other automation tools
