# Continuous Delivery and GitOps

## Continuous Delivery vs. Continuous Deployment

These terms are related but distinct:

| Term | Definition |
|---|---|
| **Continuous Integration (CI)** | Automatically build and test on every commit |
| **Continuous Delivery (CD)** | Code is always in a deployable state; deployment to production is a manual trigger |
| **Continuous Deployment** | Every commit that passes CI is automatically deployed to production |

Most organizations practice **Continuous Delivery** — the ability to deploy at any time — while keeping production deployments as an intentional act (even if automated).

---

## Continuous Delivery Principles

1. **Build quality in** — problems are caught early; don't test at the end
2. **Work in small batches** — smaller changes are easier to test, debug, and roll back
3. **Automate repeatable processes** — humans for creative work; machines for repetitive work
4. **Relentlessly pursue continuous improvement** — measure and improve delivery metrics
5. **Everyone is responsible for delivery** — not just ops or a release team

---

## Deployment Strategies

Different strategies balance risk and speed:

### Rolling Update

Replace old pods with new ones gradually. Default Kubernetes strategy.

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1    # Max pods unavailable at a time
      maxSurge: 1          # Max extra pods during update
```

Pros: Zero downtime, gradual rollout
Cons: Briefly runs two versions simultaneously; hard to roll back quickly

### Blue-Green Deployment

Run two identical production environments (blue and green). Switch traffic from old (blue) to new (green) all at once.

```
         ┌─────────────┐
Users ──►│  Load Bal.  │
         └──────┬──────┘
                │ switch
    ┌───────────┼───────────┐
    ▼                       ▼
┌──────────┐         ┌──────────┐
│  Blue    │         │  Green   │
│ (v1.0)  │         │ (v2.0)  │
└──────────┘         └──────────┘
```

Pros: Instant rollback (switch back to blue); clean cutover
Cons: Requires double the resources

### Canary Release

Route a small percentage of traffic to the new version. Gradually increase if healthy.

```
Users ──► 95% → v1.0 (stable)
      └──►  5% → v2.0 (canary)
```

Tools: Argo Rollouts, Flagger, service mesh traffic splitting (Istio, Linkerd)

Pros: Real user traffic tests new version; limit blast radius
Cons: Two versions running simultaneously; requires traffic splitting infrastructure

### Feature Flags

Deploy code with features disabled. Enable for specific users or percentages without redeployment.

Tools: LaunchDarkly, Flagsmith, Unleash, Flipt

---

## GitOps

### What Is GitOps?

GitOps is an operational framework where **Git is the single source of truth for infrastructure and application state**.

Coined by Weaveworks in 2017. The four GitOps principles (OpenGitOps):

1. **Declarative**: System state is expressed declaratively (YAML manifests, not imperative scripts)
2. **Versioned and immutable**: Desired state is stored in Git with full history; changes are commits
3. **Pulled automatically**: Software agents pull desired state from Git (not pushed by CI pipelines)
4. **Continuously reconciled**: Software agents detect and correct drift from desired state

### GitOps vs. Traditional CI/CD

| Traditional Push-Based CD | GitOps Pull-Based CD |
|---|---|
| CI pipeline pushes deployments to cluster | Operator in cluster pulls from Git |
| CI system needs cluster credentials | Cluster credentials never leave the cluster |
| Hard to detect configuration drift | Drift is detected and corrected automatically |
| Less auditability | Every change is a Git commit with author and message |

### GitOps Workflow

```
Developer opens PR
        ↓
CI pipeline runs tests, builds image, updates manifest with new image tag
        ↓
PR is reviewed and merged to main
        ↓
GitOps operator detects change in Git
        ↓
Operator applies new desired state to cluster
        ↓
Operator monitors and reconciles (heals drift)
```

### ArgoCD

ArgoCD is the most popular GitOps continuous delivery tool for Kubernetes.

Key concepts:
- **Application**: An ArgoCD resource that maps a Git repo path to a Kubernetes cluster/namespace
- **Sync**: The act of applying Git state to the cluster
- **Health**: ArgoCD tracks health of deployed resources
- **Sync policies**: Manual or automatic sync; prune (delete removed resources); self-heal

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/my-org/my-app-config
    targetRevision: HEAD
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true       # Delete resources removed from Git
      selfHeal: true    # Revert manual cluster changes
```

ArgoCD UI provides:
- Visual diff between Git state and cluster state
- Deployment history with rollback
- Health status of all resources
- Manual sync trigger

### Flux

Flux is another popular GitOps operator, now a CNCF graduated project.

Flux components:
- **Source Controller**: Watches Git repos, Helm repos, OCI registries
- **Kustomize Controller**: Applies Kustomize resources
- **Helm Controller**: Manages Helm releases
- **Image Automation Controller**: Updates image tags in Git automatically

```yaml
# Flux GitRepository
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/my-org/my-app-config
  ref:
    branch: main
```

### ArgoCD vs. Flux

| Feature | ArgoCD | Flux |
|---|---|---|
| UI | Yes (built-in) | No (third-party) |
| Multi-cluster | Yes (ArgoCD manages remote clusters) | Yes (each cluster runs its own Flux) |
| Architecture | Central server + per-cluster agents | Fully distributed |
| Helm support | Yes | Yes |
| Kustomize support | Yes | Yes |
| Image update automation | ArgoCD Image Updater | Flux Image Automation Controller |

---

## Environment Promotion in GitOps

GitOps promotes changes through environments by updating Git state for each environment.

```
Git Repository Structure:
environments/
├── dev/
│   └── my-app/
│       └── kustomization.yaml   (image: my-app:abc123)
├── staging/
│   └── my-app/
│       └── kustomization.yaml   (image: my-app:xyz789)
└── production/
    └── my-app/
        └── kustomization.yaml   (image: my-app:def456)
```

Promoting from dev to staging = creating a PR that updates the image tag in `staging/my-app/kustomization.yaml`.

---

## Key Takeaways

- Continuous Delivery = always deployable; Continuous Deployment = automatically deployed
- Deployment strategies: rolling (default), blue-green (instant rollback), canary (gradual rollout)
- GitOps principles: declarative, versioned, pulled, continuously reconciled
- GitOps is pull-based: cluster operators pull from Git; CI systems never push to clusters
- ArgoCD: central UI-based GitOps operator; most popular tool
- Flux: distributed, composable GitOps; CNCF graduated project
- Environment promotion = PR updating image tag in environment-specific Git path
