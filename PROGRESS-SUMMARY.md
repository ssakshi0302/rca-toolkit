# Progress Summary: RCA Toolkit Development

**Date**: 2026-05-09  
**Status**: Phase 1 & 2 Complete ✅

---

## Completed: Option 2 (Organize Project) ✅

### File Organization
- ✅ Moved Temporal-specific files to `.claude/context/temporal/`
- ✅ Created `.claude/config/temporal-config.yaml`
- ✅ Created `rca-toolkit-draft/` structure
- ✅ Documented new structure in `PROJECT-STRUCTURE.md`

### Temporal-Specific Knowledge (Private)
**Location**: `.claude/context/temporal/`
- `temporal-metrics-complete-catalog.md` (89+ metrics)
- `temporal-argus-patterns.md` (query patterns)
- `temporal-splunk-patterns.md` (log queries)
- `temporal-service-architecture.md` (dependencies)
- `temporal-incidents.md` (incident patterns)

### Team Config (Private)
**Location**: `.claude/config/temporal-config.yaml`
- Team metadata
- Services (frontend/history/matching/worker)
- Infrastructure mapping
- Knowledge file paths
- Alert → service → root cause mapping
- Runbook pattern definitions
- Environment priorities

---

## Completed: Option 1 (Implement Core Skill) ✅

### Core Skill (~500 lines)
**Location**: `rca-toolkit-draft/skills/rca-analyzer/skill.yaml`

**Features**:
- Single RCA analysis
- Batch mode (parallel processing)
- Pattern detection (≥2 similar RCAs)
- Opt-in runbook generation
- Config-driven (team-agnostic)
- Backup system integration
- Error handling

**Components**:
- Argument parsing (50 lines)
- Config loading (50 lines)
- Single RCA analysis (100 lines)
- Batch orchestration (80 lines)
- Pattern detection (60 lines)
- Runbook generation (80 lines)
- Output formatting (50 lines)
- Error handling (30 lines)

**Total**: ~500 lines (target met)

### Runbook Templates
**Location**: `rca-toolkit-draft/templates/runbook/`

1. **diagnosis-template.md**:
   - Symptoms
   - Diagnosis steps (with decision points)
   - Cross-service correlation
   - Common pitfalls
   - Related incidents

2. **remediation-template.md**:
   - Prerequisites & safety checks
   - Remediation steps (with validation)
   - Verification checklist
   - Rollback plan
   - Post-incident actions

### Config Templates
**Location**: `rca-toolkit-draft/templates/config/`

1. **team-config-schema.yaml**: Full schema definition
2. **team-config-example.yaml**: Minimal working example

### Documentation
**Location**: `rca-toolkit-draft/docs/`

1. **quick-start.md**: 5-10 minute setup guide
2. **team-onboarding.md**: Comprehensive onboarding (1-2 hours)
3. **runbook-spec.md**: Full specification (copied from artifacts)

### Skill README
**Location**: `rca-toolkit-draft/skills/rca-analyzer/README.md`
- Usage examples
- Configuration requirements
- Output format
- Requirements

---

## Repository Structure Ready for Extraction

```
rca-toolkit-draft/                      # Ready to become rca-toolkit repo
├── skills/rca-analyzer/
│   ├── skill.yaml                      # <500 line core skill
│   └── README.md                       # Usage guide
├── templates/
│   ├── runbook/
│   │   ├── diagnosis-template.md       # Generic diagnosis pattern
│   │   └── remediation-template.md     # Generic remediation pattern
│   └── config/
│       ├── team-config-schema.yaml     # Schema definition
│       └── team-config-example.yaml    # Minimal example
└── docs/
    ├── quick-start.md                  # 5-10 min setup
    ├── team-onboarding.md              # 1-2 hour onboarding
    └── runbook-spec.md                 # Full specification

Total: ~100KB (minimal, tool-focused)
```

---

## Private Workspace Organization

