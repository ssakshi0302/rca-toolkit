# ✅ RCA Analyzer System: READY

**Date**: 2026-05-08  
**Status**: Ready for batch processing

---

## What's Been Built

### 1. RCA Analyzer Skill ✅
**File**: `.claude/skills/temporal-rca-analyzer.md`

**Capabilities**:
- Analyzes Google Doc RCAs
- Analyzes ICC Slack channels
- Extracts timelines (TTD, diagnosis time, TTR)
- Identifies delays (detection, diagnosis, resolution)
- Captures diagnostic details (metrics, logs, actions)
- Captures remediation details (commands, validation)
- Spawns parallel subagents for batch processing
- Creates runbooks for recurring patterns
- Updates executive report

**Supports**:
- Single RCA: `/temporal-rca-analyzer <url>`
- Batch: `/temporal-rca-analyzer --batch <urls-or-folder>`
- Environment filter: `--env=prod,esvc`

### 2. Directory Structure ✅
```
incident-triaging-evaluation/
├── .claude/skills/temporal-rca-analyzer.md   # Skill definition
├── rca-urls-example.txt                       # Template for batch input
├── QUICK-START.md                             # Usage guide
├── research/
│   ├── incident-automation-executive-report.md  # Executive summary
│   └── past rca/
│       ├── rca-analysis-1.md  ✅ (ESVC1 DB CPU)
│       ├── rca-analysis-2.md  ✅ (Mesh routing)
│       ├── rca-analysis-3.md  ✅ (Archival storm)
│       ├── rca-analysis-4.md  ✅ (Karpenter)
│       └── incident-analysis-synthesis.md
└── runbooks/
    ├── metrics-catalog.md                     # Metrics/logs catalog
    ├── diagnosis/                             # (auto-created as patterns emerge)
    └── remediation/                           # (auto-created as patterns emerge)
```

### 3. Existing Analysis ✅
**4 RCAs analyzed**:
- RCA #1: ESVC1 DB CPU Saturation (TTD: 20h, TTR: 4d 6h)
- RCA #2: Istio Mesh Misconfiguration (TTD: 17min, TTR: 25h)
- RCA #3: Regrello Archival Retry Storm (TTD: 3 days)
- RCA #4: Karpenter Node Join Failure

**Executive Report**: `research/incident-automation-executive-report.md`
- Average TTD: 26.4 hours (detection bottleneck)
- Average TTR: 51 hours
- ROI estimate: $186K/year

### 4. Runbook Templates ✅
**Metrics Catalog**: `runbooks/metrics-catalog.md`
- Documents useful metrics, logs, dashboards
- Tracks missing metrics (gaps)
- Updated automatically as new RCAs analyzed

---

## How It Works

### Single RCA (Quick Analysis)
```bash
/temporal-rca-analyzer https://docs.google.com/document/d/YOUR_DOC_ID
```

**Process**:
1. Reads Google Doc (or Slack ICC)
2. Extracts timeline, delays, diagnostic/remediation details
3. Writes `research/past rca/rca-analysis-{N}.md`
4. Updates metrics catalog if new findings

**Duration**: ~2-3 minutes

---

### Batch Processing (10-15 RCAs)
```bash
# Step 1: Add URLs to rca-urls-example.txt
# Step 2: Run batch
/temporal-rca-analyzer --batch rca-urls-example.txt --env=prod,esvc
```

**Process**:
1. **Parse input**: List of URLs or Google Drive folder
2. **Filter**: By environment (if `--env` flag)
3. **Spawn subagents**: 5-10 parallel agents (background)
   - Each agent: Reads 1 RCA → Extracts data → Writes analysis
   - Google Workspace MCP: Reads Google Docs
   - Slack MCP: Reads ICC channels
4. **Monitor**: Agents run in background (no blocking)
5. **Synthesize**: After all complete
   - Aggregate findings
   - Update executive report
   - Create runbooks for recurring patterns
6. **Notify**: "Analysis complete - 10 RCAs processed"

**Duration**: ~15-20 minutes for 10 RCAs

---

## What You Get

### Per RCA: Individual Analysis
**File**: `research/past rca/rca-analysis-{N}.md`

**Structure** (simplified, executive-focused):
```markdown
# RCA #{N}: [Title]

## Summary
- Date, service, environment, severity
- TTD: Xh | Diagnosis: Xh | TTR: Xh

## Where We're Lacking (Key Insight)
- **Detection**: [specific delay] → [opportunity]
- **Diagnosis**: [specific delay] → [opportunity]  
- **Resolution**: [specific delay] → [opportunity]

## Diagnostic Details
- Metrics: [what was checked]
- Logs: [queries used]
- Actions: [step-by-step]

## What Fixed It
- Action: [1-liner]
- Commands: [exact commands]
- Reproducible? [YES/NO]

## Automation Opportunities
- Quick wins: [1-2 week implementations]
- Long-term: [3-6 month investments]
```

### After Batch: Executive Report (Updated)
**File**: `research/incident-automation-executive-report.md`

**Structure**:
```markdown
## Executive Summary
[3-4 bullets: findings, ROI]

## Where We're Lacking (Data-Driven)
### Detection (Biggest Bottleneck)
- Average TTD: Xh
- Top 3 causes: [with %]
- Automation impact: [time savings]

### Diagnosis
- Average time: Xh
- Top 3 delays: [with %]
- Quick wins: [runbooks]

### Resolution
- Average TTR: Xh
- Top bottleneck: [e.g., approvals]
- Automation potential: [specific]

## ROI Estimate
[Time × frequency = savings]

## Recommendations
[3-5 action items, prioritized]
```

