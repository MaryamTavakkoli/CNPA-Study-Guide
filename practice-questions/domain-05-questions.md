# Domain 5: Internal Developer Portals and Developer Experience — Practice Questions

**Topics covered:** Service catalogs, Backstage, developer portals, AI/ML in platform engineering

---

**Question 1**

A platform team deploys Backstage as an Internal Developer Portal. A developer opens the software catalog and clicks on a microservice they own. Which type of information would they MOST expect to find there?

A. The live CPU and memory metrics for the service's production pods  
B. Service ownership, a link to the source repository, documentation, deployed environments, recent CI build status, and API specifications  
C. The Kubernetes YAML manifests defining the service's Deployments  
D. A list of all Kubernetes cluster nodes the service has ever been scheduled on  

**Answer: B**
Backstage's software catalog aggregates metadata about services — ownership, repos, docs, API specs, dependencies, and integrated plugin data (CI status, deployments, SLOs). The goal is a single pane of glass, not a real-time metrics dashboard (though plugins can embed those).

---

**Question 2**

A Backstage Software Template is created for a new microservice. A developer fills in the template form and clicks "Create." What is the expected outcome?

A. Backstage sends a Slack message to the platform team, who then manually scaffold the repository  
B. Backstage executes the template's scaffolding steps, creating a Git repository pre-populated with code, CI pipeline configuration, Kubernetes manifests, and registering the service in the catalog — all automatically  
C. Backstage provisions a Kubernetes namespace and waits for the developer to push code manually  
D. The template generates a static PDF document with setup instructions for the developer to follow  

**Answer: B**
Software Templates (Golden Paths) in Backstage automate the creation of new services. They execute actions — creating repos, adding files, triggering CI, registering catalog entries — so developers start with a fully configured, opinionated project structure in minutes.

---

**Question 3**

In the context of platform engineering, what is the primary purpose of a "Golden Path"?

A. To enforce a mandatory, unchangeable set of tools that all teams must use, with no exceptions permitted  
B. To provide a pre-built, opinionated, and well-supported path for common development tasks that reduces cognitive load, while allowing teams to deviate when they have a justified reason  
C. To document the historical evolution of the platform from its initial inception to its current state  
D. To define the networking path that requests take from the internet to a backend microservice  

**Answer: B**
A Golden Path is an opinionated but not mandatory route that embodies platform best practices. It reduces toil by providing ready-made templates, but teams should be able to leave the path when their use case demands it. Mandatory paths without escape hatches create friction and shadow IT.

---

**Question 4**

How does AI/ML integration in an Internal Developer Portal primarily improve developer productivity?

A. By replacing the need for developers to write any code, using AI to generate full application architectures automatically  
B. By offering capabilities such as AI-assisted search across the service catalog, intelligent code generation from templates, and automated documentation summarisation — reducing time spent on context switching and knowledge discovery  
C. By training a machine learning model on production traffic to automatically scale services  
D. By using AI to review and approve all pull requests before they are merged to main  

**Answer: B**
AI integration in IDPs (e.g., AI-powered search, LLM-generated docs summaries, code scaffolding suggestions) addresses developer experience pain points around discoverability and boilerplate. The goal is reducing cognitive load, not replacing developer judgment.

