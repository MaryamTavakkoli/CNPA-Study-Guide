# DORA Metrics In Depth

## Overview

The DORA (DevOps Research and Assessment) metrics are the most widely validated set of software delivery performance indicators in the industry. They originate from the research published in *Accelerate: The Science of Lean Software and DevOps* (Forsgren, Humble & Kim, 2018) and the annual *State of DevOps Report* produced by the DORA research program (now at Google).

DORA metrics define **four key metrics** that, when taken together, characterize the speed and stability of software delivery. They are foundational to platform engineering because they provide an objective, evidence-based language for measuring the impact of platform investments.

---

## The Four Key DORA Metrics

### 1. Deployment Frequency (DF)

**Definition:** How often an organization successfully releases to production.

**What it measures:** The throughput (speed) of the software delivery system.

| Elite | High | Medium | Low |
|-------|------|--------|-----|
| Multiple deploys per day | Once per day to once per week | Once per week to once per month | Fewer than once per month |

**How to measure:**

```
Deployment Frequency = Number of production deployments / Time period

Measurement sources:
  - CI/CD pipeline records (GitHub Actions, Jenkins, Argo CD)
  - Deployment tracking tools (e.g., Argo CD sync events)
  - Release management systems

Example query (Prometheus, if using deployment events):
  increase(deployment_total{env="production"}[30d]) / 30
  → Average daily deployment count over 30 days
```

**Platform team impact on DF:**
- Automated CI/CD reduces manual steps, enabling more frequent releases
- Golden paths with built-in deployment pipelines make deployment the easy path
- Feature flags decouple deployment from release, allowing trunk-based development
- Canary deployments and progressive delivery reduce deployment risk, encouraging more frequent releases

### 2. Lead Time for Changes (LT)

**Definition:** The time it takes for a commit to reach production.

**What it measures:** The end-to-end efficiency of the delivery pipeline (throughput + cycle time).

| Elite | High | Medium | Low |
|-------|------|--------|-----|
| Less than one hour | One day to one week | One week to one month | Six months to one year |

**How to measure:**

```
Lead Time = Production Deployment Timestamp − Code Commit Timestamp

Measurement sources:
  - Git commit timestamps
  - CI/CD pipeline completion timestamps
  - Deployment timestamps (CD system)

Calculation:
  LT per PR = merge_to_main_time + pipeline_duration + deployment_duration

  Aggregate as:
    - p50 (median): typical experience
    - p90: worst common experience
    - p99: worst-case outliers
```

**Lead time breakdown:**

```
Code Commit
  │
  ▼ ← Code review time (can be days or minutes)
PR Merged to Main
  │
  ▼ ← CI pipeline time (build, test, scan)
Artifact Published
  │
  ▼ ← Deployment pipeline time (staging → canary → production)
Production Deployment
  │
  ▼
Lead Time = entire duration above
```

**Platform team impact on LT:**
- Fast, reliable CI pipelines (caching, parallelism) reduce pipeline time from 40 minutes to 5
- Automated code quality gates replace manual review steps
- GitOps with automated sync reduces deployment lag
- Staging environment parity with production reduces "works on staging, fails on prod" rework

### 3. Mean Time to Restore (MTTR)

**Definition:** The average time it takes to restore service after a production incident.

**What it measures:** The stability and resilience of the system.

> Note: Some DORA materials use "Mean Time to Recovery" interchangeably. Both refer to the same metric.

| Elite | High | Medium | Low |
|-------|------|--------|-----|
| Less than one hour | Less than one day | One day to one week | More than six months |

**How to measure:**

```
MTTR = Mean(Incident Resolution Time − Incident Start Time)

Where:
  Incident Start Time    = Time the incident began affecting users
                           (first alert, first error spike, or customer report)
  Incident Resolution Time = Time when service returned to normal

Measurement sources:
  - PagerDuty / OpsGenie incident records
  - Status page incident timelines
  - On-call rotation records
  - Monitoring system (first alert → all-clear)

MTTR = Σ(incident_end_time − incident_start_time) / total_incidents
```

**Important distinction — MTTR vs. MTTD vs. MTTF:**

