# Platform Efficiency and Team Productivity

## Overview

Measuring platform efficiency means quantifying both the health of the platform itself and the productivity improvements it delivers to engineering teams. This requires a multi-dimensional approach: delivery performance metrics, developer experience surveys, platform adoption data, toil tracking, and cost analysis. Together, these metrics help platform teams demonstrate value, prioritize investments, and continuously improve.

---

## What to Measure: The Measurement Stack

Platform measurement operates at three levels:

```
Level 3: Business Outcomes
  ↑
  Revenue impact, time-to-market, compliance adherence, talent retention
  
Level 2: Developer Productivity
  ↑
  DORA metrics, onboarding time, developer satisfaction, toil reduction
  
Level 1: Platform Health
  ↑
  Availability, latency, error rates, resource utilization, cost
```

A common mistake is measuring only Level 1 (infrastructure up/down) while ignoring the higher-level outcomes that actually justify platform investment.

---

## Delivery Performance Metrics (DORA)

The DORA Four Keys are the industry-standard delivery performance metrics. They measure how fast and how safely a team delivers software to production. See the dedicated [DORA metrics file](./02-dora-metrics.md) for full detail.

| Metric | What It Measures |
|--------|-----------------|
| Deployment Frequency | How often deployments reach production |
| Lead Time for Changes | Time from code commit to production |
| Mean Time to Restore (MTTR) | Time to recover from production incidents |
| Change Failure Rate | Percentage of deployments causing incidents |

---

## Developer Satisfaction Metrics

### Developer Net Promoter Score (Developer NPS)

NPS (Net Promoter Score) is a survey-based metric that asks:

> "On a scale of 0–10, how likely are you to recommend our platform/tooling to a colleague?"

| Score | Segment | Action |
|-------|---------|--------|
| 9–10 | Promoters | Understand what's working |
| 7–8 | Passives | Find and remove friction |
| 0–6 | Detractors | Investigate and fix pain points |

```
Developer NPS = (% Promoters) − (% Detractors)

Score interpretation:
  > 50 : Excellent — developers are platform advocates
  20–50: Good — room for improvement
   0–20: Acceptable — significant friction exists
  < 0  : Poor — platform is a source of frustration
```

**Collecting Developer NPS:**
- Quarterly survey (avoid survey fatigue)
- Embedded in the developer portal
- Anonymous to encourage honest feedback
- Include open-text "why?" question to contextualize scores
- Segment by team, seniority, and tenure to identify patterns

### Developer Satisfaction Survey Topics

Beyond NPS, detailed satisfaction surveys cover:

```
TOOLING
  - How satisfied are you with your CI/CD pipeline?
  - How satisfied are you with the developer portal?
  - How satisfied are you with documentation quality?

PROCESS
  - How often are you blocked waiting on another team?
  - How much time do you spend on toil per week?
  - How easy is it to onboard to a new service?

PLATFORM RELIABILITY
  - How often does the platform cause unexpected downtime for you?
  - How responsive is the platform team when you have problems?

OVERALL
  - If you could change one thing about the platform, what would it be?
```

---

## The SPACE Framework

SPACE is a multi-dimensional developer productivity framework from Microsoft Research (Forsgren et al., 2021). It argues that productivity cannot be captured by a single metric.

| Dimension | What It Measures | Example Metrics |
|-----------|-----------------|-----------------|
| **S**atisfaction & Well-being | How developers feel about their work | Developer NPS, survey scores, burnout indicators |
| **P**erformance | Outcomes and quality of work | Change failure rate, reliability, customer satisfaction |
| **A**ctivity | Volume of work completed | PRs merged, deployments, code reviews |
| **C**ommunication & Collaboration | How teams work together | Review turnaround time, documentation coverage |
| **E**fficiency & Flow | Minimal interruptions, smooth workflow | Toil %, PR cycle time, wait times, onboarding time |

### Using SPACE in Practice

SPACE metrics should be collected at **multiple levels**:

```
Individual level: Self-reported satisfaction, perceived productivity
Team level:       Deployment frequency, PR cycle time, incident frequency  
System level:     DORA metrics, platform availability, API latency
```

> Critical insight: never use activity metrics (like PRs merged) to evaluate individual developer performance. Activity is a system-level signal, not an individual performance signal.

---

## Platform Adoption Metrics

