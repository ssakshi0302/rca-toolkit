# RCA Toolkit - Ready for Testing ✅

**Date**: 2026-05-09  
**Status**: Core implementation complete, ready for validation

---

## Purpose (Refined)

**Identify automation opportunities to reduce incident response time:**
- **TTD (Time to Detect)**: Find missing alerts/metrics
- **TTX (Time to Diagnose)**: Generate runbooks for causation
- **TTR (Time to Remediate)**: Identify automation candidates
- **ROI**: Quantify time savings and cost reduction

---

## What Was Created

### Core Components ✅

1. **Skill** (<500 lines):
   - `rca-toolkit-draft/skills/rca-analyzer/skill.yaml`
   - Single RCA + batch mode
   - Pattern detection (≥2 RCAs)
   - Runbook generation (opt-in)

2. **Templates**:
   - `diagnosis-template.md` - Investigation with decision points
   - `remediation-template.md` - Action steps with rollback

3. **Config**:
   - `team-config-schema.yaml` - What teams must provide
   - `team-config-example.yaml` - Minimal working example

4. **Documentation**:
   - `README.md` - Main overview with examples
   - `PURPOSE-AND-SCOPE.md` - Detailed purpose and use cases
   - `docs/quick-start.md` - 5-10 min setup
   - `docs/team-onboarding.md` - 1-2 hour comprehensive guide
   - `docs/runbook-spec.md` - Full specification

---

## Repository Structure

```
rca-toolkit-draft/                       # Ready for extraction to rca-toolkit repo
├── README.md                            # Main overview (new ✅)
├── PURPOSE-AND-SCOPE.md                 # Detailed purpose (new ✅)
├── skills/rca-analyzer/
│   ├── skill.yaml                       # <500 line core skill ✅
│   └── README.md                        # Skill usage ✅
├── templates/
│   ├── runbook/
│   │   ├── diagnosis-template.md        # Generic diagnosis pattern ✅
│   │   └── remediation-template.md      # Generic remediation pattern ✅
│   └── config/
│       ├── team-config-schema.yaml      # Schema definition ✅
│       └── team-config-example.yaml     # Minimal example ✅
└── docs/
    ├── quick-start.md                   # 5-10 min setup ✅
    ├── team-onboarding.md               # 1-2 hour onboarding ✅
    └── runbook-spec.md                  # Full specification ✅

Total: ~100KB (tool-focused, minimal)
```

---

## Key Features

### Time Reduction Focus

**TTD (Detection)**:
- Identifies missing alerts that would detect faster
- Finds metric gaps (exists but not monitored)
- Quantifies detection delay

**TTX (Diagnosis)**:
- Generates runbooks for recurring patterns
- Extracts decision trees from RCAs
- Identifies cross-service correlation logic

**TTR (Remediation)**:
- Identifies automation candidates
- Assesses safety (preconditions + rollback)
- Generates remediation runbooks

### ROI Quantification
- Manual effort vs automated effort
- Savings per incident
- Annual projection (if recurring)

### Pattern Detection
- Batch analysis (≥2 RCAs)
- Same service + symptom + root cause
- Deterministic runbook generation

---

## Example Output

**Single RCA Analysis**:
```
✅ RCA Analysis Complete
├─ Service: temporalhistory
├─ Root Cause: capacity_exhaustion (OOMKilled)
├─ TTD: 29 min (gap: no memory pressure alert)
├─ TTX: 29 min (gap: no capacity runbook)
├─ TTR: 6h 44m (gap: manual scaling, no HPA)
└─ Automation Opportunity:
   ├─ Detection: Add memory >70% alert → saves 27 min TTD
   ├─ Diagnosis: Create capacity runbook → saves 19 min TTX
   └─ Remediation: Implement HPA → saves 6h 14m TTR
   Total savings: ~7 hours per incident
```

**Batch Analysis** (6 RCAs):
```
✅ Batch Analysis Complete
├─ RCAs Analyzed: 6
├─ Patterns: 2 recurring
│  ├─ temporalhistory-capacity_exhaustion (2x)
│  └─ temporalfrontend-mesh_routing_failure (2x)
├─ Runbooks Generated: 2
├─ Average TTD: 16.5h → 5 min target (99.5% reduction)
├─ Average TTR: 33.7h → 2h target (94% reduction)
└─ ROI: $264K-529K/year (74-90% time reduction)
```

