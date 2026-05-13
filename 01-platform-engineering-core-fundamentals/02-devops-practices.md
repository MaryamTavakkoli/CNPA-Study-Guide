# DevOps Practices in Platform Engineering

## What Is DevOps?

DevOps is a cultural and technical movement that **breaks down silos between development and operations** teams. The goal is to deliver software faster, more reliably, and with higher quality.

Core principles:
- **Collaboration** — Dev and Ops share ownership of the full delivery lifecycle
- **Automation** — eliminate manual, error-prone steps
- **Continuous improvement** — measure, learn, iterate
- **Shared responsibility** — "you build it, you run it"

---

## DevOps vs. SRE vs. Platform Engineering

These three disciplines are complementary, not competing.

| Discipline | Focus | Primary Question |
|---|---|---|
| **DevOps** | Culture and practice of Dev+Ops collaboration | How do we ship faster with higher quality? |
| **Site Reliability Engineering (SRE)** | Applying software engineering to operations | How do we keep systems reliable at scale? |
| **Platform Engineering** | Building internal products for developers | How do we reduce cognitive load on dev teams? |

**SRE operationalizes DevOps** — it gives concrete practices (error budgets, SLOs) to the DevOps philosophy.

**Platform Engineering productizes DevOps** — instead of each team implementing DevOps practices themselves, a platform team builds shared tooling and "golden paths" so developers get DevOps benefits without managing complexity.

---

## The DevOps Infinity Loop

The DevOps lifecycle is often visualized as an infinity loop (∞):

```
Plan → Code → Build → Test → Release → Deploy → Operate → Monitor
                                                              ↑           ↓
                                                           (feedback feeds back into Plan)
```

Platform engineers build the tooling that automates as much of this loop as possible.

---

## Key DevOps Practices

### Shift Left

Moving testing, security, and quality checks **earlier** in the development cycle — toward the developer, before code reaches production.

- Run unit tests on every commit
- Scan for vulnerabilities in the IDE or on PR creation
- Validate infrastructure-as-code before apply

**Why it matters**: Finding bugs early is cheaper. A bug found in production costs 100x more to fix than one found in development.

### Infrastructure as Code (IaC)

Manage infrastructure using code files tracked in version control, just like application code.

**Tools:**
- **Terraform** — provider-agnostic, HCL syntax, plan/apply workflow
- **Pulumi** — IaC using real programming languages (Python, Go, TypeScript)
- **AWS CloudFormation** — AWS-native declarative IaC
- **Crossplane** — Kubernetes-native IaC (covered in Domain 4)

```hcl
# Terraform example
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  tags = {
    Name = "web-server"
  }
}
```

Benefits of IaC:
- Reproducibility — spin up identical environments
- Version control — track who changed what and when
- Code review — changes go through PR process
- Automation — apply changes in CI/CD pipelines

### Everything as Code (EaC)

The extension of IaC to all operational concerns:

- **Configuration as Code** — app config in version-controlled files
- **Policy as Code** — security and compliance rules written as code (OPA, Kyverno)
- **Pipelines as Code** — CI/CD pipelines defined in YAML files (Jenkinsfile, `.github/workflows`)
- **Security as Code** — security scanning rules and policies in code

### Continuous Feedback

Platforms must surface information back to developers:
- Build and test results
- Deployment status
- Runtime errors and performance metrics
- Security scan results

---

## The Three Ways of DevOps

From *The Phoenix Project* and *The DevOps Handbook*:

1. **Systems Thinking (Flow)**: Optimize the whole system, not individual silos. Maximize throughput from Dev to Ops to customer.

2. **Amplify Feedback Loops**: Create fast, rich feedback from right to left (from production back to development). Shorten and amplify feedback.

3. **Culture of Continual Experimentation and Learning**: Create a culture where it's safe to take risks. Learn from failures. Deliberately practice resilience.

---

## Platform Engineering's Role in DevOps

Platform engineering teams act as **internal product teams**. Their customers are the internal development teams.

They apply product thinking to developer tooling:
- Interview internal users (developers) to understand pain points
- Build solutions that remove toil and cognitive load
- Measure adoption and developer satisfaction
- Iterate based on feedback

**Cognitive load reduction** is a primary goal. Developers should not need to understand Kubernetes networking, certificate management, or observability infrastructure — the platform handles it.

### Team Topologies

Platform engineering is closely tied to the *Team Topologies* model:

| Team Type | Role |
|---|---|
| **Stream-aligned team** | Builds and delivers a specific product/service |
| **Platform team** | Provides internal platform to reduce cognitive load on stream-aligned teams |
| **Enabling team** | Helps stream-aligned teams acquire new capabilities temporarily |
| **Complicated subsystem team** | Manages a subsystem requiring deep specialist knowledge |

The platform team's goal: make stream-aligned teams as autonomous as possible.

---

## Golden Paths

A **Golden Path** (also called "paved road") is an opinionated, supported, and well-documented path for building and deploying software. It is:

- **Pre-configured** — security, logging, monitoring, CI/CD baked in
- **Self-service** — developers can use it without asking the platform team
- **Optional but incentivized** — developers can deviate, but the golden path is easier

Example: a scaffolding CLI that generates a new service with a Dockerfile, CI pipeline, Kubernetes manifests, Grafana dashboards, and alerting rules already configured.

---

## Key Takeaways

- DevOps is culture + practice; SRE operationalizes it; platform engineering productizes it
- Shift left = move quality checks earlier, toward the developer
- IaC makes infrastructure reproducible and auditable
- The Three Ways: systems thinking, amplify feedback, continual learning
- Platform teams are internal product teams; their customers are developers
- Golden paths reduce cognitive load by providing opinionated, pre-built patterns