Adoption metrics tell you whether teams are actually using the platform capabilities you build. High adoption means the platform is delivering value; low adoption is a signal that developers find alternative paths (or the golden path is not compelling).

### Key Adoption Metrics

| Metric | Definition | Target |
|--------|-----------|--------|
| **Active portal users** | Unique devs using the portal per month | > 80% of developers |
| **Templates used** | % of new services created from golden path templates | > 90% |
| **Catalog coverage** | % of services with a `catalog-info.yaml` | > 95% |
| **TechDocs coverage** | % of services with up-to-date documentation | > 80% |
| **Self-service rate** | % of infrastructure requests fulfilled via self-service (vs. tickets) | > 85% |
| **Pipeline standardization** | % of services using the standard CI/CD pipeline | > 90% |

### Measuring Adoption

```python
# Pseudocode: catalog coverage calculation
total_services = catalog.count_entities(kind="Component", type="service")
services_with_docs = catalog.count_entities(
    kind="Component",
    type="service",
    has_annotation="backstage.io/techdocs-ref"
)
catalog_coverage = services_with_docs / total_services * 100
```

### Portal Usage Analytics

Backstage and other portals can emit usage events that are tracked via analytics:

```typescript
// Backstage analytics tracking (app-config.yaml)
app:
  analytics:
    ga4:
      measurementId: G-XXXXXXXXXX
      testMode: false
      // custom dimensions
      customDimensionsMetrics:
        - type: dimension
          index: 1
          source: user
          name: userEntityRef
        - type: dimension
          index: 2
          source: context
          name: pluginId
```

---

## Toil Reduction Measurement

Toil is manual, repetitive, automatable work that scales with service growth. Measuring toil reduction is a direct measure of platform team effectiveness.

### Toil Categories and Tracking

| Toil Category | How to Measure Reduction |
|---------------|--------------------------|
| Manual deployments | Deployment automation rate (automated/total) |
| Certificate renewals | Certificates managed by cert-manager / total |
| Secret rotation requests | Automated rotations / total rotation events |
| Infrastructure requests | Self-service fulfillment rate |
| On-call pages that required no human action | Auto-remediated pages / total pages |
| Dependency update PRs | Renovate/Dependabot PRs auto-merged / total |

### Toil Percentage Measurement

```
Platform team toil survey (weekly or biweekly):

"Estimate the percentage of your work hours this week spent on:
  [ ] Manual operational tasks (deployments, access requests, etc.)
  [ ] Reactive support (answering questions, debugging for others)
  [ ] Project work (building new capabilities)
  [ ] Meetings and planning"

Toil % = (Manual + Reactive) / Total Hours × 100

Target: < 50% toil (Google SRE standard)
Excellent: < 25% toil
```

### Before/After Toil Tracking

Document specific toil elimination efforts and their impact:

| Toil Eliminated | Before | After | Time Saved/Week |
|----------------|--------|-------|----------------|
| Manual namespace creation | 15 min/request × 40 requests/week | Automated via portal | 10 hours |
| Manual SSL certificate requests | 2 tickets/day × 30 min each | cert-manager automated | 7 hours |
| Manual staging environment setup | 4 hours × 5 requests/week | Automated via GitOps | 20 hours |
| Dependency update tickets | 8 hours/week | Renovate Bot automated | 8 hours |

---

## Cost Per Deployment

Cost per deployment normalizes infrastructure spending against delivery output. It helps justify platform investment in efficiency tools.

```
Cost Per Deployment = Total Platform Infrastructure Cost / Number of Deployments

Example:
  Monthly cloud spend:          $50,000
  Monthly deployments to prod:  2,000
  Cost per deployment:          $25

After optimizing (Spot instances, better bin-packing, cache layers):
  Monthly cloud spend:          $35,000
  Monthly deployments to prod:  3,000 (adoption increased)
  Cost per deployment:          $11.67

Improvement: 53% reduction in cost per deployment
```

### Cost Attribution

For meaningful cost metrics, you need cost attribution at the service/team level:

```yaml
# Kubernetes labels for cost attribution (OpenCost / Kubecost)
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    cost-center: "payments"       # business unit
    team: "payments-team"         # owning team
    service: "payments-api"       # service name
    environment: "production"     # environment
```

---

## Onboarding Time as a Metric

Time-to-Productivity (T2P) or Time-to-First-Deployment (T2FD) measures how quickly new developers can contribute to production.

