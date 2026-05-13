# GitOps for Application Environments

## Environments as Code

In a GitOps workflow, every environment — development, staging, production, and ephemeral preview environments — is defined as code in a Git repository. This means:

- Environments are reproducible: a lost cluster can be rebuilt by running `git clone && argocd sync`
- Environment drift is detectable and correctable
- The path from dev to prod is a series of Git operations (PRs, merges, tag pushes)
- Audit trail of every environment change exists in Git history

This document covers the practical patterns for structuring environments in GitOps, automating image updates, creating ephemeral environments, and delivering software progressively.

---

## Environment Promotion via Pull Requests

The GitOps-native way to promote a new version of a service across environments is through **Pull Requests (PRs)**. Instead of a pipeline `kubectl set image` command, the promotion is:

1. Open a PR that updates the image tag in the target environment's manifest
2. CI validates the manifest (linting, policy checks)
3. A reviewer approves (for staging → prod) or automation auto-merges (for dev → staging)
4. GitOps agent detects the merge and syncs the environment

This approach gives you:
- Peer review for every production change
- A rollback mechanism: just revert the PR
- A clear record of what was deployed when and by whom

### Promotion PR Flow

```
CI builds myapp:sha-a1b2c3d4
      │
      ▼
Automated PR: update image in dev environment
      │ auto-merged (no review required)
      ▼
ArgoCD syncs dev → myapp:sha-a1b2c3d4 running in dev
      │
      ▼
Dev tests pass (automated) → promotion PR opened for staging
      │ reviewed by team lead
      ▼
ArgoCD syncs staging → myapp:sha-a1b2c3d4 running in staging
      │
      ▼
Staging tests pass → promotion PR opened for production
      │ requires 2 approvals + product owner sign-off
      ▼
ArgoCD syncs prod → myapp:sha-a1b2c3d4 running in production
```

---

## Directory-per-Environment vs. Branch-per-Environment

These are the two primary strategies for organizing environment configuration in a GitOps repository.

### Branch-per-Environment

Each environment has its own branch (`dev`, `staging`, `main`/`prod`). Promotion is a merge from one branch to the next.

```
repo/
  branch: dev        → dev cluster
  branch: staging    → staging cluster
  branch: main       → production cluster
```

**Promotion**: `git merge dev staging`, `git merge staging main`

| Pros | Cons |
|------|------|
| Familiar Git workflow | Branch divergence — environments drift apart over time |
| Easy to see what's in each environment | Cherry-picks for hotfixes are complex |
| Tooling support is mature | Hard to apply a cross-environment change (e.g., add a label to all envs) |
| | Merge conflicts require resolution per promotion |

### Directory-per-Environment

All environments live in the same branch; each environment has its own directory.

```
platform-config/          (single branch: main)
├── services/
│   └── payments-api/
│       ├── base/          ← shared Kustomize base
│       │   ├── deployment.yaml
│       │   └── service.yaml
│       └── overlays/
│           ├── dev/
│           │   └── kustomization.yaml  ← image: myapp:sha-latest
│           ├── staging/
│           │   └── kustomization.yaml  ← image: myapp:sha-a1b2c3d4
│           └── production/
│               └── kustomization.yaml  ← image: myapp:sha-e5f6g7h8
```

**Promotion**: open a PR that updates the image tag in the target overlay.

| Pros | Cons |
|------|------|
| All environments visible in one place | Requires discipline: never let prod tag get into dev overlay |
| No branch divergence | Single branch means one PR can accidentally touch multiple envs |
| Cross-environment changes in a single PR | Kustomize or Helm overlays add tooling requirement |
| Cleaner for GitOps tools (ArgoCD, Flux) | |

> **Recommended**: Directory-per-environment is the current best practice for GitOps. It is what the CNCF GitOps working group and most ArgoCD/Flux documentation recommends.

---

## Kustomize Overlay Pattern

Kustomize is Kubernetes-native configuration management built into `kubectl`. It uses a `base` + `overlays` pattern that maps cleanly to directory-per-environment GitOps.

