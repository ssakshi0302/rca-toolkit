# Post-Incident Analysis Framework - Purpose & Scope

**Version**: 1.0  
**Last Updated**: 2026-05-09

---

## Core Purpose

**Build operational maturity through systematic incident analysis to identify automation opportunities.**

Analyzes incident RCAs to understand:
1. **Where is time spent?** (Detection, Diagnosis, Remediation breakdown)
2. **What operational gaps exist?** (missing alerts, runbooks, automation opportunities)
3. **What patterns recur?** (runbook generation for ≥2 incidents)
4. **What groundwork is needed?** (before scaling automation)

This framework extracts operational intelligence from past incidents to identify maturity gaps before investing in specific automation platforms.

---

## Time Reduction Focus

### TTD (Time to Detect)
**Goal**: Reduce time from incident start to detection

**How toolkit helps**:
- Identifies missing alerts that would have detected earlier
- Finds metric gaps (services emit but not monitored)
- Quantifies detection delay (actual vs ideal)

**Example output**:
```
Detection Gap: 29 minutes
- Why missed: No memory pressure alerting (>70% sustained)
- Fix: Add alert for container_memory_working_set_bytes >70% for 10+ min
- Expected reduction: 29 min → 2 min (27 min saved)
```

---

### TTX (Time to Diagnose / Causation)
**Goal**: Reduce time from detection to root cause identification

**How toolkit helps**:
- Extracts diagnosis steps from RCAs
- Generates deterministic runbooks for recurring patterns
- Identifies missing knowledge (cross-service correlation, sibling checks)

**Example output**:
```
Diagnosis Gap: 1h 30m
- Why slow: No runbook for capacity exhaustion, manual log analysis
- Fix: Create diagnosis runbook with decision tree
- Expected reduction: 1h 30m → 15 min (1h 15m saved)
```

---

### TTR (Time to Remediate)
**Goal**: Reduce time from root cause to resolution

**How toolkit helps**:
- Identifies manual remediation steps that could be automated
- Assesses automation safety (preconditions, rollback plans)
- Generates remediation runbooks with rollback

**Example output**:
```
Remediation Gap: 6h 44m
- Why slow: Manual scaling, no HPA/VPA, waited for approval
- Fix: Implement HPA + automated scaling with safety checks
- Expected reduction: 6h 44m → 30 min (6h 14m saved)
- Safety: Pre-check cluster capacity, gradual scale-up, auto-rollback on errors
```

---

## Automation Opportunity Identification

