# AI/ML in Platform Automation

## Overview

Artificial intelligence and machine learning are increasingly embedded in platform engineering workflows. AI assists developers with code generation, accelerates incident response, improves CI/CD pipelines, and enables conversational interfaces to platform capabilities. Platform teams also play a critical supporting role for ML workloads — providing GPU scheduling, model serving infrastructure, and MLOps tooling.

---

## AI-Assisted Development (Copilot-Style Tools)

AI coding assistants integrate directly into developer IDEs to suggest code completions, generate entire functions, write tests, and explain code. They represent one of the highest-ROI AI investments for developer productivity.

### How AI Coding Assistants Work

AI coding assistants use Large Language Models (LLMs) trained on large code corpora. They operate on:
- **Context window** — the current file, open tabs, and project structure
- **RAG (Retrieval-Augmented Generation)** — optionally augmented with codebase-specific embeddings
- **Fine-tuning** — some enterprise variants are fine-tuned on internal codebases

### Common AI Coding Tools

| Tool | Provider | Key Feature |
|------|----------|-------------|
| **GitHub Copilot** | GitHub/Microsoft | Broad IDE support, chat, PR review |
| **Amazon CodeWhisperer** | AWS | AWS SDK awareness, security scanning |
| **Tabnine** | Tabnine | On-premises deployment option, privacy focus |
| **Codeium** | Codeium | Free tier, fast completions |
| **JetBrains AI** | JetBrains | Deep JetBrains IDE integration |
| **Sourcegraph Cody** | Sourcegraph | Entire-codebase context via code graph |

### Platform Team Considerations for AI Coding Tools

- **License compliance** — AI tools trained on open-source code may surface GPL/LGPL code; platform teams should establish policies
- **Secret detection** — AI tools must not suggest or accept secrets/credentials in code
- **Enterprise deployment** — some tools offer VPC-hosted or on-premises versions to prevent code leaving the organization
- **Prompt injection in IDE** — security consideration when AI reads repo files that may contain adversarial instructions

### Developer Experience Impact

```
Before AI assistant:
  Write function signature → search docs → write boilerplate → run tests → iterate
  Time: ~45 minutes for a standard CRUD handler

After AI assistant:
  Write function signature → accept suggestion → adjust → run tests
  Time: ~10 minutes for a standard CRUD handler
```

---

## AI for Incident Response and AIOps

### AIOps Definition

AIOps (Artificial Intelligence for IT Operations) applies machine learning to operational data — logs, metrics, traces, events — to automate tasks that traditionally required human expertise.

### Key AIOps Capabilities

| Capability | Description | Tools |
|-----------|-------------|-------|
| **Anomaly Detection** | Detect deviations from baseline behavior without fixed thresholds | Prometheus + ML (Evidently AI), Datadog, Dynatrace |
| **Alert Correlation** | Group related alerts into a single incident, reducing alert fatigue | PagerDuty AIOps, Moogsoft, BigPanda |
| **Root Cause Analysis** | Automatically identify the probable root cause of an incident | Datadog Watchdog, Dynatrace Davis |
| **Predictive Scaling** | Forecast load and scale infrastructure proactively | KEDA + custom metrics, AWS Predictive Scaling |
| **Log Intelligence** | Cluster and classify log patterns; surface anomalies | Elastic ML, Splunk ITSI, Coralogix |

### AI-Assisted Incident Response Workflow

```
1. Alert fires (Prometheus → Alertmanager → PagerDuty)
2. AIOps layer correlates with related alerts (reduces 20 alerts → 1 incident)
3. AI suggests probable root cause based on recent changes + log patterns
4. Runbook automation (PagerDuty Automation Actions, Opsgenie) executes initial response steps
5. On-call engineer joins incident with AI-generated summary:
   - Timeline of events
   - Services affected and their dependency chain
   - Recent deployments that may be related
   - Suggested remediation steps
6. Post-incident: AI generates draft post-mortem from incident timeline
```

### Example: Automated Incident Summary (LLM-powered)

