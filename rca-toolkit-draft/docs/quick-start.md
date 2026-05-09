# RCA Toolkit - Quick Start

**Time**: 5-10 minutes  
**Goal**: Identify automation opportunities in your first incident RCA (reduce TTD, TTX, TTR)

---

## Prerequisites

- Claude Code with MCP access
- Google Workspace MCP installed (for reading Google Docs)
- RCA documents in Google Docs format

---

## Step 1: Install Toolkit (2 min)

```bash
# Clone repository
git clone git.soma.salesforce.com/orcaas/rca-toolkit.git
cd rca-toolkit

# Verify structure
ls skills/rca-analyzer/
# Should see: skill.yaml, README.md
```

---

## Step 2: Create Team Config (3 min)

```bash
# Copy example config
mkdir -p .claude/config
cp templates/config/team-config-example.yaml .claude/config/myteam-config.yaml

# Edit config
vim .claude/config/myteam-config.yaml
```

**Minimum required**:
```yaml
team:
  name: MyTeam
  service_prefix: myservice

services:
  - name: myservice-api

knowledge:
  metrics_catalog: path/to/your/metrics.md
  query_patterns: path/to/your/queries.md

environments:
  HIGH: [prod]
  LOW: [dev]
```

---

## Step 3: Analyze Your First RCA (2 min)

```bash
# Single RCA
/rca-analyzer https://docs.google.com/document/d/YOUR_DOC_ID
```

**Expected output**:
```
✅ RCA Analysis Complete
├─ File: research/past rca/rca-analysis-1.md
├─ Environment: prod (HIGH priority)
├─ Service: myservice-api
├─ Root Cause: database_saturation
├─ TTD: 2h 15m
├─ TTR: 4h 30m
└─ Automation Opportunity: Add DB CPU alert (saves 2h)
```

---

## Step 4: Review Analysis (1 min)

```bash
# Open generated file
cat research/past rca/rca-analysis-1.md
```

**Contains**:
- Timeline (TTD/TTX/TTR)
- Root cause
- Gaps identified
- Automation opportunities
- ROI estimate

---

## Next Steps

### Batch Analysis
Analyze multiple RCAs at once:
```bash
/rca-analyzer --batch https://docs.google.com/.../doc1, https://docs.google.com/.../doc2
```

### Generate Runbooks
Add flag to generate runbooks for recurring patterns:
```bash
/rca-analyzer --batch <urls> --generate-runbook
```

**Runbooks generated when**:
- ≥2 RCAs with same pattern
- Pattern has clear diagnosis path

### Customize
- **Templates**: Edit `templates/runbook/*.md` for your patterns
- **Config**: Add alerts, services, knowledge files
- **Patterns**: Define custom diagnosis/remediation patterns

---

## Troubleshooting

**Issue**: "Team config not found"
- **Fix**: Create `.claude/config/myteam-config.yaml` (see Step 2)

**Issue**: "Failed to read Google Doc"
- **Fix**: Check Google Workspace MCP is installed and authenticated

**Issue**: "No patterns found"
- **Fix**: Analyze ≥2 RCAs with `--batch` for pattern detection

---

## Getting Help

- **Documentation**: See `docs/team-onboarding.md` for detailed setup
- **Runbook Spec**: See `docs/runbook-spec.md` for template format
- **Examples**: See config examples in `templates/config/`

---

**Next**: [Team Onboarding Guide](team-onboarding.md) for advanced configuration
