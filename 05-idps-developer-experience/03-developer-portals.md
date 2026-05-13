# Developer Portals

## Overview

A developer portal is the unified, web-based interface through which developers interact with all platform capabilities. It surfaces the service catalog, scaffolding tools, documentation, observability data, CI/CD status, and self-service workflows in one place. Rather than switching between dozens of tools, developers have a single pane of glass tailored to their workflows.

Backstage — the open-source developer portal framework created by Spotify and donated to the CNCF — has become the de facto standard for cloud-native developer portals.

---

## Why Developer Portals Exist

Without a developer portal, developers navigate a fragmented landscape:

| Task | Separate Tool Required |
|------|----------------------|
| Find service owner | Slack, email, tribal knowledge |
| Read service documentation | Confluence, Notion, GitHub Wiki, internal sites |
| Check CI/CD status | Jenkins, GitHub Actions, Argo CD (each separately) |
| View metrics and alerts | Grafana, Datadog, Prometheus (each separately) |
| Create a new service | Manual — copy a colleague's repo, modify configs |
| Request infrastructure | Jira ticket → wait → follow up |
| Find on-call contact | PagerDuty, spreadsheet, Slack |

A developer portal collapses this fragmentation into a single, searchable, consistent interface.

---

## Backstage Architecture

Backstage is built on a **plugin architecture** — a monorepo with a core framework and an ecosystem of plugins that add functionality.

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Backstage App                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   Frontend   │  │   Backend    │  │     Plugins      │  │
│  │  (React SPA) │  │  (Node.js)   │  │ (frontend+backend│  │
│  │              │  │              │  │  per plugin)     │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│                            │                                 │
│               ┌────────────┴──────────────┐                 │
│               │     Core Services         │                 │
│               │ - Catalog API             │                 │
│               │ - Identity (Auth)         │                 │
│               │ - Search                  │                 │
│               │ - Scaffolder              │                 │
│               │ - TechDocs                │                 │
│               └────────────┬──────────────┘                 │
└────────────────────────────┼────────────────────────────────┘
                             │
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
     GitHub/GitLab      Databases          External APIs
     (source of          (PostgreSQL,      (PagerDuty,
      truth for          SQLite)           Grafana, etc.)
      catalog)
```

### Frontend

- Built with **React** and **Material UI**
- Each plugin contributes React components, pages, and sidebar items
- **App.tsx** — the root configuration file that registers plugins and routes
- **Entity pages** — composed from plugin cards (each plugin adds a tab or card to entity pages)

### Backend

- Built with **Node.js** (TypeScript)
- Provides REST APIs consumed by the frontend
- Integrates with external systems (GitHub, Kubernetes, databases)
- Runs catalog ingestion, scaffolder actions, TechDocs building

### Key Backend Services

| Service | Purpose |
|---------|---------|
| Catalog Backend | Ingests, stores, and serves catalog entities |
| Scaffolder Backend | Executes template steps (git operations, API calls) |
| TechDocs Backend | Builds and serves MkDocs documentation |
| Auth Backend | Handles authentication (OIDC, OAuth2, LDAP) |
| Search Backend | Indexes and queries catalog, docs, and other content |

### Configuration: app-config.yaml

```yaml
# app-config.yaml — primary Backstage configuration
app:
  title: MyOrg Developer Portal
  baseUrl: https://portal.myorg.io

organization:
  name: My Organization

backend:
  baseUrl: https://portal.myorg.io
  database:
    client: pg
    connection:
      host: ${POSTGRES_HOST}
      port: 5432
      user: ${POSTGRES_USER}
      password: ${POSTGRES_PASSWORD}
      database: backstage

auth:
  environment: production
  providers:
    github:
      production:
        clientId: ${AUTH_GITHUB_CLIENT_ID}
        clientSecret: ${AUTH_GITHUB_CLIENT_SECRET}

integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}

catalog:
  providers:
    github:
      my-org:
        organization: my-org
        catalogPath: /catalog-info.yaml
        filters:
          branch: main
        schedule:
          frequency: { minutes: 30 }
          timeout: { minutes: 3 }

techdocs:
  builder: external
  generator:
    runIn: docker
  publisher:
    type: awsS3
    awsS3:
      bucketName: my-org-techdocs
      region: eu-west-1
```

---

## Software Templates (Scaffolder)

The **Scaffolder** is Backstage's golden path engine. It allows platform teams to define templates that developers use to create new services, libraries, and infrastructure via a guided wizard in the portal.

### Scaffolder Execution Flow

```
1. Developer browses Templates in portal
2. Selects a template (e.g., "Go gRPC Microservice")
3. Fills in a form (service name, owner, region)
4. Backstage Scaffolder executes steps:
   - Fetch skeleton directory from template repo
   - Apply variable substitution (Nunjucks templating)
   - Create GitHub repository
   - Push rendered files
   - Register entity in catalog
   - Provision initial infrastructure (optional)
