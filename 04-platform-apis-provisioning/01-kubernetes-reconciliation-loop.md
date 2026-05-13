# Kubernetes Reconciliation Loop

## Overview

The reconciliation loop is the heartbeat of the Kubernetes control plane. Every controller in Kubernetes — whether it manages Deployments, Services, certificates, or cloud databases — operates on the same fundamental pattern: continuously compare actual state against desired state and take action to close any gap.

This model is called **declarative infrastructure**. Instead of issuing imperative commands ("create three pods now"), you declare what you want ("I want three pods running"), and the system figures out how to get there and stays there.

---

## The Control Loop Pattern

The core algorithm every controller implements is:

```
loop:
  desired  = read desired state from API server
  actual   = observe actual state of the world
  diff     = desired - actual
  if diff != empty:
    act(diff)
  sleep or wait for event
```

This is sometimes written as the **Observe → Diff → Act** cycle:

1. **Observe**: Read the current state of objects in the cluster
2. **Diff**: Compare current state to desired state (spec)
3. **Act**: Call APIs (Kubernetes or external) to move toward desired state, then update `status`

The controller then **re-queues** the object and runs again, either on a timer or when a relevant event occurs. This is why the system is **eventually consistent** — it converges over time rather than making synchronous guarantees.

---

## Level-Triggered vs. Edge-Triggered

Kubernetes controllers are **level-triggered**, not edge-triggered. This is a critical design property.

| Model | Description | Risk |
|-------|-------------|------|
| **Edge-triggered** | React to a change event | If you miss an event, you miss the action |
| **Level-triggered** | React to the current state | Re-running is always safe; no events can be missed |

Because controllers are level-triggered, **idempotency** is a hard requirement: calling the reconcile function multiple times for the same object must produce the same result. A controller that creates an external resource must first check whether it already exists.

---

## The Informer Pattern

Informers are the backbone of efficient controller operation. Rather than polling the API server on a timer, informers use a **Watch** mechanism backed by a local in-memory cache.

```
                     ┌──────────────────────────────┐
                     │         Informer             │
  API Server ──List──► SharedInformer               │
             ──Watch─►   ┌──────────────┐           │
                     │   │  Local Cache │           │
                     │   └──────────────┘           │
                     │   Event Handlers:             │
                     │     OnAdd, OnUpdate, OnDelete │
                     └──────────────────────────────┘
```

**How an Informer works:**
1. On startup, performs a **List** to populate the local cache
2. Opens a long-lived **Watch** stream for incremental updates
3. Triggers **event handlers** (Add, Update, Delete) when objects change
4. Re-syncs periodically (default: 10–12 hours) to catch any missed events

The **SharedInformer** is a key optimization: multiple controllers watching the same resource type share a single informer and cache, reducing load on the API server.

---

## Work Queues

Controllers use a **work queue** to decouple event detection (informer callbacks) from reconciliation logic. When an informer fires an event, the controller enqueues a key (`namespace/name`) rather than reconciling immediately.

Benefits of the work queue pattern:
- **Deduplication**: if an object changes five times rapidly, it is only reconciled once
- **Rate limiting**: prevents thundering-herd problems by throttling how fast items are processed
- **Retry with backoff**: failed reconciliations are re-queued with exponential backoff

```go
// Simplified controller loop in Go
func (c *Controller) processNextItem() bool {
    key, quit := c.queue.Get()
    if quit {
        return false
    }
    defer c.queue.Done(key)

    err := c.reconcile(key.(string))
    if err != nil {
        c.queue.AddRateLimited(key)  // retry with backoff
        return true
    }
    c.queue.Forget(key)
    return true
}
```

---

## Event-Driven Reconciliation

Controllers register **event handlers** with informers to feed the work queue:

```go
informer.AddEventHandler(cache.ResourceEventHandlerFuncs{
    AddFunc: func(obj interface{}) {
        key, _ := cache.MetaNamespaceKeyFunc(obj)
        queue.Add(key)
    },
    UpdateFunc: func(old, new interface{}) {
        key, _ := cache.MetaNamespaceKeyFunc(new)
        queue.Add(key)
    },
    DeleteFunc: func(obj interface{}) {
        key, _ := cache.DeletionHandlingMetaNamespaceKeyFunc(obj)
        queue.Add(key)
    },
})
```

Controllers can also **watch secondary resources** — for example, a Deployment controller watches both Deployments and their child ReplicaSets. When a ReplicaSet changes, the controller maps it back to the owning Deployment and enqueues that.

