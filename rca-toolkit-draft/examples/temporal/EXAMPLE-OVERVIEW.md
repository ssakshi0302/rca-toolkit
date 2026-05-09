# Temporal Example - Complete Workflow

**Purpose**: Show end-to-end RCA analysis workflow with real Temporal incidents

**What this demonstrates**:
- **Input**: 6 incident RCAs (Google Docs)
- **Process**: Batch analysis with rca-analyzer skill
- **Output**: Individual analyses + batch synthesis + executive report + runbooks

---

## Example Structure

```
examples/temporal/
├── EXAMPLE-OVERVIEW.md                 # This file - overview of example
├── README.md                           # How to use Temporal as reference
│
├── Knowledge Files (Team Setup)
│   ├── team-config.yaml                # Complete Temporal config
│   ├── metrics-catalog.md              # 89+ metrics
│   ├── argus-patterns.md               # Query patterns
│   └── splunk-patterns.md              # Log query patterns
│
├── Input: RCA Documents (Google Docs)
│   └── rca-analyses/
│       ├── rca-analysis-1.md           # ESVC1 DB CPU Saturation
│       ├── rca-analysis-2.md           # Istio Mesh Misconfiguration
│       ├── rca-analysis-3.md           # Regrello Archival Retry Storm
│       ├── rca-analysis-4.md           # Karpenter Node Join Failure
│       ├── rca-analysis-5.md           # Frontend WASM Panic (C2C Timeout)
│       ├── rca-analysis-6.md           # History Capacity Exhaustion
│       └── batch-synthesis-6-rcas.md   # Aggregate analysis
│
├── Output: Generated Runbooks
│   ├── runbook-capacity-exhaustion.md  # From RCA #6 pattern
│   └── runbook-wasm-panic.md           # From RCA #5 pattern
│
└── Output: Executive Report
    └── incident-automation-executive-report.md  # ROI, metrics, roadmap
```

---

## Real Incidents Analyzed

### RCA #1: Database CPU Saturation (ESVC1)
**Google Doc**: [Link to original RCA]  
**Date**: 2024-12-20  
**Environment**: aws-esvc1-useast2/foundation (HIGH PRIORITY)  
**Service**: temporalhistory  
**Root Cause**: Workload spike without indexes

**Timeline**:
- TTD: 20 hours (detection delay)
- TTX: 4 hours (diagnosis)
- TTR: 24 hours (total)

**Automation Identified**:
- Detection: Add DB CPU alert → saves 20h TTD
- Diagnosis: Create DB saturation runbook → saves 2h TTX
- Remediation: Add missing indexes + HPA → saves 4h TTR

---

### RCA #2: Istio Mesh Misconfiguration (ESVC1)
**Google Doc**: [Link to original RCA]  
**Date**: 2025-03-09  
**Environment**: aws-esvc1-useast2/foundation (HIGH PRIORITY)  
**Service**: temporalfrontend  
**Root Cause**: Mesh routing to PassthroughCluster

**Timeline**:
- TTD: 1.5 hours (detection)
- TTX: 18.5 hours (diagnosis - unclear mesh logs)
- TTR: 20 hours (total)

**Automation Identified**:
- Detection: Alert on PassthroughCluster traffic >0 → saves 1h TTD
- Diagnosis: Mesh routing failure runbook → saves 15h TTX
- Remediation: Automated mesh config validation → prevents recurrence

---

### RCA #3: Archival Retry Storm (Preprod)
**Google Doc**: [Link to original RCA]  
**Date**: 2025-05-20  
**Environment**: fdev1-uswest2 (LOW PRIORITY - preprod)  
**Service**: temporalworker (archival)  
**Root Cause**: Regrello service outage → archival retry storm

**Timeline**:
- TTD: 3 days (detection delay)
- TTX: 30 minutes (diagnosis)
- TTR: 3 days (total - delayed action on preprod)

**Automation Identified**:
- Detection: Archival queue depth alert → saves 3 days TTD
- Diagnosis: Archival backlog runbook → saves 15 min TTX
- Remediation: Automated backoff/circuit breaker → prevents storm

---

### RCA #4: Karpenter Node Join Failure (Prod)
**Google Doc**: [Link to original RCA]  
**Date**: 2025-06-15  
**Environment**: aws-prod3-useast1 (HIGH PRIORITY)  
**Service**: temporalhistory (impacted by node capacity)  
**Root Cause**: Karpenter failed to join nodes, capacity exhausted

**Timeline**:
- TTD: 15 minutes (detection)
- TTX: 45 minutes (diagnosis)
- TTR: 1 hour (total)

**Automation Identified**:
- Detection: Node join failure alert → saves 10 min TTD
- Diagnosis: Node provisioning runbook → saves 20 min TTX
- Remediation: Karpenter monitoring + fallback → prevents recurrence

---

### RCA #5: Frontend WASM Panic (C2C Timeout)
**Google Doc**: [Link to original RCA]  
**Date**: 2025-08-12 to 2025-08-22 (10 days)  
**Environment**: prod1/foundation (HIGH PRIORITY)  
**Service**: temporalfrontend  
**Root Cause**: C2C auth timeout → WASM panic on missing headers