```
incident-triaging-evaluation/           # PRIVATE (your learning)
├── .claude/
│   ├── context/
│   │   ├── temporal/                   # Temporal-specific (128KB)
│   │   ├── platforms/                  # Platform evaluation
│   │   └── *.md                        # Generic patterns
│   ├── config/
│   │   └── temporal-config.yaml        # Temporal team config
│   └── artifacts/                      # Generated analysis
├── research/                           # 6 RCA analyses + reports (288KB)
├── runbooks/                           # Generated runbooks
├── rca-toolkit-draft/                  # Components to extract
├── backups/                            # Automatic backups (220KB)
└── scripts/                            # Utilities

Total: ~648KB (your depth + learning)
```

---

## Next Steps

### Phase 3: Test Skill (Next)
- [ ] Test skill with 1 existing RCA (validate extraction logic)
- [ ] Test batch mode with 2-3 RCAs (validate pattern detection)
- [ ] Test runbook generation (validate deterministic logic)
- [ ] Refine based on results

### Phase 4: Extract to Shared Repo (Later)
- [ ] Create `git.soma.salesforce.com/orcaas/rca-toolkit`
- [ ] Move `rca-toolkit-draft/` contents to new repo
- [ ] Set access: Private, org-level
- [ ] Add CODEOWNERS

### Phase 5: Announce & Share (Later)
- [ ] Create README for rca-toolkit
- [ ] Announce in #platform-reliability or similar
- [ ] Onboard 1-2 pilot teams
- [ ] Iterate based on feedback

---

## Repository Decision

**Confirmed**: `rca-toolkit` (not post-incident-toolkit, not rca-automation-toolkit)

**Access**: Private repository, org-level visibility (not team-restricted)

**Ownership**: Sakshi Mehrotra (creator), OrcaaS team (maintainers)

---

## Key Achievements

1. ✅ Organized project with clear private/shared boundaries
2. ✅ Implemented <500 line skill (modular, config-driven)
3. ✅ Created deterministic runbook templates
4. ✅ Built comprehensive documentation (quick-start + onboarding)
5. ✅ Defined team config schema (team-agnostic)
6. ✅ Preserved all private work (Temporal depth, RCA analyses)

---

## Skill Features Summary

**Input**: Google Doc URL(s) + team config
**Output**: 
- Structured RCA analysis (TTD/TTX/TTR, gaps, ROI)
- Pattern detection (≥2 similar RCAs)
- Deterministic runbooks (opt-in)

**Modes**:
- Single RCA analysis
- Batch mode (parallel)
- With/without runbook generation

**Team-Agnostic**:
- Config-driven (teams provide their knowledge files)
- Generic templates (teams can customize)
- Works for any distributed system (not Temporal-specific)

---

## Files Created/Modified

### Created (9 files)
1. `.claude/config/temporal-config.yaml`
2. `rca-toolkit-draft/skills/rca-analyzer/skill.yaml`
3. `rca-toolkit-draft/skills/rca-analyzer/README.md`
4. `rca-toolkit-draft/templates/runbook/diagnosis-template.md`
5. `rca-toolkit-draft/templates/runbook/remediation-template.md`
6. `rca-toolkit-draft/templates/config/team-config-schema.yaml`
7. `rca-toolkit-draft/templates/config/team-config-example.yaml`
8. `rca-toolkit-draft/docs/quick-start.md`
9. `rca-toolkit-draft/docs/team-onboarding.md`

### Moved (6 files)
- `temporal-*.md` → `.claude/context/temporal/`

### Updated (1 file)
- `PROJECT-STRUCTURE.md` (reorganized structure documentation)

---

**Time Invested**: ~2 hours  
**Lines of Code**: ~500 (skill) + ~500 (templates/docs)  
**Ready for**: Testing phase

---

**Owner**: Sakshi Mehrotra  
**Next Session**: Test skill with existing RCAs
