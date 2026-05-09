# Executive Summary Specification

**Purpose**: Define requirements for producing executive-ready incident automation analysis reports

**Target Audience**: Leadership, Engineering Managers, Decision-makers

**Reading Time**: 3-5 minutes (1-2 pages maximum)

---

## Core Principles

1. **Scannable**: Tables and bullets over paragraphs
2. **Data-Driven**: Numbers, timelines, concrete metrics (no marketing language)
3. **Action-Oriented**: Clear "Implement X" items with expected impact
4. **Concise**: 1-2 pages maximum, minimal text

---

## Required Sections

### 1. Header

**Format**:
```markdown
# [Team/Service] Incident Automation: Action Plan

**Date**: YYYY-MM-DD | **Analysis Period**: [date range] | **RCAs Analyzed**: [count]
```

**Required**:
- Clear title (action-oriented, not "analysis" or "report")
- Date of analysis
- Analysis period (incident date range)
- Number of RCAs analyzed

---

### 2. Executive Summary (3-4 lines)

**Format**:
```markdown
**[Biggest bottleneck]**: [statistic with range]
**Automation impact**: [percentage reductions]
**ROI**: $[amount]/year ([hours saved], [percentage] reduction)
```

**Requirements**:
- Identify biggest bottleneck (detection, diagnosis, or remediation)
- Quantify automation impact (percentage reductions for TTD/TTX/TTR)
- Show ROI (dollar amount, hours saved, percentage reduction)
- Each line must have concrete numbers

**Example**:
```markdown
**Detection is the bottleneck**: 75% of incidents had TTD >10 hours (range: 29 min - 3 days)
**Automation impact**: 50-95% TTD reduction, 40-70% faster diagnosis, 30-40% auto-resolution potential
**ROI**: $264K-529K/year (1,240 hours saved, 71% reduction in incident time)
```

---

### 3. Data Overview (Table)

**Format**: Single table with key metrics

| Metric | Value | Note |
|--------|-------|------|
| **RCAs Analyzed** | [count] | [date range] |
| **Average TTD** | [hours] | Range: [min] - [max] |
| **Average TTR** | [hours] | [qualifier] |
| **Detection Bottleneck** | [percentage] | [root cause] |
| **Root Cause** | [percentage] observable | [finding] |

**Requirements**:
- 5-7 rows maximum
- Bold metric names
- Include ranges where applicable
- Add context in "Note" column

---

### 4. Critical Findings (3-4 findings)

**Format**: Each finding has:
```markdown
### [Number]. [Finding Title] (Category)

**Data**:
- [RCA #N](path/to/rca-analysis-N.md) ([Brief description]): [metric] ([impact])
- [RCA #N](path/to/rca-analysis-N.md) ([Brief description]): [metric] ([impact])
- [RCA #N](path/to/rca-analysis-N.md) ([Brief description]): [metric] ([impact])

**Root Causes**:
- [Cause 1] ([specific gap])
- [Cause 2] ([specific gap])
- [Cause 3] ([specific gap])

**Impact**: [percentage] [metric] reduction with [solution]
```

**Requirements**:
- Link RCA references (markdown links to analysis files)
- Include brief description in parentheses (e.g., "DB CPU Saturation - ESVC1")
- Include concrete metrics (hours, days, percentages)
- Show root causes (not just symptoms)
- Quantify impact of fixing the issue
- 3-5 bullet points per subsection

**Categories**:
- Detection delays
- Manual correlation/diagnosis
- Recurring patterns
- Remediation bottlenecks

---

### 5. Incident Summary (Table)

**Format**:

