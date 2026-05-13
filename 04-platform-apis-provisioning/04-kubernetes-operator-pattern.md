# The Kubernetes Operator Pattern

## Overview

The **Operator pattern** extends Kubernetes to manage complex, stateful applications with the same automation and self-healing properties that Kubernetes brings to stateless workloads. An Operator packages human operational knowledge — how to install, upgrade, backup, restore, scale, and heal a specific application — as code that runs inside the cluster.

The term was coined by CoreOS in 2016. Since then, operators have become the standard packaging mechanism for cloud native data services, security tools, and platform components.

---

## What Is an Operator?

An Operator is the combination of:

1. **One or more Custom Resource Definitions (CRDs)** — the domain-specific API for the application
2. **One or more Controllers** — software that watches those CRDs and drives the application toward desired state

```
Operator = CRD(s) + Controller(s) + Operational Knowledge
```

A Kubernetes `Deployment` controller knows how to manage pods. An Operator knows how to manage, for example, a Kafka cluster — including bootstrapping, rolling upgrades, partition rebalancing, topic provisioning, and disaster recovery.

Without operators, you would need a team of Kafka experts maintaining runbooks and manually performing these operations. With an operator, that expertise is encoded once and runs continuously.

---

## The Operator Lifecycle: Observe, Diff, Act

Operators implement the same reconciliation loop as all Kubernetes controllers, but the **Act** phase is where domain knowledge lives:

```
┌──────────────────────────────────────────────────────┐
│                  Reconciliation Loop                  │
│                                                       │
│  1. OBSERVE: Read CR spec + status                    │
│              Query actual state of the application    │
│                                                       │
│  2. DIFF:    Compare desired vs actual                │
│              Determine what actions are needed        │
│                                                       │
│  3. ACT:     Create/update/delete child resources     │
│              Call application APIs (e.g., Kafka Admin)│
│              Update CR status with results            │
│              Set status.conditions                    │
└──────────────────────────────────────────────────────┘
```

The key distinction from a simple controller: the **Act** phase for an operator may involve complex, multi-step processes that require deep application knowledge — for example, knowing that you must add a Kafka broker before rebalancing partitions, or that you must take a Postgres checkpoint before doing an upgrade.

---

## Operator SDK

The **Operator SDK** (part of the Operator Framework project, CNCF) provides scaffolding and libraries for building operators in three styles:

### 1. Go-based Operators (controller-runtime)

Full control, maximum flexibility. Best for complex operators with intricate domain logic.

```bash
# Scaffold a new Go operator
operator-sdk init --domain example.com --repo github.com/example/my-operator
operator-sdk create api --group apps --version v1alpha1 --kind MyApp --resource --controller
```

The scaffolded controller uses `controller-runtime`:

```go
// controller-runtime reconciler interface
type MyAppReconciler struct {
    client.Client
    Scheme *runtime.Scheme
}

func (r *MyAppReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    log := log.FromContext(ctx)

    // 1. Fetch the MyApp instance
    myApp := &appsv1alpha1.MyApp{}
    if err := r.Get(ctx, req.NamespacedName, myApp); err != nil {
        if apierrors.IsNotFound(err) {
            return ctrl.Result{}, nil  // Object deleted, nothing to do
        }
        return ctrl.Result{}, err
    }

    // 2. Check if a Deployment already exists; if not, create one
    deployment := &appsv1.Deployment{}
    err := r.Get(ctx, types.NamespacedName{Name: myApp.Name, Namespace: myApp.Namespace}, deployment)
    if apierrors.IsNotFound(err) {
        dep := r.deploymentForMyApp(myApp)
        if err := r.Create(ctx, dep); err != nil {
            log.Error(err, "Failed to create Deployment")
            return ctrl.Result{}, err
        }
        return ctrl.Result{Requeue: true}, nil
    }

    // 3. Ensure replica count matches spec
    if *deployment.Spec.Replicas != myApp.Spec.Replicas {
        deployment.Spec.Replicas = &myApp.Spec.Replicas
        if err := r.Update(ctx, deployment); err != nil {
            return ctrl.Result{}, err
        }
    }

    // 4. Update status
    myApp.Status.ReadyReplicas = deployment.Status.ReadyReplicas
    if err := r.Status().Update(ctx, myApp); err != nil {
        return ctrl.Result{}, err
    }

    return ctrl.Result{}, nil
}
```

### 2. Ansible-based Operators

Use existing Ansible roles and playbooks as the reconciliation logic. No Go required.

```yaml
# watches.yaml — maps CR types to Ansible roles/playbooks
- version: v1alpha1
  group: apps.example.com
  kind: MyApp
  role: myapp          # runs roles/myapp/tasks/main.yml on reconcile
```