```
INCIDENT SUMMARY — Generated at 14:32 UTC

Affected Service: payments-api (production)
Impact: ~12% of payment requests returning 503
Duration: 18 minutes

Root Cause Analysis:
  - High probability (87%): Database connection pool exhaustion on payments-db
  - Correlated event: Deployment of payments-api v2.4.1 at 14:08 UTC
    (increased query complexity in new refund reconciliation feature)
  - Supporting evidence: payments_db_pool_waiting_connections rose from 2 → 847
    beginning at 14:09 UTC

Actions Taken:
  - Auto-scaled payments-api replicas from 3 → 8 (14:22 UTC) — no effect
  - Manual rollback initiated by on-call at 14:28 UTC

Suggested Remediation:
  1. Complete rollback to v2.4.0
  2. Review query in refund reconciliation service (payments/reconcile.go:234)
  3. Add database connection pool monitoring alert
```

---

## AI in CI/CD Pipelines

### Test Generation

AI can automatically generate unit tests for new code, improving test coverage without additional developer effort.

```yaml
# GitHub Actions workflow with AI test generation
name: AI Test Generation
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  generate-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Generate Tests for Changed Files
        uses: codegen-ai/generate-tests-action@v1
        with:
          changed-files: ${{ steps.changed-files.outputs.all }}
          model: gpt-4o
          framework: go-testing
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
      - name: Open PR with Generated Tests
        uses: peter-evans/create-pull-request@v6
```

### Anomaly Detection in Deployments

AI monitors deployment metrics and automatically rolls back if anomalies are detected:

```yaml
# Argo Rollouts analysis with ML-based anomaly detection
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: ml-anomaly-detection
spec:
  metrics:
    - name: error-rate-anomaly
      provider:
        job:
          spec:
            template:
              spec:
                containers:
                  - name: anomaly-detector
                    image: myorg/ml-anomaly-detector:latest
                    env:
                      - name: SERVICE_NAME
                        value: "{{ args.service-name }}"
                      - name: BASELINE_WINDOW
                        value: "7d"
                      - name: SENSITIVITY
                        value: "medium"
      successCondition: result[0] == "normal"
      failureCondition: result[0] == "anomaly"
```

### AI-Powered PR Review

AI assists human code reviewers by flagging:
- Security vulnerabilities (e.g., SQL injection, hardcoded secrets)
- Performance regressions (based on historical profiling data)
- Missing error handling
- Test coverage gaps
- Violations of internal coding standards

```yaml
# .github/workflows/ai-code-review.yaml
name: AI Code Review
on: [pull_request]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: AI Security Review
        uses: snyk/actions/scan@master
      - name: AI Code Quality Review
        uses: github/copilot-code-review-action@v1
        with:
          model: copilot-gpt-4o
```

---

## LLM-Powered Platform Chatbots

Platform chatbots expose platform capabilities through a conversational natural language interface, reducing the need to remember CLI syntax or navigate complex UIs.

### Common Chatbot Capabilities

```
Developer: "What is the current error rate for payments-api in production?"
Bot: "The error rate for payments-api in production is 0.12% over the last 15 minutes.
     This is within normal range (p95 baseline: 0.15%)."

Developer: "Create a dev environment for the feature/new-checkout branch"
Bot: "Creating environment payments-dev-new-checkout...
     Provisioning database... done
     Deploying payments-api from feature/new-checkout... done
     Environment URL: https://payments-dev-new-checkout.preview.myorg.io
     TTL: 7 days (auto-destroys on 2025-05-20)"

Developer: "Who is on call for the auth service right now?"
Bot: "The on-call engineer for auth-service is @jsmith.
     Backup: @alopez | Escalation: @platform-oncall-manager
     Active incidents: None"
```

### Architecture of an LLM-Powered Platform Bot

