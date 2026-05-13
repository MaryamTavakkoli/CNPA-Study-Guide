# The CI/CD Relationship & Fundamentals

## CI and CD Are Not the Same Thing

Continuous Integration (CI) and Continuous Delivery/Deployment (CD) are frequently written together as "CI/CD" as if they were one thing. In practice, they are two distinct disciplines with different goals, different tooling, and different owners.

| | Continuous Integration (CI) | Continuous Delivery (CD) |
|--|----------------------------|-------------------------|
| **Goal** | Verify that code changes are correct and safe to build | Get verified artifacts into production reliably and quickly |
| **Input** | Source code commit | Verified, immutable artifact |
| **Output** | Artifact (image, binary, package) | Deployed running software |
| **Trigger** | Every commit / every PR | Successful CI build (or manual approval) |
| **Failure** | Breaks the build | Blocks promotion to next environment |
| **Primary tool** | GitHub Actions, GitLab CI, Tekton, Jenkins | ArgoCD, Flux, Spinnaker, GitHub Actions CD jobs |

Understanding this distinction is critical for platform engineers, who often need to design systems that bridge the two.

---

## How CI Feeds CD: The Artifact Handoff

The handoff point between CI and CD is the **published artifact**. CI's final job is to produce and publish a verifiable artifact. CD's first action is to consume that artifact.

```
  ┌──────────────────────────────────────────────────────────┐
  │                       CI PIPELINE                        │
  │                                                          │
  │  checkout → lint → test → build image → scan → sign     │
  │                                             │            │
  │                                             ▼            │
  │                              push to registry            │
  │                    image: myapp:sha-a1b2c3d4             │
  └──────────────────────────────────────────────────────────┘
                                    │
                                    │ image digest / tag
                                    ▼
  ┌──────────────────────────────────────────────────────────┐
  │                       CD PIPELINE                        │
  │                                                          │
  │  dev deploy → dev tests → staging deploy → staging tests │
  │       → [approval gate] → prod deploy → smoke tests      │
  └──────────────────────────────────────────────────────────┘
```

The artifact (container image, binary, Helm chart) is built **once** and promoted through environments. It is never rebuilt between environments because rebuilding introduces risk: the build may produce a different output due to updated dependencies, non-deterministic compilation, or changed build tooling.

---

## Artifact Promotion

**Artifact promotion** is the act of advancing the same artifact through a sequence of environments, where each environment provides additional confidence before production.

### Why Promotion, Not Rebuild?

- A rebuild from the same source code could produce a different binary if any build-time dependency changed
- Promotion ensures exactly what was tested in staging is exactly what runs in production
- Promotion is auditable: you can trace any running image back to the specific CI build and Git commit that produced it

### Immutable Tags vs. Mutable Tags

```
ANTI-PATTERN:
  dev:    myapp:latest    ← rebuilt every commit, "latest" means nothing
  staging: myapp:latest   ← same tag, different content than dev?
  prod:   myapp:latest    ← completely opaque; what version is this?

CORRECT PATTERN:
  dev:    myapp:sha-a1b2c3d4   ← points to exact Git commit
  staging: myapp:sha-a1b2c3d4  ← same image, same digest
  prod:   myapp:sha-a1b2c3d4   ← same image, provably identical to what was tested
```

Container images should be referenced by **digest** (e.g., `myapp@sha256:abc123...`) for maximum immutability, or by a tag that includes the Git SHA or a semver version.

---

## Environment Gates

An **environment gate** is a check that must pass before an artifact can be promoted to the next environment. Gates can be automated or manual.

### Automated Gates

- All tests in the current environment pass
- Security scan shows no CRITICAL vulnerabilities
- Performance benchmark stays within defined thresholds
- SLO compliance is met for a minimum soak period (e.g., "must be healthy in staging for 1 hour")
- SBOM is present and signed
- Image digest in deployment matches the expected digest from CI

### Manual Approval Gates

Manual gates introduce a human decision point. Common placement:

- Before production deployment (especially in regulated industries)
- Before promoting to a customer-facing environment
- After a large architectural change

### Environment Gate Example (GitHub Actions)