---

## Next Steps

### Phase 3: Test Skill (Now)

**Test 1: Single RCA** (validate extraction logic)
```bash
# Pick one RCA from research/past rca/
# Extract Google Doc URL
# Run: /rca-analyzer <url> --config=.claude/config/temporal-config.yaml
```

**Expected**:
- Extracts TTD/TTX/TTR correctly
- Identifies gaps (detection/diagnosis/remediation)
- Calculates ROI
- Saves to `research/past rca/rca-analysis-7.md` (or new number)

---

**Test 2: Batch Mode** (validate pattern detection)
```bash
# Use 2-3 existing RCA URLs
# Run: /rca-analyzer --batch <url1>, <url2>, <url3>
```

**Expected**:
- Analyzes all RCAs in parallel
- Detects patterns (if same service + symptom + root cause)
- Aggregates metrics (average TTD/TTR)
- Saves batch synthesis

---

**Test 3: Runbook Generation** (validate deterministic logic)
```bash
# Use 2+ RCAs with same pattern
# Run: /rca-analyzer --batch <urls> --generate-runbook
```

**Expected**:
- Identifies pattern (≥2 occurrences)
- Generates runbook from template
- Fills placeholders (service, metrics, queries)
- Saves to `runbooks/diagnosis/<pattern>.md`

---

### Phase 4: Extract to Shared Repo (Later)

1. Create `git.soma.salesforce.com/orcaas/rca-toolkit`
2. Move `rca-toolkit-draft/` contents
3. Set access: Private, org-level
4. Add CODEOWNERS

---

### Phase 5: Announce & Share (Later)

1. Test with 1-2 pilot teams
2. Iterate based on feedback
3. Announce in #platform-reliability
4. Onboard teams

---

## Temporal Team Config

**Location**: `.claude/config/temporal-config.yaml` ✅

**Points to**:
- Temporal metrics catalog (89+ metrics)
- Argus query patterns
- Splunk query patterns
- Service architecture (dependencies)
- Alert → service → root cause mapping

**Ready for testing**: Yes, config exists and references all knowledge files

---

## Private Workspace Organization

**Temporal-specific knowledge** (stays private):
- `.claude/context/temporal/` (128KB)
  - Complete metrics catalog
  - Query patterns (Argus/Splunk)
  - Service architecture
  - Incident patterns

**RCA analyses** (stays private):
- `research/past rca/` (6 RCAs analyzed)
- All findings, gaps, ROI calculations

**Toolkit draft** (ready to extract):
- `rca-toolkit-draft/` (100KB)
- Tool-focused, team-agnostic
- No Temporal-specific content

---

## Validation Checklist

**Before extracting to shared repo**:
- [ ] Test single RCA analysis (TTD/TTX/TTR extraction)
- [ ] Test batch mode (pattern detection)
- [ ] Test runbook generation (deterministic logic)
- [ ] Validate ROI calculations
- [ ] Confirm config-driven (team-agnostic)
- [ ] Verify <500 line skill limit
- [ ] Test with Temporal config
- [ ] Review generated runbooks for quality

---

## Documentation Complete ✅

**User-facing**:
- README.md - Main overview with examples
- PURPOSE-AND-SCOPE.md - Detailed purpose
- quick-start.md - 5-10 min setup
- team-onboarding.md - Comprehensive guide

**Technical**:
- skill.yaml - Implementation spec (<500 lines)
- runbook-spec.md - Runbook format specification
- team-config-schema.yaml - Config requirements

**Templates**:
- diagnosis-template.md - Investigation pattern
- remediation-template.md - Action pattern
- team-config-example.yaml - Minimal config

---

## Summary

**Completed**:
1. ✅ Organized project (Option 2)
2. ✅ Implemented core skill (Option 1)
3. ✅ Refined purpose (TTD/TTX/TTR focus)
4. ✅ Created comprehensive documentation
5. ✅ Cleaned up artifacts (archived evaluations)

**Ready for**:
- Testing with existing 6 RCAs
- Validation of extraction logic
- Validation of pattern detection
- Validation of runbook generation

**Next session**: Test skill with real RCA URLs

---

**Time Invested**: ~3 hours  
**Lines of Code**: ~500 (skill) + ~500 (templates/docs)  
**Status**: Ready for validation testing ✅