```yaml
# roles/myapp/tasks/main.yml
- name: Ensure Deployment exists
  kubernetes.core.k8s:
    state: present
    definition:
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: "{{ ansible_operator_meta.name }}"
        namespace: "{{ ansible_operator_meta.namespace }}"
      spec:
        replicas: "{{ replicas | default(1) }}"
        selector:
          matchLabels:
            app: "{{ ansible_operator_meta.name }}"
        template:
          spec:
            containers:
            - name: app
              image: "{{ image }}"
```

### 3. Helm-based Operators

Wraps an existing Helm chart in an operator. The CR spec maps to Helm `values.yaml`. Good for packaging existing Helm charts for OLM/OperatorHub distribution.

```yaml
# watches.yaml
- group: apps.example.com
  version: v1alpha1
  kind: MyApp
  chart: helm-charts/myapp
  overrideValues:
    replicaCount: "1"
```

---

## controller-runtime

`controller-runtime` (sigs.k8s.io/controller-runtime) is the Go library that underpins the Operator SDK and most production Go operators. It provides:

- **Manager**: bootstraps and runs multiple controllers in a single binary, handles leader election, metrics, health checks
- **Client**: typed CRUD client for Kubernetes objects with caching
- **Reconciler**: the interface every controller implements (`Reconcile(ctx, Request) (Result, error)`)
- **Builder**: fluent API for declaring what resources a controller watches

```go
// Setting up a controller with controller-runtime
func (r *MyAppReconciler) SetupWithManager(mgr ctrl.Manager) error {
    return ctrl.NewControllerManagedBy(mgr).
        For(&appsv1alpha1.MyApp{}).           // Primary resource to watch
        Owns(&appsv1.Deployment{}).            // Watch owned Deployments
        Owns(&corev1.Service{}).               // Watch owned Services
        WithOptions(controller.Options{
            MaxConcurrentReconciles: 3,        // Parallel reconciliations
        }).
        Complete(r)
}
```

---

## Finalizers and Owner References

### Finalizers

A **finalizer** is a string in `metadata.finalizers` that prevents an object from being deleted until a controller removes it. This enables **pre-delete cleanup** of external resources.

```yaml
metadata:
  name: my-kafka-cluster
  finalizers:
  - kafka.strimzi.io/delete-topics
  - kafka.strimzi.io/delete-users
```

**Lifecycle:**
1. User runs `kubectl delete kafkacluster my-kafka-cluster`
2. Kubernetes sets `metadata.deletionTimestamp` (object enters terminating state)
3. Strimzi operator detects `deletionTimestamp`, performs cleanup (deletes Kafka topics, users, PVCs)
4. Operator removes finalizers from the object
5. Kubernetes garbage collects the object

**Implementation pattern:**

```go
const myFinalizer = "myapp.example.com/finalizer"

func (r *Reconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    obj := &myv1.MyApp{}
    r.Get(ctx, req.NamespacedName, obj)

    if obj.DeletionTimestamp.IsZero() {
        // Object is not being deleted — ensure finalizer is registered
        if !controllerutil.ContainsFinalizer(obj, myFinalizer) {
            controllerutil.AddFinalizer(obj, myFinalizer)
            r.Update(ctx, obj)
        }
    } else {
        // Object is being deleted — perform cleanup
        if controllerutil.ContainsFinalizer(obj, myFinalizer) {
            r.cleanupExternalResources(obj)
            controllerutil.RemoveFinalizer(obj, myFinalizer)
            r.Update(ctx, obj)
        }
    }
    // ... rest of reconcile
}
```

### Owner References

**Owner references** create parent-child relationships between Kubernetes objects. When the parent is deleted, Kubernetes automatically garbage collects all objects that have it as an owner.

```yaml
metadata:
  name: myapp-deployment
  ownerReferences:
  - apiVersion: apps.example.com/v1alpha1
    kind: MyApp
    name: myapp
    uid: 9cbf7f47-c9b2-4d06-8f31-d12387cfa4d3
    controller: true           # this controller manages this object
    blockOwnerDeletion: true   # block parent deletion until child is gone
```

Use `controllerutil.SetControllerReference(owner, owned, scheme)` in Go to set this programmatically.

---

## Operator Maturity Levels

The **Operator Capability Levels** model (from OperatorHub.io) defines five maturity levels:

| Level | Name | Capabilities |
|-------|------|-------------|
| **Level 1** | Basic Install | Automated provisioning and configuration of the application |
| **Level 2** | Seamless Upgrades | Patch and minor version upgrades supported; knows upgrade sequence |
| **Level 3** | Full Lifecycle | App lifecycle (backup, failure recovery, reconfiguration without downtime) |
| **Level 4** | Deep Insights | Metrics, alerts, log processing, workload analysis published to the platform |
| **Level 5** | Auto Pilot | Horizontal/vertical scaling, auto-config tuning, anomaly detection and remediation |

