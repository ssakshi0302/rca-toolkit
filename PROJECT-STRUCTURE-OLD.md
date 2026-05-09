# Temporal RCA Analyzer - Project Structure

**Last Updated**: 2026-05-08  
**Status**: Production-ready for demo

---

## Directory Structure

```
incident-triaging-evaluation/
│
├── 📋 Documentation
│   ├── README.md                              # Project overview
│   ├── CLAUDE.md                              # Project context for Claude
│   ├── QUICK-START.md                         # Usage guide
│   ├── DEMO.md                                # Demo preparation guide
│   ├── SYSTEM-READY.md                        # System status & architecture
│   └── PROJECT-STRUCTURE.md                   # This file
│
├── 🛠️ Skills (Executable Automation)
│   └── .claude/skills/
│       ├── temporal-rca-analyzer.md           # Main RCA analysis skill
│       └── temporal-pd-alerts.md              # PagerDuty alert analysis skill
│
├── 📊 Research Outputs
│   └── research/
│       ├── incident-automation-executive-report.md  # Executive summary
│       ├── past rca/                          # Individual RCA analyses
│       │   ├── rca-analysis-1.md              # ESVC1 DB CPU Saturation
│       │   ├── rca-analysis-2.md              # Istio Mesh Misconfiguration
│       │   ├── rca-analysis-3.md              # Regrello Archival Retry Storm
│       │   ├── rca-analysis-4.md              # Karpenter Node Join Failure
│       │   └── incident-analysis-synthesis.md # Cross-RCA patterns
│       ├── comparisons/                       # Platform comparison docs
│       ├── notes/                             # Research notes
│       └── strategy/                          # Strategy documents
│
├── 📚 Runbooks (Reusable Knowledge)
│   └── runbooks/
│       ├── metrics-catalog.md                 # All useful metrics/logs/dashboards
│       ├── diagnosis/                         # Diagnostic runbooks (auto-created)
│       └── remediation/                       # Remediation runbooks (auto-created)
│
├── 💾 Backups (Automated Versioning)
│   └── backups/
│       ├── latest/                            # Symlink to most recent backup
│       ├── YYYYMMDD_HHMMSS/                   # Timestamped backups
│       │   ├── research/
│       │   ├── runbooks/
│       │   └── backup-manifest.txt
│       └── ...
│
├── 🔧 Utilities
│   ├── rca-urls-example.txt                   # Template for batch input
│   └── setup-backup-system.sh                 # Backup script
│
└── 🗂️ Context (Platform Evaluation)
    └── .claude/context/
        ├── scope.md                           # Project scope & phases
        ├── platform-evaluation.md             # Evaluation framework
        ├── temporal-incidents.md              # Incident patterns
        └── platforms/                         # Platform-specific context
            ├── matrix.md
            ├── aiexchange.md
            └── ... (warden, icd, resolve.ai - to be created)
```

---

## Key Files

### User-Facing Documentation

| File | Purpose | Audience |
|------|---------|----------|
| `QUICK-START.md` | How to use the analyzer | Engineers, PMs |
| `DEMO.md` | Demo preparation guide | Presenters, stakeholders |
| `SYSTEM-READY.md` | Architecture & technical details | Engineers, architects |
| `PROJECT-STRUCTURE.md` | File organization | All (this file) |

### Executable Skills

| File | Purpose | Invocation |
|------|---------|------------|
| `.claude/skills/temporal-rca-analyzer.md` | RCA analysis automation | `/temporal-rca-analyzer <url>` |
| `.claude/skills/temporal-pd-alerts.md` | PagerDuty alert analysis | `/temporal-pd-alerts --days=30` |

### Research Outputs

| File | Purpose | Updated By |
|------|---------|------------|
| `research/incident-automation-executive-report.md` | Executive summary (aggregate) | Synthesis agent (auto) |
| `research/past rca/rca-analysis-{N}.md` | Individual RCA analysis | Subagents (auto) |
| `research/past rca/incident-analysis-synthesis.md` | Cross-RCA patterns | Initial manual synthesis |

### Runbooks (Knowledge Base)

| File | Purpose | Updated By |
|------|---------|------------|
| `runbooks/metrics-catalog.md` | Metrics, logs, dashboards | Subagents (auto) |
| `runbooks/diagnosis/{pattern}.md` | Diagnostic guides | Synthesis agent (auto, when ≥2 similar RCAs) |
| `runbooks/remediation/{action}.md` | Fix action guides | Synthesis agent (auto, when ≥2 similar fixes) |

