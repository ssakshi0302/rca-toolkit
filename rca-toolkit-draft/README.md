# RCA Toolkit

**Purpose**: Analyze incident RCAs to identify automation opportunities and reduce Time to Detect (TTD), Time to Diagnose (TTX), and Time to Remediate (TTR).

**Version**: 1.0  
**Status**: Ready for testing

---

## Quick Links

**For Executives**: See `EXECUTIVE-SUMMARY.md` - **1-page summary** showing 71% incident time reduction from 6 Temporal incidents

**For Detailed Results**: See `TEMPORAL-RESULTS-EXECUTIVE-SUMMARY.md` - Full 4-page analysis

**For Navigation**: See `EXECUTIVE-NAVIGATION.md` - Guide by role (execs, EMs, engineers)

**For Complete Example**: See `examples/temporal/EXAMPLE-OVERVIEW.md` - End-to-end workflow with real incidents

---

## What It Does

**Analyzes incident RCAs** from Google Docs to:
1. **Identify where time is spent** - Extract TTD, TTX, TTR from incident timeline
2. **Find automation opportunities** - Detect gaps in detection, diagnosis, remediation
3. **Quantify impact** - Calculate time savings from automation
4. **Generate runbooks** - Create deterministic runbooks for recurring patterns (≥2 incidents)

**Goal**: Reduce manual incident response time through data-driven automation identification.

---

## Quick Start

### 1. Install (2 min)
```bash
git clone git.soma.salesforce.com/orcaas/rca-toolkit.git
cd rca-toolkit
```

### 2. Create Team Config (3 min)
```bash
mkdir -p .claude/config
cp templates/config/team-config-example.yaml .claude/config/myteam-config.yaml
# Edit config: add your services, metrics catalog, query patterns
```

### 3. Analyze First RCA (2 min)
```bash
/rca-analyzer https://docs.google.com/document/d/YOUR_RCA_DOC
```

**Output**:
```
✅ RCA Analysis Complete
├─ Environment: prod (HIGH priority)
├─ Service: myservice-api
├─ Root Cause: database_saturation
├─ TTD: 2h 15m (gap: no DB CPU alert)
├─ TTX: 1h 30m (gap: no runbook for DB saturation)
├─ TTR: 4h 30m (gap: manual restart process)
└─ Automation Opportunity: Add alert + runbook → saves 2h TTD, 1h TTX
```

---

## Key Features

### Time Analysis
- **TTD (Time to Detect)**: How long until incident detected
- **TTX (Time to Diagnose)**: How long to identify root cause
- **TTR (Time to Remediate)**: How long to resolve

**Identifies gaps**: What alert/metric/runbook would have reduced each phase

### Automation Opportunities

**Detection**:
- Missing alerts identified
- Metric gaps discovered
- Expected TTD reduction calculated

**Diagnosis** (Causation):
- Runbook patterns extracted
- Cross-service correlation logic
- Expected TTX reduction calculated

**Remediation**:
- Manual steps identified
- Automation safety assessed
- Expected TTR reduction calculated

### Pattern Detection
- Analyzes ≥2 RCAs in batch mode
- Identifies recurring patterns (same service + symptom + root cause)
- Generates deterministic runbooks for patterns

### Time Savings Calculation
- Manual effort (hours per incident)
- With automation (reduced hours per incident)
- Time saved per incident
- Annual projection (if recurring)

---

## Example Output

### Single RCA Analysis
**File**: `research/past rca/rca-analysis-1.md`

```markdown
## RCA Analysis #1

**Incident**: Database CPU Saturation (prod1)
**Service**: temporalhistory
**Root Cause**: Workload spike without capacity planning

### Timeline
- Incident Start: 2025-09-06 08:00 UTC
- Detection: 08:29 UTC (TTD: 29 minutes)
- Diagnosis Complete: 08:58 UTC (TTX: 29 minutes)
- Resolution: 14:44 UTC (TTR: 6h 44m)

### Gaps Identified

**Detection Gap** (TTD: 29 min):
- **Why missed**: No memory pressure alerting
- **Fix**: Add alert for memory >70% sustained for 10+ min
- **Expected reduction**: 29 min → 2 min (27 min saved)

**Diagnosis Gap** (TTX: 29 min):
- **Why slow**: No capacity planning, manual log analysis
- **Fix**: Runbook for capacity exhaustion + HPA configuration
- **Expected reduction**: 29 min → 10 min (19 min saved)

**Remediation Gap** (TTR: 6h 44m):
- **Why slow**: Manual scaling, no HPA/VPA
- **Fix**: Automated resource scaling (HPA + alerts)
- **Expected reduction**: 6h 44m → 30 min (6h 14m saved)

### Automation Opportunity
- **Detection**: Add memory pressure + OOMKilled alerts
- **Diagnosis**: Create capacity exhaustion runbook
- **Remediation**: Implement HPA + automated scaling
- **Total savings per incident**: ~7 hours
- **Annual projection** (if recurring monthly): $168K-336K/year
```