Most production operators target Level 3-4. Level 5 (Autopilot) is rare and represents fully autonomous self-management.

---

## Real-World Operator Examples

### Prometheus Operator

Manages Prometheus instances, Alertmanager clusters, and scrape configuration through CRDs.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: platform-prometheus
  namespace: monitoring
spec:
  replicas: 2
  retention: 30d
  storage:
    volumeClaimTemplate:
      spec:
        resources:
          requests:
            storage: 100Gi
  serviceMonitorSelector:
    matchLabels:
      team: platform
  ruleSelector:
    matchLabels:
      prometheus: platform
```

```yaml
# ServiceMonitor — tells Prometheus what to scrape
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app
  labels:
    team: platform
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

### cert-manager

Automates TLS certificate management. Integrates with ACME (Let's Encrypt), Vault, and private CAs.

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: platform@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

### Strimzi Kafka Operator

Manages Apache Kafka clusters, including topics, users, and mirror-maker.

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: production-kafka
  namespace: kafka
spec:
  kafka:
    version: 3.6.0
    replicas: 3
    listeners:
    - name: plain
      port: 9092
      type: internal
      tls: false
    - name: tls
      port: 9093
      type: internal
      tls: true
    storage:
      type: persistent-claim
      size: 500Gi
      class: fast-storage
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 10Gi
  entityOperator:
    topicOperator: {}
    userOperator: {}
```

```yaml
# KafkaTopic CRD — self-service topic provisioning
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: orders
  namespace: kafka
  labels:
    strimzi.io/cluster: production-kafka
spec:
  partitions: 12
  replicas: 3
  config:
    retention.ms: "604800000"    # 7 days
    compression.type: snappy
```

### postgres-operator (Zalando)

Manages PostgreSQL clusters with streaming replication, automated failover, and connection pooling.

```yaml
apiVersion: acid.zalan.do/v1
kind: postgresql
metadata:
  name: production-postgres
  namespace: databases
spec:
  teamId: "platform"
  volume:
    size: 50Gi
  numberOfInstances: 3
  users:
    app_user:
    - login
    - createdb
  databases:
    appdb: app_user
  postgresql:
    version: "15"
  resources:
    requests:
      cpu: 500m
      memory: 500Mi
    limits:
      cpu: "1"
      memory: 1Gi
```

---

## OperatorHub

**OperatorHub.io** is the community registry for Kubernetes operators. It lists operators by category (database, security, streaming, etc.) and capability level.

Key aspects:
- Operators on OperatorHub are packaged using **OLM (Operator Lifecycle Manager)** bundles
- OLM manages operator installation, upgrades, and dependency resolution in the cluster
- Each operator bundle includes: CSVs (ClusterServiceVersions), CRDs, RBAC, install strategies
- Integrated into Red Hat OpenShift by default; installable standalone on any cluster

```bash
# Installing OLM on a cluster
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.26.0/install.sh | bash -s v0.26.0

# After OLM is installed, operators are installed via Subscriptions
```

```yaml
# Install an operator via OLM
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: prometheus
  namespace: operators
spec:
  channel: beta
  name: prometheus
  source: operatorhubio-catalog
  sourceNamespace: olm
  installPlanApproval: Automatic
```

---

## Building vs. Using Operators

| Scenario | Recommendation |
|----------|---------------|
| Managing a popular open-source project (Kafka, Postgres, Redis) | Use an existing operator from OperatorHub |
| Encoding your organization's internal application lifecycle | Build a custom operator with Operator SDK |
| Simple Helm chart packaging for OLM | Helm-based operator |
| Existing Ansible automation | Ansible-based operator |
| Complex stateful domain logic | Go-based operator with controller-runtime |
| Quick prototyping | Metacontroller (webhook-based, any language) |

---

## Summary

| Concept | Key Point |
|---------|-----------|
| Operator | CRD + Controller encoding application operational knowledge |
| Operator SDK | Scaffolding for Go, Ansible, and Helm operators |
| controller-runtime | Go library for building controllers (Manager, Client, Reconciler) |
| Finalizers | Prevent deletion; allow pre-delete cleanup of external state |
| Owner References | Enable automatic garbage collection of child resources |
| Maturity Levels | Install → Upgrades → Lifecycle → Insights → Autopilot |
| OperatorHub | Community registry; operators packaged with OLM bundles |
| Prometheus Operator | Manages Prometheus via ServiceMonitor/PrometheusRule CRDs |
| Strimzi | Manages Kafka clusters, topics, and users via CRDs |
| cert-manager | Automates TLS certificate issuance and renewal |