### Utilities

| File | Purpose | Usage |
|------|---------|-------|
| `rca-urls-example.txt` | Batch input template | Add URLs, run batch analysis |
| `setup-backup-system.sh` | Backup automation | `./setup-backup-system.sh` |

---

## File Relationships

### RCA Analysis Flow

```
Input: Google Doc RCA URL
  ↓
temporal-rca-analyzer skill
  ↓
Subagent (reads Google Doc via MCP)
  ↓
Output: research/past rca/rca-analysis-{N}.md
  ↓
Updates: runbooks/metrics-catalog.md
  ↓
(If ≥2 similar patterns)
  ↓
Creates: runbooks/diagnosis/{pattern}.md
Creates: runbooks/remediation/{action}.md
```

### Batch Processing Flow

```
Input: rca-urls-example.txt (10 URLs)
  ↓
temporal-rca-analyzer --batch
  ↓
Backup: setup-backup-system.sh (auto)
  ↓
Spawn: 5-10 subagents (parallel, background)
  ├── Subagent 1 → rca-analysis-5.md
  ├── Subagent 2 → rca-analysis-6.md
  └── ... → rca-analysis-14.md
  ↓
All complete
  ↓
Synthesis agent
  ↓
Updates: incident-automation-executive-report.md
Creates: runbooks/* (if patterns found)
  ↓
Notification: "Batch complete"
```

### PagerDuty Alert Flow

```
Command: /temporal-pd-alerts --days=30
  ↓
Reads: Slack #temporal-notifications (via Slack MCP)
  ↓
Extracts: Alert frequency, categories, resolution patterns
  ↓
Correlates: With analyzed RCAs (incident dates)
  ↓
Output: research/pagerduty-alert-analysis-{date-range}.md
  ↓
Updates: Executive report (alert → incident mapping)
```

---

## Data Flow

### Inputs (Where Data Comes From)

| Source | MCP Tool | What We Get |
|--------|----------|-------------|
| **Google Docs** | `mcp__plugin_google-workspace_vmcp-google-workspace__get_doc_as_markdown` | RCA documents |
| **Slack ICC** | `mcp__slack__slack_read_channel`, `mcp__slack__slack_read_thread` | Investigation timelines, communication |
| **Slack PD Alerts** | `mcp__slack__slack_search_public` | PagerDuty alerts, resolution patterns |

### Processing (How We Analyze)

| Component | Type | Function |
|-----------|------|----------|
| **Master skill** | Skill definition | Orchestration, batch management |
| **Worker subagents** | `general-purpose` agents | Parallel RCA processing |
| **Synthesis agent** | `general-purpose` agent | Aggregate findings, update report |

### Outputs (What We Generate)

| File Type | Location | Purpose |
|-----------|----------|---------|
| **Individual analyses** | `research/past rca/` | Per-RCA insights |
| **Executive report** | `research/` | Aggregate findings, ROI |
| **Runbooks** | `runbooks/diagnosis/`, `runbooks/remediation/` | Reusable guides |
| **Metrics catalog** | `runbooks/metrics-catalog.md` | Knowledge base |
| **PD alert analysis** | `research/` | Alert patterns, noise |

---

## Backup System

### Automatic Backups

**Triggered**: Before any batch update
**Location**: `backups/{timestamp}/`
**Contents**:
- Executive report
- All RCA analyses
- Runbooks
- Backup manifest

**Script**: `setup-backup-system.sh`

### Backup Structure

```
backups/
├── latest/                                    # Symlink to most recent
│   └── incident-automation-executive-report.md
│
├── 20260508_213541/                           # Timestamped backup
│   ├── backup-manifest.txt                    # What was backed up
│   ├── research/
│   │   ├── incident-automation-executive-report.md
│   │   └── past rca/
│   │       ├── rca-analysis-1.md
│   │       └── ...
│   └── runbooks/
│       └── metrics-catalog.md
│
└── 20260509_140022/                           # Next backup
    └── ...
```

### Restore Process

```bash
# List available backups
ls -lh backups/

# View backup manifest
cat backups/YYYYMMDD_HHMMSS/backup-manifest.txt

# Restore specific file
cp backups/YYYYMMDD_HHMMSS/research/incident-automation-executive-report.md \
   research/incident-automation-executive-report.md

# Or restore entire backup
cp -r backups/YYYYMMDD_HHMMSS/research/* research/
```

