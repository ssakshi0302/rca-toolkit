# RCA Analysis Skill & Runbook Specification

**Version**: 1.0  
**Date**: 2026-05-09  
**Purpose**: Define reusable, team-agnostic RCA analysis skill with opt-in runbook generation

---

## Skill Specification

### Design Principles

1. **<500 lines**: Core skill logic under 500 lines via modular design
2. **Team-agnostic**: Works for any distributed system (Temporal, Kafka, etc.)
3. **Config-driven**: Team-specific knowledge loaded via config files
4. **Opt-in runbook**: Ask user if runbook generation is needed
5. **Parallel processing**: Batch mode with concurrent subagents

### Skill Interface

```yaml
name: rca-analyzer
description: Analyze incident RCAs from Google Docs, extract patterns, generate runbooks

arguments:
  url: string (required)          # Google Doc URL or --batch flag
  environment: string (optional)  # prod/esvc/dev (for prioritization)
  generate_runbook: boolean       # User choice (defaults to prompt)
  team_config: string (optional)  # Path to team-specific config
```

**Examples**:
```bash
# Single RCA
/rca-analyzer https://docs.google.com/document/d/...

# Batch mode
/rca-analyzer --batch https://docs.google.com/..., https://docs.google.com/...

# With runbook generation
/rca-analyzer https://docs.google.com/... --generate-runbook

# Custom team config
/rca-analyzer https://docs.google.com/... --team-config=.claude/config/temporal-config.yaml
```

### Skill Workflow

```
1. Parse arguments (single URL or --batch)
2. If generate_runbook not specified:
   → Ask user: "Generate runbook for recurring patterns? (y/n)"
3. Load team config (if specified) or use defaults
4. For single RCA:
   → Spawn analysis subagent (Google Docs MCP)
   → Extract structured data
   → Save to research/past rca/rca-analysis-N.md
   → If runbook requested + pattern matches ≥2 RCAs:
      → Generate deterministic runbook
5. For batch:
   → Spawn N parallel subagents
   → Aggregate results
   → Generate synthesis report
   → If runbook requested:
      → Identify common patterns (≥2 occurrences)
      → Generate runbooks for each pattern
6. Output:
   → Summary (TTD/TTX/TTR, gaps, ROI)
   → File paths (RCA analysis, runbooks if generated)
```

### Modular Components

**Core skill** (<500 lines):
- Argument parsing
- Subagent orchestration
- Output aggregation
- Runbook trigger logic

**External modules** (loaded via config):
- Team-specific metric catalogs
- Log query patterns
- Service architecture knowledge
- Alert → service mapping

---

## Runbook Specification

### Design Principles

1. **Deterministic**: Same pattern → same runbook (no LLM creativity)
2. **Structural**: Fixed template with placeholders
3. **Step-by-step**: One action per step, with validation
4. **Actionable**: Commands, queries, decision points
5. **Team-agnostic template**: Placeholders filled from team config

### Runbook Template Structure

```markdown
# Runbook: <Pattern Name>

**Pattern ID**: <pattern-type>-<service>-<root-cause>
**Trigger**: <PagerDuty alert name or symptom>
**Frequency**: <N incidents in M days>
**Last Occurrence**: <YYYY-MM-DD>
**Average TTD**: <duration> | **Average TTR**: <duration>

---

## Symptoms

**User Impact**:
- <Concrete symptom 1>
- <Concrete symptom 2>

**System Indicators**:
- Alert: <alert name>
- Metric: <metric name> shows <threshold>
- Logs: <log pattern>

---

## Diagnosis Steps

### Step 1: Verify Scope
**Action**: Check if issue is isolated to <service/cluster/namespace>
**Command**:
```
<query command with placeholders>
```
**Expected Result**: <what to look for>
**Decision Point**:
- If isolated → Proceed to Step 2
- If widespread → Escalate to <team> (check <sibling service>)

### Step 2: Identify Root Cause
**Action**: <specific investigation action>
**Command**:
```
<query command>
```
**Expected Result**: <what indicates root cause>
**Decision Point**:
- If <condition A> → Root cause is <X>, proceed to Remediation A
- If <condition B> → Root cause is <Y>, proceed to Remediation B

### Step 3: Validate Impact
**Action**: Check current error rate and latency
**Command**:
```
<monitoring query>
```
**Expected Result**: Error rate >5% or latency >P99 threshold
**Decision Point**:
- If actively impacting → Proceed to immediate remediation
- If resolved → Skip to verification

---

## Remediation

### Remediation A: <Action Name>
**Trigger Condition**: <when to use this>
**Safety Check**:
- [ ] Verify <precondition 1>
- [ ] Confirm <precondition 2>

**Steps**:
1. <Action 1>
   ```
   <command>
   ```
   **Expected Output**: <what success looks like>

2. <Action 2>
   ```
   <command>
   ```
   **Wait**: <duration> for propagation

3. Verify remediation:
   ```
   <validation command>
   ```
   **Success Criteria**: <metric/log pattern>

**Rollback Plan** (if remediation fails):
```
<rollback commands>
```

### Remediation B: <Alternative Action>
(Same structure as Remediation A)

---

## Verification

**Check 1: Error Rate**
```
<query>
```
**Expected**: Error rate <1% for 5 minutes

**Check 2: Latency**
```
<query>
```
**Expected**: P99 latency back to baseline

**Check 3: User Impact**
```
<query or manual check>
```
**Expected**: No customer reports in #incidents-channel

---

## Prevention

**Immediate** (do now):
- [ ] <Action to prevent immediate recurrence>

**Short-term** (this week):
- [ ] <Monitoring improvement>
- [ ] <Alert tuning>

**Long-term** (this quarter):
- [ ] <Architectural fix>
- [ ] <Capacity planning>

---

## Related Incidents

- RCA #<N>: <title> (<date>) - <URL>
- RCA #<M>: <title> (<date>) - <URL>

---

## Metadata

**Generated**: <timestamp>
**Pattern Confidence**: <HIGH/MEDIUM/LOW> (<N incidents match>)
**Team**: <team name from config>
**Services**: <comma-separated list>
**Root Cause Category**: <category from taxonomy>
```

