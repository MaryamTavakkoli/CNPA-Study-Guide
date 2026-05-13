# Further Reading and Resources

A curated collection of books, official documentation, courses, and community resources to support CNPA exam preparation and ongoing learning in platform engineering.

---

## Books

### Platform Engineering and DevOps

**Team Topologies: Organising Business and Technology Teams for Fast Flow**
- Authors: Matthew Skelton and Manuel Pais
- Publisher: IT Revolution Press, 2019
- Why read it: Foundational framework for understanding platform teams, stream-aligned teams, and interaction modes. Core conceptual underpinning of the CNPA exam's platform architecture domain.

**Platform Engineering: A Guide for Technical, Product, and People Leaders**
- Authors: Camille Fournier and Ian Nowland
- Publisher: O'Reilly Media, 2024
- Why read it: A practical guide to building and operating internal developer platforms, written by practitioners. Covers platform-as-a-product thinking, metrics, and team structure.

**The DevOps Handbook: How to Create World-Class Agility, Reliability, and Security in Technology Organisations**
- Authors: Gene Kim, Patrick Debois, John Willis, Jez Humble
- Publisher: IT Revolution Press (2nd ed. 2021)
- Why read it: Comprehensive treatment of DevOps principles and practices. Covers the Three Ways (flow, feedback, continual learning) and provides the cultural and process context for CI/CD and platform engineering.

**Accelerate: The Science of Lean Software and DevOps**
- Authors: Nicole Forsgren, Jez Humble, Gene Kim
- Publisher: IT Revolution Press, 2018
- Why read it: The research foundation for DORA metrics. Essential reading for Domain 6 (Measuring Your Platform). Explains the statistical evidence linking DevOps practices to organisational performance.

**Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation**
- Authors: Jez Humble and David Farley
- Publisher: Addison-Wesley, 2010
- Why read it: The definitive text on continuous delivery pipelines, deployment pipeline patterns, and environment management. Core reference for Domain 3.

**The Site Reliability Workbook: Practical Ways to Implement SRE**
- Authors: Betsy Beyer, Niall Richard Murphy, David K. Rensin, Kent Kawahara, Stephen Thorne (Google)
- Publisher: O'Reilly Media, 2018
- Why read it: Practical implementation guide for SLOs, error budgets, and observability practices (Domain 2). Complements the original SRE Book with worked examples.

**Site Reliability Engineering: How Google Runs Production Systems**
- Authors: Betsy Beyer, Chris Jones, Jennifer Petoff, Niall Richard Murphy (Google)
- Publisher: O'Reilly Media, 2016
- Why read it: Defines SLI, SLO, SLA, error budgets, and the four golden signals. Available free online at sre.google/sre-book.

**Cloud Native Patterns: Designing Change-Tolerant Software**
- Author: Cornelia Davis
- Publisher: Manning Publications, 2019
- Why read it: Explains cloud native design patterns (microservices, dynamic configuration, service discovery) with Kubernetes examples. Useful background for Domains 1 and 4.

**Kubernetes in Action (2nd Edition)**
- Author: Marko Lukša
- Publisher: Manning Publications, 2024
- Why read it: The most comprehensive technical Kubernetes reference. Essential for understanding Deployments, StatefulSets, RBAC, CRDs, and the control plane — all tested in the CNPA exam.

**Production Kubernetes: Building Successful Application Platforms**
- Authors: Josh Rosso, Rich Lander, Alex Brand, John Harris
- Publisher: O'Reilly Media, 2021
- Why read it: Covers Kubernetes in production: multi-tenancy, security, policy, networking, and storage — directly relevant to Domains 1, 2, and 4.

---

## CNCF Project Documentation

The official documentation for CNCF projects is the highest-fidelity study material for tool-specific exam questions.

### GitOps and CD

- **Argo CD Documentation** — https://argo-cd.readthedocs.io
  Core concepts: Application, ApplicationSet, sync policies, health checks, RBAC.

- **Flux Documentation** — https://fluxcd.io/flux
  GitOps toolkit, Kustomization and HelmRelease CRDs, image automation.

- **Argo Rollouts Documentation** — https://argoproj.github.io/rollouts
  Canary and blue-green strategies, AnalysisTemplates, progressive delivery.

### CI

- **Tekton Documentation** — https://tekton.dev/docs
  Task, Pipeline, PipelineRun, Trigger concepts and YAML examples.

### Policy

- **OPA Documentation** — https://www.openpolicyagent.org/docs
  Rego language reference, integration patterns.

- **Gatekeeper Documentation** — https://open-policy-agent.github.io/gatekeeper
  ConstraintTemplate authoring, constraint enforcement modes, audit.

- **Kyverno Documentation** — https://kyverno.io/docs
  Policy types (validate, mutate, generate, verify image), ClusterPolicy vs Policy.

### Observability

- **Prometheus Documentation** — https://prometheus.io/docs
  Data model, PromQL, alerting rules, recording rules.

- **OpenTelemetry Documentation** — https://opentelemetry.io/docs
  Signals (traces, metrics, logs), SDK instrumentation, Collector configuration.

- **Jaeger Documentation** — https://www.jaegertracing.io/docs
  Architecture, sampling strategies, trace context propagation.

### Security

- **Falco Documentation** — https://falco.org/docs
  Rule syntax, built-in rules, eBPF driver, Kubernetes audit log support.

- **Trivy Documentation** — https://aquasecurity.github.io/trivy
  Scanning modes (image, fs, repo, k8s), SBOM generation, CI integration.

- **cert-manager Documentation** — https://cert-manager.io/docs
  Issuers, Certificate resources, ACME, Vault integration.

### Infrastructure Provisioning

