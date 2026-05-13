# CNPA Study Guide

## Certified Cloud Native Platform Engineering Associate (CNPA)

> Linux Foundation Certification — [Exam Page](https://training.linuxfoundation.org/certification/certified-cloud-native-platform-engineering-associate-cnpa/)

This repository contains structured study material covering every domain of the CNPA exam. Reading through each section in order will build the knowledge needed to pass the exam.

---

## Exam Overview

| Property | Detail |
|---|---|
| Format | Online, proctored, multiple-choice |
| Duration | 120 minutes |
| Level | Beginner/Associate |
| Prerequisites | None |

---

## Domains & Weights

| # | Domain | Weight |
|---|---|---|
| 1 | [Platform Engineering Core Fundamentals](./01-platform-engineering-core-fundamentals/README.md) | 36% |
| 2 | [Platform Observability, Security, and Conformance](./02-platform-observability-security-conformance/README.md) | 20% |
| 3 | [Continuous Delivery & Platform Engineering](./03-continuous-delivery-platform-engineering/README.md) | 16% |
| 4 | [Platform APIs and Provisioning Infrastructure](./04-platform-apis-provisioning/README.md) | 12% |
| 5 | [IDPs and Developer Experience](./05-idps-developer-experience/README.md) | 8% |
| 6 | [Measuring Your Platform](./06-measuring-your-platform/README.md) | 8% |

---

## How to Use This Guide

1. Work through domains **in order** — earlier domains provide foundation for later ones.
2. Each domain folder has a `README.md` overview and individual topic files.
3. After each domain, attempt the practice questions in [`practice-questions/`](./practice-questions/).
4. Use the [Glossary](./resources/glossary.md) when you encounter unfamiliar terms.
5. Finish with the [Full Practice Exam](./practice-questions/full-practice-exam.md).

---

## Repository Structure

```
cnpa-study-guide/
├── 01-platform-engineering-core-fundamentals/   # 36% of exam
│   ├── 01-declarative-resource-management.md
│   ├── 02-devops-practices.md
│   ├── 03-application-environments.md
│   ├── 04-platform-architecture.md
│   ├── 05-platform-goals-and-objectives.md
│   ├── 06-continuous-integration-fundamentals.md
│   └── 07-continuous-delivery-and-gitops.md
├── 02-platform-observability-security-conformance/  # 20%
│   ├── 01-observability-fundamentals.md
│   ├── 02-secure-service-communication.md
│   ├── 03-policy-engines.md
│   ├── 04-kubernetes-security-essentials.md
│   └── 05-security-in-cicd.md
├── 03-continuous-delivery-platform-engineering/  # 16%
│   ├── 01-ci-pipelines-overview.md
│   ├── 02-incident-response.md
│   ├── 03-cicd-relationship-fundamentals.md
│   ├── 04-gitops-basics.md
│   └── 05-gitops-application-environments.md
├── 04-platform-apis-provisioning/               # 12%
│   ├── 01-kubernetes-reconciliation-loop.md
│   ├── 02-apis-self-service-crds.md
│   ├── 03-infrastructure-provisioning.md
│   └── 04-kubernetes-operator-pattern.md
├── 05-idps-developer-experience/               # 8%
│   ├── 01-simplified-access.md
│   ├── 02-api-driven-service-catalogs.md
│   ├── 03-developer-portals.md
│   └── 04-ai-ml-platform-automation.md
├── 06-measuring-your-platform/                 # 8%
│   ├── 01-platform-efficiency-productivity.md
│   └── 02-dora-metrics.md
├── practice-questions/
│   ├── domain-01-questions.md
│   ├── domain-02-questions.md
│   ├── domain-03-questions.md
│   ├── domain-04-questions.md
│   ├── domain-05-questions.md
│   ├── domain-06-questions.md
│   └── full-practice-exam.md
└── resources/
    ├── glossary.md
    ├── tools-and-technologies.md
    └── further-reading.md
```