### Measuring Onboarding Time

```
T2FD = Date of First Production Deployment − Hire/Start Date

Measurement method:
  1. Record start dates in HR system
  2. Record first production deployment date per developer (from CI/CD logs)
  3. Calculate delta
  4. Segment by:
     - Seniority (junior vs. senior)
     - Team (some teams may have harder domains)
     - Platform experience (internal transfer vs. external hire)
```

### Onboarding Funnel Analysis

```
Step 1: Dev portal access granted                    → Day 0 (target: automated)
Step 2: First catalog browse                         → Day 0-1
Step 3: First service created from template          → Day 1-2 (target: same day)
Step 4: First PR merged to a service                 → Day 2-5
Step 5: First deployment to staging                  → Day 3-7
Step 6: First deployment to production               → Day 5-10 (target: < 5 days)
```

Each step can be measured from portal analytics and CI/CD logs, creating a funnel view that identifies where new developers get stuck.

---

## Feedback Mechanisms

### Quantitative Feedback

| Method | Cadence | What It Captures |
|--------|---------|-----------------|
| Developer NPS survey | Quarterly | Overall platform satisfaction trend |
| Pulse surveys (3–5 questions) | Monthly | Targeted satisfaction on specific areas |
| Portal usage analytics | Continuous | Adoption, feature utilization, drop-off points |
| CI/CD metrics | Continuous | Pipeline performance, failure rates |
| On-call data | Continuous | Incident frequency, alert noise |

### Qualitative Feedback

| Method | Cadence | What It Captures |
|--------|---------|-----------------|
| 1:1 interviews with developers | Quarterly | Deep insight, unarticulated pain points |
| Platform team office hours | Weekly | Real-time feedback, relationship building |
| Retrospectives with stream-aligned teams | Per sprint | Sprint-level friction and wins |
| User research sessions | Quarterly | Usability of new portal features |
| Slack/Teams feedback channel | Continuous | Passive, unstructured feedback |

### Closing the Feedback Loop

Feedback is only valuable if it drives action and the action is communicated back:

```
1. Collect feedback (survey, interview, analytics)
2. Triage and categorize (tooling friction, documentation, process, missing capability)
3. Prioritize (impact × effort matrix)
4. Act (platform backlog items)
5. Communicate ("Based on your feedback, we shipped X this quarter")
6. Re-measure (did the action improve the metric?)
```

A **developer platform changelog** or **release notes** shared in Slack or the portal is an effective way to demonstrate that feedback is heard and acted upon.

---

## Building a Platform Metrics Dashboard

### Recommended Dashboard Sections

```
SECTION 1: Delivery Performance (DORA)
  - Deployment frequency trend (30d, 90d, 1y)
  - Lead time distribution (p50, p90, p99)
  - MTTR trend
  - Change failure rate trend
  - DORA classification (Elite/High/Medium/Low)

SECTION 2: Developer Experience
  - Developer NPS trend (quarterly)
  - Toil percentage (self-reported, trend)
  - Time to first deployment (new hires, trend)

SECTION 3: Platform Adoption
  - Active portal users / total developers
  - Catalog coverage %
  - Template adoption %
  - Self-service rate %

SECTION 4: Platform Health
  - Platform availability (SLO compliance)
  - API latency (p99)
  - Provisioning time (environments, services)

SECTION 5: Cost Efficiency
  - Cost per deployment (trend)
  - Total platform cost / team served
  - Cost by team/service (attribution)
```

---

## Exam Tips

- **Developer NPS formula**: `% Promoters − % Detractors` (scores 9–10 are promoters; 0–6 are detractors; 7–8 are passives)
- **SPACE** has five dimensions: Satisfaction, Performance, Activity, Communication, Efficiency
- **Never use Activity metrics** (lines of code, PRs merged) to evaluate individual performance
- **Toil** must be: manual, repetitive, automatable, tactical, devoid of lasting value, scaling linearly
- **T2FD** (Time to First Deployment) is the key onboarding metric
- **Cost per deployment** normalizes cloud spend to delivery output
- **Platform adoption rate** is a leading indicator of platform value
- **Feedback loops must be closed** — collecting feedback without action erodes trust
- Know the difference between **quantitative metrics** (dashboards, CI/CD data) and **qualitative feedback** (interviews, surveys)
