# Domain 2: Platform Observability, Security, and Conformance

**Exam Weight: 20%**

## Topics

| # | Topic | File |
|---|---|---|
| 1 | Observability Fundamentals (Traces, Metrics, Logs, Events) | [01-observability-fundamentals.md](./01-observability-fundamentals.md) |
| 2 | Secure Service Communication | [02-secure-service-communication.md](./02-secure-service-communication.md) |
| 3 | Policy Engines for Platform Governance | [03-policy-engines.md](./03-policy-engines.md) |
| 4 | Kubernetes Security Essentials | [04-kubernetes-security-essentials.md](./04-kubernetes-security-essentials.md) |
| 5 | Security in CI/CD Pipelines | [05-security-in-cicd.md](./05-security-in-cicd.md) |

## Key Concepts to Know

- The three (or four) pillars of observability: metrics, logs, traces, events
- OpenTelemetry as the vendor-neutral observability standard
- Prometheus metrics model (counters, gauges, histograms, summaries)
- mTLS and how it secures service-to-service communication
- What a service mesh does and why (Istio, Linkerd)
- Policy-as-code: OPA/Gatekeeper vs. Kyverno
- Kubernetes RBAC model (Roles, ClusterRoles, Bindings)
- Pod security: SecurityContext, PodSecurity admission
- Supply chain security: image signing, SBOM, admission webhooks
- Security scanning in CI: SAST, DAST, SCA, container scanning