### Base: shared configuration

```yaml
# services/payments-api/base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payments-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: payments-api
  template:
    metadata:
      labels:
        app: payments-api
    spec:
      containers:
        - name: payments-api
          image: payments-api:PLACEHOLDER   # overridden per environment
          ports:
            - containerPort: 8080
```

```yaml
# services/payments-api/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
```

### Production overlay: environment-specific overrides

```yaml
# services/payments-api/overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
  - ../../base
replicas:
  - name: payments-api
    count: 5              # prod gets more replicas
images:
  - name: payments-api
    newTag: sha-e5f6g7h8  # pinned image tag for this environment
patches:
  - target:
      kind: Deployment
      name: payments-api
    patch: |-
      - op: add
        path: /spec/template/spec/containers/0/resources
        value:
          requests:
            cpu: 500m
            memory: 512Mi
          limits:
            cpu: 1000m
            memory: 1Gi
```

---

## Image Update Automation

Manually opening PRs to update image tags is workable but slow and error-prone at scale. **Image update automation** tools watch a container registry for new image tags and automatically open PRs (or commit directly) to the GitOps repository.

### Flux Image Automation

Flux provides two CRDs for image update automation:

1. **`ImageRepository`**: tells Flux to poll a registry for tags
2. **`ImagePolicy`**: selects the latest tag matching a semver range or regex
3. **`ImageUpdateAutomation`**: commits the new tag to the Git repository

```yaml
# ImageRepository: poll the registry
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: payments-api
  namespace: flux-system
spec:
  image: ghcr.io/myorg/payments-api
  interval: 1m0s

---
# ImagePolicy: select latest patch release in the 2.x series
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: payments-api
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: payments-api
  policy:
    semver:
      range: ">=2.0.0 <3.0.0"

---
# ImageUpdateAutomation: write the new tag back to Git
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 1m0s
  sourceRef:
    kind: GitRepository
    name: flux-system
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        email: fluxcdbot@example.com
        name: fluxcdbot
      messageTemplate: "chore: update {{range .Updated.Images}}{{println .}}{{end}}"
    push:
      branch: main
```

Then annotate the manifest to tell Flux where to update:

```yaml
# In the Kustomization overlay
images:
  - name: payments-api
    newTag: sha-a1b2c3d4  # {"$imagepolicy": "flux-system:payments-api:tag"}
```

### ArgoCD Image Updater

ArgoCD Image Updater is a separate tool (not built into ArgoCD core) that polls registries and updates `Application` annotations or commits to Git.

```yaml
# ArgoCD Application with image updater annotations
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: payments-api
  annotations:
    argocd-image-updater.argoproj.io/image-list: payments-api=ghcr.io/myorg/payments-api
    argocd-image-updater.argoproj.io/payments-api.update-strategy: semver
    argocd-image-updater.argoproj.io/payments-api.allow-tags: regexp:^[0-9]+\.[0-9]+\.[0-9]+$
    argocd-image-updater.argoproj.io/write-back-method: git   # commit to Git repo
```

### Comparison

| Feature | Flux Image Automation | ArgoCD Image Updater |
|---------|----------------------|---------------------|
| Built-in to tool | Yes (Flux) | No (separate install) |
| Commits to Git | Yes | Yes |
| Semver policy | Yes | Yes |
| Regex tag filter | Yes | Yes |
| Multi-registry | Yes | Yes |
| Maturity | Stable | Beta |

---

## Ephemeral Environments

An **ephemeral environment** (also called a preview environment or PR environment) is a fully functional, isolated deployment of an application that is automatically created for each pull request and destroyed when the PR is closed or merged.

### Benefits

- Reviewers can interact with the actual deployed application, not just read code
- Integration issues are caught before merge
- Product owners can verify features in a real environment
- No long-lived shared dev environment becomes a bottleneck

### How Platform Teams Provision Ephemeral Environments

The typical flow:

