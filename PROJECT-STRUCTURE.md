# Project Structure & Organization

**Project**: incident-triaging-evaluation  
**Purpose**: Private workspace for Temporal incident automation research & learning  
**Last Updated**: 2026-05-09

---

## Directory Structure

```
incident-triaging-evaluation/          # PRIVATE workspace (your learning)
│
├── .claude/
│   ├── context/
│   │   ├── temporal/                  # Temporal-specific knowledge (128KB)
│   │   │   ├── temporal-metrics-complete-catalog.md    # 89+ metrics
│   │   │   ├── temporal-argus-patterns.md              # Argus query patterns
│   │   │   ├── temporal-splunk-patterns.md             # Splunk query patterns
│   │   │   ├── temporal-service-architecture.md        # Service dependencies
│   │   │   ├── temporal-incidents.md                   # Incident patterns
│   │   │   └── temporal-doctor-extraction-proposal.md  # Source reference
│   │   ├── platforms/                 # Platform evaluation research
│   │   │   ├── matrix.md
│   │   │   ├── aiexchange.md
│   │   │   └── README.md
│   │   ├── knowledge-sources.md       # Generic: where data comes from
│   │   ├── platform-evaluation.md     # Generic: how to evaluate platforms
│   │   └── scope.md                   # Project scope
│   ├── config/
│   │   └── temporal-config.yaml       # Temporal team config (points to knowledge files)
│   └── artifacts/                     # Generated analysis outputs
│       ├── skill-and-runbook-spec.md
│       ├── project-organization-final.md
│       └── metrics-validation-rca-vs-catalog.md
│
├── research/                          # Your RCA analyses & learning (288KB)
│   ├── past rca/                      # 6 RCA analyses (PRIVATE)
│   │   ├── rca-analysis-1.md
│   │   ├── rca-analysis-2.md
│   │   ├── rca-analysis-3.md
│   │   ├── rca-analysis-4.md
│   │   ├── rca-analysis-5.md
│   │   ├── rca-analysis-6.md
│   │   └── incident-analysis-synthesis.md
│   ├── comparisons/                   # Platform evaluations (PRIVATE)
│   │   ├── platform-comparison-comprehensive.md
│   │   ├── platform-comparison-strategic.md
│   │   └── ... (multiple versions)
│   ├── incident-automation-executive-report.md
│   └── batch-synthesis-6-rcas.md
│
├── runbooks/                          # Your generated runbooks
│   └── metrics-catalog.md             # Metrics used in RCA analyses
│
├── rca-toolkit-draft/                 # Components to extract to rca-toolkit repo
│   ├── skills/
│   │   └── rca-analyzer/              # <500 line skill (to be created)
│   ├── templates/
│   │   ├── runbook/                   # Generic runbook templates
│   │   └── config/                    # Team config schema/examples
│   └── docs/                          # Usage documentation
│
├── scripts/
│   └── setup-backup-system.sh         # Backup utility (PRIVATE)
│
├── backups/                           # Automatic backups (220KB)
│   ├── 20260508_213541/
│   └── 20260509_002352/
│
├── DEMO.md                            # Demo preparation (PRIVATE)
├── COMMANDS-CHEATSHEET.md             # Quick reference (PRIVATE)
├── PROJECT-STRUCTURE.md               # This file
├── CLAUDE.md                          # Project context for Claude
└── README.md                          # Project overview

Total: ~648KB (your depth + learning artifacts)
```

---

## Content Categories

### Private (Stays Here)
**Your learning & depth** - Never shared:
- All 6 RCA analyses (`research/past rca/`)
- Temporal-specific knowledge files (`.claude/context/temporal/`)
- Platform evaluation research (`research/comparisons/`)
- Executive reports & synthesis
- Backups, demos, cheatsheets
- Scripts (backup system)

### Draft (Extract to rca-toolkit)
**Components to move to shared repo**:
- RCA analyzer skill (`rca-toolkit-draft/skills/`)
- Generic runbook templates (`rca-toolkit-draft/templates/`)
- Usage documentation (`rca-toolkit-draft/docs/`)

### Generic (Reusable Concepts)
**Knowledge that applies broadly**:
- `.claude/context/knowledge-sources.md` - Generic data sources concept
- `.claude/context/platform-evaluation.md` - Generic evaluation framework
- Template structures (diagnosis/remediation patterns)

---

## Temporal-Specific Knowledge Files

### Location: `.claude/context/temporal/`

| File | Size | Purpose | Source |
|------|------|---------|--------|
| `temporal-metrics-complete-catalog.md` | ~50KB | 89+ metrics (Counter/Histogram/Gauge) | temporal-doctor PR#3 |
| `temporal-argus-patterns.md` | ~20KB | Argus query patterns, transforms | temporal-doctor PR#3 |
| `temporal-splunk-patterns.md` | ~20KB | Splunk query patterns, batching | temporal-doctor PR#3 |
| `temporal-service-architecture.md` | ~20KB | Service dependencies, cascades | temporal-doctor PR#3 |
| `temporal-incidents.md` | ~10KB | Temporal incident patterns | Original research |
| `temporal-doctor-extraction-proposal.md` | ~10KB | Extraction plan/metadata | Planning doc |

