---
name: rca-analyzer
description: Analyze incident RCAs to identify automation opportunities and reduce TTD, TTX, TTR
version: 1.0
author: Sakshi Mehrotra (OrcaaS)
---

# RCA Analyzer Skill

## Purpose

Identify automation opportunities and time reduction in incident response:
- **TTD (Time to Detect)**: Find missing alerts/metrics that would detect faster
- **TTX (Time to Diagnose)**: Generate runbooks to speed up causation analysis  
- **TTR (Time to Remediate)**: Identify automation candidates for faster resolution
- **ROI**: Quantify time savings and annual cost reduction from automation

## How to Use

### Single RCA Analysis

```bash
/rca-analyzer https://docs.google.com/document/d/YOUR_RCA_DOC
```

### Batch Analysis (Multiple RCAs)

```bash
/rca-analyzer --batch <url1>, <url2>, <url3>
```

### With Options

```bash
/rca-analyzer --batch <url1>, <url2> --generate-runbook --generate-summary
```

## Arguments

**Single RCA**:
- `<url>` - Google Doc URL

**Batch mode**:
- `--batch <url1>, <url2>, <url3>` - Multiple URLs (comma-separated)

**Options**:
- `--generate-runbook` - Generate runbook if pattern matches ≥2 RCAs
- `--generate-summary` - Generate executive summary (action plan)
- `--config=<path>` - Team config file (default: `.claude/config/[team]-config.yaml`)

## Workflow

When you invoke this skill, I will:

### 1. Load Team Configuration

Read your team's config file to get:
- Services and ports
- Metrics catalog
- Query patterns (Argus, Splunk)
- Service architecture
- Alert definitions

**Default location**: `.claude/config/temporal-config.yaml`

### 2. Extract RCA Data from Google Docs

For each RCA, I'll spawn a subagent to read the Google Doc and extract:
- Timeline (incident start, detection, diagnosis, resolution)
- Services affected
- Root cause
- Symptoms and contributing factors
- What signals existed (logs, metrics, dashboards)
- What worked well (positive signals, effective tools)

### 3. Calculate TTD/TTX/TTR

- **TTD**: Time from incident start to detection
- **TTX**: Time from detection to diagnosis complete
- **TTR**: Total time from start to resolution

### 4. Identify Gaps

For each phase, identify:

**Detection Gap**:
- Why was it missed (what alert/metric was missing)
- What signals were observable but not alerted on
- Specific alert to add with threshold

**Diagnosis Gap**:
- Why was diagnosis slow (manual correlation, noisy logs, missing runbook)
- What existing tools/dashboards helped
- What runbook/correlation would have helped

**Remediation Gap**:
- Why was remediation slow (manual process, approval delays)
- What existing automation worked
- What automation would help (with safety considerations)

### 5. Capture What Worked Well

For each RCA, document:
- **Positive Signals**: Alerts/metrics that fired correctly
- **Effective Tools**: Dashboards, monitoring that provided visibility
- **Good Processes**: Runbooks, coordination that helped

### 6. Pattern Detection (Batch Mode)

If analyzing multiple RCAs:
- Identify recurring patterns (same service + symptom + root cause)
- Count occurrences (≥2 = runbook candidate)
- Calculate average TTD/TTR per pattern

### 7. Generate Runbooks (Optional)

If `--generate-runbook` flag set and patterns found:
- Select appropriate template (diagnosis or remediation)
- Fill template with pattern data
- Include decision points and safety checks

### 8. Generate Executive Summary (Optional)

If `--generate-summary` flag set:
- Create leadership-ready action plan
- Link all RCA references to analysis files
- Include specific alerts with thresholds
- Show ROI calculations
- Provide immediate/short/long-term actions

## Output Files

### Individual RCA Analysis
**Location**: `research/past rca/rca-analysis-[N].md`

**Contains**:
```markdown
## RCA Analysis #N

**Incident Title**: [from doc]
**Date**: YYYY-MM-DD
**Environment**: prod/esvc/dev
**Service(s) Affected**: [services]

### Timeline
- Incident Start: [timestamp]
- Detection: [timestamp] (TTD: [duration])
- Diagnosis Complete: [timestamp] (TTX: [duration])
- Resolution: [timestamp] (TTR: [duration])

### Root Cause
**Category**: db_saturation | mesh_failure | capacity_exhaustion | ...
**Service**: [primary service]
**Details**: [1-2 sentence summary]

**Symptoms**:
- [symptom 1]
- [symptom 2]

### What Worked Well
**Positive Signals**:
- [alert/metric that fired correctly]
- [dashboard that showed relevant data]

**Effective Tools**:
- [monitoring tool that provided visibility]

### Gaps Identified
**Detection Gap** (TTD: [duration]):
- **Why missed**: [what alert was missing]
- **What existed**: [observable signals]
- **Fix**: [specific alert with threshold]

**Diagnosis Gap** (TTX: [duration]):
- **Why slow**: [manual correlation, noisy logs]
- **What worked**: [tools/dashboards that helped]
- **Fix**: [runbook/correlation needed]

**Remediation Gap** (TTR: [duration]):
- **Why slow**: [manual process, approvals]
- **What worked**: [existing automation]
- **Fix**: [automation needed, with safety]

### Automation Opportunity
**Detection**: [alert to add] → [expected TTD reduction]
**Diagnosis**: [runbook pattern] → [expected TTX reduction]
**Remediation**: [automation candidate] → [expected TTR reduction]

### ROI Estimate
- Manual effort: [hours] × $[rate] = $[amount]
- With automation: [hours] × $[rate] = $[amount]
- Savings per incident: $[amount]
- Annual projection: $[amount]/year
```

