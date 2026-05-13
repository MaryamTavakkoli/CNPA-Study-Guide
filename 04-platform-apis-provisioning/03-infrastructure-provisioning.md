# Infrastructure Provisioning with Kubernetes

## Overview

Cloud native platform engineering increasingly treats infrastructure as just another Kubernetes resource. Rather than managing cloud resources through separate CLI tools, web consoles, or IaC pipelines outside the cluster, teams can provision and manage infrastructure — databases, message queues, DNS records, virtual machines — using `kubectl apply` and GitOps workflows. This document covers the major approaches: Crossplane, the Terraform Operator, and the Pulumi Kubernetes Operator.

---

## Crossplane

Crossplane is a CNCF graduated project that turns any Kubernetes cluster into a **universal control plane for infrastructure**. Its core insight: the Kubernetes reconciliation loop is a general-purpose infrastructure management engine — not just for scheduling containers.

### Core Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      Kubernetes Cluster                  │
│                                                          │
│  Application Team          Platform Team                 │
│  ┌─────────────┐          ┌────────────────────────┐    │
│  │   Claim     │────────► │  Composite Resource    │    │
│  │ (ns-scoped) │          │ (cluster-scoped XR)    │    │
│  └─────────────┘          └────────────┬───────────┘    │
│                                        │ Composition     │
│                            ┌───────────▼──────────┐     │
│                            │   Managed Resources  │     │
│                            │  (RDSInstance,       │     │
│                            │   SecurityGroup,     │     │
│                            │   SubnetGroup)       │     │
│                            └───────────┬──────────┘     │
└────────────────────────────────────────┼────────────────┘
                                         │ Provider reconciles
                                         ▼
                              AWS / GCP / Azure / etc.
```

### Providers

A **Provider** is a Crossplane package that installs CRDs for a specific cloud platform and runs a controller that reconciles those resources against the real API.

```yaml
# Install the AWS provider
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-aws-rds
spec:
  package: xpkg.upbound.io/upbound/provider-aws-rds:v1.1.0
```

```yaml
# Configure provider credentials
apiVersion: aws.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: aws-creds
      key: creds
```

Popular providers: AWS, GCP, Azure, Helm, Kubernetes, Vault, GitHub, Confluent, MongoDB Atlas.

### Managed Resources

A **Managed Resource (MR)** is a CRD-defined Kubernetes object that represents exactly one external resource. Managed resources are **cluster-scoped**.

```yaml
# A Managed Resource — maps 1:1 to an AWS RDS instance
apiVersion: rds.aws.upbound.io/v1beta1
kind: Instance
metadata:
  name: production-postgres
  annotations:
    crossplane.io/external-name: prod-postgres-01  # name in AWS
spec:
  forProvider:
    region: us-east-1
    instanceClass: db.t3.medium
    engine: postgres
    engineVersion: "15.4"
    allocatedStorage: 100
    dbName: appdb
    username: admin
    passwordSecretRef:
      namespace: crossplane-system
      name: rds-password
      key: password
    multiAz: true
    skipFinalSnapshot: false
  writeConnectionSecretToRef:
    namespace: crossplane-system
    name: production-postgres-connection
```

The Crossplane controller watches this object and reconciles it against AWS. If the instance is deleted externally, Crossplane re-creates it. If the spec changes, Crossplane updates the instance.

### Composite Resources (XRs) and XRDs

An **XRD (Composite Resource Definition)** defines a new composite resource type — like a CRD but specifically for Crossplane. Platform teams author XRDs to create opinionated, reusable infrastructure abstractions.

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: xpostgresqlinstances.platform.example.org
spec:
  group: platform.example.org
  names:
    kind: XPostgreSQLInstance
    plural: xpostgresqlinstances
  claimNames:
    kind: PostgreSQLInstance    # namespace-scoped claim
    plural: postgresqlinstances
  versions:
  - name: v1alpha1
    served: true
    referenceable: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              parameters:
                type: object
                required: ["storageGB"]
                properties:
                  storageGB:
                    type: integer
                    minimum: 20
                    maximum: 500
                  region:
                    type: string
                    default: us-east-1
                  size:
                    type: string
                    enum: ["small", "medium", "large"]
                    default: small
```