**Total**: ~130KB of Temporal operational knowledge

---

## RCA Toolkit Draft Structure

### Location: `rca-toolkit-draft/`

**Purpose**: Components ready to extract to `rca-toolkit` shared repository

```
rca-toolkit-draft/
├── skills/rca-analyzer/
│   ├── skill.yaml                     # TO CREATE: <500 line skill
│   └── README.md                      # TO CREATE: Usage guide
├── templates/
│   ├── runbook/
│   │   ├── diagnosis-template.md      # TO CREATE: Generic diagnosis pattern
│   │   └── remediation-template.md    # TO CREATE: Generic remediation pattern
│   └── config/
│       ├── team-config-schema.yaml    # TO CREATE: What teams must provide
│       └── team-config-example.yaml   # TO CREATE: Minimal example
└── docs/
    ├── quick-start.md                 # TO CREATE: 5-minute setup
    ├── team-onboarding.md             # TO CREATE: How to create team config
    └── runbook-spec.md                # TO COPY: From .claude/artifacts/
```

**Next steps**:
1. Implement skill (<500 lines)
2. Create generic templates (2 patterns)
3. Write usage docs
4. Test with Temporal config
5. Extract to `rca-toolkit` repo

---

## Workflow

### Development Cycle

1. **Analyze RCAs** in `research/past rca/` (your workspace)
2. **Extract patterns** → identify reusable logic
3. **Create templates** in `rca-toolkit-draft/`
4. **Test** using `.claude/config/temporal-config.yaml`
5. **Refine** based on results
6. **Extract** to shared `rca-toolkit` repo when ready

### File Movement Rules

**From incident-triaging-evaluation → rca-toolkit**:
- ✅ Generic skill logic (<500 lines)
- ✅ Generic templates (diagnosis/remediation patterns)
- ✅ Usage documentation (quick start, onboarding)
- ✅ Config schema (what teams provide)

**Stays in incident-triaging-evaluation**:
- ❌ All RCA analyses
- ❌ All Temporal-specific knowledge
- ❌ All platform evaluations
- ❌ All learning artifacts
- ❌ Backup system, demos, cheatsheets

---

## Configuration System

### Temporal Team Config: `.claude/config/temporal-config.yaml`

**Purpose**: Points skill to Temporal-specific knowledge files

**Key sections**:
- `team`: Team metadata
- `services`: Temporal services (frontend/history/matching/worker)
- `knowledge`: Paths to Temporal-specific files
- `alerts`: PD alert → service → root cause mapping
- `runbook_patterns`: Pattern → template mapping
- `environments`: Priority levels (HIGH/MEDIUM/LOW)

**This config stays PRIVATE** (not shared to rca-toolkit)

### Generic Team Config (rca-toolkit)

**Location**: `rca-toolkit-draft/templates/config/team-config-example.yaml`

**Purpose**: Minimal example showing what teams must provide

**Format**: Same structure, different paths (teams point to their own files)

---

## Size Guidelines

### Private Workspace (incident-triaging-evaluation)
- **Current**: ~648KB
- **Expected growth**: 1-2MB (as you analyze more RCAs)
- **No size limit**: This is your learning space

### Shared Toolkit (rca-toolkit)
- **Target**: 50-100KB total
- **Strict limit**: Keep minimal (skill + templates + docs only)
- **No team-specific content**: Teams provide their own

---

## Next Actions

**Phase 1: Complete Organization** ✅ Done
- [x] Move Temporal files to `.claude/context/temporal/`
- [x] Create `.claude/config/temporal-config.yaml`
- [x] Create `rca-toolkit-draft/` structure
- [x] Document new structure

**Phase 2: Implement Skill** (Next)
- [ ] Create `rca-toolkit-draft/skills/rca-analyzer/skill.yaml` (<500 lines)
- [ ] Create generic runbook templates (2 patterns)
- [ ] Write usage docs
- [ ] Test with 6 existing RCAs

**Phase 3: Extract & Share** (Later)
- [ ] Create `rca-toolkit` repository
- [ ] Move draft components to rca-toolkit
- [ ] Announce toolkit availability

---

## Maintenance

**Review structure**: When adding new RCAs or knowledge files  
**Clean backups**: Monthly (keep last 5 only)  
**Update config**: When Temporal adds new services/alerts  
**Refine templates**: As patterns become clearer (≥2 RCAs per pattern)

---

**Last Updated**: 2026-05-09  
**Owner**: Sakshi Mehrotra