| Term | Definition |
|------|-----------|
| **MTTD** (Mean Time to Detect) | Time from incident start to first alert/detection |
| **MTTR** (Mean Time to Restore) | Time from incident start to full service restoration |
| **MTTF** (Mean Time to Failure) | Average time between failures (reliability measure) |
| **MTBF** (Mean Time Between Failures) | Average time between the end of one failure and start of next |

**Platform team impact on MTTR:**
- Automated rollback capabilities (Argo Rollouts, Flagger) can restore service in < 5 minutes
- Runbook automation (PagerDuty Automation Actions) reduces manual intervention steps
- Observability tooling (distributed tracing, correlation) reduces time to root cause
- Canary deployments limit the blast radius of bad deployments
- Feature flags allow instant "off switches" for problematic features

### 4. Change Failure Rate (CFR)

**Definition:** The percentage of deployments to production that result in a service degradation requiring hotfix, rollback, or incident.

**What it measures:** The quality and stability of the delivery system.

| Elite | High | Medium | Low |
|-------|------|--------|-----|
| 0–5% | 5–10% | 10–15% | 46–60% |

**How to measure:**

```
Change Failure Rate = (Failed Deployments / Total Deployments) × 100

Where "failed deployment" = a deployment that:
  - Required a hotfix within 24 hours
  - Caused an incident (P1/P2 incident linked to a deployment)
  - Was rolled back

Measurement sources:
  - Link incidents to deployments (via deployment tracking in PagerDuty/JIRA)
  - Tag CI/CD runs with "caused incident" when linked to incident ticket
  - Post-incident reviews that identify root cause as a deployment

Example (automated linking):
  - Each deployment tagged with git SHA
  - Incidents tagged with triggering git SHA
  - CFR calculated from linked records
```

**Platform team impact on CFR:**
- Comprehensive automated test suites catch regressions before production
- Static analysis and SAST in CI reduces security and quality defects reaching production
- Progressive delivery (canary, blue/green) limits exposure of bad changes
- Database migration validation prevents schema-related incidents
- Contract testing prevents API compatibility issues between services

---

## Elite vs. Low Performer Summary

| Metric | Elite Performers | Low Performers | Difference |
|--------|-----------------|---------------|------------|
| Deployment Frequency | Multiple/day | Quarterly/yearly | 208x faster |
| Lead Time | < 1 hour | 6+ months | 2,604x faster |
| MTTR | < 1 hour | > 1 week | 168x faster |
| Change Failure Rate | 0–5% | 46–60% | 7x lower |

> These numbers come from the 2019 *Accelerate State of DevOps* report. Elite performers also show significantly higher organizational performance, commercial success, and employee satisfaction.

---

## The Speed-Stability Trade-off — a Myth

A common misconception is that deploying more frequently means accepting more failures. DORA research consistently demonstrates the opposite:

```
Intuition:  More deployments → More failures (speed vs. stability trade-off)
Research:   Elite performers → Higher DF AND lower CFR AND lower MTTR

Why? 
  - Smaller deployments are easier to test and review
  - Failures are isolated to smaller changes, easier to diagnose
  - Practice makes deployment routine, reducing human error
  - Platform investments in automation improve both speed AND quality
```

This insight is the core justification for investing in platform engineering: the same capabilities (CI/CD automation, progressive delivery, observability) improve all four DORA metrics simultaneously.

---

## Measuring DORA Metrics in Practice

### Tooling for DORA Measurement

| Tool | What It Provides |
|------|-----------------|
| **Four Keys (Google)** | Open-source DORA metric collector (GitHub/GitLab → BigQuery → Looker) |
| **DORA Quick Check** | Google's online DORA assessment tool |
| **LinearB** | Commercial DORA metric tracking for engineering teams |
| **Sleuth** | Commercial DORA-focused deployment tracking |
| **Haystack** | Engineering analytics platform (DORA + SPACE) |
| **Argo CD metrics** | Native metrics for deployment frequency and success rates |
| **GitHub Insights** | Built-in PR cycle time and deployment frequency |

### Google Four Keys Project