5. Developer receives links to new repo and catalog entry
```

### Built-in Scaffolder Actions

| Action | Description |
|--------|-------------|
| `fetch:template` | Fetches and renders a Cookiecutter-style template |
| `fetch:plain` | Copies files without templating |
| `publish:github` | Creates a GitHub repo and pushes files |
| `publish:gitlab` | Creates a GitLab project and pushes files |
| `catalog:register` | Registers a new entity in the catalog |
| `github:actions:dispatch` | Triggers a GitHub Actions workflow |
| `http:backstage:request` | Makes an HTTP request to Backstage APIs |
| `debug:log` | Logs a message (useful for debugging templates) |

### Custom Scaffolder Actions

Platform teams can write custom actions for organization-specific operations:

```typescript
// custom-action: create-jira-project
import { createTemplateAction } from '@backstage/plugin-scaffolder-node';
import { JiraClient } from '../clients/jira';

export const createJiraProjectAction = () => {
  return createTemplateAction<{ projectKey: string; projectName: string }>({
    id: 'myorg:jira:create-project',
    description: 'Creates a Jira project for the new service',
    schema: {
      input: {
        required: ['projectKey', 'projectName'],
        properties: {
          projectKey: { type: 'string' },
          projectName: { type: 'string' },
        },
      },
    },
    async handler(ctx) {
      const client = new JiraClient(ctx.secrets?.jiraToken);
      await client.createProject({
        key: ctx.input.projectKey,
        name: ctx.input.projectName,
      });
      ctx.output('projectUrl', `https://jira.myorg.io/projects/${ctx.input.projectKey}`);
    },
  });
};
```

---

## Key Backstage Plugins

Plugins extend Backstage with integrations to external tools. They are typically installed as npm packages and registered in the Backstage app.

### Kubernetes Plugin

Provides visibility into Kubernetes resources directly on entity pages. Shows pod status, deployments, replica sets, and Kubernetes events.

```yaml
# catalog-info.yaml — link service to Kubernetes resources
metadata:
  annotations:
    backstage.io/kubernetes-id: payments-api
    backstage.io/kubernetes-namespace: payments
    backstage.io/kubernetes-label-selector: app=payments-api
```

```yaml
# app-config.yaml — Kubernetes cluster configuration
kubernetes:
  serviceLocatorMethod:
    type: multiTenant
  clusterLocatorMethods:
    - type: config
      clusters:
        - url: https://k8s-prod.myorg.io
          name: production
          authProvider: serviceAccount
          serviceAccountToken: ${K8S_SA_TOKEN}
          caData: ${K8S_CA_DATA}
```

### Argo CD Plugin

Surfaces Argo CD application status, sync state, and deployment history on entity pages.

```yaml
# catalog-info.yaml — link to Argo CD application
metadata:
  annotations:
    argocd/app-name: payments-api-production
    # or for multiple apps:
    argocd/app-selector: service=payments-api
```

### GitHub Actions Plugin

Shows CI/CD workflow runs and their statuses directly in Backstage.

```yaml
# catalog-info.yaml — link to GitHub Actions
metadata:
  annotations:
    github.com/project-slug: my-org/payments-api
```

### Grafana Plugin

Embeds Grafana dashboards and surfaces active alerts on entity pages.

```yaml
# catalog-info.yaml — link to Grafana dashboards
metadata:
  annotations:
    grafana/dashboard-selector: "title contains 'Payments'"
    grafana/alert-label-selector: service=payments-api
```

### PagerDuty Plugin

Shows on-call schedule, active incidents, and allows triggering incidents directly from Backstage.

```yaml
# catalog-info.yaml — link to PagerDuty service
metadata:
  annotations:
    pagerduty.com/service-id: P7K2MN1
