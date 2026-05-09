# Post-Incident Analysis Framework for Temporal Operations

**A structured approach to operational maturity through systematic incident analysis**

**Version**: 1.0  
**Status**: Operational - Temporal team pilot complete

---

## Overview

Temporal incidents already contain meaningful operational patterns. This framework extracts those patterns to identify operational maturity gaps before investing in specific AIOps or agentic automation platforms.

**Core principle**: AI systems are only as effective as the operational signals, observability, and runbook quality available to them.

---

## Vision

This framework analyzes historical incidents and RCAs to identify operational maturity gaps across:

**Detection**: Missing alerts, metric gaps, observable signals not monitored  
**Diagnosis**: Manual correlation, missing runbooks, signal interpretation  
**Remediation**: Manual processes, approval bottlenecks, automation candidates  
**Operational Learning**: Pattern recognition, knowledge capture, recurrence prevention

**Long-term direction**: Enable signal-aware causation and automated reasoning for Temporal operations, built on a foundation of structured operational intelligence.

---

## Why This Framework Exists

### The Problem

Before scaling automation, we need structured understanding of:
- Recurring incident patterns (where do we see the same failures?)
- Missing metrics and signals (what observable data exists but isn't monitored?)
- Alerting gaps (which signals should trigger alerts?)
- Runbook quality (what institutional knowledge exists only in people's heads?)
- Automation opportunities (what can be safely automated?)
- Remediation workflows (where are the approval bottlenecks?)

### The Approach

Rather than immediately adopting an AIOps platform, we systematically analyze past incidents to understand:
1. What operational signals already exist
2. Where detection/diagnosis delays occur
3. Which patterns recur frequently
4. What can be improved with current tooling
5. What groundwork is needed for future automation

---

## Key Learnings (Temporal Team - 6 RCAs)

### Observability Already Exists
- **100% of incidents had observable signals** in existing systems (Argus, Splunk, Grafana)
- Delays were caused by missing alerting logic, not missing data
- Most metrics existed; they simply weren't configured to alert

### Detection Delay is the Largest Bottleneck
- **75% of incidents had detection time >10 hours** (range: 29 min - 3 days)
- Average detection time: 16.5 hours
- Once detected and focused, diagnosis was relatively fast (1-14 hours)
- **Insight**: Improving detection has higher leverage than optimizing diagnosis

### Missing Alerts Create Operational Load
- **10 specific missing alerts identified** across 6 incidents
- Examples: DB CPU >80%, memory pressure >70%, OOMKilled events, queue drain rate
- Each missing alert added hours to detection time
- **Insight**: Alerts are operational debt - missing alerts compound over time

### Patterns Enable Operational Intelligence
- **2 recurring patterns found** in 6 RCAs (33% recurrence rate)
- Same service + symptom + root cause = runbook opportunity
- Patterns convert reactive knowledge into proactive tooling
- **Insight**: RCA corpus is a knowledge mine, not a document archive

### CARs Without Prioritization Lead to Recurrence
- RCA #2 was identical to an incident 4 months prior
- CAR existed but wasn't prioritized by recurrence risk
- **Insight**: CARs need cross-RCA prioritization, not per-incident tracking

---

## Phased Maturity Model

### Phase 1: Observational Insights (Current)
**Goal**: Understand operational gaps through structured RCA analysis

**Activities**:
- Analyze historical incidents (6 RCAs completed for Temporal)
- Identify missing alerts, metrics, and runbooks
- Document recurring patterns
- Quantify detection and remediation delays

**Output**: Operational intelligence baseline (what we know vs what we need)

**Status**: ✅ Complete for Temporal team

---

### Phase 2: Guided Triage Assistance (Next 60-90 days)
**Goal**: Reduce manual correlation and diagnosis time

**Activities**:
- Implement missing alerts (10 identified for Temporal)
- Create runbooks for recurring patterns (2 patterns → 4 runbooks)
- Integrate RCA corpus for historical pattern matching
- Build signal correlation chains (automated causation)
- Implement CAR prioritization process

**Output**: Reduced diagnosis time through structured guidance and historical knowledge

**Status**: Design complete, implementation planned

---

### Phase 3: Automated Reasoning & Remediation (6-12 months)
**Goal**: Enable safe, context-aware automation

**Prerequisites** (from Phase 1 & 2):
- Comprehensive alert coverage
- High-quality runbook library
- Historical pattern database
- Signal correlation logic
- CAR prioritization process

**Activities**:
- Automated signal correlation and causation
- AI-assisted triage and diagnosis (with guardrails)
- Guided remediation workflows (human-in-loop)
- Pilot low-risk automated remediation (30-40% of cases)

**Output**: Operational intelligence platform with automation capabilities

**Status**: Groundwork in progress

---

## What This Framework Does

### Analyzes Post-Incident RCAs to Extract:

**1. Time Breakdown** (TTD, TTX, TTR)
- How long until incident detected
- How long to identify root cause
- How long to resolve
- Where time is lost in each phase

**2. Operational Gaps**
- **Detection**: What alert/metric was missing
- **Diagnosis**: What runbook/knowledge would have helped
- **Remediation**: What automation would have accelerated resolution

**3. Pattern Recognition**
- Recurring incidents (same service + symptom + root cause)
- Frequency and severity scoring
- Runbook generation candidates

**4. Automation Opportunities**
- Safe automation candidates (low-risk, high-frequency)
- Approval workflow bottlenecks
- Manual process elimination candidates

**5. Operational Intelligence**
- Observable signals not monitored
- CAR prioritization by recurrence risk
- Historical pattern matching for future incidents

---

## Preliminary Results (Temporal Team)

**Context**: 6 production RCAs analyzed (July-April 2026)

### Detection Gaps
- 10 missing alerts identified (DB CPU, memory pressure, OOMKilled, queue depth, etc.)
- Detection delays: 29 minutes to 3 days (average: 16.5 hours)
- Preliminary analysis indicates significant opportunity to reduce detection time through targeted alerting

### Diagnosis Patterns
- 2 recurring patterns identified (capacity exhaustion, mesh routing failure)
- 4 runbook candidates generated
- Manual correlation took 1-14 hours per incident
- Opportunity to reduce diagnosis time through historical pattern matching

### Remediation Bottlenecks
- 3 automation candidates identified (HPA implementation, config validation, graceful timeout)
- Manual approval processes added 1-4 days
- CAR recurrence: 1 incident recurred 4 months later (CAR existed but not prioritized)

### Operational Learning
- 100% of incidents had observable signals before detection
- Detection delay was consistently the largest bottleneck
- Patterns emerged from just 6 RCAs - corpus analysis scales with data

---

## Quick Start

### 1. Install
```bash
git clone git.soma.salesforce.com/orcaas/post-incident-analysis-framework.git
cd post-incident-analysis-framework
```

### 2. Configure Your Team
```bash
mkdir -p .claude/config
cp templates/config/team-config-example.yaml .claude/config/myteam-config.yaml
# Edit: add your services, metrics catalog, query patterns
```

### 3. Analyze First RCA
```bash
/rca-analyzer https://docs.google.com/document/d/YOUR_RCA_DOC
```

**Output**: Structured analysis identifying detection gaps, diagnosis delays, and automation opportunities

---

## Use Cases

### 1. Post-Incident Operational Review
**When**: After RCA is written  
**Goal**: Extract operational learnings beyond incident resolution

**Questions answered**:
- What signals existed but didn't alert?
- What knowledge would have accelerated diagnosis?
- Is this pattern recurring?
- What CARs should be prioritized?

---

### 2. Quarterly Automation Planning
**When**: Planning operational improvements  
**Goal**: Data-driven prioritization of automation work

**Questions answered**:
- Which patterns recur most frequently?
- What automation has highest impact?
- Where are the biggest operational gaps?
- Which CARs prevent recurrence?

---

### 3. Operational Maturity Assessment
**When**: Evaluating team/service reliability  
**Goal**: Baseline operational capabilities before platform investment

**Questions answered**:
- What percentage of incidents have detection gaps?
- How mature is our runbook library?
- Are CARs tracked and prioritized?
- What operational intelligence exists in RCA corpus?

---

## Repository Structure

```
post-incident-analysis-framework/
├── README.md                    # This file
├── EXECUTIVE-SUMMARY.md         # 1-page leadership overview
├── skills/rca-analyzer/         # Analysis framework (<500 lines)
├── templates/
│   ├── runbook/                 # Diagnosis/remediation templates
│   └── config/                  # Team configuration
├── docs/
│   ├── quick-start.md           # 5-10 minute setup
│   ├── team-onboarding.md       # Comprehensive guide
│   └── executive-summary-spec.md # Leadership communication format
└── examples/
    └── temporal/                # Complete Temporal pilot (6 RCAs)
        ├── EXAMPLE-OVERVIEW.md  # Full workflow walkthrough
        ├── rca-analyses/        # Individual analyses + synthesis
        ├── runbooks/            # Generated runbooks (2 patterns)
        └── knowledge/           # Metrics, queries, architecture
```

---

## Documentation

**For Executives**: `EXECUTIVE-SUMMARY.md` - Strategic overview and operational impact  
**For Engineering Managers**: `examples/temporal/EXAMPLE-OVERVIEW.md` - Complete pilot walkthrough  
**For Teams**: `docs/quick-start.md` - Hands-on setup and first analysis  
**For Detailed Spec**: `docs/team-onboarding.md` - Comprehensive onboarding

---

## Team Configuration

**Location**: `.claude/config/<team>-config.yaml`

**Minimum required**:
```yaml
team:
  name: MyTeam
  service_prefix: myservice

services:
  - name: myservice-api

knowledge:
  metrics_catalog: path/to/metrics.md
  query_patterns: path/to/queries.md
  service_architecture: path/to/architecture.md

environments:
  HIGH: [prod, esvc]
  LOW: [dev]
```

**Purpose**: Team-agnostic framework adapts to your services, metrics, and operational context

---

## Technical Architecture

**Design Principles**:
- **Team-agnostic**: Works for any distributed system
- **Config-driven**: Teams provide their operational knowledge
- **Modular**: Single RCA analysis, batch processing, pattern detection
- **Deterministic**: Reproducible results, clear reasoning

**Components**:
- RCA extraction (Google Docs → structured data)
- Gap analysis (detection, diagnosis, remediation)
- Pattern detection (≥2 similar incidents → runbook)
- CAR tracking (prioritization by recurrence risk)

**Size**: Core framework <500 lines (deliberately lightweight)

---

## License & Ownership

**Owner**: Sakshi Mehrotra (OrcaaS)  
**Maintainers**: OrcaaS Temporal team  
**Access**: Private, org-level (all Salesforce teams)  
**Repository**: `git.soma.salesforce.com/orcaas/post-incident-analysis-framework`

---

## Getting Help

**Issues**: Create issue in this repository  
**Questions**: Slack #temporal-reliability (or your team channel)  
**Owner**: Sakshi Mehrotra (OrcaaS)

---

## Next Steps

1. **Review pilot results**: `examples/temporal/EXAMPLE-OVERVIEW.md`
2. **Understand framework**: `EXECUTIVE-SUMMARY.md`
3. **Start analyzing**: `docs/quick-start.md`
4. **Share learnings**: Contribute patterns back to framework

---

**Philosophy**: Build operational maturity before scaling automation. Understand the signals before building the system.