```
┌──────────────┐     ┌─────────────────────────────────────┐
│  Slack /     │────▶│         LLM Orchestration Layer      │
│  Teams /     │     │  (LangChain / LlamaIndex / custom)   │
│  Portal UI   │     └──────────────┬──────────────────────┘
└──────────────┘                    │
                          ┌─────────┴──────────┐
                          ▼                    ▼
                   ┌─────────────┐    ┌──────────────────┐
                   │  LLM (GPT-4 │    │   Tool Calls     │
                   │  Claude,    │    │ - Catalog API    │
                   │  Llama3)    │    │ - Metrics API    │
                   └─────────────┘    │ - PagerDuty API  │
                                      │ - Argo CD API    │
                                      │ - Kubernetes API │
                                      └──────────────────┘
```

### Implementation Considerations

- **Authentication** — bot must authenticate with platform APIs using service account credentials; users' permissions must be enforced (bot should not escalate privileges)
- **Audit logging** — all bot actions must be logged with the requesting user's identity
- **Confirmation for destructive actions** — bot should ask for explicit confirmation before deleting environments or triggering rollbacks
- **Hallucination mitigation** — ground responses in real API data; do not let the LLM invent infrastructure state
- **Rate limiting** — prevent LLM abuse and runaway API calls

---

## Responsible AI in Platform Engineering

### Key Principles

| Principle | Platform Implication |
|-----------|---------------------|
| **Transparency** | Developers should know when AI is involved in decisions affecting their code or infrastructure |
| **Human oversight** | AI recommendations should require human confirmation for high-impact actions (deployments, rollbacks, deletions) |
| **Fairness** | AI tools used in code review should not introduce bias against specific coding styles or languages |
| **Privacy** | Code sent to external AI APIs may be a data protection concern; prefer on-premises or VPC-hosted models for sensitive workloads |
| **Accountability** | AI-generated code or actions must have a clear human owner responsible for the outcome |
| **Security** | AI tools must not exfiltrate secrets, internal APIs, or proprietary algorithms |

### Data Classification for AI Tool Usage

```
CONFIDENTIAL / PROPRIETARY CODE
  → Use only on-premises or VPC-hosted AI models
  → Contractual data processing agreements required

INTERNAL / SENSITIVE CODE  
  → Enterprise AI tools with DPA (GitHub Copilot Enterprise, etc.)
  → Code snippets not retained for training

PUBLIC / OPEN SOURCE CODE
  → Any AI coding assistant acceptable
```

---

## MLOps and Platform Engineering for ML Workloads

Platform engineering teams increasingly support data science and ML teams who need specialized infrastructure for training, serving, and monitoring models.

### What MLOps Is

MLOps (Machine Learning Operations) applies DevOps principles to ML workflows:
- Versioned datasets and models (like versioned code)
- Automated model training pipelines (like CI/CD)
- Reproducible experiments
- Model monitoring in production (data drift, model drift)
- Automated retraining triggers

### Platform Team Responsibilities for ML Workloads

| Responsibility | Tools |
|---------------|-------|
| GPU node provisioning | NVIDIA GPU Operator, AWS EC2 P/G instances, GKE GPU node pools |
| GPU scheduling | Kubernetes device plugins, NVIDIA MIG, Time-slicing |
| Distributed training | Kubeflow Training Operator, Ray on Kubernetes |
| Model serving | KServe, Seldon Core, BentoML, NVIDIA Triton |
| Experiment tracking | MLflow, Weights & Biases, Neptune |
| Feature stores | Feast, Tecton, Hopsworks |
| Model registry | MLflow Model Registry, Kubeflow Model Registry |
| Data pipeline orchestration | Apache Airflow, Prefect, Dagster |
| Model monitoring | Evidently AI, Arize Phoenix, Fiddler |

### GPU Scheduling on Kubernetes

