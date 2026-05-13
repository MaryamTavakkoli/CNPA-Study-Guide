# Domain 3: Continuous Delivery in Platform Engineering — Practice Questions

**Topics covered:** CD pipelines, incident response, CI/CD relationship, GitOps basics, environment promotion

---

**Question 1**

What is the key distinction between Continuous Delivery and Continuous Deployment?

A. Continuous Delivery requires manual approval before each production release; Continuous Deployment automatically releases every successful build to production  
B. Continuous Delivery is only applicable to frontend applications; Continuous Deployment is for backend services  
C. Continuous Delivery uses push-based GitOps; Continuous Deployment uses pull-based GitOps  
D. They are synonymous terms used interchangeably in the industry  

**Answer: A**
Continuous Delivery ensures software is always in a releasable state with a human gate before production. Continuous Deployment removes that gate, automatically deploying every change that passes all pipeline stages. The choice depends on organisational risk tolerance.

---

**Question 2**

During an incident, a platform team identifies that a recent deployment caused a spike in 5xx errors. Which CD practice enables the fastest recovery with the least manual intervention?

A. Rebuilding the container image from scratch and redeploying  
B. Automated rollback triggered by a failed health check or error-rate threshold in the deployment pipeline  
C. Opening a ticket for the operations team to investigate the issue  
D. Scaling down the affected Deployment to zero replicas until the issue is diagnosed  

**Answer: B**
Automated rollback based on post-deployment health checks (e.g., Argo Rollouts with analysis metrics) minimises Mean Time to Recovery (MTTR). It eliminates the delay of manual detection and intervention during a live incident.

---

**Question 3**

A team wants to use a canary deployment strategy to reduce risk when releasing a new version. Which behaviour correctly describes a canary release?

A. The new version is deployed to a subset of users or traffic while the majority continues to receive the stable version, allowing gradual validation before a full rollout  
B. The new version is deployed to production only during off-peak hours and immediately replaces the old version  
C. Two identical environments are maintained; traffic is shifted 100% to the new environment at a specific cutover time  
D. The new version is deployed only to staging and never to production until manual testing is complete  

**Answer: A**
A canary release routes a small percentage of traffic to the new version. Metrics are monitored; if the canary is healthy, traffic is incrementally shifted. If issues arise, the canary is rolled back without impacting the majority of users.

---

**Question 4**

In a GitOps-driven CD workflow using Argo CD, what triggers a synchronisation of the cluster state?

A. A developer manually runs `kubectl apply` after merging a pull request  
B. Argo CD detects a difference between the desired state in Git and the live state in the cluster, either via polling or a webhook notification  
C. A cron job runs nightly to apply all pending manifests from Git  
D. A CI pipeline pushes manifests directly to the Kubernetes API server  

**Answer: B**
Argo CD continuously compares the desired state (Git) with the live state (cluster). Sync can be triggered automatically on drift detection (auto-sync) or manually. Git webhooks can also push notifications to Argo CD to trigger immediate synchronisation.

---

**Question 5**

A team practices "shift-left" security in their CD pipeline. Which of the following activities is most aligned with this principle?

A. Running vulnerability scans on container images after they are deployed to production  
B. Performing penetration testing once per year during a scheduled maintenance window  
C. Integrating SAST, dependency vulnerability scanning, and container image scanning into the CI pipeline so issues are caught before artifacts are promoted  
D. Delegating all security testing to a separate security team that reviews changes quarterly  

**Answer: C**
"Shift-left" means moving security activities earlier in the development lifecycle — ideally into CI. Catching vulnerabilities before an artifact is built or promoted is cheaper and faster to remediate than finding them post-deployment.

---

**Question 6**

Which metric is most directly improved by implementing automated deployment pipelines with rollback capabilities?

A. Deployment Frequency  
B. Mean Time to Recovery (MTTR)  
C. Change Lead Time  
D. Change Failure Rate  

**Answer: B**
Automated rollback directly reduces MTTR by shortening the time between incident detection and service restoration. While other DORA metrics benefit from CI/CD practices, MTTR is most directly impacted by fast, automated recovery mechanisms.

---

**Question 7**

A platform team needs to promote a Helm chart from staging to production. Using a GitOps approach with separate environment folders, what is the correct sequence?

A. Run `helm upgrade` against the production cluster directly from a developer laptop  
B. Copy the staging values file to the production folder in Git, open a pull request, get it approved, and merge — allowing the GitOps operator to apply the change  
C. Delete the staging deployment and re-deploy it in the production namespace  
D. Export the staging cluster state using `kubectl get all -o yaml` and apply it to production  

**Answer: B**
GitOps environment promotion is a Git operation. Changes to the production environment folder (image tags, values) are made via a pull request, enabling review and audit. The GitOps operator (Argo CD, Flux) then reconciles the cluster to the new desired state.

---

**Question 8**

An incident post-mortem identifies that the root cause of a production outage was a misconfigured environment variable introduced in a deployment. Which CD safeguard would most likely have prevented this incident?

A. Increasing the number of replicas in production  
B. Automated integration and smoke tests in a staging environment that validate application behaviour with the new configuration before promoting to production  
C. Using a blue-green deployment strategy at a higher cadence  
D. Storing environment variables in plain text in the application repository  

**Answer: B**
Staging environment tests that exercise the application with real configuration values (or production-like values) catch misconfiguration before promotion. A smoke test validating key application endpoints would surface a misconfigured env var early in the pipeline.

