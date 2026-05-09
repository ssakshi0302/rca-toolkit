# Post-Incident Analysis Framework

**Operational intelligence extraction from historical incidents**

---

## Overview

Historical incidents contain recurring operational patterns. This framework analyzes incidents and RCAs to identify:

- **Observability gaps** - missing metrics, dashboards, signal visibility
- **Alerting gaps** - signals exist but alerts not configured
- **Runbook gaps** - repeated diagnosis steps without documentation
- **Recurring patterns** - same failures across multiple incidents
- **Automation opportunities** - manual processes suitable for automation

**Core principle**: Understand operational foundation before scaling automation. AI/agentic systems depend on signal quality, observability maturity, and runbook coverage.

---

## Operational Flow

```
Incident / RCA
      ↓
Pattern Extraction
      ↓
Gap Identification
(metrics / alerts / runbooks)
      ↓
Operational Insights
      ↓
Automation Opportunities
```

**Framework observes** → **Engineers prioritize** → **Teams execute**

---

## What This Framework Does

### 1. Extracts Operational Patterns
- Recurring incidents (same service + symptom + root cause)
- Common failure modes across services
- Repeated diagnosis workflows
- Manual correlation patterns

### 2. Identifies Observable Gaps
- Missing alerts (signal exists, alert missing)
- Observability blind spots (metric/dashboard missing)
- Runbook opportunities (repeated steps undocumented)
- Process bottlenecks (approval delays, manual workflows)

### 3. Documents Time Breakdown
- Detection delays (incident start → detection)
- Diagnosis time (detection → root cause)
- Remediation time (root cause → resolution)
- Where time is lost in each phase

### 4. Generates Operational Intelligence
- Pattern frequency and severity
- Runbook generation candidates (≥2 occurrences)
- Automation readiness assessment
- CAR prioritization by recurrence risk

---

## Key Learnings (Temporal Team - 6 RCAs)

### Observable Signals Already Exist
- 100% of incidents had observable signals in existing systems
- Delays caused by missing alerting logic, not missing data
- Most metrics existed but weren't configured to alert

### Detection Delay is the Bottleneck
- 75% of incidents had detection time >10 hours
- Average detection: 16.5 hours
- Once detected, diagnosis was relatively fast (1-14 hours)
- **Insight**: Detection has higher leverage than diagnosis optimization

### Missing Alerts Create Operational Load
- 10 specific missing alerts identified across 6 incidents
- Examples: DB CPU >80%, memory pressure >70%, OOMKilled events, queue drain rate
- Each missing alert added hours to detection time
- **Insight**: Missing alerts compound over time

### Patterns Enable Operational Intelligence
- 2 recurring patterns found in 6 RCAs (33% recurrence rate)
- Same service + symptom + root cause = runbook opportunity
- Patterns convert reactive knowledge into proactive tooling
- **Insight**: RCA corpus is operational intelligence, not archive

### CARs Without Prioritization Lead to Recurrence
- RCA #2 was identical to incident 4 months prior
- CAR existed but wasn't prioritized by recurrence risk
- **Insight**: CARs need cross-RCA prioritization

---

## Phased Approach

### Phase 1: Observational Insights (Current)
**Goal**: Understand operational gaps through incident analysis

**Activities**:
- Analyze historical incidents
- Identify missing alerts, metrics, runbooks
- Document recurring patterns
- Quantify detection and remediation delays

**Output**: Operational intelligence baseline

**Status**: ✅ Complete for Temporal team (6 RCAs analyzed)

---

### Phase 2: Guided Triage Assistance (60-90 days)
**Goal**: Reduce manual correlation and diagnosis time

**Activities**:
- Implement missing alerts
- Create runbooks for recurring patterns
- Integrate RCA corpus for historical pattern matching
- Build signal correlation chains
- Implement CAR prioritization process

**Output**: Reduced diagnosis time through structured guidance

---

### Phase 3: Automated Reasoning & Remediation (6-12 months)
**Goal**: Enable safe, context-aware automation

**Prerequisites**:
- Comprehensive alert coverage
- High-quality runbook library
- Historical pattern database
- Signal correlation logic
- CAR prioritization process

**Activities**:
- Automated signal correlation and causation
- AI-assisted triage and diagnosis (with guardrails)
- Guided remediation workflows (human-in-loop)
- Pilot low-risk automated remediation

