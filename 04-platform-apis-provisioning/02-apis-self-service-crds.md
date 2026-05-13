# Kubernetes APIs, Self-Service & Custom Resource Definitions (CRDs)

## Overview

One of Kubernetes' most powerful architectural decisions is that its API is **extensible**. Rather than hardcoding every possible resource type, Kubernetes provides mechanisms to add new API types at runtime. Custom Resource Definitions (CRDs) are the primary and most widely-used extension mechanism, forming the foundation of virtually every platform-engineering tool in the cloud native ecosystem.

---

## What Is a CRD?

A **Custom Resource Definition** is a Kubernetes resource (of kind `CustomResourceDefinition`) that registers a new resource type with the Kubernetes API server. Once a CRD is installed in a cluster, users can create instances of that new type — called **Custom Resources (CRs)** — using the same `kubectl`, API calls, and RBAC patterns they use for built-in types.

```yaml
# This is the CRD itself — it defines the schema
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: certificates.cert-manager.io
spec:
  group: cert-manager.io
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required: ["secretName", "issuerRef"]
            properties:
              secretName:
                type: string
              dnsNames:
                type: array
                items:
                  type: string
              issuerRef:
                type: object
                properties:
                  name:
                    type: string
                  kind:
                    type: string
  scope: Namespaced
  names:
    plural: certificates
    singular: certificate
    kind: Certificate
    shortNames: ["cert"]
```

```yaml
# This is a Custom Resource — an instance of the CRD above
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: my-app-tls
  namespace: production
spec:
  secretName: my-app-tls-secret
  dnsNames:
  - app.example.com
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
```

---

## CRD vs. Custom Resource

| Concept | Role | Analogy |
|---------|------|---------|
| `CustomResourceDefinition` | Defines the schema/type | A Go struct definition or a database table schema |
| `Custom Resource` | An instance of that type | A struct value or a database row |
| `Controller/Operator` | Watches CRs and acts on them | Application business logic |

A CRD alone does nothing useful. It is the **controller** (or operator) that watches custom resources and takes action that gives them meaning.

---

## CRD Schema Validation

CRDs use **OpenAPI v3 schema validation** to enforce structure. The API server validates every Custom Resource against the schema before persisting it to etcd.

Key schema features:

```yaml
schema:
  openAPIV3Schema:
    type: object
    properties:
      spec:
        type: object
        required:
        - replicas
        - image
        properties:
          replicas:
            type: integer
            minimum: 1
            maximum: 100
            default: 1          # default values (since Kubernetes 1.17)
          image:
            type: string
            pattern: '^[a-z0-9/:.@-]+$'
          resources:
            type: object
            properties:
              memory:
                type: string
                # x-kubernetes-int-or-string: true  # for IntOrString fields
    x-kubernetes-preserve-unknown-fields: false  # strict mode
```

**Structural schema** (required since Kubernetes 1.16) means:
- Types must be specified for all fields
- Unknown fields are pruned by default (improving security)
- Defaults and validation rules are enforced

**CEL validation** (since Kubernetes 1.25+, stable 1.29) allows complex cross-field validation rules:

```yaml
x-kubernetes-validations:
- rule: "self.minReplicas <= self.maxReplicas"
  message: "minReplicas must be less than or equal to maxReplicas"
```

---

## Custom Resources vs. Built-In Resources

| Aspect | Built-In Resources | Custom Resources |
|--------|-------------------|-----------------|
| Schema location | Compiled into API server | Registered via CRD |
| Versioning | Kubernetes release cycle | Independent versioning |
| Storage | etcd (same as built-ins) | etcd (same as built-ins) |
| RBAC | Same RBAC system | Same RBAC system |
| kubectl support | Full (get, describe, edit, etc.) | Full (same mechanisms) |
| Discovery | Built into API groups | Registered in API discovery |
| Validation | Go struct tags + admission | OpenAPI v3 + CEL |

Custom resources are **first-class citizens** in Kubernetes. They appear in `kubectl api-resources`, support label selectors, have `metadata.generation`, support finalizers and owner references — everything built-in resources have.

---

## CRDs as the Foundation of Self-Service Platforms

Platform teams use CRDs to expose **opinionated, domain-specific abstractions** to application teams. Instead of exposing raw cloud APIs or complex Kubernetes internals, the platform team defines a simple CRD that:

1. Captures only what the application team needs to care about
2. Enforces platform defaults and constraints via schema validation
3. Triggers automation (via a controller) to do the actual work

**Example: A `PostgreSQLDatabase` CRD**

```yaml
# What the application developer writes (simple, opinionated)
apiVersion: platform.company.io/v1alpha1
kind: PostgreSQLDatabase
metadata:
  name: my-app-db
  namespace: team-alpha
spec:
  size: small          # "small", "medium", "large" — not raw instance sizes
  version: "15"
  backupEnabled: true
```

