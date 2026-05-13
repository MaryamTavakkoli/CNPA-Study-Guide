# Policy Engines for Platform Governance

## What Is Policy as Code?

**Policy as Code** means expressing organizational rules and compliance requirements as machine-readable code that is automatically enforced.

Instead of:
- Documentation no one reads
- Manual reviews that miss things
- Firefighting after violations are discovered

You get:
- Automated enforcement at admission time (before resources are created)
- Consistent application across all teams and clusters
- Version-controlled policies with change history
- Immediate developer feedback

---

## Admission Controllers

Kubernetes **admission controllers** are plugins that intercept API requests before they are persisted to etcd.

```
kubectl apply → API Server → Authentication → Authorization → Admission Control → etcd
```

Two types:
- **Validating admission webhooks**: Allow or reject a request
- **Mutating admission webhooks**: Modify a request (e.g., inject sidecars, add labels)

Policy engines hook into Kubernetes via **validating and mutating webhook admission controllers**.

---

## OPA (Open Policy Agent) and Gatekeeper

### OPA

**Open Policy Agent** is a general-purpose policy engine. It uses **Rego** as its policy language.

OPA can enforce policies for:
- Kubernetes (via Gatekeeper)
- Terraform plans
- API authorization
- CI/CD pipelines
- Microservices authorization

### Gatekeeper

**OPA Gatekeeper** is the Kubernetes-native integration for OPA. It provides:
- CRDs for defining **constraint templates** (policy logic in Rego)
- CRDs for defining **constraints** (policy instances with parameters)
- Audit mode: scan existing resources for violations

```yaml
# ConstraintTemplate: defines the policy logic
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        openAPIV3Schema:
          properties:
            labels:
              type: array
              items: {type: string}
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        violation[{"msg": msg}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("Missing required labels: %v", [missing])
        }
```

```yaml
# Constraint: instantiates the policy
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-team-label
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels: ["team", "environment"]
```

**Common OPA/Gatekeeper use cases:**
- Require labels on all resources
- Prohibit `latest` image tag
- Require resource limits on all containers
- Restrict which image registries are allowed
- Enforce naming conventions

---

## Kyverno

Kyverno is a Kubernetes-native policy engine that uses **YAML** instead of Rego. This makes it more accessible to Kubernetes operators who are already familiar with YAML.

Kyverno policy types:
- **Validate**: Reject non-compliant resources
- **Mutate**: Modify resources (add labels, inject configs)
- **Generate**: Create related resources (e.g., create a NetworkPolicy when a Namespace is created)
- **VerifyImages**: Verify container image signatures

```yaml
# Kyverno policy: require resource limits
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-resource-limits
spec:
  validationFailureAction: Enforce
  rules:
    - name: check-resource-limits
      match:
        any:
          - resources:
              kinds: [Pod]
      validate:
        message: "Resource limits are required for all containers."
        pattern:
          spec:
            containers:
              - name: "*"
                resources:
                  limits:
                    memory: "?*"
                    cpu: "?*"
```

```yaml
# Kyverno mutate: add default labels
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-default-labels
spec:
  rules:
    - name: add-managed-by-label
      match:
        any:
          - resources:
              kinds: [Deployment]
      mutate:
        patchStrategicMerge:
          metadata:
            labels:
              managed-by: platform-team
```

```yaml
# Kyverno generate: create NetworkPolicy on Namespace creation
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-default-network-policy
spec:
  rules:
    - name: default-deny
      match:
        any:
          - resources:
              kinds: [Namespace]
      generate:
        apiVersion: networking.k8s.io/v1
        kind: NetworkPolicy
        name: default-deny-all
        namespace: "{{request.object.metadata.name}}"
        data:
          spec:
            podSelector: {}
            policyTypes: [Ingress, Egress]
```

---

## OPA/Gatekeeper vs. Kyverno

| Feature | OPA/Gatekeeper | Kyverno |
|---|---|---|
| Policy language | Rego (purpose-built) | YAML |
| Learning curve | Higher (Rego is new) | Lower (familiar YAML) |
| Flexibility | Very high (Rego is general-purpose) | High within YAML structure |
| Mutate policies | Via mutation webhook | Native support |
| Generate policies | No | Yes |
| Image verification | No (use separate tool) | Yes (built-in) |
| Use outside Kubernetes | Yes (Terraform, APIs) | No |
| CNCF status | Graduated | Incubating |

---

## Policy Enforcement Modes

Both tools support two enforcement modes:

| Mode | Behavior |
|---|---|
| **Enforce / Fail** | Reject non-compliant resources at admission time |
| **Audit / Warn** | Allow but report violations; good for gradual rollout |

Start in audit mode to detect existing violations before switching to enforce mode.

---

## Platform Governance Use Cases

Policies the platform team typically enforces:

| Policy | Why |
|---|---|
| No `latest` image tag | Ensures deterministic deployments |
| Required resource requests/limits | Prevents noisy neighbors |
| Required labels (team, app, env) | Enables cost allocation, routing |
| Allowed image registries | Prevent pulling from untrusted sources |
| No privileged containers | Security posture |
| Required liveness/readiness probes | Ensures proper health checking |
| Network policies required | Defense in depth |
| Required annotations for runbooks | Operational excellence |

---

## Key Takeaways

- Policy as Code enforces rules automatically at admission time — no manual review needed
- Kubernetes admission webhooks are how policies intercept resource creation/updates
- OPA/Gatekeeper uses Rego (powerful, steep learning curve); Kyverno uses YAML (easier to start)
- Kyverno uniquely supports generate policies (creating resources from events)
- Start policies in audit mode; switch to enforce after validating against existing workloads
- Platform teams use policies to enforce security, labeling, resource management, and registry trust
