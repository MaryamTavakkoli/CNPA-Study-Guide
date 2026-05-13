# Secure Service Communication

## Why Service-to-Service Security Matters

In a microservices architecture, services communicate over the network. Without encryption and authentication:
- Traffic can be intercepted (man-in-the-middle)
- Any service in the cluster can talk to any other service
- It's impossible to know which service made a request

Kubernetes provides no built-in encryption for east-west (service-to-service) traffic.

---

## TLS and mTLS

### TLS (Transport Layer Security)

TLS encrypts traffic between a client and server. The server proves its identity with a certificate. Used for north-south traffic (users → services).

### mTLS (Mutual TLS)

**Mutual TLS** extends TLS so that **both sides** present certificates and authenticate each other.

```
Client                         Server
  │─── ClientHello ──────────►│
  │◄── ServerHello + Cert ────│   Server proves identity
  │─── Client Cert ──────────►│   Client proves identity
  │─── Both verify certs ────►│
  │◄── Encrypted channel ────►│
```

With mTLS:
- Services know **who** they're talking to (authentication)
- All traffic is **encrypted** in transit
- Access control can be enforced based on service identity

---

## Service Mesh

A **service mesh** injects a **sidecar proxy** (Envoy is standard) into every pod. The proxy handles all network traffic, providing mTLS, traffic management, and observability transparently — without changing application code.

```
Pod A                          Pod B
┌──────────────────┐          ┌──────────────────┐
│ App Container    │          │ App Container    │
│                  │          │                  │
│ Sidecar (Envoy) ◄────mTLS──► Sidecar (Envoy) │
└──────────────────┘          └──────────────────┘
         │                              │
         └──────── Control Plane ───────┘
                  (Istiod / Linkerd)
```

### Service Mesh Capabilities

| Capability | Description |
|---|---|
| **mTLS** | Automatic mutual TLS between all services |
| **Traffic management** | Load balancing, retries, circuit breakers, timeouts |
| **Canary deployments** | Route % of traffic to new version |
| **Observability** | Automatic metrics, traces, and logs for all service calls |
| **Authorization policies** | Define which services can talk to which |

### Istio

Istio is the most feature-rich service mesh.

Components:
- **Istiod** (control plane): Manages certificates, pushes config to proxies
- **Envoy sidecar** (data plane): The proxy injected into each pod

Key resources:

```yaml
# Allow only frontend to call backend
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: backend-authz
  namespace: production
spec:
  selector:
    matchLabels:
      app: backend
  rules:
    - from:
        - source:
            principals: ["cluster.local/ns/production/sa/frontend"]
```

```yaml
# Traffic routing: 90% stable, 10% canary
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts: [my-app]
  http:
    - route:
        - destination:
            host: my-app
            subset: stable
          weight: 90
        - destination:
            host: my-app
            subset: canary
          weight: 10
```

### Linkerd

Linkerd is lighter-weight than Istio. It uses a Rust-based micro-proxy (not Envoy) which has lower resource usage and latency overhead.

Key differences:
- Simpler to operate
- Fewer features than Istio
- Better performance profile (lower latency overhead)
- Uses its own proxy instead of Envoy

### When to Use a Service Mesh

Use a service mesh when you need:
- Automatic mTLS across all services
- Fine-grained traffic management (canaries, circuit breakers)
- Service-to-service authorization policies
- Automatic distributed tracing without code changes

Tradeoffs:
- Adds operational complexity
- Sidecar injection adds latency (small but non-zero)
- Every pod has an extra container consuming CPU/memory

---

## Secrets Management

Secrets are sensitive values: passwords, API keys, TLS certificates.

### Kubernetes Secrets (built-in)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=   # base64 encoded
  password: cGFzc3dvcmQ=
```

Limitations of native Kubernetes Secrets:
- Only **base64 encoded**, not encrypted at rest by default
- Stored in etcd; requires etcd encryption configuration
- No automatic rotation
- No audit trail for access

### External Secrets Operator (ESO)

ESO connects Kubernetes to external secret stores and syncs secrets as Kubernetes Secret objects.

Supported backends:
- **HashiCorp Vault**
- **AWS Secrets Manager**
- **Google Secret Manager**
- **Azure Key Vault**

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: db-credentials
  data:
    - secretKey: password
      remoteRef:
        key: secret/production/db
        property: password
```

### HashiCorp Vault

Vault is the most widely used secret management solution:
- Centralized secret storage with audit logging
- Dynamic secrets (generates short-lived credentials on demand)
- Secret leasing and renewal
- Kubernetes authentication (pods authenticate using service account tokens)

---

## Network Policies

Kubernetes NetworkPolicy objects restrict pod-to-pod communication. By default, all pods can talk to all other pods.

```yaml
# Allow only frontend pods to reach backend on port 8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-allow-frontend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

**Important**: NetworkPolicy requires a CNI plugin that supports it (Calico, Cilium, Weave). The default Kubernetes CNI (kubenet) does not enforce network policies.

**Default deny all** pattern — start by blocking everything, then allow what's needed:

```yaml
# Deny all ingress and egress by default in a namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}  # applies to all pods
  policyTypes:
    - Ingress
    - Egress
```

---

## Key Takeaways

- mTLS encrypts and authenticates both sides of service-to-service communication
- Service meshes (Istio, Linkerd) automate mTLS via sidecar proxies without code changes
- Istio is feature-rich; Linkerd is lightweight — choose based on complexity needs
- Kubernetes Secrets are only base64-encoded; use External Secrets Operator + Vault for production
- NetworkPolicy restricts pod-to-pod traffic; default is allow-all; best practice is default-deny
