# Observability Fundamentals

## What Is Observability?

Observability is the ability to understand the internal state of a system by examining its external outputs. A system is observable if you can answer the question "why is this broken?" without needing to redeploy code or add new instrumentation.

Observability vs. monitoring:
- **Monitoring**: Watching predefined metrics; alerts when known thresholds are crossed
- **Observability**: Understanding unknown failure modes; allows exploration of novel problems

---

## The Four Pillars

### 1. Metrics

Metrics are **numeric measurements over time**. They are aggregated and efficient to store.

**Prometheus** is the standard for Kubernetes metrics.

Metric types:
| Type | Description | Example |
|---|---|---|
| **Counter** | Monotonically increasing; never decreases | `http_requests_total` |
| **Gauge** | Can go up or down | `memory_usage_bytes`, `active_connections` |
| **Histogram** | Samples observations; buckets for distribution | `http_request_duration_seconds` |
| **Summary** | Like histogram but calculates quantiles client-side | `rpc_duration_seconds` |

Prometheus uses a **pull model**: it scrapes `/metrics` endpoints on a schedule.

```yaml
# Prometheus scrape config
scrape_configs:
  - job_name: 'my-app'
    static_configs:
      - targets: ['my-app:9090']
```

**PromQL** — Prometheus Query Language:
```promql
# Request rate per second over the last 5 minutes
rate(http_requests_total[5m])

# 99th percentile latency
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))

# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])
```

**Grafana** is used to visualize Prometheus metrics with dashboards.

### 2. Logs

Logs are **timestamped records of discrete events**. Useful for debugging specific requests.

Best practices:
- Use **structured logging** (JSON) — machine-parseable, enables filtering and querying
- Include correlation IDs (trace IDs) to link logs to traces
- Log at appropriate levels: DEBUG, INFO, WARN, ERROR

**Log aggregation stack options:**
- **ELK/EFK**: Elasticsearch + Logstash/Fluentd + Kibana
- **Loki + Grafana**: Loki indexes log labels (not full text); efficient storage; integrates with Grafana
- **Splunk**, **Datadog**: Commercial options

**Loki** is Kubernetes-native and popular in cloud-native stacks because:
- Stores only labels (not full log content) in the index — very efficient
- Log content is stored compressed in object storage
- Queries with **LogQL** (similar to PromQL)

### 3. Traces

Traces capture the **journey of a request** through a distributed system. A trace is made of **spans** — each span represents a unit of work in one service.

```
Request comes in
│
└── API Gateway (span, 50ms total)
    ├── Auth Service call (span, 10ms)
    ├── Product Service call (span, 20ms)
    │   └── Database query (span, 15ms)
    └── Response sent
```

Key concepts:
- **Trace ID**: Unique identifier for the entire request flow
- **Span ID**: Identifier for one unit of work
- **Parent span**: The span that called this span
- **Span attributes**: Key-value metadata (HTTP status, DB query, user ID)
- **Span events**: Timestamped events within a span

**Distributed tracing tools:**
- **Jaeger**: Open-source, CNCF graduated
- **Tempo** (Grafana): Storage backend for traces
- **Zipkin**: Early open-source tracing system

### 4. Events

Events are **significant occurrences** in the system. In Kubernetes:

```bash
kubectl get events -n production
# Shows: pod scheduled, image pulled, container started, liveness probe failed, etc.
```

Events differ from logs in that they represent system-level state changes, not application log lines.

---

## OpenTelemetry (OTel)

OpenTelemetry is the CNCF standard for telemetry — a unified framework for metrics, logs, and traces.

Why it matters:
- **Vendor-neutral**: Instrument once, send to any backend (Prometheus, Jaeger, Datadog, etc.)
- **Standardized**: One API/SDK instead of per-vendor instrumentation
- **Auto-instrumentation**: Many languages support zero-code instrumentation

Components:
- **API**: Language-specific interfaces for creating telemetry
- **SDK**: Implementation of the API
- **Collector**: Receives, processes, and exports telemetry data
- **Instrumentation libraries**: Auto-instrument popular frameworks

```
App (OTel SDK)
      │
      ▼
OTel Collector  ──► Prometheus (metrics)
                ──► Jaeger / Tempo (traces)
                ──► Loki / Elasticsearch (logs)
```

The **OTel Collector** runs as a sidecar or DaemonSet in Kubernetes, collecting telemetry from applications and forwarding to backends.

---

## The USE Method and RED Method

### USE Method (for infrastructure)
- **Utilization**: % time the resource is busy
- **Saturation**: Queue depth or excess requests
- **Errors**: Error rate

### RED Method (for services)
- **Rate**: Requests per second
- **Errors**: Error rate
- **Duration**: Latency (distribution)

### The Four Golden Signals (Google SRE)
1. **Latency**: Time to service a request
2. **Traffic**: How much demand is on the system
3. **Errors**: Rate of failed requests
4. **Saturation**: How full the service is (CPU, memory, queue depth)

---

## Service Level Objectives (SLOs)

SLOs define reliability targets:

- **SLI (Service Level Indicator)**: A metric that measures reliability (e.g., request success rate)
- **SLO (Service Level Objective)**: The target for the SLI (e.g., 99.9% requests succeed)
- **SLA (Service Level Agreement)**: The contractual commitment, often with financial penalties
- **Error Budget**: 100% - SLO = the allowed unreliability (1 - 0.999 = 0.1% of requests can fail)

Error budgets gate risky deployments: if the error budget is depleted, freeze feature deployments and focus on reliability.

---

## Alerting

Good alerts are:
- **Actionable**: Someone can do something about it
- **Symptom-based**: Alert on user-visible problems (high error rate), not causes (CPU usage)
- **Not too noisy**: Alert fatigue causes important alerts to be ignored

**Prometheus Alertmanager** handles routing alerts to notification channels (PagerDuty, Slack, email).

```yaml
# Prometheus alert rule
groups:
  - name: my-app
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate on {{ $labels.service }}"
```

---

## Key Takeaways

- Observability = metrics + logs + traces + events
- Metrics are efficient aggregates; logs are detailed event records; traces follow requests across services
- OpenTelemetry is the vendor-neutral standard for all telemetry signals
- Prometheus pulls metrics; Grafana visualizes them
- RED method for services (Rate, Errors, Duration); USE for infrastructure
- SLOs define reliability targets; error budgets gate risky changes
- Alert on symptoms (high error rate), not causes (CPU usage)