---

## MCP Integration

### Google Workspace MCP
**Used by**: RCA analyzer
**Functions**:
- `get_doc_as_markdown`: Read RCA Google Docs
- `list_drive_items`: List RCAs in folder (for batch)

### Slack MCP
**Used by**: RCA analyzer, PD alerts analyzer
**Functions**:
- `slack_search_public`: Search channels for PD alerts
- `slack_read_thread`: Read ICC investigation threads
- `slack_read_channel`: Read full channel history

### Future MCPs (Not Yet Used)
- **GUS MCP**: Link PRB records automatically
- **Argus/Monitoring MCP**: Fetch metrics directly (if available)

---

## Optimization Status

### ✅ Implemented Optimizations

1. **Parallel Processing**: 5-10 subagents process RCAs simultaneously
2. **Background Execution**: Agents run non-blocking (`run_in_background: true`)
3. **Isolated Context**: Each subagent gets fresh context (no overflow)
4. **Automatic Backups**: No manual backup needed
5. **Incremental Updates**: Executive report updated, not overwritten
6. **MCP Integration**: Direct read from Google Docs & Slack (no manual export)

### 🔄 Future Optimizations

1. **Caching**: Cache Google Doc content to avoid re-reads
2. **Incremental RCA Processing**: Only reprocess changed RCAs
3. **GUS Integration**: Automatic PRB linking
4. **Argus Integration**: Fetch metrics directly for diagnosis
5. **Real-time Monitoring**: Stream PagerDuty alerts live

---

## Current Status

| Component | Status | Notes |
|-----------|--------|-------|
| **RCA Analyzer Skill** | ✅ Ready | Batch & single mode |
| **PD Alerts Skill** | ✅ Ready | 30-day default |
| **Backup System** | ✅ Ready | Automatic timestamped backups |
| **4 RCAs Analyzed** | ✅ Complete | ESVC1, Mesh, Archival, Karpenter |
| **Executive Report** | ✅ Complete | $186K ROI estimate |
| **Runbooks** | ⏳ Auto-created | Created when ≥2 similar patterns |
| **Demo Guide** | ✅ Ready | 10-15 min demo flow |
| **Batch Processing** | ✅ Tested | Used for first 4 RCAs |

**READY FOR**: Batch processing 10-15 more RCAs ✅

---

## Demo Preparation

### Files to Know
1. `DEMO.md` - Demo flow script
2. `research/incident-automation-executive-report.md` - Key numbers
3. `research/past rca/rca-analysis-2.md` - Good example (mesh routing)

### Commands to Know
```bash
# Single RCA
/temporal-rca-analyzer <url>

# Batch RCAs
/temporal-rca-analyzer --batch rca-urls-example.txt --env=prod,esvc

# PD alerts (last 30 days)
/temporal-pd-alerts --days=30

# Manual backup
./setup-backup-system.sh

# Show structure
tree -L 2 -I '.git|backups'
```

### Key Numbers to Remember
- **4 RCAs analyzed** (current)
- **Average TTD**: 26.4 hours (detection bottleneck)
- **Average TTR**: 51 hours
- **ROI estimate**: $186K/year (1,240 hours saved)
- **Target**: 15 RCAs for statistical confidence

---

## Next Steps

### Immediate (This Week)
1. Get 10-15 RCA URLs from stakeholders
2. Run batch analysis: `/temporal-rca-analyzer --batch <urls> --env=prod,esvc`
3. Review updated executive report
4. Optional: Run PD alert analysis for same time period

### Short-term (2-4 Weeks)
1. Present findings to stakeholders (use DEMO.md)
2. Identify top 3 incident patterns
3. Create Tier 1 skills for those patterns (diagnosis runbooks)
4. Measure impact (time to diagnose reduction)

### Medium-term (2-3 Months)
1. Platform evaluation (Matrix, Warden, ICD, Resolve.ai)
2. Score platforms against requirements (from RCA findings)
3. PoC planning (test automation on real incidents)

### Long-term (6-12 Months)
1. Deploy Tier 2 AI-assisted diagnosis
2. Pilot Tier 3 autonomous detection/remediation
3. Expand to other teams (reuse skills)

---

**Maintained by**: Sakshi Mehrotra  
**Last Backup**: 2026-05-08 21:35:41 IST  
**System Status**: PRODUCTION-READY ✅
