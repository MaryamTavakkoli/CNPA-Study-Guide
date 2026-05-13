# Continuous Integration Fundamentals

## What Is Continuous Integration?

**Continuous Integration (CI)** is the practice of frequently merging developer code changes into a shared repository, with each merge triggering an automated build and test sequence.

Core principle: **integrate early and often** to detect problems early, before they compound.

Origin: Coined by Grady Booch, popularized by Martin Fowler and Kent Beck in Extreme Programming.

---

## CI Workflow

```
Developer pushes code
        ↓
CI system detects change (webhook)
        ↓
Clone repository at that commit
        ↓
Install dependencies
        ↓
Run linting / static analysis
        ↓
Compile / build
        ↓
Run unit tests
        ↓
Run integration tests
        ↓
Build container image
        ↓
Scan image for vulnerabilities
        ↓
Push image to registry (if all passed)
        ↓
Notify developer of result (pass/fail)
```

If any step fails, the pipeline stops and notifies the developer. The code is not promoted.

---

## CI Practices

### Commit Frequently

The longer between commits, the larger the merge conflict and the harder it is to isolate failures. Commit and push at least daily, ideally multiple times per day.

### Keep the Build Fast

A CI pipeline that takes 45 minutes provides slow feedback. Aim for:
- Unit tests: under 5 minutes
- Full pipeline including integration tests: under 15-20 minutes

Strategies to keep it fast:
- Run tests in parallel
- Cache dependencies
- Only run expensive tests when relevant files change

### Never Commit Broken Code

The team's CI pipeline is shared. A broken build blocks everyone. Practices:
- Run tests locally before pushing
- Use pre-commit hooks for fast checks (linting, formatting)
- Fix broken builds immediately — this is the highest priority

### Trunk-Based Development

In trunk-based development, all developers commit directly to the main branch (or very short-lived feature branches). This minimizes merge conflicts and keeps everyone in sync.

Contrast with **GitFlow** (long-lived feature branches), which can lead to "integration hell" when merging.

### Build Once, Deploy Many

Build the container image once in CI. Promote the same immutable image through environments. Never rebuild an image when deploying to a new environment — differences in build environment, dependencies, or base images can introduce bugs.

---

## CI Pipeline Components

### Source Triggers

CI systems watch for events:
- Push to a branch
- Pull request creation or update
- Tag creation
- Schedule (nightly builds)
- Manual trigger

### Stages and Steps

A pipeline is composed of **stages** (logical groups) containing **steps** (individual commands).

```yaml
# GitHub Actions example
name: CI Pipeline
on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt
      - run: pytest --cov=src tests/

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t my-app:${{ github.sha }} .
      - name: Push to registry
        run: docker push my-app:${{ github.sha }}
```

### Artifacts

CI pipelines produce **artifacts**:
- Container images (pushed to registry)
- Binary files (stored in artifact repository)
- Test reports
- Coverage reports
- Security scan results

Artifacts are identified by **version** — typically the Git commit SHA. This ensures traceability: you can always trace a running container back to the exact commit that produced it.

---

## CI Tools

| Tool | Model | Notes |
|---|---|---|
| **GitHub Actions** | Cloud, YAML | Tightly integrated with GitHub; marketplace of actions |
| **GitLab CI** | Cloud/self-hosted, YAML | Built into GitLab; powerful with merge requests |
| **Jenkins** | Self-hosted, Groovy/YAML | Oldest, most flexible; high operational overhead |
| **Tekton** | Kubernetes-native, YAML | Cloud-native; runs pipelines as Kubernetes pods |
| **CircleCI** | Cloud, YAML | Popular SaaS CI |
| **Argo Workflows** | Kubernetes-native | Workflow engine; often paired with Argo CD |
| **Drone** | Self-hosted/cloud | Container-first CI |

### Tekton (Cloud-Native CI)

Tekton is particularly relevant to platform engineering because it is Kubernetes-native. Pipelines run as Kubernetes pods.

Key Tekton concepts:
- **Task**: A series of steps (each step is a container)
- **Pipeline**: A series of tasks
- **TaskRun / PipelineRun**: An instance of a Task/Pipeline running
- **Trigger**: An event that starts a PipelineRun

```yaml
# Tekton Task example
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: run-tests
spec:
  steps:
    - name: test
      image: python:3.11
      command: ["pytest"]
      args: ["tests/"]
```

---

## Code Quality Gates

CI pipelines enforce quality gates — checks that must pass before code is merged:

| Gate | Tool Examples |
|---|---|
| Linting | ESLint, flake8, golangci-lint |
| Formatting | Prettier, gofmt, black |
| Unit tests | pytest, Jest, JUnit |
| Test coverage | Coverage.py, Istanbul |
| Static analysis (SAST) | SonarQube, CodeClimate |
| Dependency scanning | Snyk, Dependabot |
| Container image scanning | Trivy, Clair, Snyk |
| License compliance | FOSSA, license-checker |

---

## Branch Protection and Pull Requests

Platform teams configure **branch protection rules** on the main branch:
- Required status checks (CI must pass)
- Required approvals (code review)
- No force pushes
- Signed commits (optional)

This ensures no code reaches main without passing CI.

---

## Key Takeaways

- CI = frequent merges + automated build/test on every commit
- Fast pipelines = fast feedback; keep CI under 15-20 minutes
- Build the container image once in CI; promote the same image through environments
- Trunk-based development minimizes merge conflicts and integration problems
- Quality gates (linting, tests, scans) enforce standards automatically
- Tekton is the Kubernetes-native CI tool most relevant to platform engineering
