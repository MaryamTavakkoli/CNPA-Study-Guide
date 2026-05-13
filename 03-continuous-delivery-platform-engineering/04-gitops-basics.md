# GitOps Fundamentals

## What Is GitOps?

**GitOps** is an operational framework that applies DevOps best practices — version control, collaboration, compliance, CI/CD — to infrastructure and application deployment. The core idea is simple: if you can describe your desired system state completely in Git, then Git becomes your system of record for operations.

The term was coined by Weaveworks in 2017. The CNCF's **OpenGitOps** working group has since produced a formal, vendor-neutral definition built on four principles.

---

## The Four OpenGitOps Principles

These are the canonical definition of GitOps. Exam questions will reference these directly.

### Principle 1: Declarative

> "A system managed by GitOps must have its desired state expressed declaratively."

**What it means**: You describe *what* you want (desired state), not *how* to get there (imperative commands). Kubernetes YAML manifests, Helm values files, and Terraform HCL are declarative. `kubectl apply` commands in a shell script are not.

**Why it matters**: Declarative descriptions are idempotent — applying the same state twice produces the same result. This is essential for automated reconciliation.

```yaml
# Declarative: describes desired state
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: myapp
          image: myapp:sha-a1b2c3d4
```

vs.

```bash
# Imperative: describes steps (NOT GitOps)
kubectl scale deployment myapp --replicas=3
kubectl set image deployment/myapp myapp=myapp:sha-a1b2c3d4
```

---

### Principle 2: Versioned and Immutable

> "The desired state is stored in a way that enforces immutability, versioning, and retains a complete version history."

**What it means**: Git itself is the storage mechanism. Every change to desired state is a Git commit with a SHA, author, timestamp, and message. You cannot change history without leaving a trace. You can always roll back by reverting a commit.

**Why it matters**: Git's immutable history gives you a complete audit trail of every change to your infrastructure — who changed what, when, and why (commit message). This is invaluable for compliance, debugging, and post-mortems.

```bash
# Git history is your deployment audit trail
git log --oneline services/payments/

a1b2c3d feat: increase payments-api replicas to 5
e4f5a6b fix: rollback to previous image after memory leak
7c8d9e0 feat: deploy payments-api v2.3.1
```

---

### Principle 3: Pulled Automatically

> "Software agents automatically pull the desired state declarations from the source."

**What it means**: A software agent running *inside* the target environment (typically in the Kubernetes cluster) watches the Git repository and pulls changes. The pipeline does *not* push to the cluster.

**Why it matters**: The pull model is more secure. The cluster's API server does not need to be exposed to the CI system. The agent running in the cluster has cluster-internal credentials and communicates outbound to Git. This is far better than giving a CI pipeline full cluster admin credentials.

```
PUSH MODEL (traditional CD):
  CI Pipeline ──[kubectl apply]──► Kubernetes API Server
  (CI needs cluster credentials; cluster API must be reachable from CI)

PULL MODEL (GitOps):
  Git ◄──[watch]── GitOps Agent (in cluster)
                         │
                         └──[kubectl apply]──► Kubernetes API Server
  (agent needs only Git read access + in-cluster permissions)
```

---

### Principle 4: Continuously Reconciled

> "Software agents continuously observe actual system state and attempt to apply the desired state."

**What it means**: The agent doesn't just apply changes once when a Git commit arrives. It runs a continuous control loop, comparing the actual state of the cluster to the desired state in Git. If they differ (for any reason — manual change, node failure, someone ran `kubectl edit`), the agent reconciles back to desired state.

**Why it matters**: This is the GitOps answer to **drift**. In a traditional CD system, if someone makes an emergency manual change to production, it persists. In a GitOps system, the agent will detect and revert it (or alert on it).