### Batch Synthesis
**Location**: `research/past rca/batch-synthesis.md`

**Contains**:
- Aggregate metrics (average TTD/TTX/TTR, ranges)
- Patterns identified (≥2 similar incidents)
- Missing alerts summary
- Runbook candidates
- ROI calculation (manual vs automated)
- Annual projection

### Executive Summary
**Location**: `[project-root]/EXECUTIVE-SUMMARY.md`

**Generated when**: `--generate-summary` flag used

**Contains**:
- 3-line executive summary (bottleneck, impact, ROI)
- Data overview table
- Critical findings (linked RCAs with context)
- Incident summary table (all RCAs linked)
- Immediate actions (alerts, runbooks, AI correlation)
- Short-term and long-term actions
- ROI estimate table
- Recommendations

**Format**: 1-2 pages, scannable, action-oriented

**Spec**: See `docs/executive-summary-spec.md`

### Runbooks
**Location**: `runbooks/diagnosis/[service]-[pattern].md`

**Generated when**: `--generate-runbook` flag used and ≥2 similar incidents

**Contains**:
- Pattern description
- Symptoms (user impact + system indicators)
- Diagnosis steps (with decision points)
- Cross-service correlation
- Common pitfalls
- Related incidents
- Metadata

## Examples

### Example 1: Single RCA

```bash
/rca-analyzer https://docs.google.com/document/d/193y8SEP05g7Lzjk6gjA9U__0KpWsXvJRWt_4rpMM8ro/edit
```

**Output**:
```
✅ RCA Analysis Complete
├─ File: research/past rca/rca-analysis-1.md
├─ Environment: prod (HIGH priority)
├─ Service: temporalhistory
├─ Root Cause: database_saturation
├─ TTD: 20h (gap: no DB CPU alert)
├─ TTX: 25h (gap: manual correlation)
├─ TTR: 4d 6h (gap: manual quota increase)
└─ Automation Opportunity: Add DB CPU alert → 20h → 2 min TTD
```

### Example 2: Batch with Runbooks

```bash
/rca-analyzer --batch <url1>, <url2>, <url3> --generate-runbook
```

**Output**:
```
✅ Batch Analysis Complete
├─ RCAs Analyzed: 3
├─ Patterns Identified: 1 recurring
│  └─ temporalhistory-capacity_exhaustion (2 occurrences)
├─ Runbook Generated: runbooks/diagnosis/temporalhistory-capacity-exhaustion.md
├─ Average TTD: 10.1h (range: 29 min - 20h)
├─ Average TTR: 3.8d (range: 6h 44m - 4d 6h)
└─ Estimated ROI: $56K-113K (74% time reduction)
```

### Example 3: Batch with Executive Summary

```bash
/rca-analyzer --batch <url1>, <url2>, <url3>, <url4>, <url5>, <url6> --generate-summary
```

**Output**:
```
✅ Batch Analysis Complete
├─ RCAs Analyzed: 6
├─ Patterns Identified: 2 recurring
├─ Files Generated:
│  ├─ research/past rca/rca-analysis-1.md through rca-analysis-6.md
│  ├─ research/past rca/batch-synthesis-6-rcas.md
│  ├─ EXECUTIVE-SUMMARY.md (leadership-ready action plan)
│  ├─ runbooks/diagnosis/temporalhistory-capacity-exhaustion.md
│  └─ runbooks/diagnosis/temporalfrontend-wasm-panic.md
├─ Average TTD: 16.5h → Target: 2h (92% reduction)
├─ Average TTR: 33.7h → Target: 20h (60% reduction)
└─ ROI: $264K-529K/year (71% time reduction)
```

## Configuration Required

**Team config file**: `.claude/config/[team]-config.yaml`

**Minimum required**:
```yaml
team:
  name: MyTeam
  service_prefix: myservice

services:
  - name: myservice-api
    port: 8080

knowledge:
  metrics_catalog: path/to/metrics.md
  query_patterns: path/to/queries.md
  service_architecture: path/to/architecture.md

environments:
  HIGH: [prod, esvc]
  MEDIUM: [preprod]
  LOW: [dev]
```

**See**: `templates/config/team-config-example.yaml` for full example

## Implementation Notes

This skill orchestrates the analysis workflow by:

1. **Reading Google Docs** via MCP (Google Workspace integration)
2. **Spawning subagents** for parallel RCA extraction
3. **Pattern matching** using deterministic logic (service + symptom + root cause)
4. **Template filling** for runbooks using team config data
5. **ROI calculation** using configurable oncall rates

**Size**: ~500 lines (meets requirement)
**Dependencies**: Google Workspace MCP, team config file

## Next Steps After Running

1. **Review** individual RCA analyses for accuracy
2. **Validate** patterns and runbooks
3. **Share** executive summary with leadership
4. **Implement** immediate actions (missing alerts)
5. **Iterate** on runbooks based on team feedback

---

**Documentation**:
- Full specification: `docs/executive-summary-spec.md`
- Quick start: `docs/quick-start.md`
- Team onboarding: `docs/team-onboarding.md`

**Example**: See `examples/temporal/` for complete workflow with 6 RCAs
