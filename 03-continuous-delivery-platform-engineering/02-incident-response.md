# Incident Response in Platform Engineering

## Why Incident Response Belongs in Platform Engineering

Platform engineering teams are responsible for the underlying infrastructure and tooling that *other teams* depend on. When the platform breaks — a Kubernetes cluster is unavailable, the CI system is down, the artifact registry is unreachable, an IDP returns 500s — every team building on that platform is blocked. This makes platform incidents uniquely high-blast-radius events, and it makes a rigorous incident response process critical.

Platform teams must own:

- **Detection**: instrumentation and alerting for platform components
- **Response process**: a defined, practiced workflow for working incidents
- **Communication**: keeping internal customers (application teams) informed
- **Learning**: blameless post-mortems that produce lasting improvements
- **Prevention**: closing the loop from post-mortems back to the CI/CD pipeline (e.g., new reliability tests)

---

## The Incident Lifecycle

```
  ┌────────────┐     ┌──────────┐     ┌─────────────┐     ┌────────────┐     ┌─────────────┐
  │  Detection  │────►│  Triage  │────►│  Mitigation │────►│ Resolution │────►│ Post-mortem │
  └────────────┘     └──────────┘     └─────────────┘     └────────────┘     └─────────────┘
       │                  │                  │                   │                   │
  Alert fires         Confirm &          Stop the             Root cause          Action
  or user             assign SEV         bleeding             eliminated          items
  reports             & IC                                    & verified          tracked
```

### Phase Details

| Phase | Key Activities | Output |
|-------|---------------|--------|
| **Detection** | Alert fires (PagerDuty/OpsGenie), user report, synthetic monitor failure | Incident ticket opened |
| **Triage** | Confirm impact, assign severity (SEV), page responders, appoint Incident Commander | SEV assigned, IC named |
| **Mitigation** | Apply fastest fix to stop customer impact (rollback, traffic reroute, feature flag off) | Customer impact reduced or eliminated |
| **Resolution** | Identify and fix root cause; verify fix; restore full service | All metrics return to healthy baseline |
| **Post-mortem** | Write blameless analysis; identify action items; share broadly | Published post-mortem doc, tracked action items |

---

## Severity Levels

Severity (SEV) levels provide a shared vocabulary for communicating incident urgency. They determine response time SLAs, who gets paged, and what communication channels are used.

| SEV | Description | Response Time | Who is Paged |
|-----|------------|--------------|--------------|
| **SEV-1** | Complete service outage; all customers affected; revenue/safety impact | Immediate (< 5 min) | On-call + manager + escalation path |
| **SEV-2** | Major functionality degraded; significant customer subset affected | < 15 min | On-call engineer + team lead |
| **SEV-3** | Minor functionality impaired; workaround available; small user subset | < 1 hour | On-call engineer |
| **SEV-4** | Minimal impact; cosmetic issue or low-traffic path; no workaround needed | Next business day | Ticket only; no page |

> **Exam tip**: SEV definitions vary by organization. What matters for the exam is understanding the *purpose* of severity levels (routing, urgency, communication) not memorizing any specific org's thresholds.

---

## The Incident Commander Role

Every non-trivial incident should have a designated **Incident Commander (IC)**. The IC's job is *coordination*, not technical fixing:

- Delegates investigation tasks to subject matter experts (SMEs)
- Manages the incident communication channel
- Decides when to escalate severity
- Calls the all-clear when the incident is resolved
- Ensures the post-mortem is scheduled

A common mistake is having the IC also be the person debugging — this leads to tunnel vision and poor communication.

---

## Runbooks

A **runbook** is a pre-written, step-by-step guide for responding to a known failure mode. Good runbooks:

- Are linked directly from the alert that fires
- Contain copy-paste commands (avoid ambiguity under stress)
- Include decision trees for branching paths
- Specify who to escalate to if the runbook doesn't resolve the issue
- Are stored in version control and treated as living documents

### Runbook Structure (Example)

```markdown
# Runbook: ArgoCD Sync Stuck

## Alert
`ArgoCD application has been OutOfSync for > 30 minutes`

## Impact
New deployments are not reaching production. Currently running version is unaffected.

## Triage Steps
1. Open ArgoCD UI → check the affected Application resource
2. Look for error message in the Sync Status panel
3. Check `kubectl get events -n argocd` for recent errors

## Common Causes & Fixes

### Cause: Webhook delivery failure
```bash
# Re-trigger sync manually
argocd app sync <app-name> --force
```

### Cause: Destination cluster unreachable
```bash
argocd cluster list
kubectl get secret -n argocd -l argocd.argoproj.io/secret-type=cluster
```

### Cause: Resource conflict (another controller managing the resource)
- Check if the resource has annotations from another tool
- Escalate to platform team lead

## Escalation
If not resolved in 30 minutes: page @platform-oncall-lead
```