### Compositions

A **Composition** defines how an XR (Composite Resource) maps to a set of Managed Resources. This is where the platform team's infrastructure expertise lives.

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: xpostgresqlinstances.aws.platform.example.org
  labels:
    provider: aws
    environment: production
spec:
  compositeTypeRef:
    apiVersion: platform.example.org/v1alpha1
    kind: XPostgreSQLInstance
  resources:
  - name: rds-instance
    base:
      apiVersion: rds.aws.upbound.io/v1beta1
      kind: Instance
      spec:
        forProvider:
          region: us-east-1
          engine: postgres
          engineVersion: "15.4"
          multiAz: true
    patches:
    - type: FromCompositeFieldPath
      fromFieldPath: "spec.parameters.storageGB"
      toFieldPath: "spec.forProvider.allocatedStorage"
    - type: FromCompositeFieldPath
      fromFieldPath: "spec.parameters.region"
      toFieldPath: "spec.forProvider.region"
    - type: FromCompositeFieldPath
      fromFieldPath: "spec.parameters.size"
      toFieldPath: "spec.forProvider.instanceClass"
      transforms:
      - type: map
        map:
          small: db.t3.medium
          medium: db.m5.large
          large: db.r5.2xlarge
  - name: subnet-group
    base:
      apiVersion: rds.aws.upbound.io/v1beta1
      kind: SubnetGroup
      spec:
        forProvider:
          region: us-east-1
          description: "Managed by Crossplane"
```

### Claims (Namespace-Scoped Self-Service)

A **Claim** is the namespace-scoped counterpart to a Composite Resource. Application developers use claims to request infrastructure without needing cluster-level access.

```yaml
# What the application team writes (in their own namespace)
apiVersion: platform.example.org/v1alpha1
kind: PostgreSQLInstance
metadata:
  name: my-app-db
  namespace: team-alpha
spec:
  parameters:
    storageGB: 50
    size: medium
    region: eu-west-1
  writeConnectionSecretToRef:
    name: my-app-db-connection    # connection details landed here
```

The platform team controls the Composition; the application team controls the Claim. This is **self-service infrastructure with guardrails**.

### Crossplane Resource Hierarchy Summary

| Resource | Scope | Author | Purpose |
|----------|-------|--------|---------|
| Provider | Cluster | Platform team | Install cloud provider CRDs + controller |
| ProviderConfig | Cluster | Platform team | Configure credentials |
| Managed Resource | Cluster | Platform team / automation | Direct 1:1 mapping to cloud resource |
| XRD | Cluster | Platform team | Define composite resource schema |
| Composition | Cluster | Platform team | Map composite → managed resources |
| XR (Composite Resource) | Cluster | Automation | Instantiation of a composition |
| Claim | Namespace | Application team | Self-service request for a composite resource |

---

## Terraform Operator

The **Terraform Operator** (by HashiCorp or community variants like tf-controller by Weaveworks) runs Terraform inside Kubernetes, enabling GitOps-driven infrastructure provisioning.

```yaml
# tf-controller example
apiVersion: infra.contrib.fluxcd.io/v1alpha2
kind: Terraform
metadata:
  name: aws-vpc
  namespace: platform
spec:
  interval: 1h
  approvePlan: auto
  path: ./terraform/aws-vpc
  sourceRef:
    kind: GitRepository
    name: platform-infra
    namespace: flux-system
  vars:
  - name: vpc_cidr
    value: "10.0.0.0/16"
  - name: environment
    value: production
  writeOutputsToSecret:
    name: aws-vpc-outputs
```

Key characteristics:
- Leverages existing Terraform modules without rewriting
- State stored in Kubernetes Secrets or remote backends (S3, Terraform Cloud)
- Supports plan approval workflows (manual or automatic)
- Plan and apply phases visible as Kubernetes status conditions

---

## Pulumi Kubernetes Operator

The **Pulumi Kubernetes Operator** runs Pulumi programs inside Kubernetes using a `Stack` CRD:

```yaml
apiVersion: pulumi.com/v1
kind: Stack
metadata:
  name: production-infrastructure
  namespace: pulumi-system
