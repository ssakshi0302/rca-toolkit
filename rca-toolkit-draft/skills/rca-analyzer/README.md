# RCA Analyzer Skill

**Version**: 1.0  
**Purpose**: Identify automation opportunities to reduce Time to Detect (TTD), Time to Diagnose (TTX), and Time to Remediate (TTR)

---

## Usage

### Single RCA Analysis
```bash
/rca-analyzer <google-doc-url>
```

**Example**:
```bash
/rca-analyzer https://docs.google.com/document/d/1abc...
```

### Batch Analysis
```bash
/rca-analyzer --batch <url1>, <url2>, <url3>
```

**Example**:
```bash
/rca-analyzer --batch https://docs.google.com/document/d/1abc..., https://docs.google.com/document/d/2def...
```

### With Runbook Generation
```bash
/rca-analyzer <url> --generate-runbook
```

### Custom Team Config
```bash
/rca-analyzer <url> --config=.claude/config/my-team-config.yaml
```

---

## What It Does

1. **Reads RCA** from Google Doc (via MCP)
2. **Extracts structured data**:
   - Incident timeline (TTD/TTX/TTR)
   - Root cause
   - Services affected
   - Gaps (detection, diagnosis, remediation)
3. **Identifies patterns** (≥2 similar RCAs)
4. **Generates runbooks** (opt-in, deterministic)
5. **Saves analysis** to `research/past rca/`

---

## Configuration

**Required**: Team config file (YAML)

**Location**: `.claude/config/<team>-config.yaml`

**Schema**: See `../../templates/config/team-config-schema.yaml`

---

## Output

### RCA Analysis File
**Location**: `research/past rca/rca-analysis-N.md`

**Format**:
- Incident summary
- Timeline (TTD/TTX/TTR)
- Root cause
- Gaps identified
- Automation opportunities
- ROI estimate

### Runbook (if requested)
**Location**: `runbooks/diagnosis/<pattern>.md` or `runbooks/remediation/<pattern>.md`

**Format**: Deterministic, step-by-step, with rollback plans

---

## Requirements

- Claude Code with MCP access
- Google Workspace MCP (for reading Google Docs)
- Team config file with paths to knowledge files