| RCA # | Incident Type | TTD | Diagnosis | TTR | Detection Gap | Resolution |
|-------|---------------|-----|-----------|-----|---------------|------------|
| **[#1](path/to/rca-analysis-1.md)** | [brief description with context] | [time] | [time] | [time] | [gap] | [action] |
| **[#2](path/to/rca-analysis-2.md)** | [brief description with context] | [time] | [time] | [time] | [gap] | [action] |

**Requirements**:
- One row per RCA
- Link RCA numbers to analysis files (markdown links)
- Brief descriptions with context (e.g., "DB CPU Saturation (ESVC1 namespace workload spike)")
- Include TTD, diagnosis time, TTR
- Show what was missing (detection gap)
- Show what was done (resolution)
- Add "Key Observations" bullet list below table

**Key Observations** (3-5 bullets):
- Percentage with long TTD (e.g., "75% had TTD >10 hours")
- Observable signals statement (e.g., "100% had observable signals")
- Diagnosis vs detection comparison

---

### 6. Immediate Actions (Next 30 Days)

**Format**: Action tables with specific details

#### 6.1 Implement Missing Alerts

| Alert | Threshold | Expected Impact | RCA Reference |
|-------|-----------|-----------------|---------------|
| **[Alert name]** | [specific threshold] | [TTD before] → [TTD after] | #[number] |

**Requirements**:
- Specific alert names (not "availability alert")
- Concrete thresholds (e.g., ">80% for 10+ min")
- Show before/after TTD
- Reference which RCA(s) demonstrated the gap
- 5-10 alerts typical

**Examples**:
- ❌ Generic: "Add availability alert"
- ✅ Specific: "DB CPU utilization >80% for 10+ min | 20h → 2 min TTD | [#1](path/to/rca-analysis-1.md)"
- ✅ With context: "[RCA #1](path/to/rca-analysis-1.md) (DB CPU Saturation - ESVC1): 20h TTD"

---

#### 6.2 Create Runbooks

| Pattern | Services | Diagnosis Steps | Remediation |
|---------|----------|-----------------|-------------|
| **[Pattern name]** | [service list] | [step 1] → [step 2] → [step 3] | [action] |

**Requirements**:
- Pattern name (e.g., "Capacity exhaustion")
- Services affected
- High-level diagnosis flow (3-4 steps with arrows)
- Remediation action
- 3-5 runbooks typical

---

#### 6.3 Other Quick Wins

**Format**: Bullet list with brief description and impact

**Examples**:
- Pre-flight validation (capacity checks before config changes)
- Recurring alert tracking (>2x in 24h)
- Dashboard improvements (add missing metrics)

---

### 7. Short-Term Actions (60-90 Days)

**Format**: Numbered list with subsections

```markdown
### [Number]. [Action Title]

**Description**: [1-2 sentences]

**Expected impact**: [percentage] [metric]
```

**Requirements**:
- 3-5 actions
- Each has clear description and quantified impact
- Examples: Signal correlation, pattern matching, guided remediation

---

### 8. Long-Term Actions (6-12 Months)

**Format**: Same as short-term

**Requirements**:
- 2-4 actions
- Focus on automation, capacity planning, advanced features
- Examples: Auto-remediation, capacity planning, smart alerting

---

### 9. ROI Estimate (Table)

**Format**:

| Phase | Current Avg | Target | Reduction | Annual Time Saved* |
|-------|-------------|--------|-----------|-------------------|
| **Detection (TTD)** | [hours] | [hours] | [percentage] | ~[hours] |
| **Diagnosis (TTX)** | [hours] | [hours] | [percentage] | ~[hours] |
| **Resolution (TTR)** | [hours] | [hours] | [percentage] | ~[hours] |
| **Total** | [hours]/incident | [hours]/incident | **[percentage]** | **~[hours]/year** |

**Requirements**:
- Show current average, target, percentage reduction
- Calculate annual time saved
- Add footnote: "Assumes [N] incidents/year"
- Include business impact bullets below table:
  - Reduced customer impact
  - Reduced oncall burden
  - Prevented incidents
  - Cost savings (hours × hourly rate)

---

### 10. Recommendations

**Format**:
```markdown
**Approve**:
1. **[Action 1]** ([category]) → **[percentage] [metric] reduction**
2. **[Action 2]** ([category]) → **[percentage] [metric] reduction**
3. **[Action 3]** ([category]) → **[percentage] [metric] reduction**

**Expected impact**: $[amount]/year savings, [percentage] reduction in incident time

**Risk**: [level] ([reasoning])

**Timeline**:
- Immediate (alerts): [timeframe]
- Short-term (runbooks + automation): [timeframe]
- Long-term (capacity planning, auto-remediation): [timeframe]
```

**Requirements**:
- 3-4 key approvals needed
- Each shows category (detection/diagnosis/remediation) and quantified impact
- Overall expected impact (dollars and percentage)
- Risk assessment (low/medium/high with reasoning)
- Timeline breakdown by phase

---

### 11. Appendix: Data Sources

**Format**: Bullet list

**Requirements**:
- RCA documents (count, brief descriptions)
- PagerDuty/alert data (sample size)
- ICC/communication channels (message count)
- Individual analysis files (paths)
- Synthesis documents (paths)

---

## Style Guidelines

### DO

✅ **Use specific thresholds**: "DB CPU >80% for 10+ min"  
✅ **Show before/after**: "20h → 2 min TTD"  
✅ **Link RCAs**: "[RCA #1](path/to/rca-analysis-1.md) (Brief context)"  
✅ **Add context to RCA references**: "(DB CPU Saturation - ESVC1)"  
✅ **Quantify everything**: "75%", "16.5 hours", "$264K/year"  
✅ **Use tables**: Alert tables, runbook tables, ROI tables  
✅ **Use bullets**: Findings, observations, actions  
✅ **Bold key terms**: Metric names, RCA numbers, percentages  
✅ **Include ranges**: "(range: 29 min - 3 days)"

### DON'T

❌ **Generic statements**: "Add availability alert"  
❌ **Marketing language**: "game-changer", "critical", "exciting"  
❌ **Text-heavy paragraphs**: Use tables and bullets  
❌ **Vague impacts**: "Faster detection" (use percentages)  
❌ **Platform comparisons**: Focus on actions, not tool evaluation  
❌ **Future evaluation steps**: This is an action plan, not a research proposal

---

## Length Requirements

**Target**: 1-2 pages (3-5 minute read)

**Section Lengths**:
- Executive Summary: 3-4 lines
- Data Overview: 5-7 rows (table)
- Critical Findings: 3-4 findings (1/2 page total)
- Incident Summary: 1 table + 3-5 observations
- Immediate Actions: 1-2 tables + bullets
- Short-Term Actions: 3-5 items
- Long-Term Actions: 2-4 items
- ROI Estimate: 1 table + 4 bullets
- Recommendations: 3-4 approvals + impact + timeline
- Appendix: 5-7 bullets

**Maximum**: 2 pages when rendered

---

## Data Requirements

### Input Data Needed

**From RCA Analysis**:
- TTD, TTX, TTR per incident
- Detection gaps (what alert was missing)
- Root causes (why it happened)
- Resolution actions (what was done)
- Observable signals (what metrics existed)

**From Batch Synthesis**:
- Average TTD/TTX/TTR
- Recurring patterns (≥2 similar incidents)
- Alert frequency (PagerDuty data)
- Incident count (for annual projection)

**From Team Config**:
- Services and ports
- Metrics catalog
- Query patterns
- Alert definitions

### Calculations Required

**Time Reduction**:
```
Reduction % = (Current - Target) / Current × 100
```

**Annual Time Saved**:
```
Annual Hours = (Current - Target) × Incidents/Year × Affected %
```

**Cost Savings**:
```
Savings = Annual Hours × Hourly Rate
Range = Annual Hours × [Low Rate, High Rate]
```

**Typical Rates**: $200-400/hour (oncall + follow-up + opportunity cost)

---

## Validation Checklist

Before finalizing executive summary:

- [ ] Title is action-oriented (not "analysis" or "report")
- [ ] Executive summary has 3 concrete numbers (bottleneck %, automation impact %, ROI $)
- [ ] All metrics have ranges where applicable
- [ ] All alerts have specific thresholds (not generic "availability")
- [ ] All findings reference specific RCAs
- [ ] All impacts are quantified (percentages or hours)
- [ ] Tables are used for structured data (alerts, runbooks, ROI)
- [ ] Bullets are used for lists (findings, observations, actions)
- [ ] No marketing language ("game-changer", "exciting", etc.)
- [ ] No platform comparison or evaluation steps
- [ ] Length is 1-2 pages maximum
- [ ] Appendix lists all data sources

---

## Example Output

See: `rca-toolkit-draft/EXECUTIVE-SUMMARY.md`

**Structure**:
1. Action-oriented title with metadata
2. 3-line executive summary (bottleneck, impact, ROI)
3. Data overview table
4. 4 critical findings (data, root causes, impact)
5. Incident summary table
6. Immediate actions (alert table, runbook table)
7. Short-term actions (3 items)
8. Long-term actions (3 items)
9. ROI table with business impact
10. Recommendations (3 approvals, impact, risk, timeline)
11. Appendix (data sources)

**Total length**: ~2 pages, 3-5 minute read

---

## Integration with RCA Analyzer Skill

**When to generate**: After batch analysis with `--generate-summary` flag

**Required inputs**:
- Batch synthesis file (aggregate metrics)
- Individual RCA analyses (TTD/TTX/TTR, gaps, root causes)
- Team config (services, metrics, alerts)

**Template location**: `templates/executive-summary-template.md`

**Output location**: `[project-root]/EXECUTIVE-SUMMARY.md`

---

**Last Updated**: 2026-05-09  
**Status**: Production-ready
