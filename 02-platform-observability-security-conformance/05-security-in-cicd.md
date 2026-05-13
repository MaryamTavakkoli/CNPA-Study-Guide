# Security in CI/CD Pipelines

## Why CI/CD Security Matters

CI/CD pipelines have elevated privileges — they build, test, and deploy code. A compromised pipeline can:
- Inject malicious code into artifacts
- Steal secrets and credentials
- Deploy backdoored images to production
- Exfiltrate source code

The SolarWinds attack (2020) was a supply chain attack via a compromised CI/CD pipeline.

---

## Shift Left Security (DevSecOps)

**DevSecOps** integrates security into every phase of the development lifecycle rather than treating it as a gate at the end.

The "shift left" metaphor: move security checks earlier (left) in the pipeline so developers get feedback faster and fixing is cheaper.

```
Commit → Build → Test → Stage → Deploy → Run
  ↑        ↑       ↑       ↑
  IDE    Pre-commit  CI   Gate checks
  (SAST)  (hooks)  (scans) (policy)
```

---

## Security Scanning in CI

### SAST — Static Application Security Testing

Analyzes source code without executing it. Finds:
- Hardcoded credentials
- SQL injection, XSS vulnerabilities
- Insecure cryptography usage
- Dangerous function calls

Tools:
- **Semgrep**: Fast, customizable SAST; YAML rules; free tier
- **SonarQube / SonarCloud**: Comprehensive code quality and security
- **Bandit**: Python-specific security linter
- **gosec**: Go security scanner
- **CodeQL**: GitHub's semantic code analysis engine

```yaml
# GitHub Actions: Semgrep SAST
- name: Run Semgrep
  uses: semgrep/semgrep-action@v1
  with:
    config: p/security-audit p/owasp-top-ten
```

### DAST — Dynamic Application Security Testing

Tests the running application by simulating attacks. Finds vulnerabilities that only appear at runtime.

Tools:
- **OWASP ZAP**: Open-source web app scanner
- **Burp Suite**: Commercial; powerful for manual + automated testing

DAST typically runs in staging environments, not directly in CI.

### SCA — Software Composition Analysis

Scans third-party dependencies for known vulnerabilities.

Tools:
- **Dependabot** (GitHub): Automated PRs to update vulnerable dependencies
- **Snyk**: Comprehensive; IDE + CI integration
- **OWASP Dependency-Check**: Open-source
- **Trivy**: Also scans filesystem and lock files

```bash
# Trivy scan of filesystem dependencies
trivy fs --scanners vuln .
```

### Container Image Scanning

Scans container images for OS-level and application-level vulnerabilities.

```bash
trivy image --severity HIGH,CRITICAL my-app:latest
```

Best practices:
- Scan in CI on every PR
- Block merges if CRITICAL vulnerabilities exist
- Re-scan images in the registry regularly (new CVEs are discovered daily)
- Use minimal base images to reduce attack surface

### Secret Scanning

Prevents committing secrets to Git.

Tools:
- **GitLeaks**: Scans git history for secrets
- **detect-secrets** (Yelp): Pre-commit hook
- **Trufflehog**: Deep scan for high-entropy strings
- **GitHub Secret Scanning**: Built into GitHub; alerts on detected secrets

```bash
# Pre-commit hook with detect-secrets
detect-secrets scan --all-files > .secrets.baseline
detect-secrets audit .secrets.baseline
```

```yaml
# GitHub Actions: GitLeaks scan
- name: GitLeaks scan
  uses: gitleaks/gitleaks-action@v2
```

---

## Supply Chain Security

The software supply chain is everything that goes into producing a deployed artifact: source code, dependencies, build tools, CI pipelines, registries.

### SLSA Framework (Supply chain Levels for Software Artifacts)

SLSA (pronounced "salsa") is a framework for supply chain security, providing levels of assurance:

| Level | Description |
|---|---|
| **SLSA 1** | Provenance exists; build process is documented |
| **SLSA 2** | Provenance is signed; build is tamper-resistant |
| **SLSA 3** | Source is two-party reviewed; build is hermetic |
| **SLSA 4** | Highest: reproducible builds, two-party review |

### SBOM (Software Bill of Materials)

An SBOM is a formal list of all components (libraries, tools, OS packages) in a software artifact. Similar to a food ingredient label.

Why it matters:
- When a new CVE is discovered, you can quickly identify which of your services are affected
- Required by some regulations (US Executive Order on Improving Cybersecurity)

Tools for generating SBOMs:
- **Syft**: Generates SBOMs from container images, filesystems
- **Trivy**: Can generate SBOM in CycloneDX or SPDX format
- **Microsoft SBOM Tool**

Formats: **SPDX** (Linux Foundation), **CycloneDX** (OWASP)

```bash
# Generate SBOM with Syft
syft my-app:latest -o spdx-json > sbom.json
```

### Image Signing (Sigstore/Cosign)

Sign images in CI to create a verifiable chain of trust:

```yaml
# GitHub Actions: sign image after push
- name: Sign image with Cosign
  run: |
    cosign sign --yes \
      --key env://COSIGN_PRIVATE_KEY \
      ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}@${{ steps.push.outputs.digest }}
  env:
    COSIGN_PRIVATE_KEY: ${{ secrets.COSIGN_PRIVATE_KEY }}
```

At deploy time, Kyverno or Connaisseur verifies the signature before admitting the pod.

---

## Securing the Pipeline Itself

### Secrets in CI

Never hardcode secrets in pipeline YAML. Use the CI system's secret management:
- GitHub Actions: Secrets in repository or organization settings
- GitLab CI: CI/CD variables (masked and protected)
- Tekton: Kubernetes Secrets mounted into pipeline pods

```yaml
# GitHub Actions: use secrets from repository settings
- name: Push image
  run: docker push $IMAGE
  env:
    REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
```

### Least Privilege for Pipeline Identities

- Pipeline service accounts should only have permissions needed for their specific task
- Deploy pipeline: read/write to target namespace only
- Build pipeline: push to registry only
- Use **Workload Identity** (OIDC federation) instead of static credentials where possible

### OIDC in GitHub Actions (Keyless Auth)

Instead of storing cloud credentials as secrets, GitHub Actions can use OIDC tokens to authenticate directly with cloud providers:

```yaml
- name: Configure AWS credentials via OIDC
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789:role/github-actions-deploy
    aws-region: eu-west-1
```

No static credentials stored in GitHub. AWS validates the GitHub OIDC token.

### Dependency Pinning

Pin exact versions (including hashes) for all CI dependencies to prevent supply chain attacks:

```yaml
# Pin to specific commit SHA, not a mutable tag
uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4.2.2
```

---

## Key Takeaways

- CI/CD pipelines have privileged access; a compromised pipeline is a critical vulnerability
- Shift left: SAST in the IDE and pre-commit; SCA and image scanning in CI; DAST in staging
- SAST scans source code; SCA scans dependencies; image scanning scans container layers
- SBOM is a full ingredient list of your software; required for rapid CVE response
- Sign images with Cosign/Sigstore; verify signatures at admission with Kyverno
- Use OIDC for keyless authentication in CI — no static cloud credentials needed
- SLSA framework provides supply chain security maturity levels (1-4)