spec:
  stack: myorg/production-infrastructure/prod
  projectRepo: https://github.com/myorg/infrastructure
  branch: refs/heads/main
  envRefs:
    AWS_REGION:
      type: Literal
      literal:
        value: us-east-1
    AWS_ACCESS_KEY_ID:
      type: Secret
      secret:
        name: aws-credentials
        key: access-key-id
  destroyOnFinalize: false
```

Key characteristics:
- Uses general-purpose programming languages (TypeScript, Python, Go, .NET)
- Pulumi state stored in Pulumi Cloud or self-hosted backends
- Supports stack references for cross-stack outputs

---

## Crossplane vs. Terraform: Comparison

| Dimension | Crossplane | Terraform |
|-----------|------------|-----------|
| **Paradigm** | Kubernetes-native control plane | Declarative IaC with state file |
| **State management** | etcd (Kubernetes objects) | State file (local, S3, Terraform Cloud) |
| **Drift detection** | Continuous (reconciliation loop) | On-demand (`terraform plan`) |
| **Multi-tenancy** | Built-in (namespace scoping, RBAC) | Requires external tooling |
| **Abstraction layer** | Compositions + XRDs | Modules |
| **Ecosystem** | Growing (100+ providers) | Mature (thousands of providers) |
| **Learning curve** | Higher (new concepts) | Lower for teams already using Terraform |
| **Runtime requirements** | Kubernetes cluster | CI/CD runner with Terraform binary |
| **Secret handling** | Kubernetes Secrets + ESO | Vault provider, env vars |
| **Best for** | Platform teams building self-service APIs | Teams with existing Terraform investment |

---

## Namespace-Scoped vs. Cluster-Scoped Resources

This distinction is critical for multi-tenancy in provisioning platforms:

| Scope | Examples | Access | Use Case |
|-------|----------|--------|----------|
| **Cluster-scoped** | Managed Resources, XRs, CRDs, Nodes | Cluster-admin or explicit ClusterRole | Platform team management, cross-namespace shared resources |
| **Namespace-scoped** | Claims, Pods, Services, Secrets | Namespace RBAC (Role + RoleBinding) | Application team self-service |

**Why this matters for Crossplane:**
- Application teams get `create`/`list` on Claims in their namespace only
- Platform team manages Compositions and Managed Resources cluster-wide
- Connection secrets can be propagated from cluster-scoped XR to namespace-scoped Secrets for application use

---

## How Platform Teams Expose Infrastructure Self-Service via XRDs

A production self-service platform workflow:

```
1. Platform team defines XRD
      └─► Registers `PostgreSQLInstance` claim type

2. Platform team creates Composition(s)
      └─► aws.platform.example.org (production)
      └─► local.platform.example.org (development/minikube)

3. Platform team assigns RBAC
      └─► Role: create/list/get/watch PostgreSQLInstance in team namespaces

4. Developer creates a Claim in their namespace
      └─► `kubectl apply -f db-claim.yaml`

5. Crossplane reconciles
      └─► Creates XR (cluster-scoped)
      └─► Composition renders Managed Resources
      └─► Provider creates RDS instance in AWS

6. Connection details available as a Secret
      └─► App references it via envFrom/volumeMount
```

This entire flow is **GitOps-compatible**: the claim YAML lives in the application team's Git repository, and Argo CD or Flux applies it to the cluster.

---

## Summary

| Tool | Model | Strength |
|------|-------|----------|
| **Crossplane** | Kubernetes-native control plane | Built-in multi-tenancy, continuous drift detection, composable platform APIs |
| **Terraform Operator** | IaC pipeline in Kubernetes | Reuse existing Terraform modules, familiar HCL syntax |
| **Pulumi Operator** | IaC pipeline in Kubernetes | General-purpose languages, cross-stack references |

For platform engineering, **Crossplane is the recommended cloud native approach** when building self-service platforms, because it provides native Kubernetes multi-tenancy, composable abstractions, and continuous reconciliation without requiring separate pipeline infrastructure.
