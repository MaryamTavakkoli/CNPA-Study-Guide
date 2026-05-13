# Domain 4: Platform APIs and Provisioning — Practice Questions

**Topics covered:** Reconciliation loop, CRDs, Crossplane, Operator pattern

---

**Question 1**

A platform team extends the Kubernetes API by defining a custom `Database` resource. Developers can now create a Database object in YAML just like a Deployment. What Kubernetes feature makes this possible?

A. ConfigMaps  
B. Custom Resource Definitions (CRDs)  
C. PersistentVolumeClaims  
D. Admission Webhooks  

**Answer: B**
CRDs extend the Kubernetes API with custom resource types. Once a CRD is registered, users can create, update, and delete instances of the custom type using standard Kubernetes tooling (kubectl, GitOps operators), and controllers can act on those resources.

---

**Question 2**

The Kubernetes control loop follows the "observe, diff, act" pattern. In the context of an Operator managing a PostgreSQL cluster, which scenario correctly illustrates the "act" phase?

A. The Operator reads the current `PostgresCluster` resource spec from the Kubernetes API  
B. The Operator compares the desired replica count in the spec with the number of running Pods  
C. The Operator creates additional Pods to match the desired replica count after detecting a deficit  
D. The Operator logs a warning event when it cannot schedule a new Pod  

**Answer: C**
The reconciliation loop: observe (read current state), diff (compare with desired state), act (take action to close the gap). Creating Pods to reach the desired replica count is the "act" phase — the controller is actively driving toward the declared desired state.

---

**Question 3**

A developer submits a Crossplane `SQLInstance` Composite Resource Claim (XRC). What happens next in the Crossplane provisioning model?

A. Crossplane deploys a container running PostgreSQL inside the Kubernetes cluster  
B. Crossplane's composite resource controller maps the claim to a Composition, which assembles a set of managed resources (e.g., a cloud SQL instance and its networking config) and provisions them in the target cloud provider  
C. Crossplane delegates provisioning to an external Terraform script triggered by a webhook  
D. The claim is queued in etcd and a human operator must approve it before resources are created  

**Answer: B**
In Crossplane, a Composite Resource Claim triggers a Composition — a template that maps the claim to one or more Managed Resources. The provider (e.g., provider-gcp, provider-aws) then reconciles those managed resources against the actual cloud infrastructure.

---

**Question 4**

What is the primary advantage of using the Operator pattern over a simple Helm chart for managing a stateful application like a message broker?

A. Operators use less cluster memory than Helm charts because they do not store release history  
B. Operators encode operational knowledge — such as ordered upgrades, backup procedures, and failure recovery — into controller logic that responds to state changes automatically, beyond what a Helm chart's templating can express  
C. Helm charts cannot manage StatefulSets, so Operators are the only option for stateful workloads  
D. Operators bypass the Kubernetes scheduler for better Pod placement  

**Answer: B**
Helm is excellent for templating and initial deployment but is stateless after install. Operators continuously watch resources and respond to events (node failures, schema migrations, replica scaling) using domain-specific logic that goes far beyond what a Helm lifecycle hook can express.

---

**Question 5**

A platform engineer wants to enable self-service provisioning of cloud storage buckets for developers. Using Crossplane, which resource type should the engineer expose to developers?

A. A Managed Resource (MR) that directly represents a cloud provider resource  
B. A Composite Resource Claim (XRC) backed by a Composition that abstracts the underlying cloud provider details  
C. A Kubernetes Secret containing cloud provider credentials  
D. A PersistentVolumeClaim bound to a StorageClass  

**Answer: B**
Exposing MRs directly would give developers access to low-level, provider-specific resources — a security and complexity concern. XRCs expose a simplified, organisation-defined API. The Composition translates the claim into the correct set of provider-specific managed resources, hiding cloud details from the developer.

---

**Question 6**

An Operator's controller is in a reconciliation loop but cannot reach the external API it depends on due to a network outage. Which behaviour is most consistent with best practices for Operator design?

A. The controller crashes and relies on Kubernetes to restart the Pod  
B. The controller marks the resource status as `Degraded` or `Unknown`, sets a retry backoff, and continues reconciling once connectivity is restored without losing the desired state  
C. The controller deletes the custom resource and notifies the developer to re-submit it after the outage  
D. The controller bypasses the external API and applies a hardcoded default configuration  

**Answer: B**
Well-designed Operators use status conditions to communicate resource health and implement exponential backoff retries. The desired state (the custom resource spec) is preserved; the controller simply cannot act on it until conditions allow. This is idiomatic Kubernetes controller behaviour.