### Bonus: Runbooks (Auto-Created for Patterns)
If ≥2 RCAs show same pattern:
- **Diagnosis runbook**: `runbooks/diagnosis/[pattern].md`
- **Remediation runbook**: `runbooks/remediation/[action].md`

---

## Agent Architecture

### Master Skill (Orchestrator)
**Type**: Skill (`.claude/skills/temporal-rca-analyzer.md`)
**Role**: 
- Parse input (URLs, folder, filters)
- Spawn subagents
- Monitor progress
- Trigger synthesis

### Worker Subagents (Parallel Processors)
**Type**: `general-purpose` agents (spawned dynamically)
**Count**: 5-10 at a time
**Mode**: `run_in_background: true`
**Role**:
- Read Google Doc (via Google Workspace MCP)
- Read ICC Slack (via Slack MCP)
- Extract timeline, delays, diagnostic/remediation details
- Write individual analysis file
- Update metrics catalog

**Why subagents?**
- ✅ Parallel: 10 RCAs in 20 min (vs. 30 min sequential)
- ✅ Isolation: Each RCA gets fresh context
- ✅ Background: Don't block user
- ✅ Scalable: Handle 50+ RCAs without overflow

### Synthesis Agent (Aggregator)
**Type**: `general-purpose` agent (spawned after workers complete)
**Role**:
- Read all `rca-analysis-{N}.md` files
- Aggregate: Average TTD/TTR, top delays
- Identify patterns (≥2 similar incidents)
- Create runbooks for recurring patterns
- Update executive report

---

## Design Decisions (Based on Your Input)

### ✅ Simplified (Per Your Request)
- No complex weighted scoring initially
- Focus: "Where are we lacking?" (detection, diagnosis, resolution)
- Executive-ready output (not academic analysis)

### ✅ Flexible Input (Per Your Request)
- Google Docs ✅
- Slack ICC channels ✅
- URL lists ✅
- Google Drive folders ✅

### ✅ Slack MCP Integration (Per Your Request)
- Reads ICC channels for communication analysis
- Extracts investigation timelines
- Captures dead-end investigations

### ✅ Runbooks + Metrics (Per Your Request)
- Captures specific metrics/logs/actions that helped
- Creates reusable diagnostic guides
- Foundation for future automation (skills, AI)

---

## Next Steps

### Immediate: Batch Process 10-15 More RCAs

**You provide**:
- Google Doc RCA URLs (paste into `rca-urls-example.txt`)
- OR Google Drive folder link
- Optional: ICC Slack channels (if documented in RCAs or known)

**I execute**:
```bash
/temporal-rca-analyzer --batch rca-urls-example.txt --env=prod,esvc
```

**Output** (~20 min later):
- 10-15 individual RCA analyses
- Updated executive report (aggregate findings)
- Runbooks for recurring patterns (if ≥2 similar incidents)
- Metrics catalog updates

### After Batch: Executive Presentation

**What you'll have**:
- Data-driven answer: "Where are we lacking?" (detection, diagnosis, or resolution)
- Specific bottlenecks with frequency data
- Automation ROI estimate (time saved × incident frequency)
- Prioritized recommendations (quick wins vs. long-term)

**Use for**:
- Leadership updates
- Platform evaluation (requirements derived from findings)
- PoC planning (test automation on worst bottlenecks)

---

## Future Extensions (Not Implemented Yet)

### Support Query Analyzer (Module 2)
**Similar architecture**, different rules:
- Analyze #temporal-support Slack channel
- Categorize recurring customer queries
- Identify self-service opportunities
- On-call burden analysis

**When ready**: Separate project (same skill pattern)

---

## Questions?

### "Can I start with 5 RCAs instead of 10?"
✅ Yes. Patterns still valuable with smaller sample.

### "What if RCA doesn't have clear timeline?"
✅ Skill marks as "Unknown", still extracts other insights.

### "Do I need to export Slack threads manually?"
✅ No. Skill uses Slack MCP to read ICC channels directly.

### "Can I re-run analysis if I find more info?"
✅ Yes. Just re-run the skill with updated RCA URLs.

---

## Ready Status

| Component | Status |
|-----------|--------|
| Skill definition | ✅ Ready |
| Directory structure | ✅ Ready |
| Metrics catalog template | ✅ Ready |
| Batch input template | ✅ Ready |
| Existing analyses (4 RCAs) | ✅ Complete |
| Executive report | ✅ Complete |
| Runbooks | ⏳ Auto-created as patterns emerge |
| Subagent architecture | ✅ Tested (used for first 4 RCAs) |
| Google Workspace MCP | ✅ Configured |
| Slack MCP | ✅ Configured |

**READY FOR BATCH PROCESSING** ✅

---

## What I Need From You

**To kick off batch analysis**:

1. **RCA URLs** (10-15 recommended):
   - Google Doc links, OR
   - Google Drive folder link, OR
   - Paste URLs into `rca-urls-example.txt`

2. **Optional filters**:
   - Focus production only? (add `--env=prod,esvc`)
   - Specific date range? (mention it, I'll filter)

3. **Confirmation**:
   - Ready to start? Just say "go" and share the URLs

---

**Status**: READY. Waiting for your RCA URLs to begin batch processing.