### Platform teams should build runbooks for:
- Cluster API server degradation
- etcd compaction and disk pressure
- Registry push failures
- CI system (Tekton/Actions runner) outages
- Certificate expiry warnings
- Node Not Ready conditions
- PVC provisioning failures

---

## On-Call

On-call is the practice of having an engineer available to respond to incidents outside business hours. Platform engineering teams almost always have an on-call rotation.

### On-Call Best Practices

- **Rotation**: spread the on-call burden across the team on a weekly or bi-weekly schedule
- **Handoff**: outgoing on-call engineer briefs incoming on-call about ongoing issues and recent changes
- **Tooling**: engineers have mobile access to dashboards, runbooks, and escalation paths
- **Toil budget**: track on-call interrupt rate; high rates indicate a need for automation or reliability investment
- **Shadow shifts**: new team members shadow before going primary on-call

### On-Call Fatigue

Excessive paging is a serious team health issue. Platform teams should track:

- **Alert volume**: pages per week per engineer
- **MTTA** (Mean Time to Acknowledge): how long before an alert is acknowledged
- **MTTR** (Mean Time to Resolve): how long incidents last
- **Noise ratio**: what percentage of pages were actionable vs. noisy/false positives

Addressing alert fatigue typically involves: tuning alert thresholds, adding runbooks, automating common mitigations, and investing in reliability improvements that SLO data highlights.

---

## Blameless Post-Mortems

A **blameless post-mortem** (also called a blameless retrospective or incident review) is a structured analysis of an incident conducted without attributing fault to individuals. The philosophy, originating at Google and widely adopted in SRE culture, holds that:

> "Incidents are the result of systemic failures, not individual error. Humans make mistakes in systems that allow those mistakes to have large consequences."

### Why Blameless Matters

If engineers fear punishment for incidents, they hide information, avoid risky improvements, and don't honestly report near-misses. A blameless culture produces more honest post-mortems, which produce better action items, which produce more reliable systems.

### Post-Mortem Document Structure

```
## Incident Summary
One paragraph: what happened, when, how long, what was impacted.

## Timeline
| Time (UTC) | Event |
|-----------|-------|
| 14:03     | Alert fires: ArgoCD sync failure rate > 50% |
| 14:08     | On-call acknowledges alert |
| 14:12     | SEV-2 declared; IC assigned |
| 14:35     | Root cause identified: expired registry credentials |
| 14:41     | Credentials rotated; sync recovery begins |
| 15:02     | All applications synced; incident resolved |

## Root Cause Analysis
Describe the technical root cause (the "what").
Then describe contributing factors (the "why"):
- Credential expiry was not monitored
- Rotation runbook was outdated and referenced a deleted secret name
- No automated credential rotation was in place

## Impact
- Duration: 59 minutes
- Services affected: 12 ArgoCD-managed applications
- Customer impact: No new deployments during incident window; no production traffic impact
- Error budget consumed: 2.3% of monthly budget

## What Went Well
- On-call response was fast (< 5 min acknowledge)
- Incident channel was active and well-documented
- Rollback was available if needed

## What Could Be Improved
- No monitoring on credential expiry
- Runbook was stale

## Action Items
| Action | Owner | Due Date | Priority |
|--------|-------|----------|----------|
| Add Prometheus alert for registry credential expiry < 7 days | @alice | 2026-05-20 | P1 |
| Implement automated credential rotation via External Secrets Operator | @bob | 2026-05-27 | P1 |
| Audit and update all runbooks referencing registry credentials | @carol | 2026-05-20 | P2 |
```

### Post-Mortem Anti-Patterns

- **Blame**: naming individuals as the cause of an incident
- **No action items**: analysis with no follow-through
- **Vague action items**: "improve monitoring" with no owner or due date
- **Not sharing widely**: post-mortems locked to the immediate team miss organizational learning
- **Skipping the post-mortem**: "it was a one-off" — every incident has learnings

---

## SLOs, Error Budgets, and Their Relationship to Incidents

**Service Level Objectives (SLOs)** are the primary reliability target for a platform service (e.g., "the Kubernetes API server will be available 99.9% of the time over a rolling 30-day window"). The **error budget** is the allowed amount of unreliability: 100% - SLO = error budget (0.1% = ~43 minutes/month for 99.9%).

### How Incidents Connect to Error Budgets