- **Crossplane Documentation** — https://docs.crossplane.io
  Concepts: Providers, Managed Resources, Compositions, Composite Resources, Claims.

- **Cluster API Documentation** — https://cluster-api.sigs.k8s.io/introduction
  Declarative cluster lifecycle management.

### Developer Portals

- **Backstage Documentation** — https://backstage.io/docs
  Software catalog, Software Templates, TechDocs, plugin development.

### Image Security / Supply Chain

- **Sigstore / Cosign Documentation** — https://docs.sigstore.dev
  Keyless signing, attestations, policy enforcement with Cosign.

- **SLSA Framework** — https://slsa.dev
  Supply chain security levels, provenance, build requirements.

### Core Kubernetes

- **Kubernetes Official Documentation** — https://kubernetes.io/docs
  Concepts, Tasks, and API reference. Pay particular attention to:
  - Workloads (Deployments, StatefulSets, DaemonSets)
  - Configuration (ConfigMaps, Secrets)
  - Security (RBAC, Pod Security Admission, NetworkPolicy, ServiceAccounts)
  - Extending Kubernetes (CRDs, Admission Webhooks, Operators)
  - Resource Management (LimitRange, ResourceQuota)

---

## CNCF Landscape and Trail Map

- **CNCF Cloud Native Landscape** — https://landscape.cncf.io
  Interactive map of all CNCF-hosted and ecosystem projects, organised by capability area. Use this to understand how tools relate to each other.

- **CNCF Trail Map** — https://github.com/cncf/trailmap
  A recommended path through cloud native adoption, from containerisation to observability. Useful for understanding the order and priority of platform capabilities.

- **CNCF Technology Radar** — https://radar.cncf.io
  Community-sourced assessments of which tools are in "adopt," "trial," "assess," or "hold" categories, based on practitioner experience.

---

## Related Linux Foundation Courses and Certifications

These certifications from the Linux Foundation / CNCF complement the CNPA and share exam content:

| Certification | Abbreviation | Focus | Relationship to CNPA |
|--------------|--------------|-------|----------------------|
| Certified Kubernetes Administrator | CKA | Kubernetes cluster operations | Strong foundation for Domains 1 and 4 |
| Certified Kubernetes Application Developer | CKAD | Deploying and configuring applications on Kubernetes | Relevant to Domains 1, 2, and 3 |
| Certified Kubernetes Security Specialist | CKS | Kubernetes and cluster security | Deep dive into Domain 2 content |
| GitOps Fundamentals (LFS169) | — | Argo CD and Flux GitOps | Direct preparation for Domain 1 and 3 GitOps content |
| GitOps at Scale (LFS269) | — | Advanced GitOps patterns | Advanced content for Domain 1 and 3 |
| Introduction to DevSecOps for Managers (LFS180) | — | Security in DevOps pipelines | Relevant to Domain 2 (CI security, supply chain) |
| Prometheus and Grafana: Alerting and Monitoring (LFS241) | — | Prometheus, Grafana, Alertmanager | Core preparation for Domain 2 observability |

**Linux Foundation Training Portal:** https://training.linuxfoundation.org

---

## Community Resources

### Blogs and News

- **CNCF Blog** — https://www.cncf.io/blog
  Project announcements, case studies, and practitioner experience reports from CNCF members and end users.

- **The New Stack** — https://thenewstack.io
  News and analysis on cloud native technologies, platform engineering, DevOps, and Kubernetes.

- **InfoQ Cloud** — https://www.infoq.com/cloud
  In-depth articles and conference presentations on cloud native architecture and platform engineering.

- **Platform Engineering Community Blog** — https://platformengineering.org/blog
  Articles specifically focused on Internal Developer Platforms, Backstage, Golden Paths, and platform team organisation.

### Communities and Forums

- **CNCF Slack** — https://slack.cncf.io
  Active Slack workspace with channels for every major CNCF project (`#argo-cd`, `#flux`, `#prometheus`, `#backstage`, etc.) and general cloud native discussion. Excellent for getting answers from project maintainers.

- **Platform Engineering Slack** — https://platformengineering.org/slack-rd
  Community focused specifically on IDPs, developer experience, and platform teams.

- **Kubernetes Slack** — https://kubernetes.slack.com
  Official Kubernetes community Slack with channels for SIGs, API machinery, security, and general Kubernetes questions.

- **KubeWeekly Newsletter** — https://www.cncf.io/kubeweekly
  Weekly curated newsletter of Kubernetes and cloud native content, maintained by the CNCF.

### Conferences and Talks

- **KubeCon + CloudNativeCon** — https://events.linuxfoundation.org/kubecon-cloudnativecon
  The flagship CNCF conference. All past session recordings are available free on the CNCF YouTube channel. Keynotes and project deep-dives are excellent study material.

- **PlatformCon** — https://platformcon.com
  Annual virtual conference focused entirely on platform engineering. Session recordings cover IDPs, Golden Paths, Backstage, and measuring developer productivity.

- **CNCF YouTube Channel** — https://www.youtube.com/@cncf
  Recordings of all KubeCon sessions, project demos, and community meetings.

### Reference Papers and Reports

- **DORA State of DevOps Report** (annual) — https://dora.dev/research
  The annual research report covering DevOps and software delivery performance. Essential reading for Domain 6. The 2023 and 2024 reports introduce updated performance profiles and the DORA Core model.

- **Gartner Platform Engineering Hype Cycle** — Available via Gartner subscription
  Tracks the maturity and adoption of platform engineering practices in enterprise organisations.

- **Humanitec State of Platform Engineering Report** (annual) — https://humanitec.com/state-of-platform-engineering
  Annual survey of platform engineering practitioners covering tooling adoption, team structures, and maturity levels.