---

## Eventual Consistency

The Kubernetes API makes **no synchronous guarantee** that a change will be in effect immediately. When you apply a manifest, the API server validates and persists it to etcd, then returns success. What happens next is asynchronous:

1. Controller detects the change via Watch
2. Controller enqueues the object
3. Controller reconciles — may call external APIs that themselves take time
4. Status is updated to reflect progress

This means:
- A Pod being "Running" in `kubectl get pods` does not mean the application inside is ready
- `status.conditions` fields exist precisely to communicate sub-steps of convergence
- Operators should surface meaningful conditions (e.g., `Ready`, `Synced`, `Degraded`)

---

## Idempotency Requirements

Because reconciliation can be triggered at any time for any reason, controllers **must** handle being called repeatedly without side effects:

```yaml
# A controller creating this Secret must:
# 1. Check if the Secret already exists before creating
# 2. Compare existing content before updating
# 3. Not fail if the Secret already has the desired content
apiVersion: v1
kind: Secret
metadata:
  name: app-credentials
  namespace: production
type: Opaque
data:
  password: dXNlcjpwYXNz
```

Common idempotency patterns:
- Use `CreateOrUpdate` (controller-runtime) rather than `Create`
- Use server-side apply for field ownership tracking
- Hash resource contents and store in annotations to detect whether an update is needed
- Never generate random values (UUIDs, tokens) inside a reconcile function without checking if they already exist

---

## Reading Controller Logs

Controllers emit structured logs. Key things to look for:

```
# Successful reconciliation
{"level":"info","ts":"2024-01-15T10:23:11Z","msg":"Reconciling","controller":"deployment","namespace":"production","name":"frontend"}
{"level":"info","ts":"2024-01-15T10:23:11Z","msg":"Reconciliation complete","requeue":false}

# Requeue after error
{"level":"error","ts":"2024-01-15T10:23:12Z","msg":"Reconciliation failed","error":"connection refused","requeue-after":"30s"}

# Rate limiting
{"level":"info","ts":"2024-01-15T10:23:13Z","msg":"Reconciler rate-limited","wait":"5s"}
```

Key log fields:
| Field | Meaning |
|-------|---------|
| `controller` | Which controller is running |
| `namespace/name` | The object being reconciled |
| `requeue` | Whether the object will be re-processed |
| `requeue-after` | Scheduled requeue delay |
| `error` | The error that caused a failure |

Use `kubectl logs -n <namespace> <controller-pod>` with `-f` to follow in real time. Increase verbosity with `--v=4` or higher for the API server component to see Watch stream events.

---

## Common Controller Patterns

### 1. Owned Resources (Parent-Child)
A controller creates child resources and sets an **owner reference** on them. When the parent is deleted, Kubernetes garbage collects all children automatically.

```yaml
metadata:
  ownerReferences:
  - apiVersion: apps/v1
    kind: Deployment
    name: frontend
    uid: 9cbf7f47-...
    controller: true
    blockOwnerDeletion: true
```

### 2. Status Conditions
Controllers communicate sub-state using `status.conditions`:

```yaml
status:
  conditions:
  - type: Ready
    status: "True"
    lastTransitionTime: "2024-01-15T10:23:11Z"
    reason: AllReplicasAvailable
    message: "Deployment has minimum availability."
  - type: Progressing
    status: "False"
    reason: NewReplicaSetAvailable
```

### 3. Finalizers
Before deleting a resource, a controller may need to clean up external state (e.g., delete a cloud database). Finalizers prevent deletion until cleanup completes:

```yaml
metadata:
  finalizers:
  - database.example.com/delete-db-instance
```

The controller removes the finalizer only after external cleanup succeeds, at which point Kubernetes proceeds with deletion.

### 4. Generation and ObservedGeneration
`metadata.generation` increments on every spec change. Controllers set `status.observedGeneration` to match when they have finished reconciling that version, giving a reliable signal of "has the latest desired state been applied?"

---

## Summary

| Concept | Key Point |
|---------|-----------|
| Control Loop | Observe → Diff → Act, runs continuously |
| Level-triggered | React to state, not events — idempotency is required |
| Informer | List+Watch with local cache — efficient, no polling |
| Work Queue | Deduplicates and rate-limits reconciliation |
| Eventual Consistency | Changes converge asynchronously; use `status.conditions` |
| Finalizers | Block deletion until cleanup is complete |
| Owner References | Enable automatic garbage collection of child resources |
