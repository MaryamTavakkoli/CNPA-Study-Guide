# Declarative Resource Management

## What Is It?

Declarative resource management means you describe **what you want** (the desired state), not **how to get there** (the steps). The system is responsible for achieving and maintaining that state.

**Imperative:** "Create a pod, then attach a volume, then expose port 8080"
**Declarative:** "I want a pod running image X with a volume and port 8080 exposed"

---

## Why It Matters for Platform Engineering

Declarative management is the foundation of Kubernetes and GitOps. Platform engineers use it to:

- Make infrastructure **reproducible** — the same manifest always produces the same result
- Enable **self-healing** — controllers continuously reconcile actual state with desired state
- Support **auditable** changes — all changes are manifest edits tracked in Git
- Simplify **automation** — pipelines apply manifests; no scripted imperative steps

---

## Kubernetes as a Declarative System

Kubernetes resources are described in YAML manifests. The API server stores desired state in `etcd`. Controllers read desired state and act to produce it.

### Core Resource Types

| Resource | Purpose |
|---|---|
| `Pod` | Smallest deployable unit; one or more containers |
| `Deployment` | Manages a ReplicaSet; handles rolling updates |
| `ReplicaSet` | Ensures N identical pod replicas are running |
| `StatefulSet` | Like Deployment but for stateful workloads (stable identity, ordered operations) |
| `DaemonSet` | Runs one pod per node (e.g., log collectors, monitoring agents) |
| `Job` / `CronJob` | Run-to-completion workloads |
| `Service` | Stable network endpoint in front of pods |
| `ConfigMap` | Non-sensitive configuration data |
| `Secret` | Sensitive configuration data (base64-encoded, not encrypted by default) |
| `Namespace` | Virtual cluster partition for resource isolation |
| `PersistentVolume` (PV) | A piece of storage in the cluster |
| `PersistentVolumeClaim` (PVC) | A request for storage by a workload |

### Anatomy of a Kubernetes Manifest

```yaml
apiVersion: apps/v1        # API group and version
kind: Deployment           # Resource type
metadata:
  name: my-app
  namespace: production
  labels:
    app: my-app
spec:                      # Desired state
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:1.2.3
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

### Key Fields

- **`apiVersion`**: Which API group handles this resource (e.g., `apps/v1`, `v1`, `batch/v1`)
- **`kind`**: The resource type
- **`metadata.labels`**: Key-value pairs used for selection and grouping
- **`metadata.annotations`**: Non-identifying metadata (tool configs, descriptions)
- **`spec`**: The desired state — this is what you declare
- **`status`**: The observed/current state — written by Kubernetes, not by you

---

## The Reconciliation Loop

Every Kubernetes controller runs a **reconciliation loop**:

```
Observe current state → Compare to desired state → Act to close the gap → Repeat
```

This loop runs continuously. If a pod crashes, the controller notices the gap and creates a replacement. This is why Kubernetes is called "self-healing."

---

## Labels and Selectors

Labels are how resources find and relate to each other.

```yaml
# Service selects pods with matching labels
apiVersion: v1
kind: Service
spec:
  selector:
    app: my-app        # Matches pods with this label
    tier: frontend
```

**Label selectors:**
- **Equality-based**: `app=my-app`, `env!=staging`
- **Set-based**: `env in (prod, staging)`, `tier notin (frontend)`

---

## Namespaces

Namespaces provide logical isolation within a cluster.

```bash
kubectl get pods -n production
kubectl get pods --all-namespaces
```

Default namespaces:
- `default` — resources without a specified namespace
- `kube-system` — Kubernetes system components
- `kube-public` — publicly readable data
- `kube-node-lease` — node heartbeat leases

Platform engineering uses namespaces to separate teams, environments, or tenants.

---

## Helm — Declarative Package Management

Helm is the package manager for Kubernetes. It bundles related manifests into a **chart**.

```bash
helm install my-release stable/nginx     # Install a chart
helm upgrade my-release stable/nginx     # Upgrade
helm rollback my-release 1               # Rollback to revision 1
helm list                                # List installed releases
```

A Helm chart structure:
```
my-chart/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration values
├── templates/          # Kubernetes manifest templates
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl
```

Values override at install time:
```bash
helm install my-release my-chart --set replicaCount=3
helm install my-release my-chart -f custom-values.yaml
```

---

## Kustomize — Declarative Configuration Overlays

Kustomize allows you to customize manifests without templates. It works with a base configuration and environment-specific overlays.

```
base/
├── kustomization.yaml
├── deployment.yaml
└── service.yaml

overlays/
├── staging/
│   ├── kustomization.yaml   # patches for staging
└── production/
    ├── kustomization.yaml   # patches for production
```

```yaml
# overlays/production/kustomization.yaml
resources:
  - ../../base
patches:
  - path: replica-patch.yaml
images:
  - name: my-app
    newTag: "1.2.3"
```

```bash
kubectl apply -k overlays/production/
```

Kustomize is built into `kubectl` (`kubectl apply -k`). It is favored in GitOps workflows.

---

## Key Takeaways

- Declarative = describe desired state; imperative = describe steps
- Kubernetes stores desired state in etcd; controllers reconcile actual to desired
- Labels and selectors connect resources to each other
- Helm packages sets of manifests; Kustomize patches them for different environments
- Namespaces logically partition a cluster
