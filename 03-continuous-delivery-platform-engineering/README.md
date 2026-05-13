# Domain 03: Continuous Delivery & Platform Engineering

> **Exam Weight: 16%**

This domain covers the practices, tools, and patterns that platform engineering teams use to deliver software reliably and repeatedly. It spans the full spectrum from individual CI pipeline stages through GitOps-based deployment automation and incident response when things go wrong.

---

## Topics in This Domain

| # | Topic | File |
|---|-------|------|
| 1 | CI Pipeline Anatomy & Templatization | [01-ci-pipelines-overview.md](./01-ci-pipelines-overview.md) |
| 2 | Incident Response in Platform Engineering | [02-incident-response.md](./02-incident-response.md) |
| 3 | The CI/CD Relationship & Fundamentals | [03-cicd-relationship-fundamentals.md](./03-cicd-relationship-fundamentals.md) |
| 4 | GitOps Basics & OpenGitOps Principles | [04-gitops-basics.md](./04-gitops-basics.md) |
| 5 | GitOps for Application Environments | [05-gitops-application-environments.md](./05-gitops-application-environments.md) |

---

## Key Concepts at a Glance

### Continuous Integration (CI)
- **Pipeline as Code**: Pipeline definitions live in version control alongside application code
- **Stages**: Logical groupings of jobs — build, test, security scan, publish
- **Artifacts**: Immutable outputs (container images, binaries, SBOMs) passed between stages
- **Caching**: Dependency caches (npm, Maven, Go modules) reduce redundant downloads
- **Matrix builds**: Run the same job across multiple OS/runtime version combinations
- **Templatization**: Platform teams provide reusable pipeline templates so application teams don't start from scratch

### Continuous Delivery / Deployment (CD)
- **Deployment pipeline**: Extends the build pipeline — takes a verified artifact through environment promotion
- **Artifact promotion**: The *same* image/binary is promoted through dev → staging → prod; never rebuilt
- **Environment gates**: Automated or manual checks that must pass before promotion continues
- **Approval workflows**: Human-in-the-loop gates, typically for production
- **Trunk-based development**: Short-lived branches merged frequently to keep the mainline always deployable
- **Feature flags**: Decouple deployment from feature release; enable dark launches and progressive rollout

### GitOps
- **Git as source of truth**: Desired state of the system is fully described in Git
- **Pull model**: Agents running *inside* the cluster reconcile actual state to desired state
- **Reconciliation loop**: Continuous compare-and-apply cycle performed by tools like ArgoCD or Flux
- **Drift detection**: Any out-of-band change to a live environment is detected and can be auto-corrected
- **Image tag strategies**: `latest` is an anti-pattern; use digests or immutable semver tags
- **App of Apps**: ArgoCD pattern to manage multiple `Application` resources from a single parent app

### Incident Response
- **Incident lifecycle**: Detection → Triage → Mitigation → Resolution → Post-mortem
- **Severity levels (SEV)**: Standardized impact ratings (SEV-1 = critical, SEV-4 = low) drive response urgency
- **Runbooks**: Pre-written, step-by-step operational guides for known failure modes
- **Blameless post-mortems**: Focus on systemic improvements, not individual fault
- **SLO / Error Budget**: Burning through error budget triggers incident escalation; healthy budget allows risk-taking
- **ChatOps**: Incident management via chat (Slack + PagerDuty/OpsGenie bots) for full audit trail

---

## How These Topics Connect

```
┌─────────────────────────────────────────────────────────────┐
│                   Developer Commits Code                    │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              CI Pipeline  (build → test → scan → publish)   │
│         GitHub Actions / GitLab CI / Tekton / Jenkins       │
└───────────────────────────┬─────────────────────────────────┘
                            │  immutable artifact (image digest)
                            ▼
┌─────────────────────────────────────────────────────────────┐
│           CD Pipeline / GitOps Reconciliation               │
│    Git PR updates image tag → ArgoCD/Flux detects drift     │
│    dev ──► staging ──► prod  (environment gates/approvals)  │
└───────────────────────────┬─────────────────────────────────┘
                            │
               ┌────────────┴────────────┐
               │                         │
               ▼                         ▼
   Progressive Delivery            Incident Response
   (Argo Rollouts / Flagger)   (PagerDuty / runbooks /
   canary, blue-green, A/B       post-mortems / SLOs)
```

---

## Exam Tips for This Domain

1. **Understand the pull vs. push distinction** — GitOps uses a pull model (agent pulls from Git), traditional CD pipelines use a push model (pipeline pushes to the cluster). Know why pull is preferred for security and auditability.
2. **Know the four OpenGitOps principles** by name and be able to apply them to scenarios.
3. **Artifact promotion** — the exam distinguishes between rebuilding vs. promoting the same artifact. Always promote; never rebuild.
4. **Blameless post-mortems** — understand the five sections (summary, timeline, root cause, impact, action items) and why blame is counterproductive.
5. **SLO/Error Budget connection** — a depleted error budget should freeze risky deployments and trigger incident review. This connects CI/CD policy to observability.
6. **Ephemeral environments** — understand how they are created per PR and torn down on merge.
7. **Templatization over duplication** — platform teams reduce cognitive load by providing pipeline templates (GitHub Actions reusable workflows, GitLab CI `include` templates, Tekton `Pipeline` objects).