```
Reconciliation Loop:
┌─────────────────────────────────────────────┐
│                                             │
│  1. Read desired state from Git             │
│  2. Read actual state from cluster          │
│  3. Compare: desired vs. actual             │
│  4. If diff:                                │
│       a. Apply changes to converge to desired│
│       b. Report sync status                 │
│  5. Wait (30s / 3m / configurable)          │
│  6. Go to 1                                 │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Git as Source of Truth

"Git is the source of truth" is more than a slogan. In a GitOps system:

- No change to a production system is valid unless it originates as a Git commit
- The current state of Git represents the desired current state of production
- If Git and production disagree, Git wins (or an alert fires)
- Disaster recovery means restoring the cluster and letting the GitOps agent converge it back to the Git state

This implies that **the quality of your Git repository** directly affects the reliability of your system. Platform teams must:

- Enforce branch protection (no direct commits to the main branch in the config repo)
- Require code reviews for all changes to the config repo
- Use signed commits for compliance-sensitive environments
- Ensure the config repo is highly available (hosted on a managed Git service)

---

## Pull vs. Push Model: Deep Comparison

| Dimension | Pull (GitOps) | Push (Traditional CD) |
|-----------|--------------|----------------------|
| **Initiator** | Agent inside cluster | CI/CD pipeline outside cluster |
| **Credentials** | Agent has cluster-internal access | CI system needs cluster API credentials |
| **Network** | Agent calls out to Git (outbound) | CI calls in to cluster API (inbound) |
| **Cluster exposure** | API server not exposed to CI | API server must be reachable from CI |
| **Drift handling** | Continuous reconciliation detects & fixes drift | No drift detection; manual intervention |
| **Audit trail** | Git history + agent sync logs | CI pipeline logs |
| **Rollback** | `git revert` → agent converges | Re-run pipeline or manual kubectl |
| **Multi-cluster** | Agent per cluster; each pulls from Git | CI must have credentials for all clusters |
| **Security posture** | Better (no external credential exposure) | Weaker (CI holds cluster credentials) |

---

## Drift Detection

**Drift** occurs when the actual state of a system deviates from the desired state declared in Git. Common causes:

- Manual emergency patches (`kubectl edit`, `kubectl exec` with state changes)
- External controllers modifying resources
- Node failure causing replicas to drop
- Someone directly applying a manifest without going through Git

GitOps tools handle drift in one of two modes:

| Mode | Behavior |
|------|---------|
| **Auto-sync** | Agent detects drift and automatically reconciles to desired state. Risky for some changes; excellent for stateless workloads. |
| **Manual sync with alert** | Agent detects drift and fires an alert; human approves the re-sync. Preferred when auto-correction could be dangerous. |

### ArgoCD Sync Policy

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: payments-api
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/myorg/platform-config
    targetRevision: HEAD
    path: services/payments-api/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: payments
  syncPolicy:
    automated:
      prune: true        # remove resources deleted from Git
      selfHeal: true     # revert manual changes (drift correction)
    syncOptions:
      - CreateNamespace=true
```

---

## The Role of Image Tags in GitOps

Image tags are the critical link between CI (which builds images) and GitOps (which deploys them).

### Anti-patterns

```yaml
# ANTI-PATTERN: mutable tag
image: myapp:latest
# "latest" is re-pointed on every build. The GitOps agent sees no change
# in the manifest even when a new image is pushed.

# ANTI-PATTERN: branch name tag
image: myapp:main
# Same problem — tag pointer changes, manifest doesn't.
```

### Correct Patterns

```yaml
# PATTERN 1: Git SHA tag (precise, immutable)
image: myapp:sha-a1b2c3d4e5f6

# PATTERN 2: Semantic version tag (for release-based workflows)
image: myapp:2.3.1

# PATTERN 3: Image digest (most immutable — digest is the hash of the image content)
image: myapp@sha256:abc123def456...
```

### How Image Tags Flow Through GitOps

```
CI builds image → pushes to registry as myapp:sha-a1b2c3d4
CI (or image update automation) updates Git manifest:
  image: myapp:sha-a1b2c3d4  ← commit to config repo
ArgoCD/Flux detects change in config repo
Agent syncs: kubectl apply with new image tag
New pods roll out with updated image
```

---

## Multi-Repo vs. Mono-Repo GitOps Strategies

### Mono-Repo GitOps

All application manifests and platform configuration in a single repository.

```
platform-config/
├── clusters/
│   ├── prod/
│   └── staging/
├── services/
│   ├── payments-api/
│   ├── orders-api/
│   └── inventory-api/
└── infrastructure/
    ├── namespaces/
    └── rbac/
```

**Pros**: Single place to look; easy cross-service changes; simple access control  
**Cons**: At scale, the repo becomes large; many teams fighting over the same repo; CI runs on every change regardless of scope

### Multi-Repo GitOps