### Runbook Determinism Rules

**Fixed sections** (always present):
- Symptoms
- Diagnosis Steps (with decision points)
- Remediation (with rollback)
- Verification
- Prevention

**Variable sections** (pattern-dependent):
- Number of diagnosis steps (2-5)
- Number of remediation options (1-3)
- Commands/queries (filled from team config)

**Placeholders** (replaced at generation time):
- `<service>` → from RCA analysis
- `<metric name>` → from team config
- `<query command>` → from query pattern templates
- `<team>` → from team config
- `<alert name>` → from PD alert mapping

**Decision point structure** (deterministic):
```
**Decision Point**:
- If <condition from RCA pattern> → <action A>
- If <condition from RCA pattern> → <action B>
```

### Runbook Generation Logic

**When to generate**:
1. User opted in via flag or prompt
2. Pattern identified in ≥2 RCAs
3. Pattern has clear diagnosis path (not "unknown root cause")

**Pattern matching** (deterministic):
```python
pattern_key = f"{service}_{symptom}_{root_cause_category}"
# Example: "temporalhistory_capacity_exhaustion_oomkilled"
# Example: "temporalfrontend_high_errors_mesh_routing_failure"
```

**Template selection**:
- Diagnosis pattern: DB saturation, mesh failure, capacity exhaustion, archival backlog
- Remediation pattern: Rolling restart, resource scaling, config change, cache clear

**Filename convention**:
```
runbooks/
  diagnosis/
    <pattern-key>.md
  remediation/
    <pattern-key>.md
```

---

## Team Configuration Schema

```yaml
# .claude/config/temporal-config.yaml (example)
team:
  name: OrcaaS
  service_prefix: temporal
  pagerduty_service: Temporal Production

services:
  - name: temporalfrontend
    port: 7233
    metrics_scope: temporal.temporalfrontend
  - name: temporalhistory
    port: 7234
    metrics_scope: temporal.temporalhistory
  - name: temporalmatching
    port: 7235
    metrics_scope: temporal.temporalmatching

metrics:
  catalog_path: .claude/context/temporal-metrics-complete-catalog.md
  query_patterns: .claude/context/temporal-argus-patterns.md

logs:
  splunk_index: distapps
  query_patterns: .claude/context/temporal-splunk-patterns.md

architecture:
  dependency_graph: .claude/context/temporal-service-architecture.md

runbook_templates:
  diagnosis:
    db_saturation: templates/diagnosis/db-saturation.md
    mesh_failure: templates/diagnosis/mesh-failure.md
  remediation:
    rolling_restart: templates/remediation/rolling-restart.md
    resource_scaling: templates/remediation/resource-scaling.md

alert_mapping:
  temporal-frontend-availability-low:
    services: [temporalfrontend]
    common_causes: [mesh_failure, db_saturation, deployment_issue]
  temporal-db-cpu-high:
    services: [temporalhistory, temporalmatching]
    common_causes: [workload_spike, archival_load, missing_index]
```

---

## Skill Size Calculation

**Core skill logic** (~400 lines):
- Argument parsing: 50 lines
- Config loading: 50 lines
- Single RCA analysis: 100 lines
- Batch mode orchestration: 100 lines
- Runbook trigger logic: 50 lines
- Output formatting: 50 lines

**External modules** (not counted in 500-line limit):
- Team config loaders
- Template engines
- Pattern matchers
- Metric query builders

---

## Success Criteria

**Skill**:
- ✅ Analyzes single RCA in <2 minutes
- ✅ Batch processes 5+ RCAs in parallel (<5 minutes total)
- ✅ Extracts TTD/TTX/TTR accurately
- ✅ Identifies patterns across ≥2 RCAs
- ✅ Respects <500 line limit

**Runbook**:
- ✅ Deterministic (same pattern → same structure)
- ✅ Actionable (every step has command/query)
- ✅ Step-by-step (one action, validate, next)
- ✅ Rollback plans included
- ✅ Decision points explicit
- ✅ Team-agnostic (works with any config)

---

## Next Steps

1. **Implement core skill** (<500 lines)
2. **Create template library** (diagnosis/remediation patterns)
3. **Extract team config** from current Temporal knowledge
4. **Test with existing 6 RCAs** (validate runbook generation)
5. **Document for other teams** (onboarding guide)