```

### Additional Notable Plugins

| Plugin | Purpose |
|--------|---------|
| `@backstage/plugin-cost-insights` | Show cloud cost data per team/service |
| `@backstage/plugin-lighthouse` | Web performance audits |
| `@roadiehq/backstage-plugin-security-insights` | GitHub security advisories |
| `@backstage/plugin-sonarqube` | Code quality metrics from SonarQube |
| `@backstage/plugin-vault` | HashiCorp Vault secret browsing |
| `@backstage/plugin-datadog` | Datadog dashboards and monitors |
| `@backstage/plugin-sentry` | Sentry error tracking |

---

## CNCF Projects in the IDP Space

| Project | CNCF Maturity | Role in IDP |
|---------|---------------|-------------|
| **Backstage** | Incubating | Developer portal framework |
| **Crossplane** | Graduated | Infrastructure self-service (Kubernetes-native) |
| **Argo CD** | Graduated | GitOps deployment (shown in portal) |
| **Argo Workflows** | Incubating | Pipeline automation |
| **Tekton** | Incubating | Cloud-native CI/CD pipelines |
| **Flux** | Graduated | GitOps (alternative to Argo CD) |
| **KubeVela** | Incubating | Application-centric delivery |
| **Dapr** | Incubating | Distributed application runtime |
| **OpenTelemetry** | Graduated | Observability data collection |

### Non-CNCF IDP Tools Worth Knowing

| Tool | Type | Key Feature |
|------|------|-------------|
| **Port.io** | Commercial SaaS | No-code service catalog with Scorecards |
| **Cortex** | Commercial SaaS | Service catalog with maturity scoring |
| **OpsLevel** | Commercial SaaS | Service maturity and reliability |
| **Humanitec** | Commercial | Platform orchestrator |
| **Radius** | Open source (Microsoft) | Cloud-agnostic application model |

---

## How Portals Improve Onboarding

New developer onboarding is one of the highest-impact use cases for a developer portal. The time from "first day" to "first production deployment" (T2FPD) is a measurable, meaningful metric.

### Onboarding Journey With and Without a Portal

**Without Portal:**
1. Read a 200-page wiki (outdated)
2. Ask colleagues where things are
3. Request GitHub access via ticket (2-day SLA)
4. Request cluster credentials via ticket (3-day SLA)
5. Clone a "reference" repo and delete unwanted files
6. Manually configure CI/CD
7. First deployment: week 3

**With Portal:**
1. Log in to developer portal (SSO, day one)
2. Browse service catalog to understand the landscape
3. Read TechDocs for team-specific onboarding guide
4. Use Scaffolder to create first service from golden path template (20 minutes)
5. New repo created, CI/CD configured, catalog entry registered automatically
6. First deployment: day one or two

### Onboarding Accelerators in Backstage

| Feature | Onboarding Benefit |
|---------|-------------------|
| Service catalog | Understand existing services without asking |
| TechDocs | Find runbooks and architecture docs without searching |
| Scaffolder templates | Create first service with zero prior platform knowledge |
| Team pages | Find colleagues, Slack channels, on-call contacts |
| Kubernetes plugin | Understand deployment state without `kubectl` expertise |
| Search | Full-text search across catalog, docs, and APIs |

---

## Deploying and Operating Backstage

### Deployment Topology

```
┌─────────────────────────────────────────────┐
│              Kubernetes Cluster             │
│  ┌────────────────┐  ┌────────────────────┐ │
│  │ backstage pod  │  │  PostgreSQL (RDS)  │ │
│  │ (frontend +    │  │  or Cloud SQL      │ │
│  │  backend)      │  └────────────────────┘ │
│  └────────────────┘                         │
│          │                                  │
│  ┌───────┴──────────────────────────────┐   │
│  │  Ingress (TLS termination)           │   │
│  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
         │
   Object Storage (S3/GCS)
   for TechDocs static sites
```

### Backstage Helm Chart

```yaml
# values.yaml for official Backstage Helm chart
backstage:
  image:
    registry: ghcr.io
    repository: my-org/backstage
    tag: latest
  appConfig:
    app:
      baseUrl: https://portal.myorg.io
    backend:
      baseUrl: https://portal.myorg.io

postgresql:
  enabled: true
  auth:
    secretKeys:
      adminPasswordKey: postgres-password

ingress:
  enabled: true
  host: portal.myorg.io
  tls:
    - secretName: backstage-tls
      hosts:
        - portal.myorg.io
```

---

## Exam Tips

- **Backstage is CNCF Incubating** — not Graduated (as of 2024–2025)
- **Plugins** are the extension mechanism — know what each major plugin does
- **Scaffolder templates** are for golden paths — they create repos, register catalog entries, provision resources
- **TechDocs** is docs-as-code using MkDocs — documentation lives with the code
- **app-config.yaml** is the primary Backstage configuration file
- Know the **difference between frontend and backend plugins**
- **Entity pages** are composed from plugin tabs and cards — each plugin contributes a view
- A developer portal's primary value is **reducing cognitive load and onboarding time**
- **Crossplane** (CNCF Graduated) is the standard for Kubernetes-native self-service infrastructure
