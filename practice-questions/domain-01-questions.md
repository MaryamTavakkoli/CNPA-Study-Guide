# Domain 1: Platform Engineering Core Fundamentals — Practice Questions

**Topics covered:** Declarative management, DevOps practices, environments, platform architecture, CI, GitOps

---

**Question 1**

A platform team wants to ensure that the desired state of all cluster resources is always stored in version control and automatically applied. Which approach best describes this practice?

A. Continuous Integration with automated unit tests  
B. GitOps, using Git as the single source of truth for declarative infrastructure  
C. Imperative scripting with Ansible playbooks stored in a wiki  
D. Manual kubectl apply commands triggered by a ticketing system  

**Answer: B**
GitOps treats Git as the source of truth for declarative infrastructure and application state. Changes are made via pull requests and an operator continuously reconciles the live state to match the desired state in Git.

---

**Question 2**

Which of the following best defines a "platform" in the context of cloud native platform engineering?

A. A set of virtual machines provisioned by the infrastructure team on demand  
B. A self-service layer of APIs, tools, and services that enables developers to build, run, and manage applications without dealing with underlying complexity  
C. A monolithic deployment pipeline managed exclusively by the operations team  
D. A container runtime installed on every developer laptop  

**Answer: B**
A platform abstracts infrastructure complexity and provides self-service capabilities to developers. It acts as a product consumed by internal development teams, enabling them to focus on application logic rather than operational concerns.

---

**Question 3**

In a declarative management model, what is the primary responsibility of the platform operator or controller?

A. To execute step-by-step imperative commands provided by developers  
B. To continuously compare the observed cluster state with the desired state and reconcile any differences  
C. To provision resources only when a developer submits a ticket  
D. To replace CI pipelines by building container images directly  

**Answer: B**
Declarative management relies on a control loop (reconciliation loop) that watches for drift between the desired state (declared in manifests) and the observed state of the system, taking corrective action automatically.

---

**Question 4**

A team adopts trunk-based development with short-lived feature branches. Which CI practice best complements this model?

A. Running a full regression suite only on the main branch once per week  
B. Merging feature branches into a long-lived integration branch before testing  
C. Triggering an automated build and test pipeline on every commit, with merge blocked if tests fail  
D. Disabling automated tests to speed up delivery  

**Answer: C**
Trunk-based development pairs with frequent, fast CI pipelines. Every commit should trigger builds and tests, and a failing pipeline should block merges to maintain a consistently deployable trunk.

---

**Question 5**

Which of the following is a defining characteristic of the "DevOps" practice relevant to platform engineering?

A. Strict separation of development and operations teams to avoid responsibility overlap  
B. Shared ownership of the full software delivery lifecycle, including building, deploying, and operating applications  
C. Delegating all operational concerns to a dedicated SRE team with no developer involvement  
D. Replacing agile practices with a waterfall model for more predictable delivery  

**Answer: B**
DevOps breaks down silos between development and operations, fostering shared ownership across the full delivery lifecycle. Platform engineering supports this by providing self-service tools that let developers own more of the operational stack.

---

**Question 6**

A developer wants to promote an application from a staging environment to production. In a GitOps model, which action initiates this promotion?

A. Logging into the cluster and running `kubectl set image`  
B. Sending an email to the operations team requesting a deployment  
C. Opening a pull request that updates the image tag or configuration in the production Git repository or branch  
D. Uploading a binary artifact directly to the production cluster via FTP  

**Answer: C**
In GitOps, all changes — including environment promotions — are made through Git. A pull request updates the desired state for the target environment; once merged, the GitOps operator (e.g., Argo CD) applies the change.

---

**Question 7**

Which statement best describes the difference between a "platform team" and an "enabling team" in the context of Team Topologies?

A. Platform teams build products consumed by stream-aligned teams; enabling teams provide expertise to help teams adopt new capabilities  
B. Platform teams write application code; enabling teams manage infrastructure  
C. Both team types are interchangeable and have no distinguishing responsibilities  
D. Enabling teams replace platform teams once a platform reaches maturity  

**Answer: A**
Team Topologies distinguishes platform teams (which build and operate an internal platform as a product) from enabling teams (which act as consultants or coaches, helping other teams adopt practices and capabilities without building the platform themselves).

---

**Question 8**

A company runs separate Kubernetes clusters for development, staging, and production environments. What is the primary reason for maintaining environment parity?