**Timeline**:
- Incident 1 (Aug 12): TTD: 1 min, TTX: 32 min, TTR: 50 min
- Incident 2 (Aug 22 - Escalation): TTD: 1 min, TTX: 8 hours, TTR: 22.9 hours
- **Total duration**: 10 days (with 2-3 daily rolling restarts)

**Automation Identified**:
- Detection: WASM panic alert + recurring alert tracking → saves 10 days diagnosis
- Diagnosis: WASM panic runbook with C2C correlation → saves 8h TTX
- Remediation: Graceful timeout handling + auto-restart → prevents recurrence

**Runbook Generated**: `runbook-wasm-panic.md` ✅

---

### RCA #6: History Capacity Exhaustion (OOMKilled)
**Google Doc**: [Link to original RCA]  
**Date**: 2025-09-06  
**Environment**: prod8-cacentral1/cdp2 (HIGH PRIORITY)  
**Service**: temporalhistory  
**Root Cause**: Workflow surge (50k in 8h), no HPA, 2Gi memory limit

**Timeline**:
- TTD: 29 minutes (detection delay)
- TTX: 2 hours 15 minutes (diagnosis)
- TTR: 6 hours 44 minutes (total)

**Automation Identified**:
- Detection: Memory pressure alert (>70%) → saves 27 min TTD
- Diagnosis: Capacity exhaustion runbook → saves 2h TTX
- Remediation: HPA + resource scaling → saves 6h TTR

**Runbook Generated**: `runbook-capacity-exhaustion.md` ✅

---

## Batch Analysis Summary

**Command used**:
```bash
/rca-analyzer --batch \
  https://docs.google.com/.../rca1, \
  https://docs.google.com/.../rca2, \
  https://docs.google.com/.../rca3, \
  https://docs.google.com/.../rca4, \
  https://docs.google.com/.../rca5, \
  https://docs.google.com/.../rca6 \
  --generate-runbook \
  --config=.claude/config/temporal-config.yaml
```

**Results** (from `batch-synthesis-6-rcas.md`):
```
✅ Batch Analysis Complete

RCAs Analyzed: 6
├─ Production (HIGH): 4 incidents
├─ Preprod (MEDIUM): 1 incident
└─ Dev (LOW): 1 incident

Environments:
├─ aws-esvc1-useast2 (2 incidents)
├─ prod1/foundation (1 incident)
├─ prod3-useast1 (1 incident)
├─ prod8-cacentral1 (1 incident)
└─ fdev1-uswest2 (1 incident - preprod)

Services Affected:
├─ temporalfrontend (2 incidents)
├─ temporalhistory (3 incidents)
└─ temporalworker (1 incident)

Patterns Identified: 2 recurring
├─ capacity_exhaustion (2 occurrences: RCA #1, #6)
└─ wasm_panic (1 occurrence but 10-day duration: RCA #5)

Runbooks Generated: 2
├─ runbooks/diagnosis/temporalhistory-capacity-exhaustion.md
└─ runbooks/diagnosis/temporalfrontend-wasm-panic.md

Time Reduction Potential:
├─ Average TTD: 16.5 hours → Target: 5 min (99.5% reduction)
├─ Average TTX: Variable (10 days worst case) → Target: 15 min
└─ Average TTR: 33.7 hours → Target: 2 hours (94% reduction)

ROI: $264K-529K/year (74-90% time reduction from automation)
```

---

## Key Insights from Example

