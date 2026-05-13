# CI Pipeline Anatomy & Platform Templatization

## What Is a CI Pipeline?

A **Continuous Integration (CI) pipeline** is an automated sequence of steps that validates every change committed to a version-controlled repository. Its primary goal is to give developers fast, reliable feedback that their change does not break existing behavior and meets quality standards before it can be merged or deployed.

From a platform engineering perspective, CI pipelines are not just a developer convenience — they are the first gate in a production-readiness chain. Platform teams are responsible for:

- Providing standardized, reusable pipeline templates
- Enforcing security scanning, SBOM generation, and signing as non-optional steps
- Surfacing pipeline health as a platform metric
- Ensuring pipelines are fast enough that developers don't bypass them

---

## Pipeline as Code

**Pipeline as Code** means the pipeline definition is stored in the same Git repository as the application source. This gives you:

- **Auditability**: every pipeline change is a commit with an author, timestamp, and message
- **Reviewability**: pipeline changes go through the same pull request process as application changes
- **Reproducibility**: any checkout of the repo at any commit can reproduce the pipeline behavior of that point in time
- **Testability**: pipeline definitions can be linted and validated in a pre-merge check

Each major CI system has its own pipeline-as-code format:

| Tool | File Location | Format |
|------|--------------|--------|
| GitHub Actions | `.github/workflows/*.yml` | YAML |
| GitLab CI | `.gitlab-ci.yml` | YAML |
| Tekton | Kubernetes CRDs in-cluster or in a config repo | YAML (Kubernetes manifests) |
| Jenkins | `Jenkinsfile` | Groovy DSL or declarative YAML |
| CircleCI | `.circleci/config.yml` | YAML |

---

## Pipeline Anatomy: Stages, Jobs, and Steps

A well-structured CI pipeline is organized into **stages** that execute in sequence, with **jobs** within each stage potentially running in parallel.

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Source   │───►│  Build   │───►│   Test   │───►│ Publish  │
│  (trigger)│    │          │    │          │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
                                   │
                          ┌────────┴────────┐
                          │                 │
                     Unit Tests       Integration
                    (parallel)          Tests
                                      (parallel)
```

### Typical Stage Breakdown

| Stage | Purpose | Fail behavior |
|-------|---------|---------------|
| **Lint / Static Analysis** | Code style, formatting, type checking | Fail fast — cheap to run |
| **Build** | Compile source, build container image | Required for downstream stages |
| **Unit Test** | Fast, isolated tests with no external dependencies | Must pass; coverage threshold |
| **Integration Test** | Tests with real databases, queues, etc. | Must pass; can be slower |
| **Security Scan** | SAST, SCA (dependency CVEs), secret scanning | Configurable severity threshold |
| **Publish** | Push image to registry, publish package | Only on clean builds from main/protected branches |

---

## Artifacts

An **artifact** is an immutable output produced by a pipeline stage and consumed by subsequent stages or external systems. Treating artifacts as immutable is fundamental to reproducible builds and safe artifact promotion.

### Types of Artifacts

- **Container images**: pushed to an OCI registry (Docker Hub, ECR, GAR, Harbor)
- **Build binaries**: compiled executables, JAR files, wheel packages
- **Test reports**: JUnit XML, coverage HTML — consumed by the CI platform UI
- **SBOMs** (Software Bill of Materials): machine-readable inventory of all dependencies (CycloneDX, SPDX)
- **Attestations / signatures**: Cosign signatures proving the image was built by a trusted pipeline

### Artifact Passing in GitHub Actions

```yaml
# Producer job
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build binary
        run: go build -o ./dist/app ./cmd/app
      - uses: actions/upload-artifact@v4
        with:
          name: app-binary
          path: ./dist/app

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app-binary
      - name: Run integration tests
        run: ./app --test
```

---

## Caching

**Caching** saves time by reusing previously downloaded or computed assets — primarily dependency files — across pipeline runs. Without caching, every run downloads the same npm/Maven/Go packages from the internet, wasting minutes and creating external dependencies.

### Caching in GitHub Actions

```yaml
- name: Cache Go modules
  uses: actions/cache@v4
  with:
    path: |
      ~/.cache/go-build
      ~/go/pkg/mod
    key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
    restore-keys: |
      ${{ runner.os }}-go-
```

**Key design**: the cache key includes a hash of the lock file (`go.sum`, `package-lock.json`, `pom.xml`). When dependencies change, a new cache is created. `restore-keys` provides fallback to a partial cache hit, which is faster than a cold start.

### Caching in GitLab CI

```yaml
cache:
  key:
    files:
      - package-lock.json
  paths:
    - node_modules/
  policy: pull-push   # pull on start, push on finish
```

---

## Parallelism

Running jobs in parallel reduces total pipeline duration. Platform teams tune parallelism to balance speed against infrastructure cost.

### Fan-out / Fan-in Pattern

```yaml
# GitLab CI example: parallel test jobs that fan back in
stages:
  - test
  - report

unit-test:
  stage: test
  script: pytest tests/unit/

integration-test:
  stage: test
  script: pytest tests/integration/

lint:
  stage: test
  script: flake8 src/

coverage-report:
  stage: report       # only runs after ALL test jobs succeed
  needs:
    - unit-test
    - integration-test
  script: coverage report
```

---

## Matrix Builds

A **matrix build** runs the same job definition across a combination of parameters — operating systems, language runtime versions, database versions, etc. This ensures your software works across its full support matrix without duplicating job definitions.

### GitHub Actions Matrix

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false    # don't cancel all matrix jobs if one fails
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        go-version: ["1.21", "1.22", "1.23"]
        exclude:
          - os: windows-latest
            go-version: "1.21"   # known incompatibility
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: ${{ matrix.go-version }}
      - run: go test ./...
```