```
1. Developer opens PR #42 for the payments-api service
2. CI pipeline runs: build, test, push image → payments-api:pr-42-sha-a1b2c3d4
3. PR creation webhook triggers ArgoCD ApplicationSet with PR generator
4. ApplicationSet creates Application: payments-api-pr-42
5. Namespace created: payments-pr-42
6. All dependent services spun up (or pointed at shared staging deps)
7. ArgoCD syncs payments-api:pr-42-sha-a1b2c3d4 into namespace payments-pr-42
8. Ingress created: pr-42-payments.preview.example.com
9. CI posts a comment on the PR with the preview URL
10. PR merged → GitHub webhook fires → ApplicationSet deletes Application
11. ArgoCD prunes all resources in payments-pr-42 → namespace deleted
```

### ArgoCD ApplicationSet with PR Generator

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: payments-api-previews
  namespace: argocd
spec:
  generators:
    - pullRequest:
        github:
          owner: myorg
          repo: payments-api
          tokenRef:
            secretName: github-token
            key: token
          labels:
            - preview   # only PRs with the "preview" label get an environment
        requeueAfterSeconds: 60
  template:
    metadata:
      name: 'payments-api-pr-{{number}}'
    spec:
      source:
        repoURL: https://github.com/myorg/payments-api
        targetRevision: '{{head_sha}}'
        path: deploy/preview
        helm:
          parameters:
            - name: image.tag
              value: 'pr-{{number}}-{{head_sha}}'
            - name: ingress.host
              value: 'pr-{{number}}-payments.preview.example.com'
      destination:
        server: https://kubernetes.default.svc
        namespace: 'payments-pr-{{number}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
```

### Cost Management for Ephemeral Environments

Ephemeral environments can be expensive if not managed. Platform teams should:

- **Auto-destroy on PR close/merge**: critical to avoid zombie environments
- **Inactivity TTL**: destroy environments with no traffic after N hours
- **Resource quotas**: enforce CPU/memory limits on preview namespaces
- **Shared services**: point preview envs at shared (read-only) staging databases rather than spinning up new instances per PR
- **Spot/preemptible nodes**: run ephemeral environment workloads on cheaper node pools

---

## Progressive Delivery

**Progressive Delivery** is an extension of continuous delivery that controls the blast radius of deployments by gradually routing traffic to new versions and automatically rolling back if metrics degrade.

### Why Progressive Delivery?

Even with thorough testing, production traffic patterns, data, and scale can surface issues that never appeared in staging. Progressive delivery limits how many users are affected by a bad release and provides automated safeguards.

### Deployment Strategies

| Strategy | Traffic split | Rollback mechanism | Best for |
|----------|-------------|-------------------|---------|
| **Canary** | New version gets small % of traffic; gradually increases | Auto-rollback on metric failure | Low-risk incremental rollouts |
| **Blue/Green** | 100% traffic on "blue"; deploy to "green"; switch all traffic | Switch back to blue instantly | Zero-downtime major releases |
| **A/B Testing** | User-based routing (feature flag or header) | Disable the test group | Feature comparison experiments |
| **Shadow/Mirroring** | All traffic mirrored to new version; responses discarded | No rollback needed (no real traffic) | Safe testing of new version with real traffic |

### Argo Rollouts

**Argo Rollouts** is a Kubernetes controller that replaces the standard `Deployment` resource with a `Rollout` resource that implements progressive delivery strategies.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: payments-api
spec:
  replicas: 10
  strategy:
    canary:
      canaryService: payments-api-canary
      stableService: payments-api-stable
      trafficRouting:
        nginx:
          stableIngress: payments-api-ingress
      steps:
        - setWeight: 10        # send 10% of traffic to canary
        - pause: {duration: 5m}
        - analysis:
            templates:
              - templateName: success-rate
        - setWeight: 25
        - pause: {duration: 10m}
        - analysis:
            templates:
              - templateName: success-rate
        - setWeight: 50
        - pause: {duration: 10m}
        - setWeight: 100       # full rollout
  selector:
    matchLabels:
      app: payments-api
  template:
    metadata:
      labels:
        app: payments-api
    spec:
      containers:
        - name: payments-api
          image: payments-api:sha-a1b2c3d4

---
# AnalysisTemplate: automated rollback trigger
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  metrics:
    - name: success-rate
      interval: 1m
      successCondition: result[0] >= 0.95    # 95% success rate required
      failureLimit: 3
      provider:
        prometheus:
          address: http://prometheus.monitoring.svc.cluster.local:9090
          query: |
            sum(rate(http_requests_total{app="payments-api",status!~"5.."}[5m]))
            /
            sum(rate(http_requests_total{app="payments-api"}[5m]))
```

