# Quick Start Guide

**Goal**: Set up the framework and analyze your first incident RCA

---

## Prerequisites

- Claude Code with MCP access
- Google Workspace MCP installed (for reading Google Docs)
- RCA documents in Google Docs format

---

## Step 1: Clone Repository

```bash
git clone git.soma.salesforce.com/orcaas/post-incident-analysis-framework.git
cd post-incident-analysis-framework

# Verify structure
ls skills/rca-analyzer/
# Should see: skill.yaml, README.md
```

---

## Step 2: Create Team Config

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
  metrics_catalog: path/to/metrics.md
  query_patterns: path/to/queries.md

environments:
  HIGH: [prod, esvc]
  LOW: [dev]
```

**Optional but recommended**:
- Service architecture documentation
- Common failure patterns
- Alert definitions

---

## Step 3: Analyze First RCA

```bash
/rca-analyzer https://docs.google.com/document/d/YOUR_RCA_DOC
```

**What it does**:
- Extracts incident timeline (TTD, TTX, TTR)
- Identifies missing alerts/metrics
- Documents diagnosis workflow
- Suggests runbook opportunities
- Identifies automation candidates

**Output location**: `.claude/artifacts/rca-analysis-[timestamp].md`

---

## Step 4: Review Output

The analysis will identify:

### Detection Gaps
- Missing alerts (signal exists, alert missing)
- Observability blind spots
- Detection delay breakdown

### Diagnosis Patterns
- Manual correlation steps
- Runbook opportunities
- Knowledge gaps

### Remediation Opportunities
- Manual processes
- Approval bottlenecks
- Automation candidates

### Recurring Patterns
- Similar to past incidents (if corpus available)
- Runbook generation trigger (≥2 occurrences)

---

## Next Steps

### Single RCA Analysis
If this is your first analysis, review the output and consider:
- Which missing alerts should be implemented?
- What runbook would have helped diagnosis?
- Is this pattern recurring?

### Batch Analysis
If you have multiple RCAs, analyze in batch:
```bash
/rca-analyzer --batch <url1>, <url2>, <url3>
```

This enables:
- Pattern detection across incidents (≥2 occurrences)
- Runbook generation for recurring patterns
- Aggregate metrics and trends
- CAR prioritization by recurrence risk

---

## Common Setup Issues

### MCP Access
**Problem**: "Cannot read Google Doc"  
**Fix**: Install Google Workspace MCP, grant permissions

### Team Config Not Found
**Problem**: Skill says "team config missing"  
**Fix**: Verify `.claude/config/myteam-config.yaml` exists and has required fields

### No Patterns Detected
**Expected**: Single RCA won't show patterns (need ≥2 similar incidents)  
**Action**: Run batch analysis on multiple RCAs

---

## Example Output

See `examples/temporal/rca-analyses/` for complete analysis examples from Temporal team (6 RCAs).

**Individual analysis**: `rca-analysis-1.md` through `rca-analysis-6.md`  
**Batch synthesis**: `batch-synthesis-6-rcas.md`  
**Generated runbooks**: `../runbooks/`

---

## Customization

### Metrics Catalog
Document your services' metrics in `knowledge/metrics-catalog.md`:
```markdown
## Service Health
- `api_request_total` (Counter) - Total requests
- `api_request_errors` (Counter) - Error count
- `api_request_duration` (Histogram) - Request latency
```

### Query Patterns
Document common queries in `knowledge/query-patterns.md`:
```markdown
## Error Rate Query
scope=myservice metric=api_request_errors | groupby{service, environment}
```

### Service Architecture
Document service dependencies in `knowledge/service-architecture.md`:
```markdown
## Dependencies
- Frontend → API → DB
- Worker → Queue → API
```

---

## Questions?

**Setup issues**: Check `docs/team-onboarding.md` for detailed guide  
**Framework questions**: Review `README.md`  
**Temporal examples**: Explore `examples/temporal/`