```yaml
# Pod spec requesting GPU resources
apiVersion: v1
kind: Pod
metadata:
  name: model-training-job
spec:
  containers:
    - name: trainer
      image: pytorch/pytorch:2.1.0-cuda12.1-cudnn8-runtime
      resources:
        limits:
          nvidia.com/gpu: "2"          # Request 2 GPUs
          memory: "32Gi"
          cpu: "8"
        requests:
          nvidia.com/gpu: "2"
          memory: "32Gi"
          cpu: "8"
      env:
        - name: NCCL_DEBUG
          value: INFO
  nodeSelector:
    cloud.google.com/gke-accelerator: nvidia-tesla-a100
  tolerations:
    - key: nvidia.com/gpu
      operator: Exists
      effect: NoSchedule
```

### KServe Model Serving

KServe (formerly KFServing) is a CNCF Incubating project for serving ML models on Kubernetes.

```yaml
# KServe InferenceService for a TensorFlow model
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: fraud-detection-model
  namespace: ml-serving
spec:
  predictor:
    tensorflow:
      storageUri: s3://my-org-models/fraud-detection/v3
      resources:
        requests:
          cpu: "2"
          memory: "4Gi"
        limits:
          nvidia.com/gpu: "1"
  transformer:
    containers:
      - name: feature-transformer
        image: myorg/fraud-feature-transformer:latest
```

### Feature Stores

A feature store is a centralized repository for ML features — pre-computed values derived from raw data that are used for both model training and serving.

```python
# Feast feature store definition
from feast import Entity, FeatureView, Field, FileSource
from feast.types import Float32, Int64

customer = Entity(name="customer_id", join_keys=["customer_id"])

transaction_features = FeatureView(
    name="transaction_features",
    entities=[customer],
    ttl=timedelta(days=1),
    schema=[
        Field(name="transaction_count_7d", dtype=Int64),
        Field(name="avg_transaction_amount_30d", dtype=Float32),
        Field(name="fraud_risk_score", dtype=Float32),
    ],
    source=FileSource(
        path="s3://my-org-features/transactions.parquet",
        timestamp_field="event_timestamp",
    ),
)
```

### MLflow for Experiment Tracking and Model Registry

```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import GradientBoostingClassifier

mlflow.set_tracking_uri("https://mlflow.myorg.io")
mlflow.set_experiment("fraud-detection-v3")

with mlflow.start_run():
    model = GradientBoostingClassifier(n_estimators=100, max_depth=5)
    model.fit(X_train, y_train)
    
    # Log parameters
    mlflow.log_param("n_estimators", 100)
    mlflow.log_param("max_depth", 5)
    
    # Log metrics
    mlflow.log_metric("accuracy", accuracy)
    mlflow.log_metric("f1_score", f1)
    mlflow.log_metric("auc_roc", auc)
    
    # Register model
    mlflow.sklearn.log_model(
        model,
        "fraud-detection",
        registered_model_name="fraud-detection"
    )
```

---

## CNCF ML/AI Projects

| Project | CNCF Maturity | Role |
|---------|---------------|------|
| **KServe** | Incubating | Model serving |
| **Kubeflow** | Not CNCF (CNCF adjacent) | ML platform on Kubernetes |
| **Argo Workflows** | CNCF Incubating | ML pipeline orchestration |
| **Flyte** | CNCF Incubating | Data and ML workflow platform |
| **Ray** | Not CNCF | Distributed compute for ML |
| **Feast** | Not CNCF | Feature store |

---

## Exam Tips

- **AIOps** = applying ML to IT operations data (metrics, logs, events, alerts)
- **Alert correlation** is a key AIOps feature — grouping related alerts into one incident
- **KServe** is the CNCF Incubating project for model serving
- **Feature stores** solve the training-serving skew problem (same features in training and inference)
- **GPU scheduling** in Kubernetes uses device plugins and resource limits (`nvidia.com/gpu`)
- AI tools in platform context must respect **data classification policies** — sensitive code should not go to external APIs
- **LLM chatbots** must enforce user permissions — the bot should not escalate user privileges
- **MLflow** is the dominant open-source tool for experiment tracking and model registry
- Responsible AI requires **human-in-the-loop** for high-impact automated actions
- Know the **difference between MLOps (ML-specific) and DevOps (general software delivery)**