If the success rate drops below 95% during the analysis step, Argo Rollouts automatically aborts the rollout and returns all traffic to the stable version.

### Flagger

**Flagger** is a Kubernetes operator (originally from Weaveworks, now CNCF) that automates progressive delivery for Flux users but also works standalone.

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: payments-api
  namespace: payments
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: payments-api
  service:
    port: 8080
  analysis:
    interval: 1m
    threshold: 5           # max number of failed metric checks before rollback
    maxWeight: 50          # max canary traffic weight (%)
    stepWeight: 10         # increase by 10% per interval
    metrics:
      - name: request-success-rate
        thresholdRange:
          min: 99
        interval: 1m
      - name: request-duration
        thresholdRange:
          max: 500         # p99 latency must stay under 500ms
        interval: 1m
```

### Argo Rollouts vs. Flagger

| Dimension | Argo Rollouts | Flagger |
|-----------|--------------|---------|
| **Primary ecosystem** | ArgoCD | Flux |
| **Resource type** | Replaces `Deployment` with `Rollout` | Wraps existing `Deployment` |
| **Migration** | Requires manifest change to `Rollout` | Non-invasive (wraps existing resources) |
| **Traffic routing** | Nginx, Istio, ALB, etc. | Nginx, Istio, App Mesh, Gloo, etc. |
| **UI** | Argo Rollouts Dashboard | No built-in UI |
| **CNCF status** | Incubating | CNCF Sandbox |

---

## Putting It All Together: Full GitOps Environment Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                         Git Repository                          │
│                                                                 │
│  services/payments-api/                                         │
│    overlays/                                                    │
│      dev/           ← image: sha-b2c3d4e5 (latest merged)      │
│      staging/       ← image: sha-a1b2c3d4 (promoted from dev)  │
│      production/    ← image: sha-e5f6g7h8 (promoted from stg)  │
│      pr-42/         ← image: pr-42-sha-x1y2z3 (ephemeral)      │
└─────────────────────────────────────────────────────────────────┘
         │              │              │              │
    ArgoCD App     ArgoCD App     ArgoCD App     ArgoCD App
    payments-dev   payments-stg   payments-prod  payments-pr-42
         │              │              │              │
    dev cluster    staging cluster  prod cluster   dev cluster
    (namespace:    (namespace:     (Argo Rollouts  (namespace:
     payments-dev)  payments-stg)   canary deploy)  payments-pr-42)
```

---

## Key Exam Takeaways

- **Directory-per-environment** is preferred over branch-per-environment for GitOps
- **Kustomize overlays** are the standard mechanism for environment-specific configuration
- **Flux Image Automation** uses `ImageRepository`, `ImagePolicy`, and `ImageUpdateAutomation` CRDs
- **ArgoCD Image Updater** is a separate component that commits tag changes back to Git
- **Ephemeral environments** are created per PR using ArgoCD `ApplicationSet` with the PR generator
- **Progressive Delivery** reduces blast radius: Argo Rollouts (ArgoCD ecosystem) and Flagger (Flux ecosystem) are the primary tools
- **Canary deployments** route a small percentage of traffic to the new version; auto-rollback triggers on metric failures
- **Blue/Green** keeps both versions live simultaneously; traffic switch is instantaneous
- Image tag updates to Git manifests are the atomic unit of environment promotion in GitOps