**Output**: Operational intelligence platform with automation capabilities

---

## Initial Findings (Temporal Team)

**Context**: 6 production RCAs analyzed (July 2025 - April 2026)

### Detection Gaps
- 10 missing alerts identified (DB CPU, memory pressure, OOMKilled, queue depth)
- Detection delays: 29 minutes to 3 days (average: 16.5 hours)
- Preliminary analysis indicates substantial opportunity to reduce detection time

### Diagnosis Patterns
- 2 recurring patterns identified (capacity exhaustion, mesh routing failure)
- 4 runbook candidates generated
- Manual correlation took 1-14 hours per incident
- Opportunity to reduce diagnosis time through pattern matching

### Remediation Bottlenecks
- 3 automation candidates identified (HPA implementation, config validation, graceful timeout)
- Manual approval processes added 1-4 days
- 1 incident recurred 4 months later (CAR existed but not prioritized)

### Operational Learning
- 100% of incidents had observable signals before detection
- Detection delay was consistently the largest bottleneck
- Patterns emerged from just 6 RCAs

---

## Quick Start

### 1. Clone Repository
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

**Output**: Structured analysis identifying detection gaps, diagnosis delays, automation opportunities

---

## Use Cases

### Post-Incident Operational Review
**When**: After RCA is written  
**Goal**: Extract operational learnings beyond incident resolution

**Questions answered**:
- What signals existed but didn't alert?
- What knowledge would have accelerated diagnosis?
- Is this pattern recurring?
- What CARs should be prioritized?

---

### Quarterly Automation Planning
**When**: Planning operational improvements  
**Goal**: Data-driven prioritization of automation work

**Questions answered**:
- Which patterns recur most frequently?
- Where are the biggest operational gaps?
- What automation has highest impact?
- Which CARs prevent recurrence?

---

### Operational Maturity Assessment
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
├── OPERATIONAL-FINDINGS.md      # Temporal team analysis results
├── skills/rca-analyzer/         # Analysis framework
├── templates/
│   ├── runbook/                 # Diagnosis/remediation templates
│   └── config/                  # Team configuration
├── docs/
│   ├── quick-start.md           # Setup guide
│   └── team-onboarding.md       # Comprehensive onboarding
└── examples/
    └── temporal/                # Complete Temporal pilot (6 RCAs)
        ├── EXAMPLE-OVERVIEW.md  # Workflow walkthrough
        ├── rca-analyses/        # Individual analyses + synthesis
        └── runbooks/            # Generated runbooks
```

---

## Suggested Reading Paths

**New to framework**: Start with this README, then review `examples/temporal/EXAMPLE-OVERVIEW.md`

**Setting up for your team**: Follow `docs/quick-start.md`, then `docs/team-onboarding.md`

**Reviewing Temporal findings**: Read `OPERATIONAL-FINDINGS.md`, then explore `examples/temporal/rca-analyses/`

**Understanding patterns**: Review `examples/temporal/runbooks/` for generated runbook examples

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

**Purpose**: Framework adapts to your services, metrics, and operational context

---

## Technical Architecture

**Design**:
- Team-agnostic (works for any distributed system)
- Config-driven (teams provide operational knowledge)
- Modular (single RCA analysis, batch processing, pattern detection)
- Deterministic (reproducible results, clear reasoning)

**Components**:
- RCA extraction (Google Docs → structured data)
- Gap analysis (detection, diagnosis, remediation)
- Pattern detection (≥2 similar incidents → runbook)
- CAR tracking (prioritization by recurrence risk)

**Size**: Core framework <500 lines

---

## Contributing

**Maintainers**: OrcaaS Temporal team  
**Repository**: `git.soma.salesforce.com/orcaas/post-incident-analysis-framework`

**Issues**: Create issue in this repository  
**Questions**: Slack #temporal-reliability

---

## Next Steps

1. Review Temporal pilot: `examples/temporal/EXAMPLE-OVERVIEW.md`
2. Review operational findings: `OPERATIONAL-FINDINGS.md`
3. Set up for your team: `docs/quick-start.md`
4. Contribute patterns back to framework

---

**Philosophy**: Build operational maturity before scaling automation. Understand the signals before building the system.

**Version**: 1.0  
**Status**: Operational - Temporal team pilot complete