```yaml
# GitHub Actions: use 'environment' with required reviewers for manual approval
jobs:
  deploy-staging:
    environment: staging
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to staging
        run: helm upgrade --install myapp ./chart --set image.tag=${{ env.IMAGE_TAG }}

  integration-test-staging:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - name: Run E2E tests against staging
        run: pytest tests/e2e/ --base-url=https://staging.example.com

  deploy-production:
    needs: integration-test-staging
    environment: production    # <- requires manual approval from configured reviewers
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to production
        run: helm upgrade --install myapp ./chart --set image.tag=${{ env.IMAGE_TAG }}
```

In this example, the `production` environment in GitHub requires approval from designated reviewers before the deploy step runs.

---

## Deployment Pipelines vs. Build Pipelines

These terms are often used interchangeably, but they refer to different scopes:

| | Build Pipeline | Deployment Pipeline |
|--|---------------|---------------------|
| **Scope** | Source code → artifact | Artifact → running service |
| **Runs** | On every commit | On artifact promotion trigger |
| **Outputs** | Container image, SBOM, test report | Running pods, updated Kubernetes resources |
| **Failure effect** | PR blocked from merging | Rollout stops; previous version stays running |
| **Tooling** | GitHub Actions, Tekton, Jenkins | ArgoCD, Flux, Spinnaker, Helm |
| **Auditability** | Git commit + CI run logs | Deployment history, GitOps audit trail |

A complete CI/CD system chains these together: build pipeline produces artifact → deployment pipeline promotes artifact.

---

## Trunk-Based Development

**Trunk-based development (TBD)** is a source control practice where all developers commit to a single shared branch (the "trunk" or `main`) frequently — at least once per day. Feature branches, if used at all, are very short-lived (hours to a day or two) and are never allowed to drift far from main.

### Why TBD Enables CI/CD

- With all code on one branch, there is one definitive version of the software at all times
- Long-lived feature branches create "merge hell" — large diffs that are expensive to review and risky to integrate
- Frequent integration to main means CI runs on nearly-production-ready code, not code that has been isolated for weeks
- It forces developers to keep changes small and incremental — which is safer to deploy

### TBD vs. GitFlow

| Aspect | Trunk-Based Development | GitFlow |
|--------|------------------------|---------|
| Primary branch | `main` (always deployable) | `develop` (not deployable) |
| Feature branches | Short-lived (< 1 day ideally) | Long-lived (days to weeks) |
| Release cadence | Continuous | Batched releases |
| Merge frequency | Many times per day | Once per feature |
| Integration risk | Low (small diffs) | High (large diffs) |
| CI effectiveness | High | Reduced (merge conflicts mask CI) |
| Suitable for | Continuous delivery teams | Coordinated release teams |

### Branch Policies That Support TBD

```yaml
# GitHub branch protection rule (conceptual)
branch: main
rules:
  - require_pull_request_reviews: 1
  - require_status_checks:
      - ci/build
      - ci/test
      - ci/security-scan
  - require_up_to_date_before_merge: true
  - delete_branch_on_merge: true          # enforce short-lived branches
  - restrict_pushes_to_matching_branches: true   # no direct commits to main
```

---

## Feature Flags as a Deployment Decoupler

**Feature flags** (also called feature toggles or feature switches) are boolean conditions in application code that enable or disable functionality at runtime, without a new deployment.

### Why Feature Flags Matter for CI/CD

The traditional problem: you can't deploy code that isn't finished. Feature flags solve this by allowing *incomplete* code to be deployed to production while being hidden from users.

```
Without feature flags:
  commit → branch → [weeks pass] → big merge → big deploy → big risk

With feature flags:
  commit → deploy (flag OFF) → [days pass] → turn flag ON → flag cleanup
  (code ships continuously; release is decoupled from deployment)
```

### Feature Flag Use Cases

| Use Case | Description |
|----------|------------|
| **Dark launch** | Code runs in production but output is discarded; validates performance without user impact |
| **Canary release** | Flag enabled for 1% of users; gradually increased as confidence grows |
| **Kill switch** | Disable a misbehaving feature instantly without a rollback deployment |
| **A/B testing** | Show different versions of a feature to different user groups |
| **Operational control** | Disable expensive features during high load events |
| **Beta access** | Enable features for specific users, teams, or accounts |