Application manifests live in each application's own repository; infrastructure has its own repository.

```
payments-api-repo/
├── src/
├── Dockerfile
├── .github/workflows/
└── deploy/        ← manifests for this service only

orders-api-repo/
└── deploy/

platform-config-repo/
├── infrastructure/
├── rbac/
└── cluster-addons/
```

**Pros**: Clear ownership; teams are autonomous; smaller, faster repos; security boundaries  
**Cons**: Cross-service changes require multiple PRs; harder to get a global view

### Recommended Pattern

Most mature organizations use a **hybrid**: application code + application manifests in the app repo, but a separate **platform config repo** for infrastructure, RBAC, cluster add-ons, and cross-cutting concerns.

---

## ArgoCD App of Apps Pattern

The **App of Apps** pattern solves a bootstrapping problem: if ArgoCD manages `Application` resources, how do you manage ArgoCD itself and all those `Application` resources via GitOps?

The solution: create a single "parent" `Application` that points to a directory of `Application` manifests. ArgoCD syncs the parent, which creates all the child `Application` resources, which then each sync their own services.

```yaml
# Parent Application (the "App of Apps")
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/myorg/platform-config
    targetRevision: HEAD
    path: argocd/applications     # directory containing child Application manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

```yaml
# argocd/applications/payments-api.yaml  (child Application)
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: payments-api
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/myorg/platform-config
    targetRevision: HEAD
    path: services/payments-api/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: payments
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

```yaml
# argocd/applications/orders-api.yaml  (another child Application)
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: orders-api
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/myorg/platform-config
    targetRevision: HEAD
    path: services/orders-api/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: orders
```

### App of Apps Tree

```
root-app (ArgoCD Application)
├── payments-api (Application)
│   └── Deployment, Service, HPA, PodDisruptionBudget
├── orders-api (Application)
│   └── Deployment, Service, ConfigMap
├── inventory-api (Application)
│   └── Deployment, Service
└── platform-addons (Application)
    └── cert-manager, external-secrets, metrics-server
```

Adding a new service is as simple as adding a new `Application` manifest to the `argocd/applications/` directory and opening a PR.

### ApplicationSet: The App of Apps at Scale

**ApplicationSet** is an ArgoCD resource that generates `Application` resources dynamically using generators (Git directory, cluster list, list generator, PR generator). It is the scalable evolution of the App of Apps pattern.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: all-services
  namespace: argocd
spec:
  generators:
    - git:
        repoURL: https://github.com/myorg/platform-config
        revision: HEAD
        directories:
          - path: services/*   # generate one Application per service directory
  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      source:
        repoURL: https://github.com/myorg/platform-config
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.basename}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

---

## GitOps Tools: ArgoCD vs. Flux

| Feature | ArgoCD | Flux |
|---------|--------|------|
| **Architecture** | Centralized UI + API + controllers | Controller-only; GitOps toolkit |
| **UI** | Rich web UI out of the box | Third-party dashboards (Weave GitOps) |
| **Multi-tenancy** | Strong with Projects + RBAC | Strong with multi-tenancy add-on |
| **Kustomize** | Native support | Native support |
| **Helm** | Native support | Native support (HelmRelease CRD) |
| **Image update automation** | ArgoCD Image Updater (separate) | Built-in (Flux Image Automation) |
| **Progressive delivery** | Argo Rollouts (separate project) | Flagger integration |
| **CNCF status** | Graduated | Graduated |
| **Primary CRD** | `Application` | `GitRepository`, `Kustomization`, `HelmRelease` |

---

## Key Exam Takeaways

- Know all **four OpenGitOps principles** by name: Declarative, Versioned & Immutable, Pulled Automatically, Continuously Reconciled
- **Pull vs. push**: GitOps uses pull; understand why this is more secure
- **Drift detection** is a core GitOps capability — not all CD systems have it
- Image tags must be **immutable** — `latest` is an anti-pattern in GitOps
- **App of Apps** is ArgoCD's pattern for managing multiple `Application` resources via GitOps
- **ApplicationSet** generates `Application` resources dynamically and is the scalable evolution of App of Apps
- Both ArgoCD and Flux are CNCF **Graduated** projects
- Multi-repo vs. mono-repo is a design decision; hybrid is common (app code + app manifests in app repo; platform config in separate repo)