A. To reduce cloud costs by sharing the same cluster resources across all workloads  
B. To ensure that behavior validated in lower environments accurately predicts behavior in production, reducing deployment surprises  
C. To allow developers to deploy directly to production without passing through staging  
D. To comply with Kubernetes licensing requirements that mandate separate clusters  

**Answer: B**
Environment parity — keeping development, staging, and production as similar as possible — reduces the "it works on my machine" problem. Differences between environments are a common source of production incidents.

---

**Question 9**

Which of the following Kubernetes objects is most closely associated with declarative application configuration management?

A. PersistentVolumeClaim  
B. HorizontalPodAutoscaler  
C. ConfigMap and Helm Chart (values-based templating)  
D. ServiceAccount  

**Answer: C**
ConfigMaps and Helm Charts are foundational to declarative configuration management. Helm allows parameterized, version-controlled manifests; ConfigMaps externalise configuration from container images. Together they represent core declarative practices.

---

**Question 10**

What problem does Kustomize solve in a GitOps workflow?

A. It builds container images from Dockerfiles and pushes them to a registry  
B. It provides environment-specific overlays on top of base Kubernetes manifests without duplicating configuration  
C. It monitors Kubernetes clusters and sends alerts when resources drift from desired state  
D. It replaces Helm charts by providing a package repository for Kubernetes applications  

**Answer: B**
Kustomize uses a base + overlays model. A base manifest defines the common configuration; overlays (for dev, staging, prod) patch only the differences. This avoids copy-pasting manifests while keeping everything in Git.

---

**Question 11**

A platform architect is designing the CI stage of a delivery pipeline. Which capability should the CI system provide that is MOST directly related to platform engineering goals?

A. Writing application business logic as a service  
B. Publishing versioned, tested artifacts (container images, Helm charts) to a registry upon successful build  
C. Directly modifying production Kubernetes resources when a commit is pushed  
D. Managing DNS records for all application endpoints  

**Answer: B**
CI's role in a platform engineering context is to produce reliable, tested, versioned artifacts. These artifacts are then consumed by the CD/GitOps layer. Publishing to a registry is the handoff point between CI and CD.

---

**Question 12**

Which of the following describes the "pull" model in GitOps, as opposed to a "push" model?

A. Developers push code to Git and the CI system pushes manifests directly to the cluster  
B. An agent running inside the cluster continuously pulls desired state from Git and applies it, rather than an external system pushing changes into the cluster  
C. Operations teams pull logs from the cluster to a central log aggregation system  
D. Developers pull base images from a registry and push custom images to production  

**Answer: B**
The pull model (used by Argo CD and Flux) places an agent inside the cluster. This agent polls Git and applies changes, which improves security (no external credentials needed to access the cluster) and provides continuous drift detection.

---

**Question 13**

A platform team wants to enforce that all Kubernetes Deployments include resource limits. Where is the MOST appropriate place to enforce this?

A. In every developer's local IDE as a linting rule  
B. In a Git pre-commit hook only  
C. As an admission controller policy (e.g., OPA/Gatekeeper or Kyverno) enforced at the Kubernetes API server  
D. As a comment in the team's coding standards document  

**Answer: C**
Admission controllers enforce policies at the cluster level before resources are persisted. This is the most reliable enforcement layer because it is independent of developer tooling and cannot be bypassed by skipping local checks.

---

**Question 14**

Which of the following best describes "platform as a product" thinking?

A. Selling access to the internal developer platform to external customers  
B. Treating the internal platform like a commercial product: gathering developer feedback, defining a roadmap, measuring adoption, and iterating based on user needs  
C. Purchasing a commercial SaaS platform and making it available to developers  
D. Packaging the platform as a container and distributing it via a registry  

**Answer: B**
"Platform as a product" applies product management disciplines to internal platforms. This means user research, a prioritised backlog, versioned releases, SLOs for platform services, and feedback loops with developer users.

---

**Question 15**

In a multi-tenant Kubernetes cluster, which mechanism is primarily used to provide logical isolation between different teams' workloads?

A. Separate Docker networks per team  
B. Kubernetes Namespaces combined with RBAC policies and Network Policies  
C. Running each team's workloads in a separate container runtime  
D. Assigning each team a dedicated physical node with taints  

**Answer: B**
Namespaces provide a logical boundary within a cluster. Combined with RBAC (to control who can access resources in a namespace) and Network Policies (to control traffic between namespaces), they are the primary multi-tenancy mechanism in Kubernetes.
