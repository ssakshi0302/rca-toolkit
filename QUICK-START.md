# Quick Start: RCA Analyzer

## What This Does
Analyzes Temporal RCAs to identify:
- **Where we're lacking**: Detection, diagnosis, or resolution?
- **What delays incidents**: Specific bottlenecks
- **What can be automated**: Quick wins and ROI

## Current Status

### ✅ Completed (4 RCAs Analyzed)
- RCA #1: ESVC1 DB CPU Saturation (TTD: 20h, TTR: 4d 6h)
- RCA #2: Istio Mesh Misconfiguration (TTD: 17min, TTR: 25h)
- RCA #3: Regrello Archival Retry Storm (TTD: 3 days)
- RCA #4: Karpenter Node Join Failure

**Output**: 
- Individual analyses: `.agents/artifacts/rca-analysis-{1-4}.md`
- Executive summary: `research/incident-automation-executive-report.md`

**Key Finding**: Average TTD 26.4h (detection is biggest bottleneck), estimated ROI $186K/year

---

## How to Analyze More RCAs

### Option 1: Single RCA (Quick Test)
```bash
/temporal-rca-analyzer https://docs.google.com/document/d/YOUR_DOC_ID
```

### Option 2: Batch (10-15 RCAs)

**Step 1**: Add your RCA URLs to `rca-urls-example.txt`
```
# One URL per line
https://docs.google.com/document/d/DOC_ID_1
https://docs.google.com/document/d/DOC_ID_2
...
```

**Step 2**: Run batch analysis
```bash
/temporal-rca-analyzer --batch rca-urls-example.txt
```

**Step 3**: Wait for completion (agents run in background, ~15-20 min for 10 RCAs)

**Step 4**: Review outputs
- Individual analyses: `research/past rca/rca-analysis-{5-15}.md`
- Updated executive report: `research/incident-automation-executive-report.md`

### Option 3: Google Drive Folder
```bash
/temporal-rca-analyzer --batch https://drive.google.com/drive/folders/YOUR_FOLDER_ID
```

### Option 4: Focus Production Only
```bash
/temporal-rca-analyzer --batch rca-urls-example.txt --env=prod,esvc
```

---

## What You'll Get

### Per RCA: Individual Analysis File
**Location**: `research/past rca/rca-analysis-{N}.md`

**Contents**:
- Timeline breakdown (TTD, diagnosis time, TTR)
- Delay analysis (what slowed each phase)
- Diagnostic details (metrics/logs that helped)
- What fixed it (commands, reproducibility)
- Automation opportunities

### After Batch: Updated Executive Report
**Location**: `research/incident-automation-executive-report.md`

**Contents**:
- Aggregate metrics (average TTD, TTR across all RCAs)
- Top delay causes (ranked by frequency)
- Automation ROI estimate
- Prioritized recommendations

### Bonus: Runbooks (Auto-Created)
**Location**: `runbooks/`

If ≥2 RCAs show same pattern:
- **Diagnosis runbook**: `runbooks/diagnosis/[pattern].md`
- **Remediation runbook**: `runbooks/remediation/[action].md`
- **Metrics catalog** updated: `runbooks/metrics-catalog.md`

---

## Example Workflow

```bash
# 1. You have 12 RCA Google Docs
# 2. Paste URLs into rca-urls-example.txt
# 3. Run analyzer
/temporal-rca-analyzer --batch rca-urls-example.txt --env=prod,esvc

# 4. Agents spawn (5-10 parallel, run in background)
# → You continue other work

# 5. ~20 minutes later, notification: "Analysis complete"

# 6. Review executive report
cat research/incident-automation-executive-report.md

# 7. Check new RCA analyses
ls research/past\ rca/rca-analysis-*.md

# 8. Check if runbooks created
ls runbooks/diagnosis/
ls runbooks/remediation/
```

---

## Files Created

```
incident-triaging-evaluation/
├── rca-urls-example.txt              # ← Add your RCA URLs here
├── research/
│   ├── incident-automation-executive-report.md  # ← Main output (updated)
│   └── past rca/
│       ├── rca-analysis-1.md through rca-analysis-N.md
│       └── incident-analysis-synthesis.md
├── runbooks/
│   ├── metrics-catalog.md            # ← All useful metrics/logs
│   ├── diagnosis/
│   │   └── [pattern-name].md         # ← Auto-created from patterns
│   └── remediation/
│       └── [action-name].md          # ← Auto-created from patterns
└── .claude/
    └── skills/
        └── temporal-rca-analyzer.md  # ← The skill definition
```

---

## Next Steps

1. **Now**: Add 10-15 RCA URLs to `rca-urls-example.txt`
2. **Run**: `/temporal-rca-analyzer --batch rca-urls-example.txt --env=prod,esvc`
3. **Wait**: ~20 min (agents work in background)
4. **Review**: Executive report + individual analyses
5. **Present**: Use executive report for stakeholder discussions

---

## Questions?

- **Where are the first 4 RCA analyses?** → `.agents/artifacts/` (they'll be moved to `research/past rca/` in next batch)
- **Can I analyze Slack ICC channels?** → Yes: `/temporal-rca-analyzer --icc=#icc-12345678`
- **What if I don't have 10 RCAs yet?** → Start with 3-5, patterns still valuable
- **How long does it take?** → ~2-3 min per RCA, parallel processing = ~20 min for 10 RCAs

---

## Ready to Scale Up?

**Share your RCA Google Doc URLs or Google Drive folder link**, and I'll kick off the batch analysis.
