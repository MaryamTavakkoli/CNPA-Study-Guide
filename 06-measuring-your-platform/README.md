# Domain 6: Measuring Your Platform

**Exam Weight: 8%**

This domain covers how platform engineering teams measure the effectiveness, efficiency, and impact of their platforms. It encompasses engineering productivity metrics, developer satisfaction measurement, DORA metrics, the SPACE and DevEx frameworks, and techniques for making the business case for platform investment.

---

## Topic List

| # | Topic | File |
|---|-------|------|
| 1 | Platform Efficiency and Team Productivity | [01-platform-efficiency-productivity.md](./01-platform-efficiency-productivity.md) |
| 2 | DORA Metrics In Depth | [02-dora-metrics.md](./02-dora-metrics.md) |

---

## Key Concepts

### Why Measurement Matters in Platform Engineering

Platform teams are often considered cost centers. Without clear metrics, platform investments are difficult to justify and the team's impact is invisible. Effective measurement enables:

- **Making the business case** for platform investment and headcount
- **Identifying bottlenecks** in the software delivery pipeline
- **Demonstrating ROI** of platform initiatives (e.g., golden paths reduced service creation time from 2 weeks to 2 hours)
- **Tracking improvement** over time and validating that platform changes are having the desired effect
- **Aligning with product teams** around shared delivery performance goals
- **Detecting degradation** before it becomes a crisis

### The Measurement Challenge

Measuring platform value is difficult because:
- Platform teams are **multipliers**, not direct producers — their output is the productivity of other teams
- Developer productivity is **hard to quantify** — lines of code, story points, and PR counts are poor proxies
- **Attribution** is complex — many factors affect delivery performance (org structure, technical debt, staffing)
- **Lagging vs. leading indicators** — the best outcomes metrics (DORA) reflect delivery performance, which has many causes

### Measurement Frameworks at a Glance

| Framework | Focus | Key Metrics |
|-----------|-------|-------------|
| **DORA** | Software delivery performance | Deployment Frequency, Lead Time, MTTR, Change Failure Rate |
| **SPACE** | Developer productivity (multi-dimensional) | Satisfaction, Performance, Activity, Communication, Efficiency |
| **DevEx** | Developer experience quality | Feedback loops, cognitive load, flow state |
| **Core 4 + 1** | DORA + reliability | DORA metrics + availability |
| **Platform SLOs** | Platform reliability | Uptime, API latency, provisioning time |

---

## Core Metrics Categories

### Delivery Performance Metrics (DORA)

```
Deployment Frequency     → How often do we deploy to production?
Lead Time for Changes    → How long from commit to production?
Mean Time to Restore     → How long to recover from an incident?
Change Failure Rate      → What % of deployments cause incidents?
```

### Developer Experience Metrics

```
Developer NPS            → Net Promoter Score for the platform
Time to First Deployment → How fast can a new developer deploy?
Toil Percentage          → What % of time is spent on unvalued work?
P2P (PR to Production)   → How fast do code changes reach prod?
Platform Adoption Rate   → What % of teams use platform capabilities?
```

### Platform Health Metrics

```
Platform Availability    → SLO/SLA compliance
Provisioning Time        → How long to create an environment?
Platform API Latency     → Response time for self-service APIs
Incident Count           → How often does the platform have incidents?
Cost per Deployment      → Total platform cost / number of deployments
```

### Team and Business Metrics

```
Onboarding Time          → Days from hire to first production deployment
Developer Satisfaction   → Survey-based satisfaction scores
Retention/Attrition      → Developer turnover rates
Feature Cycle Time       → Idea to production for a typical feature
```

---

## The Measurement Lifecycle

```
1. DEFINE    → What outcomes do we care about? (align with business goals)
2. INSTRUMENT → How do we collect the data? (tooling, surveys, APIs)
3. BASELINE  → What is the current state? (establish starting point)
4. TARGET    → What does "better" look like? (set improvement goals)
5. MEASURE   → Collect data regularly (dashboards, automated collection)
6. ACT       → Use data to prioritize platform investments
7. VALIDATE  → Did our actions improve the metrics? (closed-loop learning)
```

---

## Quick Reference: Metric Benchmarks

### DORA Performance Levels (2023 State of DevOps Report)

| Metric | Elite | High | Medium | Low |
|--------|-------|------|--------|-----|
| Deployment Frequency | Multiple/day | Weekly–monthly | Monthly | < Monthly |
| Lead Time for Changes | < 1 hour | 1 day–1 week | 1 week–1 month | > 6 months |
| MTTR | < 1 hour | < 1 day | 1 day–1 week | > 1 week |
| Change Failure Rate | 0–5% | 5–10% | 10–15% | > 15% |

### Developer Satisfaction Benchmarks

| Metric | Good | Acceptable | Needs Work |
|--------|------|-----------|------------|
| Developer NPS | > 50 | 20–50 | < 20 |
| Toil Percentage | < 25% | 25–50% | > 50% |
| Onboarding Time | < 1 day | 1–5 days | > 1 week |

---

## Further Reading

- [DORA Research Program](https://dora.dev/)
- [Accelerate: The Science of Lean Software and DevOps — Forsgren, Humble, Kim](https://itrevolution.com/accelerate-book/)
- [SPACE Framework Paper (Microsoft Research)](https://dl.acm.org/doi/10.1145/3453928)
- [Developer Experience Framework — GitHub Research](https://queue.acm.org/detail.cfm?id=3595878)
- [Google SRE Book — Eliminating Toil](https://sre.google/sre-book/eliminating-toil/)
- [CNCF Platforms White Paper](https://tag-app-delivery.cncf.io/whitepapers/platforms/)