### Batch Analysis with Runbook
**Command**: `/rca-analyzer --batch <url1>, <url2> --generate-runbook`

**Output**:
```
✅ Batch Analysis Complete
├─ RCAs Analyzed: 6
├─ Patterns Identified: 2 recurring
│  ├─ temporalhistory-capacity_exhaustion (2 occurrences)
│  └─ temporalfrontend-mesh_routing_failure (2 occurrences)
├─ Runbooks Generated: 2
│  ├─ runbooks/diagnosis/temporalhistory-capacity-exhaustion.md
│  └─ runbooks/diagnosis/temporalfrontend-mesh-routing-failure.md
├─ Average TTD: 16.5 hours (range: 29 min - 10 days)
├─ Average TTR: 33.7 hours (range: 30 min - 10 days)
└─ Estimated impact: 1,240 hours/year saved (74-90% time reduction)
```

---

## Use Cases

### 1. Post-Incident Analysis
**When**: After incident RCA is written  
**Goal**: Identify what automation would prevent recurrence

```bash
/rca-analyzer https://docs.google.com/.../rca-doc
```

**Output**: Gaps analysis + automation opportunities

---

### 2. Quarterly Automation Planning
**When**: Planning automation roadmap  
**Goal**: Find highest-impact automation opportunities

```bash
/rca-analyzer --batch <last-quarter-rcas>
```

**Output**: Recurring patterns + time savings estimates + runbook templates

---

### 3. Runbook Generation
**When**: ≥2 incidents with same pattern  
**Goal**: Create deterministic runbooks

```bash
/rca-analyzer --batch <rcas> --generate-runbook
```

**Output**: Step-by-step runbooks with rollback plans

---

## Documentation

- **Quick Start**: `docs/quick-start.md` (5-10 minutes)
- **Team Onboarding**: `docs/team-onboarding.md` (1-2 hours)
- **Runbook Spec**: `docs/runbook-spec.md` (full specification)
- **Skill README**: `skills/rca-analyzer/README.md` (usage details)

---

## Requirements

- Claude Code with MCP access
- Google Workspace MCP (for reading Google Docs)
- Team config file (points to your metrics/queries/architecture)

---

## Team Config

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

environments:
  HIGH: [prod]
  LOW: [dev]
```

See `templates/config/team-config-example.yaml` for full example.

---

## Architecture

**Team-agnostic design**:
- Core skill (<500 lines) works for any distributed system
- Config-driven (teams provide their knowledge files)
- Generic runbook templates (teams can customize)

**Modular**:
- Single RCA analysis
- Batch mode (parallel processing)
- Pattern detection (deterministic)
- Runbook generation (opt-in)

---

## Output Files

```
your-project/
├── research/past rca/
│   ├── rca-analysis-1.md        # Individual RCA analyses
│   ├── rca-analysis-2.md
│   └── batch-synthesis.md       # Aggregate analysis
└── runbooks/
    ├── diagnosis/
    │   └── service-pattern.md   # Generated runbooks
    └── remediation/
        └── action-pattern.md
```

---

## Examples from Temporal Team

**6 RCAs analyzed** (prod incidents):
- 2 recurring patterns identified → runbooks generated
- 10 missing alerts identified → detection improvements
- 4 runbook patterns for diagnosis
- 3 automation candidates for remediation

**Automation opportunities found**:
- 10 missing alerts (detection)
- 4 runbook patterns (diagnosis)
- 3 automation candidates (remediation)

---

## Contributing

See `CONTRIBUTING.md` for:
- How to add runbook patterns
- How to customize templates
- How to share learnings with other teams

---

## Repository Structure

```
rca-toolkit/
├── README.md                    # This file
├── skills/rca-analyzer/         # Core skill (<500 lines)
├── templates/
│   ├── runbook/                 # Diagnosis/remediation templates
│   └── config/                  # Team config schema/examples
└── docs/                        # Documentation
```

---

## License & Ownership

**Owner**: Sakshi Mehrotra (OrcaaS)  
**Maintainers**: OrcaaS team  
**Access**: Private, org-level (all Salesforce teams)  
**Repository**: `git.soma.salesforce.com/orcaas/rca-toolkit`

---

## Getting Help

- **Issues**: Create issue in this repository
- **Questions**: Slack #rca-automation (internal)
- **Owner**: Sakshi Mehrotra

---

**Get Started**: See `docs/quick-start.md` for 5-minute setup