Behind the scenes, the platform controller translates `size: small` into the appropriate cloud provider instance type, network configuration, backup policy, and monitoring setup — all enforced consistently across every team.

This is the **platform API** pattern: the CRD is the interface, the controller is the implementation.

---

## Popular CRDs in the Ecosystem

### Crossplane Composite Resources (XRs)

```yaml
apiVersion: platform.example.org/v1alpha1
kind: XPostgreSQLInstance
metadata:
  name: my-db
spec:
  parameters:
    storageGB: 20
    region: us-east-1
  compositeDeletePolicy: Foreground
  writeConnectionSecretToRef:
    namespace: crossplane-system
    name: my-db-connection
```

Crossplane uses CRDs to create **multi-layer abstractions**: XRDs define composite resource schemas, Compositions define how to map them to actual managed resources.

### Argo CD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

The `Application` CRD is Argo CD's self-service interface — a developer or GitOps system creates an `Application` and Argo CD reconciles the cluster to match the Git repository.

### cert-manager Certificate

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: api-tls
  namespace: production
spec:
  secretName: api-tls-secret
  duration: 2160h     # 90 days
  renewBefore: 360h   # Renew 15 days before expiry
  dnsNames:
  - api.example.com
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
```

Application teams request certificates through a simple CRD without needing to understand ACME protocols, certificate storage, or renewal scheduling.

### External Secrets ExternalSecret

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-credentials
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: database-credentials
    creationPolicy: Owner
  data:
  - secretKey: password
    remoteRef:
      key: production/database
      property: password
```

The External Secrets Operator syncs secrets from AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager, and others into Kubernetes Secrets via a CRD abstraction.

---

## The Kubernetes API Extension Mechanism

Kubernetes supports two official extension mechanisms:

### 1. Custom Resource Definitions (CRDs)
- Easiest to implement — just register a CRD YAML
- Resources stored in **etcd** (same as built-in resources)
- Limited to what the API server natively supports (no custom storage backends)
- Best for: 99% of use cases

### 2. Aggregated API Servers
- Run a **separate API server** process alongside the main API server
- Registered via `APIService` resource; the main API server proxies requests to it
- Can have **custom storage backends** (not etcd)
- Can implement **custom admission logic**, **custom subresources**, **streaming responses**
- Best for: metrics API (metrics-server), custom authorization, specialized storage needs

```yaml
# Registering an aggregated API server
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  name: v1beta1.metrics.k8s.io
spec:
  service:
    name: metrics-server
    namespace: kube-system
    port: 443
  group: metrics.k8s.io
  version: v1beta1
  insecureSkipTLSVerify: true
  groupPriorityMinimum: 100
  versionPriority: 100
```

### Comparison

| Feature | CRDs | Aggregated API Servers |
|---------|------|----------------------|
| Complexity | Low | High |
| Storage | etcd | Pluggable |
| Custom logic | Limited (webhooks) | Full control |
| Subresources | `/status`, `/scale` | Arbitrary |
| Examples | cert-manager, Crossplane | metrics-server, custom auth |

---

## API Versioning and Conversion

CRDs support **multiple versions** with **conversion webhooks** to translate between them:

```yaml
spec:
  versions:
  - name: v1
    served: true
    storage: true     # only one version can be storage version
  - name: v1beta1
    served: true
    storage: false
  conversion:
    strategy: Webhook
    webhook:
      clientConfig:
        service:
          namespace: my-operator
          name: conversion-webhook
          path: /convert
```

This allows CRD authors to evolve their API over time without breaking existing users — the conversion webhook translates between old and new versions transparently.

---

## RBAC for Custom Resources

CRDs follow the same RBAC model as built-in resources:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: team-alpha
  name: database-provisioner
rules:
- apiGroups: ["platform.company.io"]
  resources: ["postgresqldatabases"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["platform.company.io"]
  resources: ["postgresqldatabases/status"]
  verbs: ["get"]
```

Platform teams can grant application teams permission to create only specific custom resource types — not raw infrastructure resources — implementing **least-privilege self-service**.

---

## Summary

| Concept | Key Point |
|---------|-----------|
| CRD | Registers a new resource type with the API server |
| Custom Resource | An instance of a CRD-defined type |
| Schema Validation | OpenAPI v3 + CEL; enforced server-side before persistence |
| Self-Service Platform | CRD = platform API; controller = platform implementation |
| Aggregated API Server | Full API server extension; used when CRDs are insufficient |
| API Versioning | Multiple versions + conversion webhooks enable non-breaking evolution |
| RBAC | Same system as built-in resources; enables least-privilege self-service |