This generates **8 jobs** (3 OS × 3 versions - 1 exclusion) from a single definition.

---

## Pipeline Triggers

Triggers control *when* a pipeline fires. Platform teams configure triggers to balance feedback speed against compute cost.

| Trigger Type | When it fires | Typical use |
|-------------|--------------|-------------|
| **Push** | Every commit to any branch | Full CI on feature branches |
| **Pull Request / Merge Request** | When a PR is opened or updated | Pre-merge validation |
| **Tag** | When a git tag is pushed | Release builds |
| **Schedule** | Cron expression | Nightly full regression, dependency scanning |
| **Manual** | Triggered by a user clicking a button | Production deployments, one-off jobs |
| **Webhook** | External system event | Upstream dependency updated |
| **Path filter** | Only when specific paths changed | Monorepo: only build affected services |

### Path Filter Example (GitHub Actions)

```yaml
on:
  push:
    paths:
      - 'services/payments/**'
      - 'shared/proto/**'
    branches:
      - main
```

---

## Common CI Tools

### GitHub Actions

- Native to GitHub; no separate CI infrastructure needed
- **Reusable workflows**: callable workflows defined in `.github/workflows/` that can be invoked by other workflows — the platform team's primary templatization mechanism
- **Composite actions**: package multiple steps into a single action
- Large marketplace of pre-built actions
- Matrix builds, environment secrets, OIDC-based cloud authentication

### GitLab CI

- Deeply integrated with GitLab SCM, registry, and security scanning
- **`include` keyword**: import pipeline fragments from a central template repository
- **`extends`**: inherit and override a job definition
- **Auto DevOps**: zero-configuration pipeline for common application types
- Runners: shared (GitLab-hosted) or self-managed

### Tekton

- Kubernetes-native; pipeline resources are Kubernetes CRDs (`Task`, `Pipeline`, `PipelineRun`)
- Steps run as containers in a Pod — full Kubernetes scheduling and resource controls
- **Tekton Catalog**: community library of reusable `Task` definitions
- **Tekton Chains**: supply chain security — automatically attests and signs `TaskRun` outputs
- Platform teams deploy Tekton to the cluster and provide curated `Task` libraries

```yaml
# Tekton Pipeline example
apiVersion: tekton.dev/v1
kind: Pipeline
metadata:
  name: build-and-push
spec:
  params:
    - name: image-url
      type: string
  tasks:
    - name: clone
      taskRef:
        resolver: cluster
        params:
          - name: name
            value: git-clone
    - name: build
      runAfter: [clone]
      taskRef:
        resolver: cluster
        params:
          - name: name
            value: buildah
      params:
        - name: IMAGE
          value: $(params.image-url)
```

### Jenkins

- Long-standing, highly extensible Java-based CI server
- **Declarative Pipeline** (`Jenkinsfile`): structured, opinionated syntax
- **Scripted Pipeline**: full Groovy scripting for maximum flexibility
- **Shared Libraries**: Groovy code in a separate Git repo, `@Library` imported into `Jenkinsfile` — the primary templatization mechanism
- Large plugin ecosystem; high operational overhead (platform team must manage Jenkins controllers and agents)

```groovy
// Declarative Jenkinsfile using a Shared Library
@Library('platform-pipelines@v2.3.0') _

platformBuild {
  language = 'java'
  javaVersion = '21'
  sonarEnabled = true
  publishToRegistry = true
}
```

---

## Platform Team Templatization Strategies

One of the most impactful things a platform team can do is remove the burden of pipeline authorship from application teams. Key strategies:

### 1. Reusable Workflow Templates (GitHub Actions)

Platform team maintains a `platform-workflows` repository with callable workflows:

```yaml
# In platform-workflows/.github/workflows/java-build.yml
on:
  workflow_call:
    inputs:
      java-version:
        required: false
        default: "21"
        type: string
    secrets:
      registry-password:
        required: true

jobs:
  build-test-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.java-version }}
      - run: mvn verify
      - run: mvn package -DskipTests
      # ... image build and push steps
```

Application teams reference it with three lines:

```yaml
# In application repo .github/workflows/ci.yml
jobs:
  ci:
    uses: my-org/platform-workflows/.github/workflows/java-build.yml@v2
    secrets:
      registry-password: ${{ secrets.REGISTRY_PASSWORD }}
```

### 2. GitLab CI Component Catalog

GitLab 17+ provides a **Component Catalog** — versioned, composable pipeline fragments that teams `include` by URL and version.

### 3. Backstage Software Templates

Platform teams can scaffold new services with a pre-wired `Jenkinsfile` or `.github/workflows/ci.yml` baked in via [Backstage Software Templates](https://backstage.io/docs/features/software-templates/), so new services start with a compliant pipeline on day one.

### 4. Golden Path vs. Escape Hatch

Effective platform teams offer a **golden path** (opinionated, well-supported, low-friction template) while also providing a documented **escape hatch** for teams with unusual requirements. This avoids both sprawl (every team rolling their own) and rigidity (platform becomes a bottleneck).

---

## Key Exam Takeaways

- Pipeline as Code is non-negotiable: pipelines must be in version control
- Artifacts are **immutable**; the same artifact is promoted — never rebuilt per environment
- Caching keys should include lock file hashes to ensure correctness
- `fail-fast: false` in matrix builds prevents one flaky combination from masking failures in others
- Platform teams use **reusable workflows / shared libraries / include templates** to provide golden-path pipelines
- Tekton is the cloud-native, Kubernetes-native CI tool — important for CNPA context
- Triggers should be tuned: not every event needs to run every job
