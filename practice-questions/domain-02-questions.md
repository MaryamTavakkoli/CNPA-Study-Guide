# Domain 2: Observability, Security, and Conformance — Practice Questions

**Topics covered:** Metrics, logs, traces, mTLS, service mesh, OPA/Kyverno, RBAC, image security, CI security

---

**Question 1**

A platform team deploys Prometheus and Grafana. A developer asks why metrics are preferable to logs for capacity planning dashboards. What is the most accurate explanation?

A. Metrics are human-readable text strings, making them easier to parse than structured logs  
B. Metrics are pre-aggregated numeric time-series data optimised for querying trends over time, while logs are discrete events better suited for debugging specific occurrences  
C. Prometheus can replace a logging system entirely because it stores full request payloads  
D. Logs have higher cardinality than metrics, making them better for capacity dashboards  

**Answer: B**
Metrics are compact, pre-aggregated numerical values sampled over time — ideal for trend analysis, capacity planning, and alerting. Logs record discrete events with full context, which is better for root-cause investigation but less efficient for time-series analysis.

---

**Question 2**

Which of the three pillars of observability provides the ability to follow a single request as it traverses multiple microservices?

A. Metrics  
B. Logs  
C. Distributed tracing  
D. Alerting  

**Answer: C**
Distributed tracing assigns a unique trace ID to a request at entry and propagates it through every service call. Tools like Jaeger or Tempo collect spans and reconstruct the full request path, enabling latency analysis across service boundaries.

---

**Question 3**

What is the purpose of mTLS (mutual TLS) in a service mesh?

A. To encrypt data at rest in persistent volumes  
B. To provide both server-to-client and client-to-server certificate-based authentication, ensuring that only verified services can communicate with each other  
C. To terminate TLS at the ingress layer and forward plain HTTP inside the cluster  
D. To replace Kubernetes RBAC for authorising API server requests  

**Answer: B**
mTLS requires both parties in a connection to present and verify certificates, providing mutual authentication. In a service mesh (e.g., Istio, Linkerd), this is typically applied transparently via sidecar proxies, preventing unauthorised service-to-service communication.

---

**Question 4**

An OPA (Open Policy Agent) Gatekeeper ConstraintTemplate has been deployed. A developer submits a Deployment manifest without a `team` label. Based on standard OPA/Gatekeeper behaviour, what happens?

A. The Deployment is created but flagged in an audit report  
B. The Deployment is silently modified to add a default `team` label  
C. The request is denied by the admission webhook and the developer receives an error message  
D. The constraint is only evaluated after the resource is persisted to etcd  

**Answer: C**
Gatekeeper operates as a validating admission webhook. Policies in "deny" mode reject non-compliant resources before they are persisted to the cluster. The developer receives a rejection message explaining which constraint was violated.

---

**Question 5**

A Kubernetes RBAC Role is created with `verbs: ["get", "list"]` on `resources: ["pods"]` in the `default` namespace. Which of the following is true?

A. Any user in the cluster can get and list pods in all namespaces  
B. A user bound to this Role via a RoleBinding in the `default` namespace can get and list pods only within the `default` namespace  
C. The Role grants the ability to delete pods in the `default` namespace  
D. A ClusterRoleBinding is required for this Role to take effect  

**Answer: B**
A Role is namespace-scoped. A RoleBinding in the `default` namespace grants the permissions defined in the Role only within that namespace. To grant cluster-wide access, a ClusterRole and ClusterRoleBinding would be needed.

---

**Question 6**

Which tool is specifically designed to analyse container images for known CVEs (Common Vulnerabilities and Exposures) in OS packages and application dependencies?

A. Falco  
B. Trivy  
C. OPA Gatekeeper  
D. Prometheus  

**Answer: B**
Trivy (by Aqua Security, CNCF project) is a vulnerability scanner for container images, filesystems, Git repositories, and Kubernetes clusters. It detects CVEs in OS packages, language-specific dependencies, and misconfigurations.

---

**Question 7**

A Kyverno policy is applied to a cluster with `validationFailureAction: Enforce`. What is the effect when a Pod is submitted that violates the policy?

A. The Pod is created and a warning event is generated  
B. The Pod creation request is blocked and an error is returned to the user  
C. The Pod is created in a quarantine namespace for review  
D. The Kyverno policy is disabled after three violations  

**Answer: B**
When `validationFailureAction` is set to `Enforce`, Kyverno acts as a validating admission webhook and blocks non-compliant resources. Setting it to `Audit` instead would allow creation but log the violation.

---

**Question 8**

What does a Software Bill of Materials (SBOM) provide in the context of supply chain security?

A. A financial breakdown of software licensing costs  
B. An inventory of all components, libraries, and dependencies included in a software artifact, enabling vulnerability tracking and licence compliance  
C. A list of approved container registries that developers are permitted to use  
D. A runtime security policy that prevents unauthorised binaries from executing  

**Answer: B**
An SBOM is a structured list of all software components in an artifact. It enables security teams to quickly determine whether a newly disclosed CVE (e.g., Log4Shell) affects any of their deployed software by cross-referencing component inventories.

---

**Question 9**

A service mesh is configured to enforce `PeerAuthentication` in STRICT mode cluster-wide. What is the immediate effect on a newly deployed Pod that does not have a sidecar proxy injected?

A. The Pod continues to receive traffic normally because STRICT mode only applies to external ingress  
B. Traffic to and from the Pod is blocked because the mesh cannot establish mTLS with it  
C. The Pod is automatically restarted with a sidecar injected  
D. STRICT mode is only advisory and does not block non-mTLS traffic  

**Answer: B**
In STRICT mode (Istio terminology), the mesh requires mTLS for all traffic. A Pod without a sidecar cannot participate in mTLS, so connections to it from mesh-enabled services are rejected. This enforces the zero-trust networking model.

---

**Question 10**

In CI pipeline security, what is the purpose of signing container images with a tool like Cosign (Sigstore)?

A. To compress the image layers before pushing to a registry  
B. To cryptographically attest that an image was built by a trusted pipeline and has not been tampered with since signing  
C. To scan the image for vulnerabilities before it is pushed  
D. To automatically update image tags from `latest` to a digest reference  

**Answer: B**
Cosign (part of the Sigstore project) creates a cryptographic signature tied to the image digest. Consumers can verify the signature to confirm the image's provenance — that it was produced by a known, trusted CI system and has not been modified.