### Feature Flag Example (LaunchDarkly SDK style)

```python
import ldclient

ldclient.set_config(ldclient.Config("sdk-key"))
client = ldclient.get()

user = {"key": user_id, "email": user_email}

if client.variation("new-checkout-flow", user, False):
    return new_checkout_flow(cart)
else:
    return legacy_checkout_flow(cart)
```

### Feature Flag Platforms

| Platform | Notes |
|---------|-------|
| **LaunchDarkly** | Full-featured commercial platform; SDK for every language |
| **Unleash** | Open-source; self-hosted option |
| **Flagsmith** | Open-source; cloud and self-hosted |
| **OpenFeature** | CNCF standard API for feature flags; vendor-neutral SDK |
| **Homegrown** | Simple boolean flags in environment variables or config maps (limited) |

### The OpenFeature Standard

OpenFeature is a CNCF project that defines a vendor-neutral API for feature flags. Platform teams can provide the OpenFeature SDK as a platform capability, allowing application teams to use any compliant backend (LaunchDarkly, Unleash, etc.) without changing their application code.

### Feature Flag Technical Debt

Feature flags are code. Stale flags that are never cleaned up become:
- Maintenance burden (dead code paths)
- Security risk (shadow code paths that are rarely tested)
- Confusion for new engineers

Best practice: every flag has a planned removal date and a Jira/GitHub issue for cleanup.

---

## The Full CI/CD Flow in Practice

Putting it all together, here is what a modern CI/CD flow looks like for a platform engineering team deploying a microservice:

```
1. Developer opens PR on short-lived branch
   │
   ▼
2. CI Pipeline fires on PR event
   - Lint, unit tests, SAST scan
   - All checks must pass for PR to be mergeable
   │
   ▼
3. PR merged to main (trunk-based development)
   │
   ▼
4. CI Pipeline fires on push to main
   - Full build, all tests, integration tests
   - Container image built: myapp:sha-a1b2c3d4
   - Image scanned (Trivy, Grype), signed (Cosign)
   - SBOM generated and attached
   - Image pushed to registry
   │
   ▼
5. CD is triggered (push or pull model)
   - GitOps: CI updates image tag in Git → ArgoCD detects drift → syncs to dev
   - Or: CD pipeline job runs helm upgrade
   │
   ▼
6. Dev environment: smoke tests run automatically
   │
   ▼
7. Automated gate: all tests pass, image digest verified
   │
   ▼
8. Promotion to staging (automated or manual trigger)
   - Same image digest: myapp:sha-a1b2c3d4
   - Staging-specific values overlaid (resource limits, replicas, config)
   │
   ▼
9. Staging: full E2E test suite, performance tests
   - Feature flags may differ from dev (e.g., new-checkout-flow: OFF in staging)
   │
   ▼
10. Manual approval gate (production)
    - Reviewer confirms test results, reviews change summary
    │
    ▼
11. Production deployment
    - Same image: myapp:sha-a1b2c3d4 (promoted, not rebuilt)
    - Progressive delivery: canary 5% → 25% → 100% (via Argo Rollouts)
    - Feature flag for new feature still OFF — to be enabled separately
    │
    ▼
12. Post-deployment: smoke tests, SLO monitoring
    - If error rate spikes: auto-rollback (Argo Rollouts) or manual rollback
```

---

## Key Exam Takeaways

- CI produces an artifact; CD promotes that artifact — they are sequential but distinct
- **Never rebuild an artifact** per environment; always promote the same immutable artifact
- Environment gates enforce quality and safety; they can be automated or require human approval
- **Trunk-based development** is the source control strategy that enables true CI; long-lived branches defeat CI's purpose
- **Feature flags** decouple deployment from release — code can ship to production behind a flag before the feature is "done"
- **OpenFeature** is the CNCF standard for vendor-neutral feature flag SDKs — important for platform teams
- A deployment pipeline extends the build pipeline; both together form the complete CI/CD system