### Detection Automation
**What it finds**:
- Missing alerts (metric exists, no alert)
- Missing metrics (service should emit, doesn't)
- Alert tuning opportunities (false positives, wrong thresholds)

**Output**:
```
Detection Opportunities:
1. Add alert: memory_pressure >70% for 10+ min → saves 27 min TTD
2. Add alert: OOMKilled events (any container) → saves 9h TTD
3. Add metric: per-namespace DB CPU breakdown → saves 2h TTD
```

---

### Diagnosis Automation (Runbooks)
**What it finds**:
- Recurring diagnosis patterns (≥2 incidents with same steps)
- Cross-service correlation logic (check sibling services)
- Decision trees (if condition A → check B, else check C)

**Output**:
```
Diagnosis Opportunities:
1. Generate runbook: DB CPU saturation → saves 1h 15m TTX
   - Decision tree: Check namespace QPS → slow queries → archival load
   - Sibling checks: temporalhistory + temporalmatching
2. Generate runbook: Mesh routing failure → saves 20h TTX
   - PassthroughCluster traffic check
   - Istio sidecar log analysis
```

---

### Remediation Automation
**What it finds**:
- Manual steps that could be automated (with safety checks)
- Recurring remediations (same action, multiple incidents)
- Rollback strategies (safe automation)

**Output**:
```
Remediation Opportunities:
1. Automate: Resource scaling (HPA + alerts) → saves 6h 14m TTR
   - Safety: Check cluster capacity, gradual scale-up, auto-rollback
   - Preconditions: Verify node availability, no active deployments
2. Automate: Rolling restart (with health checks) → saves 2h TTR
   - Safety: Verify cluster health, one pod at a time, rollback on errors
```

---

## Time Savings Quantification

**Calculates**:
- Current manual effort (hours per phase)
- Potential with improvements (reduced hours)
- Time saved per incident
- Annual projection (if recurring)

**Example**:
```
Incident: DB CPU Saturation
- Detection: 20h → 2 min (with DB CPU alert)
- Diagnosis: 25h → 1h (with runbook)
- Remediation: 4d 6h → 2h (with automated scaling)
- Total time saved: ~4 days per incident
```

**Batch analysis** (6 RCAs):
```
Observable signals existed in 100% of incidents
Detection delay was the largest bottleneck (75% had TTD >10h)
10 specific missing alerts identified
2 recurring patterns found (33% recurrence rate)
```

---

## Pattern Detection & Runbook Generation

**When**: ≥2 RCAs with same pattern

**Pattern definition**:
- Same service
- Same symptom (high_errors, capacity_exhaustion, etc.)
- Same root cause category

**Generates**:
1. **Diagnosis runbook** - Step-by-step investigation with decision points
2. **Remediation runbook** - Action steps with safety checks and rollback

**Example pattern**:
```
Pattern: temporalhistory + capacity_exhaustion + oomkilled
Occurrences: 2 incidents (RCA #6, RCA #8)
Runbook: runbooks/diagnosis/temporalhistory-capacity-exhaustion.md
```

---

## What Toolkit Does NOT Do

❌ **Real-time incident detection** - This is post-incident analysis, not live monitoring  
❌ **Execute automation** - Identifies opportunities, doesn't implement them  
❌ **Replace human judgment** - Runbooks guide, humans decide  
❌ **Automatic remediation** - Safety-critical, requires human approval

---

## Use Cases

### 1. Post-Incident (Immediate)
**When**: After every incident RCA  
**Goal**: Identify what automation would prevent recurrence

```bash
/rca-analyzer https://docs.google.com/.../rca
```

**Output**: Gaps analysis + automation opportunities + ROI

---

### 2. Quarterly Planning (Strategic)
**When**: Planning automation roadmap  
**Goal**: Prioritize highest-ROI automation

```bash
/rca-analyzer --batch <last-quarter-rcas>
```

**Output**: Recurring patterns + ROI estimates + runbook templates

---

### 3. Runbook Library (Operational)
**When**: ≥2 incidents with same pattern  
**Goal**: Reduce TTX for recurring issues

```bash
/rca-analyzer --batch <rcas> --generate-runbook
```

**Output**: Deterministic runbooks with decision trees + rollback plans

---

## Success Metrics

**For teams using toolkit**:
- TTD reduction: Target 95%+ (add alerts/metrics)
- TTX reduction: Target 70%+ (runbooks for common patterns)
- TTR reduction: Target 50%+ (automation for safe remediations)
- Runbook coverage: ≥80% of recurring patterns have runbooks

**Example results** (Temporal team, 6 RCAs):
- Observable signals existed in 100% of incidents
- Detection delay was the largest bottleneck (average: 16.5h)
- 10 specific missing alerts identified
- 2 recurring patterns found (33% recurrence rate)
- 4 runbook candidates generated

---

## Scope

**In scope**:
- Post-incident RCA analysis
- Time breakdown (TTD/TTX/TTR)
- Automation opportunity identification
- ROI calculation
- Runbook generation (for recurring patterns)

**Out of scope**:
- Real-time incident detection (use monitoring platforms)
- Automated remediation execution (requires approval workflows)
- RCA document creation (teams write RCAs, toolkit analyzes them)
- Platform-specific implementations (toolkit is team-agnostic)

---

## Design Principles

1. **Operational maturity first**: Understand gaps before building automation
2. **Signal-aware**: AI systems need quality observability and runbooks to be effective
3. **Data-driven**: Extract patterns from real incidents, not theoretical scenarios
4. **Team-agnostic**: Works for any distributed system (Temporal, Kafka, etc.)
5. **Safety-first**: Human-in-loop for remediation, automation for detection and guidance
6. **Actionable**: Every gap has a specific fix (alert name, metric, runbook pattern)

---

**Next**: See README.md for features and quick start
