# Application Environments and Infrastructure Concepts

## Application Environments

An **application environment** is an isolated runtime context where a version of an application runs. Environments exist to allow testing and validation before production traffic reaches code.

### Typical Environment Progression

```
Development → Integration → Staging → Production
```

| Environment | Purpose | Who Uses It |
|---|---|---|
| **Development** (dev) | Individual developer testing | Developer |
| **Integration / Testing** | Automated tests, integration testing | CI pipelines |
| **Staging / Pre-prod** | Final validation, mirrors production | QA, product owners |
| **Production** (prod) | Serves real user traffic | End users |

Some organizations add:
- **Ephemeral environments** — short-lived environments spun up per branch/PR, torn down after merge
- **Canary** — small subset of production traffic routed to new version
- **Shadow** — production traffic duplicated to a new version without affecting users

### Environment Parity

**Production parity** (from 12-Factor App) means keeping dev, staging, and production as similar as possible. Differences create bugs that appear only in certain environments.

Platform engineering helps maintain parity by:
- Using the same Kubernetes manifests across environments, with Kustomize overlays for differences
- Using container images (same image promoted through environments, not rebuilt)
- Managing environment config externally (not baked into the image)

---

## Infrastructure Concepts

### Compute Models

| Model | Description | Example |
|---|---|---|
| **Bare metal** | Physical servers, no virtualization | On-prem hardware |
| **Virtual machines (VMs)** | Hardware virtualized, OS per VM | EC2, GCE, Azure VMs |
| **Containers** | OS-level virtualization, shared kernel | Docker, containerd |
| **Serverless** | No server management; pay per invocation | AWS Lambda, Cloud Run |

**Containers vs. VMs:**
- Containers share the host OS kernel — faster startup, lower overhead
- VMs have stronger isolation — different OS per VM
- Containers are the standard unit for cloud-native workloads

### Container Fundamentals

A **container** is a lightweight, portable, isolated process. It packages an application and its dependencies.

Key concepts:
- **Container image**: A read-only template (layers) built from a `Dockerfile`
- **Container registry**: Where images are stored and distributed (Docker Hub, GCR, ECR, Quay)
- **OCI (Open Container Initiative)**: Standard for container image format and runtime — ensures portability

```dockerfile
# Example Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

Container image layers are cached. Place frequently-changing layers (COPY . .) after rarely-changing ones (RUN pip install).

### The Kubernetes Architecture

Understanding the cluster components is foundational for platform engineering.

```
┌─────────────────────────────────────────────────────────┐
│                   Control Plane                          │
│  ┌──────────────┐  ┌─────────────┐  ┌────────────────┐ │
│  │  API Server  │  │  Scheduler  │  │ Controller Mgr │ │
│  └──────────────┘  └─────────────┘  └────────────────┘ │
│  ┌─────────────────────────────────────────────────────┐ │
│  │                    etcd                             │ │
│  └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
          │               │               │
┌─────────┴──┐   ┌────────┴───┐   ┌──────┴──────┐
│   Node 1   │   │   Node 2   │   │   Node 3    │
│ ┌────────┐ │   │ ┌────────┐ │   │ ┌─────────┐ │
│ │kubelet │ │   │ │kubelet │ │   │ │ kubelet │ │
│ │kube-   │ │   │ │kube-   │ │   │ │ kube-   │ │
│ │proxy   │ │   │ │proxy   │ │   │ │ proxy   │ │
│ └────────┘ │   │ └────────┘ │   │ └─────────┘ │
└────────────┘   └────────────┘   └─────────────┘
```

**Control Plane components:**
- **API Server** (`kube-apiserver`): The front door — all kubectl commands hit this; validates and persists resources to etcd
- **etcd**: Distributed key-value store; the source of truth for all cluster state
- **Scheduler** (`kube-scheduler`): Watches for unscheduled pods and assigns them to nodes
- **Controller Manager** (`kube-controller-manager`): Runs built-in controllers (Deployment, ReplicaSet, Node, etc.)

**Node components:**
- **kubelet**: Agent on each node; ensures containers in pods are running
- **kube-proxy**: Manages network rules on nodes for Service routing
- **Container runtime**: Runs containers (containerd, CRI-O)

### Networking Fundamentals

**Services** provide stable network endpoints:

| Service Type | Description |
|---|---|
| `ClusterIP` | Internal-only; accessible within the cluster |
| `NodePort` | Exposes port on every node's IP |
| `LoadBalancer` | Creates a cloud load balancer; external access |
| `ExternalName` | Maps to an external DNS name |

**Ingress**: An API object that manages external HTTP(S) access to services. Requires an Ingress controller (nginx, Traefik, Contour, etc.).

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  rules:
    - host: my-app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app
                port:
                  number: 80
```

**DNS in Kubernetes**: CoreDNS runs in the cluster. Every Service gets a DNS name:
`<service-name>.<namespace>.svc.cluster.local`

### Storage

**Volumes** provide persistent storage for containers. Container filesystems are ephemeral by default.

- **PersistentVolume (PV)**: A piece of cluster storage provisioned by an admin or dynamically
- **PersistentVolumeClaim (PVC)**: A request for storage by a workload
- **StorageClass**: Defines the type of storage (SSD, HDD, cloud provider class)

**Access modes:**
- `ReadWriteOnce` (RWO): Mounted read-write by a single node
- `ReadOnlyMany` (ROX): Mounted read-only by many nodes
- `ReadWriteMany` (RWX): Mounted read-write by many nodes

---

## Cloud Provider Concepts

### Managed Kubernetes Services

| Provider | Service |
|---|---|
| AWS | EKS (Elastic Kubernetes Service) |
| Google Cloud | GKE (Google Kubernetes Engine) |
| Azure | AKS (Azure Kubernetes Service) |
| DigitalOcean | DOKS |

Managed services offload control plane management to the cloud provider. Platform teams manage worker nodes and workloads, not the API server or etcd.

### Multi-Cluster and Multi-Tenancy

Large organizations often run multiple clusters:
- **Per-environment clusters**: dev, staging, prod in separate clusters (strong isolation)
- **Per-region clusters**: clusters in different geographic regions
- **Namespace-based multi-tenancy**: multiple teams share a single cluster, separated by namespaces

Platform engineering must provide consistent tooling across all clusters.

---

## Key Takeaways

- Environments (dev → staging → prod) isolate risk; environment parity prevents "works on my machine"
- Container images are immutable and promoted through environments unchanged
- Kubernetes control plane (API server, etcd, scheduler, controller manager) vs. worker nodes (kubelet, kube-proxy, container runtime)
- Services provide stable network endpoints; Ingress handles HTTP routing from outside the cluster
- PVs and PVCs decouple storage provisioning from storage consumption
