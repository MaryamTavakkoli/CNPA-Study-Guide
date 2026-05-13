# Kubernetes Security Essentials

## The 4Cs of Cloud Native Security

Security is applied in layers:

```
Cloud (provider security)
  └── Cluster (Kubernetes security)
        └── Container (container security)
              └── Code (application security)
```

Each layer is responsible for its own security. A vulnerability in any layer can compromise the whole system.

---

## RBAC (Role-Based Access Control)

Kubernetes RBAC controls **who can do what to which resources**.

### Core Concepts

| Resource | Scope | Description |
|---|---|---|
| `Role` | Namespace | Defines permissions within a namespace |
| `ClusterRole` | Cluster-wide | Defines permissions across all namespaces (or cluster-scoped resources) |
| `RoleBinding` | Namespace | Grants a Role to a user/group/service account in a namespace |
| `ClusterRoleBinding` | Cluster-wide | Grants a ClusterRole cluster-wide |

### Subjects

RBAC grants permissions to:
- **User**: Human user (managed outside Kubernetes, e.g., via OIDC)
- **Group**: Group of users
- **ServiceAccount**: Identity for pods running in the cluster

```yaml
# Role: read pods in staging namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: staging
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]
```

```yaml
# RoleBinding: grant pod-reader to the monitoring service account
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: monitoring-pod-reader
  namespace: staging
subjects:
  - kind: ServiceAccount
    name: monitoring-agent
    namespace: monitoring
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Principle of Least Privilege

Grant only the permissions needed for a task. Common mistakes:
- Binding `cluster-admin` to application service accounts
- Granting `*` verbs on `*` resources
- Using the `default` service account (which may have accumulated permissions)

### Service Accounts

Every pod runs as a service account. Kubernetes auto-mounts a service account token.

```yaml
# Opt out of auto-mounting if the pod doesn't need API access
spec:
  automountServiceAccountToken: false
```

---

## Pod Security

### SecurityContext

SecurityContext defines security settings at the pod or container level.

```yaml
spec:
  securityContext:                    # Pod-level
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: app
      securityContext:                # Container-level
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop: ["ALL"]
          add: ["NET_BIND_SERVICE"]  # Only if needed
```

Key settings:
| Setting | Recommended Value | Why |
|---|---|---|
| `runAsNonRoot` | `true` | Don't run as root |
| `runAsUser` | Non-zero UID | Explicit non-root |
| `allowPrivilegeEscalation` | `false` | Prevent gaining more permissions |
| `readOnlyRootFilesystem` | `true` | Immutable container filesystem |
| `capabilities.drop` | `["ALL"]` | Remove all Linux capabilities |

### Pod Security Admission (PSA)

PSA is built into Kubernetes (replaced the deprecated PodSecurityPolicy). It enforces security profiles at the namespace level.

Three built-in profiles:
| Profile | Description |
|---|---|
| `privileged` | No restrictions |
| `baseline` | Minimal restrictions; blocks known privilege escalations |
| `restricted` | Heavily restricted; follows pod hardening best practices |

Three enforcement modes:
| Mode | Behavior |
|---|---|
| `enforce` | Reject pods that violate the policy |
| `audit` | Allow but log violations |
| `warn` | Allow but show warnings |

```yaml
# Apply restricted policy to a namespace
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

---

## Image Security

### Use Minimal Base Images

- Prefer `distroless` or `alpine` base images — fewer packages = smaller attack surface
- `scratch` for statically compiled binaries (no OS at all)

```dockerfile
# Multi-stage build: compile in full image, run in minimal
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o myapp .

FROM gcr.io/distroless/base-debian12
COPY --from=builder /app/myapp /
CMD ["/myapp"]
```

### Image Scanning

Scan images for known CVEs (Common Vulnerabilities and Exposures):
- **Trivy**: Fast, comprehensive scanner for containers, filesystems, IaC
- **Grype**: Open-source vulnerability scanner
- **Snyk**: Commercial with IDE integration
- **Clair**: Open-source, used in Quay

```bash
# Scan an image with Trivy
trivy image my-app:1.0.0

# Fail CI if critical vulnerabilities found
trivy image --exit-code 1 --severity CRITICAL my-app:1.0.0
```

### Image Signing and Verification

**Cosign** (from Sigstore) signs container images, providing a chain of trust.

```bash
# Sign an image
cosign sign --key cosign.key my-registry/my-app:1.0.0

# Verify signature
cosign verify --key cosign.pub my-registry/my-app:1.0.0
```

Kyverno can enforce that only signed images are deployed:
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-image-signatures
spec:
  rules:
    - name: verify-signature
      match:
        any:
          - resources:
              kinds: [Pod]
      verifyImages:
        - imageReferences: ["my-registry/*"]
          attestors:
            - entries:
                - keys:
                    publicKeys: |-
                      -----BEGIN PUBLIC KEY-----
                      ...
                      -----END PUBLIC KEY-----
```

---

## Secrets Encryption

By default, Kubernetes Secrets stored in etcd are **not encrypted**. Enable encryption at rest:

```yaml
# EncryptionConfiguration (applied to API server)
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources: [secrets]
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <base64-encoded-key>
      - identity: {}  # fallback: unencrypted (remove after migration)
```

Use KMS (Key Management Service) providers for managed key rotation.

---

## Audit Logging

Kubernetes API audit logs record every API call:
- Who made the request
- What resource was accessed
- What action was taken
- When it happened

```yaml
# Audit policy
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  - level: Metadata
    resources:
      - group: ""
        resources: ["secrets"]
  - level: RequestResponse
    resources:
      - group: ""
        resources: ["pods"]
  - level: None
    users: ["system:kube-proxy"]
```

Audit log levels: `None`, `Metadata`, `Request`, `RequestResponse`

---

## Key Takeaways

- 4Cs: Cloud → Cluster → Container → Code; each layer has security responsibilities
- RBAC: Roles define what can be done; Bindings assign Roles to subjects; use least privilege
- SecurityContext: run as non-root, drop all capabilities, read-only filesystem
- Pod Security Admission: namespace-level enforcement of privileged/baseline/restricted profiles
- Scan images for CVEs; sign images with Cosign; verify signatures with Kyverno
- Encrypt Secrets at rest in etcd; use external secret stores for production
- Audit logs provide full API call history for compliance and forensics