```yaml
# four-keys stack (simplified)
# Collects events from GitHub/GitLab → BigQuery → Looker/Grafana dashboard

# Event sources collected:
# - Push events (for deployment frequency)
# - Pull request events (for lead time)
# - Issues with "incident" label (for MTTR and CFR)
# - Deployment events
```

### Building a DORA Dashboard with Grafana

```yaml
# Grafana dashboard panel: Deployment Frequency
# Using CI/CD pipeline data stored in PostgreSQL

SQL query:
  SELECT
    date_trunc('week', deployed_at) as week,
    COUNT(*) as deployments,
    COUNT(*) / 7.0 as daily_rate
  FROM deployments
  WHERE environment = 'production'
    AND deployed_at > NOW() - INTERVAL '90 days'
  GROUP BY 1
  ORDER BY 1

Visualization: Bar chart with weekly deployments
Reference lines: Elite (> 7/day), High (> 1/week)
```

---

## DORA Metrics and Platform Engineering ROI

DORA metrics are the primary instrument for making the business case for platform team investment. The argument follows this logic:

```
Step 1: Establish baseline DORA metrics (current state)
Step 2: Identify platform bottlenecks contributing to each metric
Step 3: Build platform capabilities to address bottlenecks
Step 4: Re-measure DORA metrics post-investment
Step 5: Calculate business impact of improvement
```

### Example: Making the Business Case

**Before platform investment:**
```
Deployment Frequency: 2 per week (Medium)
Lead Time:           3 days (Medium)
MTTR:                6 hours (High)
CFR:                 18% (Low)
```

**Platform investments made:**
- Built standardized CI/CD pipeline (reduced pipeline time from 45min → 8min)
- Implemented canary deployments with Argo Rollouts
- Added distributed tracing (Jaeger/Tempo)
- Created one-click rollback in developer portal

**After platform investment (6 months):**
```
Deployment Frequency: 5 per day (Elite)     ↑ 17x
Lead Time:           45 minutes (Elite)     ↑ 96x
MTTR:                25 minutes (Elite)     ↑ 14x
CFR:                 4% (Elite)             ↑ 4.5x
```

**Business impact translation:**
```
Faster deployments → Features reach customers faster
                   → Competitive advantage, revenue impact
Lower CFR         → Fewer incidents → Less customer churn
                   → Engineer time saved on firefighting
Lower MTTR        → Less downtime → SLA compliance
                   → Direct revenue protection (if SLA-bound)
Shorter lead time → Faster response to market changes
                   → Faster bug fixes
```

---

## The SPACE Framework

SPACE (Forsgren et al., 2021) is a complementary framework to DORA that measures developer productivity across five dimensions.

### The Five SPACE Dimensions

#### Satisfaction and Well-being

Developer satisfaction with their work, tools, and environment. Correlated with retention and productivity.

```
Metrics:
  - Developer NPS
  - Employee Engagement Score
  - Burnout/stress indicators
  - Retention rate
  - Survey-based satisfaction (tooling, process, culture)
```

#### Performance

The outcomes and quality of work — what the team produces, not just how much activity they generate.

```
Metrics:
  - Change Failure Rate (DORA)
  - Service reliability (availability, SLO compliance)
  - Customer-reported defects
  - Security vulnerability density
  - Code review thoroughness (defects found in review vs. production)
```

#### Activity

Quantifiable actions and outputs of developer work. Useful at system level, dangerous at individual level.

```
System-level activity metrics (appropriate):
  - Deployment frequency
  - PR merge rate per team
  - Test coverage trend
  - Documentation update frequency

Individual-level activity metrics (dangerous — should NOT be used for evaluation):
  - Lines of code per developer
  - PRs merged per developer
  - Commits per day
```

#### Communication and Collaboration

How effectively developers work together and share knowledge.

```
Metrics:
  - Code review turnaround time
  - PR review depth (comments per PR)
  - Documentation coverage
  - API contract coverage
  - Internal conference/guild participation
  - Knowledge sharing sessions hosted
```

#### Efficiency and Flow

The ability to complete work with minimal interruptions, context switching, or waiting.

```
Metrics:
  - Toil percentage (manual work / total work)
  - PR cycle time (open → merge)
  - Build wait time (time in queue)
  - Environment provisioning time
  - Context switching frequency (estimated from calendar data)
  - "Time in flow" (estimated from deep-work blocks)
  - Incident interruptions per developer per week
```