### Detection Gaps (TTD)
**10 missing alerts identified**:
1. DB CPU alert (RCA #1) - 20h delay
2. Memory pressure alert (RCA #6) - 29 min delay
3. OOMKilled event alert (RCA #6) - 9h earlier detection
4. PassthroughCluster traffic alert (RCA #2) - 1h delay
5. Archival queue depth alert (RCA #3) - 3 day delay
6. WASM panic alert (RCA #5) - 10 day masking
7. Recurring alert tracking (RCA #5) - pattern detection
8. C2C auth latency alert (RCA #5) - timeout prediction
9. Node join failure alert (RCA #4) - 10 min delay
10. DB CPU by namespace (RCA #1, #6) - faster isolation

**Impact**: Average TTD 16.5h → 5 min target (99.5% reduction)

---

### Diagnosis Gaps (TTX)
**4 runbook opportunities identified**:
1. **DB CPU saturation** - RCA #1, #6 (2 occurrences)
   - Decision tree: namespace QPS → slow queries → archival load
   - Time savings: 2-4 hours per incident
   
2. **Mesh routing failure** - RCA #2
   - PassthroughCluster check → Istio sidecar logs → deployment correlation
   - Time savings: 15 hours per incident
   
3. **Capacity exhaustion** - RCA #6 (generated ✅)
   - Memory trends → HPA check → workload surge → DB correlation
   - Time savings: 2 hours per incident
   
4. **WASM panic** - RCA #5 (generated ✅)
   - WASM logs → C2C timeout correlation → mesh latency
   - Time savings: 8 hours per incident (prevented 10-day recurrence)

**Impact**: Variable TTX → 15 min target (70-90% reduction)

---

### Remediation Gaps (TTR)
**3 automation candidates identified**:
1. **Resource scaling (HPA)** - RCA #1, #6
   - Safety: Check cluster capacity, gradual scale-up, auto-rollback
   - Time savings: 4-6 hours per incident
   
2. **Config validation** - RCA #2
   - Safety: Pre-check mesh routes, validate before deploy
   - Time savings: Prevents recurrence (20h incident avoided)
   
3. **Graceful timeout handling** - RCA #5
   - Safety: WASM extension patch, test in preprod first
   - Time savings: Prevents 10-day recurrence (2-3 daily restarts)

**Impact**: Average TTR 33.7h → 2h target (94% reduction)

---

## How to Use This Example

### For Learning
1. **Read RCAs** in `rca-analyses/` to understand incident patterns
2. **Review runbooks** to see how toolkit extracts diagnosis steps
3. **Check synthesis** to see pattern detection across multiple RCAs
4. **Read executive report** to see ROI calculation

### For Adapting to Your Team
1. **Start with 2-3 RCAs** from your team
2. **Run batch analysis** using skill
3. **Compare output** to this example
4. **Identify patterns** (≥2 similar incidents)
5. **Generate runbooks** for recurring patterns

### For Demonstrations
1. **Show inputs**: 6 real RCAs with Google Doc links
2. **Show knowledge**: Metrics catalog, query patterns, team config
3. **Show outputs**: Individual analyses, batch synthesis, runbooks, executive report
4. **Show ROI**: $264K-529K/year potential savings

---

## Files in This Example

### Knowledge Files (Setup)
- `team-config.yaml` (5KB) - Complete Temporal config
- `metrics-catalog.md` (50KB) - 89+ metrics
- `argus-patterns.md` (20KB) - Query patterns
- `splunk-patterns.md` (20KB) - Log patterns

### Input RCAs (Analyzed)
- `rca-analyses/rca-analysis-1.md` (25KB) - DB CPU saturation
- `rca-analyses/rca-analysis-2.md` (30KB) - Mesh routing failure
- `rca-analyses/rca-analysis-3.md` (20KB) - Archival retry storm
- `rca-analyses/rca-analysis-4.md` (15KB) - Node join failure
- `rca-analyses/rca-analysis-5.md` (35KB) - WASM panic
- `rca-analyses/rca-analysis-6.md` (30KB) - Capacity exhaustion

### Output: Analysis & Synthesis
- `rca-analyses/batch-synthesis-6-rcas.md` (15KB) - Aggregate metrics
- `incident-automation-executive-report.md` (20KB) - Executive summary

### Output: Runbooks
- `runbook-capacity-exhaustion.md` (10KB) - Diagnosis steps
- `runbook-wasm-panic.md` (12KB) - Diagnosis steps

**Total**: ~262KB (complete example with inputs + outputs)

---

## Google Doc Links

**Original RCA documents** (requires Salesforce access):

1. **RCA #1** (DB CPU Saturation): [Google Doc URL]
2. **RCA #2** (Mesh Misconfiguration): [Google Doc URL]
3. **RCA #3** (Archival Retry Storm): [Google Doc URL]
4. **RCA #4** (Node Join Failure): [Google Doc URL]
5. **RCA #5** (WASM Panic): [Google Doc URL]
6. **RCA #6** (Capacity Exhaustion): [Google Doc URL]

**Note**: Links intentionally placeholder - replace with actual Google Doc URLs when sharing externally

---

## Key Takeaways

### For Teams Adopting Toolkit
1. **Start small**: 2-3 RCAs to validate
2. **Build knowledge incrementally**: Start with 10-20 key metrics, add more over time
3. **Focus on patterns**: ≥2 similar incidents = runbook opportunity
4. **Measure impact**: Track TTD/TTX/TTR reduction

### For Leadership
1. **ROI is quantifiable**: $264K-529K/year for Temporal (6 RCAs)
2. **Patterns are common**: 2/6 RCAs showed capacity_exhaustion pattern
3. **Detection is biggest gap**: 99.5% time reduction potential (16.5h → 5 min)
4. **Runbooks work**: 70-90% TTX reduction with deterministic guidance

### For SRE/Oncall Teams
1. **Runbooks are actionable**: Step-by-step with decision points
2. **Knowledge is portable**: Query patterns reusable across teams
3. **Automation is safe**: Runbooks include rollback plans, safety checks
4. **Patterns recur**: Same root cause across multiple incidents (capacity, mesh, etc.)

---

## Next Steps

**Try it yourself**:
1. Clone rca-toolkit repository
2. Copy this example to your workspace
3. Replace Temporal knowledge with your team's metrics/queries
4. Run against your team's RCAs
5. Generate runbooks for your patterns

**Questions?** See `docs/team-onboarding.md` for detailed setup guide

---

**Last Updated**: 2026-05-09  
**Example Source**: OrcaaS Team (Temporal Production Incidents)  
**Status**: Production-ready example with real incidents
