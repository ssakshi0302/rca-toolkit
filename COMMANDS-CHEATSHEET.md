# Temporal RCA Analyzer - Commands Cheatsheet

Quick reference for demo and daily use.

---

## Core Commands

### Analyze Single RCA
```bash
/temporal-rca-analyzer https://docs.google.com/document/d/YOUR_DOC_ID
```
**Duration**: ~2-3 minutes  
**Output**: `research/past rca/rca-analysis-{N}.md`

---

### Batch Analyze (10-15 RCAs)
```bash
# Step 1: Add URLs to file
vim rca-urls-example.txt

# Step 2: Run batch (production only)
/temporal-rca-analyzer --batch rca-urls-example.txt --env=prod,esvc
```
**Duration**: ~20 minutes for 10 RCAs  
**Output**: 
- Individual analyses: `research/past rca/rca-analysis-{5-15}.md`
- Updated: `research/incident-automation-executive-report.md`

---

### Get PagerDuty Alerts (Last 30 Days)
```bash
/temporal-pd-alerts --days=30
```
**Duration**: ~5-10 minutes  
**Output**: `research/pagerduty-alert-analysis-{date-range}.md`

---

### Manual Backup
```bash
./setup-backup-system.sh
```
**Duration**: <1 second  
**Output**: `backups/{timestamp}/`

---

## View Results

### Executive Summary
```bash
cat research/incident-automation-executive-report.md | less
```
**Key sections**: Executive Summary, Critical Findings, ROI Estimate

---

### Sample RCA Analysis
```bash
cat "research/past rca/rca-analysis-2.md" | less
```
**Good example**: Mesh routing failure (TTD 17min, but 10h diagnosis delay)

---

### List All Analyzed RCAs
```bash
ls -lh "research/past rca"/rca-analysis-*.md
```
**Count**: `wc -l` to see total

---

### View Backup History
```bash
ls -lh backups/
cat backups/latest/backup-manifest.txt
```

---

## Demo Commands

### Show Project Structure
```bash
tree -L 2 -I '.git|backups'
```
**Alternative** (if `tree` not installed):
```bash
find . -maxdepth 2 -type d | grep -v ".git" | sort
```

---

### Show Key Numbers (for presentation)
```bash
# TTD average
grep "Average TTD" research/incident-automation-executive-report.md

# ROI estimate
grep "ROI:" research/incident-automation-executive-report.md

# Count RCAs
ls "research/past rca"/rca-analysis-*.md | wc -l
```

---

### Verify Backups Working
```bash
./setup-backup-system.sh && cat backups/latest/backup-manifest.txt
```

---

## Troubleshooting

### Check Skill Exists
```bash
cat .claude/skills/temporal-rca-analyzer.md | head -20
```

---

### Check MCP Access
```bash
# Google Workspace (should show no permission errors)
# Will be tested during first RCA run

# Slack (should show no permission errors)
# Will be tested during PD alerts run
```

---

### Restore from Backup
```bash
# List available
ls -lh backups/

# View what's in backup
cat backups/YYYYMMDD_HHMMSS/backup-manifest.txt

# Restore executive report
cp backups/YYYYMMDD_HHMMSS/research/incident-automation-executive-report.md \
   research/incident-automation-executive-report.md
```

---

## Advanced Options

### Custom Time Range (PD Alerts)
```bash
/temporal-pd-alerts --start=2026-04-01 --end=2026-05-01
```

---

### Include Pre-Prod RCAs (not recommended)
```bash
/temporal-rca-analyzer --batch rca-urls-example.txt --env=all
```
**Note**: Default `--env=prod,esvc` focuses on high-priority incidents

---

### Analyze ICC Slack Channel Only
```bash
/temporal-rca-analyzer --icc=#icc-12345678
```
**Use when**: RCA discussion happened in Slack, no Google Doc

---

## File Locations (Quick Reference)

| What | Where |
|------|-------|
| Skills | `.claude/skills/` |
| Executive report | `research/incident-automation-executive-report.md` |
| RCA analyses | `research/past rca/rca-analysis-*.md` |
| Runbooks | `runbooks/diagnosis/`, `runbooks/remediation/` |
| Metrics catalog | `runbooks/metrics-catalog.md` |
| Backups | `backups/{timestamp}/` |
| Batch input | `rca-urls-example.txt` |

---

## Common Workflows

### Monthly Review
```bash
# 1. Get last month's PD alerts
/temporal-pd-alerts --days=30

# 2. Identify noisy alerts
cat research/pagerduty-alert-analysis-*.md | less

# 3. Create GUS tickets for alert tuning
```

---

### Post-Incident Batch Analysis
```bash
# 1. Collect 5-10 recent RCA URLs
vim rca-urls-example.txt

# 2. Backup current state
./setup-backup-system.sh

# 3. Run batch analysis (prod only)
/temporal-rca-analyzer --batch rca-urls-example.txt --env=prod,esvc

# 4. Wait ~20 min, review updated report
cat research/incident-automation-executive-report.md | less
```

---

### Prepare for Demo
```bash
# 1. Verify structure
tree -L 2 -I '.git|backups'

# 2. Check key numbers
grep -E "(Average TTD|ROI:)" research/incident-automation-executive-report.md

# 3. Count RCAs
ls "research/past rca"/rca-analysis-*.md | wc -l

# 4. Review sample RCA
cat "research/past rca/rca-analysis-2.md" | head -100

# 5. Open DEMO.md
less DEMO.md
```

---

**Quick Reference Created**: 2026-05-08  
**For**: Demo preparation and daily operations