### SPACE Applied to Platform Assessment

| SPACE Dimension | Platform Metric | Platform Capability |
|----------------|-----------------|---------------------|
| Satisfaction | Developer NPS | Portal UX, documentation quality |
| Performance | CFR, SLO compliance | Testing automation, progressive delivery |
| Activity | Deployment frequency | CI/CD automation, golden paths |
| Communication | Documentation coverage | TechDocs, service catalog |
| Efficiency | Toil %, provisioning time | Self-service portal, automation |

---

## The DevEx Framework

The Developer Experience (DevEx) framework (Noda et al., 2023, GitHub) identifies three core factors that drive developer experience:

### Three DevEx Core Factors

| Factor | Description | Example Good/Bad |
|--------|-------------|-----------------|
| **Feedback Loops** | Speed and quality of signals developers receive from their tools | Good: CI finishes in 3 min. Bad: CI takes 45 min, output cryptic |
| **Cognitive Load** | Mental effort required to understand and operate the system | Good: Golden paths handle complexity. Bad: Developer must know k8s internals |
| **Flow State** | Ability to work without interruption or context switching | Good: Full morning of uninterrupted coding. Bad: 12 Slack pings before 10am |

### DevEx Measurement Approach

DevEx is primarily measured through **perception surveys** (how developers feel) combined with **workflow data** (what the system reports):

```
Perceived DevEx Survey (sample questions):

"On a scale of 1-5, how would you rate...":
  1. The speed of feedback from your CI/CD pipeline
  2. How easy it is to find documentation for platform tools
  3. How often you can maintain a flow state during work
  4. Your ability to resolve incidents quickly
  5. The clarity and usefulness of error messages in your tools

"What is your biggest day-to-day frustration with the platform?"
```

---

## Relationship Between Frameworks

```
DORA          → Outcome metrics (how fast, how safely we deliver)
SPACE         → Multi-dimensional productivity (speed, quality, satisfaction, collaboration, efficiency)
DevEx         → Developer perception and experience quality
Platform SLOs → Platform reliability (uptime, latency, availability)

Recommended combination:
  - DORA for leadership conversations (business outcomes)
  - SPACE for team-level improvement discussions  
  - DevEx surveys for understanding developer pain points
  - Platform SLOs for platform team reliability accountability
```

---

## Common Pitfalls and Anti-patterns

| Anti-pattern | Problem | Better Approach |
|-------------|---------|----------------|
| Measuring only DORA without context | Metrics can be gamed (deploy trivial changes frequently) | Combine with quality metrics and qualitative feedback |
| Using activity metrics to evaluate individuals | Discourages collaboration, experimentation | Use at team/system level only |
| Reporting metrics but never acting on them | Erodes trust in the measurement program | Close the loop: measure → act → communicate → re-measure |
| Measuring only the platform team's work | Misses the actual outcome (developer productivity) | Measure outcomes of teams the platform serves |
| Big-bang DORA improvement programs | Hard to attribute change to specific actions | Incremental: identify one bottleneck, improve it, measure |
| Ignoring qualitative data | Metrics tell you what; interviews tell you why | Combine quantitative metrics with developer interviews |

---

## Exam Tips

- **Four DORA metrics**: Deployment Frequency, Lead Time for Changes, MTTR, Change Failure Rate
- **Speed and stability are NOT a trade-off** — elite performers excel at all four metrics
- **Elite DF** = multiple deploys per day; **Elite Lead Time** = < 1 hour
- **MTTR** is measured from incident start, not from when on-call is paged
- **CFR** = deployments that caused incidents / total deployments (expressed as %)
- **SPACE** = Satisfaction, Performance, Activity, Communication, Efficiency
- **DevEx** = Feedback Loops, Cognitive Load, Flow State
- Activity metrics should **never** be used to evaluate individual developers
- **Google Four Keys** is the open-source DORA metric collection project
- DORA research originally published in **Accelerate** (Forsgren, Humble, Kim, 2018)
- Platform engineering investments improve **all four DORA metrics** simultaneously
- Know how to calculate each metric and what data sources are required