```
Error Budget Consumption
        │
        ▼
 ┌─────────────────────────────────────────────────────────┐
 │  > 0% consumed     │  Incidents draw down the budget    │
 │  > 50% consumed    │  Slow down risky changes           │
 │  > 100% consumed   │  Freeze deployments; reliability   │
 │  (budget exhausted)│  work becomes top priority         │
 └─────────────────────────────────────────────────────────┘
```

### Platform SLO Examples

| Platform Component | Example SLO |
|-------------------|------------|
| Kubernetes API Server | 99.9% requests succeed in < 1s (30-day window) |
| CI Pipeline System | 99.5% pipeline runs complete without infrastructure errors |
| Artifact Registry | 99.9% push/pull operations succeed |
| Internal Developer Portal | 99.5% uptime during business hours |
| GitOps Controller | 99.9% of sync operations complete within 5 minutes of a Git change |

### Error Budget Policy

A written **error budget policy** specifies what the team will do at each burn rate:

```yaml
# Example error budget policy (pseudo-YAML for illustration)
errorBudgetPolicy:
  service: platform-kubernetes-api
  slo: "99.9% availability"
  thresholds:
    - burnRate: 1x       # consuming budget at the expected rate
      action: normal operations
    - burnRate: 5x       # consuming 5× faster than expected
      action: investigate alert; defer low-priority work
    - burnRate: 10x      # fast burn — potential SEV-2
      action: page on-call; declare incident
    - budgetRemaining: 0%
      action: freeze all non-reliability deployments; post-mortem required
```

---

## Incident Management Tooling

### PagerDuty

- Industry-standard incident alerting and on-call management platform
- **Escalation policies**: define the chain of who gets paged if no one acknowledges
- **Services**: map monitoring alerts to owning teams
- **Schedules**: manage on-call rotations
- **Incident response**: create, update, and resolve incidents; add responders
- **Analytics**: MTTA, MTTR, alert volume trends

### OpsGenie (Atlassian)

- Alternative to PagerDuty with similar core features
- Deeper integration with Jira for tracking action items
- Heartbeat monitoring: detect when a scheduled job stops sending a check-in
- **Routing rules**: complex alert routing logic based on content, time, or team

### Comparison

| Feature | PagerDuty | OpsGenie |
|---------|-----------|----------|
| On-call scheduling | Yes | Yes |
| Escalation policies | Yes | Yes |
| Alert grouping | Yes | Yes |
| Jira integration | Good | Native (Atlassian) |
| StatusPage integration | Native | Via integration |
| Pricing model | Per user | Per user |

---

## ChatOps for Incident Management

**ChatOps** means using a chat platform (typically Slack) as the central command surface for incident management. Benefits:

- **Shared context**: all responders see the same information stream in real time
- **Audit trail**: the incident channel is a timestamped log of all decisions and actions
- **Automation**: bots can create incidents, post status updates, and run runbook commands directly from chat
- **Reduced coordination overhead**: no need to context-switch to phone calls for status

### Typical Slack-based Incident Workflow

```
1. Alert fires in PagerDuty
2. PagerDuty bot creates #incident-2026-0513-argocd-sync channel
3. On-call is auto-invited; additional responders can be @mentioned
4. Bot posts initial context: alert name, link to dashboard, link to runbook
5. IC posts: "SEV-2 declared. I am IC. @alice please investigate ArgoCD logs."
6. All commands and findings posted in channel:
     /argocd sync list --stuck
     kubectl get events -n argocd | tail -20
7. Mitigation found and applied: posted in channel with timestamp
8. IC posts resolution: "Resolved at 15:02. Post-mortem scheduled for tomorrow."
9. Channel archived; link added to incident ticket
```

### Slack Bot Integrations

- **PagerDuty for Slack**: acknowledge, resolve, escalate incidents from Slack
- **OpsGenie for Slack**: similar functionality
- **Opbot / Dispatch**: incident management bots with runbook command execution
- **Statuspage**: automatically post incident updates to a public status page from Slack

---

## Key Exam Takeaways

- The incident lifecycle has five phases: Detection → Triage → Mitigation → Resolution → Post-mortem
- **SEV levels** determine urgency and escalation, not just priority in a queue
- The **Incident Commander** coordinates; they do not necessarily fix the problem
- **Runbooks** are linked from alerts and stored in version control
- **Blameless post-mortems** focus on systemic root causes and produce tracked action items
- **Error budget depletion** is an incident trigger, not just a metric to observe
- **ChatOps** provides audit trail, shared context, and automation for incident response
- Platform-specific SLOs cover the CI system, GitOps controller, registry, and IDP — not just user-facing services
