# Feature Proposal: PagerDuty Pattern Matching

**Status**: Proposed for v1.1  
**Complexity**: Medium  
**Value**: High (automatic pattern detection from live alerts)

---

## Problem

**Current flow**:
1. Incident happens → PD alert fires
2. Team resolves incident
3. Team writes RCA in Google Doc
4. **Days/weeks later**: Team runs rca-analyzer on Google Doc
5. Toolkit identifies automation opportunities

**Gap**: Pattern detection happens **after** RCA is written, not during incident response.

---

## Proposed Feature

**Add PD link analysis** to identify patterns from historical alerts:

```bash
# Analyze alert pattern
/rca-analyzer --pagerduty <pd-incident-url>

# Example
/rca-analyzer --pagerduty https://salesforce.pagerduty.com/incidents/Q1234567890
```

**What it does**:
1. Fetches PD incident data via API
2. Looks up past incidents with same alert name
3. Identifies recurring patterns (≥2 occurrences)
4. Suggests runbook if pattern exists
5. Optionally suggests similar RCAs to reference

---

## Use Cases

### Use Case 1: Live Incident Guidance
**Scenario**: Alert fires, oncall engineer receives PD notification

**Workflow**:
```bash
# Copy PD incident URL from notification
/rca-analyzer --pagerduty https://salesforce.pagerduty.com/incidents/Q1234567890 --suggest-runbook

Output:
✅ Alert Pattern Analysis
├─ Alert: temporal-history-availability-low
├─ Occurrences: 3 times in last 90 days
├─ Pattern: capacity_exhaustion (2/3 incidents)
├─ Suggested Runbook: runbooks/diagnosis/temporalhistory-capacity-exhaustion.md
└─ Similar Incidents:
   ├─ PRB-0028677 (2025-09-06) - RCA available
   ├─ PRB-0025432 (2025-08-12) - RCA available
   └─ PRB-0023891 (2025-07-05) - No RCA (auto-resolved)
```

**Benefit**: Oncall gets immediate runbook guidance during incident

---

### Use Case 2: Post-Incident Pattern Check
**Scenario**: After incident resolution, check if pattern exists before writing RCA

**Workflow**:
```bash
/rca-analyzer --pagerduty <url> --check-pattern

Output:
✅ Pattern Check
├─ Alert: temporal-frontend-availability-low
├─ Occurrences: 1 time in last 90 days
├─ Pattern: No recurring pattern (unique incident)
└─ Recommendation: Write detailed RCA, this is first occurrence
```

**Benefit**: Teams know if they need detailed RCA or can reference existing pattern

---

### Use Case 3: Runbook Coverage Check
**Scenario**: Quarterly review - which alerts lack runbooks?

**Workflow**:
```bash
/rca-analyzer --pagerduty-summary --last-90-days

Output:
✅ Alert Summary (Last 90 Days)
├─ Total Alerts: 47
├─ Unique Alerts: 12
├─ Recurring Patterns: 4
│  ├─ temporal-history-availability-low (8x) - Has Runbook ✅
│  ├─ temporal-frontend-availability-low (6x) - Has Runbook ✅
│  ├─ temporal-db-cpu-high (5x) - Missing Runbook ❌
│  └─ temporal-worker-queue-backlog (3x) - Missing Runbook ❌
└─ Recommendation: Create runbooks for 2 recurring patterns
```

**Benefit**: Identify runbook gaps based on actual alert frequency

---

## Technical Design

### Data Sources

**PagerDuty API** (via MCP):
```yaml
required_data:
  - incident_id
  - alert_name
  - created_at
  - resolved_at
  - service_name
  - urgency
  - notes (for manual diagnosis steps)
  - related_incidents (if available)
```

**Team Config** (existing):
```yaml
alerts:
  temporal-history-availability-low:
    services: [temporalhistory]
    common_causes: [capacity_exhaustion, db_saturation]
    runbook_pattern: diagnosis/capacity-exhaustion
```

**Existing RCAs** (local):
```
research/past rca/
├── rca-analysis-1.md (metadata: alert name, pattern)
├── rca-analysis-2.md
└── ...
```

---

### Pattern Matching Logic

```python
def analyze_pd_incident(pd_url, config):
    # 1. Fetch PD incident data
    incident = fetch_pd_incident(pd_url)
    alert_name = incident.alert_name
    
    # 2. Find historical incidents with same alert
    historical = query_pd_api(
        alert_name=alert_name,
        time_range="90d",
        service=incident.service
    )
    
    # 3. Match to team config patterns
    alert_config = config.alerts.get(alert_name)
    common_causes = alert_config.common_causes if alert_config else []
    
    # 4. Check for existing runbook
    runbook_pattern = alert_config.runbook_pattern if alert_config else None
    runbook_exists = check_runbook_exists(runbook_pattern)
    
    # 5. Find similar RCAs
    similar_rcas = search_local_rcas(
        alert_name=alert_name,
        services=alert_config.services,
        common_causes=common_causes
    )
    
    # 6. Output pattern analysis
    return {
        "alert": alert_name,
        "occurrences": len(historical),
        "pattern": identify_pattern(historical, similar_rcas),
        "runbook": runbook_pattern,
        "runbook_exists": runbook_exists,
        "similar_rcas": similar_rcas
    }
```

---

### Skill Arguments

**Add to existing skill**:
```yaml
Options:
  --pagerduty <url>              # Analyze PD incident pattern
  --suggest-runbook              # Suggest runbook if pattern exists
  --check-pattern                # Check if recurring pattern
  --pagerduty-summary            # Summary of all alerts
  --last-90-days                 # Time range for summary (default: 90d)
```

**Examples**:
```bash
# Live incident guidance
/rca-analyzer --pagerduty <url> --suggest-runbook

# Post-incident pattern check
/rca-analyzer --pagerduty <url> --check-pattern

# Quarterly runbook coverage review
/rca-analyzer --pagerduty-summary --last-90-days
```

---

## Implementation Phases

### Phase 1: Basic PD Integration (v1.1)
**Scope**: 
- Fetch PD incident data via API
- Match to team config alert mappings
- Suggest runbook if exists
- List similar RCAs

**Effort**: 2-3 days

**Value**: Oncall gets immediate runbook guidance

---

### Phase 2: Pattern Detection (v1.2)
**Scope**:
- Query PD for historical incidents (same alert)
- Identify recurring patterns (≥2 occurrences in 90d)
- Suggest runbook creation if pattern but no runbook

**Effort**: 2-3 days

**Value**: Automated runbook gap identification

---

### Phase 3: Runbook Coverage Dashboard (v1.3)
**Scope**:
- Summary view: all alerts, frequency, runbook coverage
- Identify top 5 alerts without runbooks
- Prioritize by frequency × impact

**Effort**: 1-2 days

**Value**: Quarterly planning data for automation roadmap

---

## Benefits

### For Oncall Engineers
- **Live guidance**: Get runbook suggestion immediately when alert fires
- **Context**: See similar past incidents during diagnosis
- **Faster resolution**: Follow proven diagnosis steps

### For Teams
- **Pattern visibility**: Know which alerts recur frequently
- **Runbook gaps**: Identify which patterns lack runbooks
- **ROI data**: Quantify alert frequency × average TTR

### For EM/Leadership
- **Coverage metrics**: % of alerts with runbooks
- **Automation roadmap**: Prioritize by frequency
- **Trend analysis**: Are alerts decreasing over time?

---

## Considerations

### Data Privacy
**Question**: PD incidents may contain customer data or sensitive info  
**Solution**: Only fetch metadata (alert name, timing, service), not incident notes

### API Rate Limits
**Question**: PD API has rate limits  
**Solution**: Cache results, batch queries, add backoff logic

### Stale Runbooks
**Question**: Runbook exists but is outdated  
**Solution**: Track runbook last-updated date, flag if >6 months old

### False Patterns
**Question**: Same alert, different root causes  
**Solution**: Require ≥2 occurrences **with same root cause category** (from RCA metadata)

---

## Open Questions

1. **Should PD analysis replace Google Doc analysis?**
   - **No**: Both are useful. PD for live guidance, Google Doc for detailed post-incident
   
2. **Should we auto-create RCAs from PD incidents?**
   - **No**: PD lacks depth. Teams should write detailed RCAs, toolkit analyzes them
   
3. **Should we integrate with ICC (Incident Command Center)?**
   - **Maybe**: ICC has richer incident data (timeline, diagnosis steps, resolution). Consider for v2.0

4. **Should we store PD data locally or query on-demand?**
   - **Query on-demand**: Avoids stale data, respects PD as source of truth

---

## Recommendation

**Implement in phases**:
1. **v1.0** (current): Focus on Google Doc analysis, get toolkit adopted
2. **v1.1** (3-6 months): Add basic PD integration (fetch + suggest runbook)
3. **v1.2** (6-9 months): Add pattern detection from PD history
4. **v1.3** (9-12 months): Add runbook coverage dashboard

**Why phased**:
- v1.0 delivers core value (RCA analysis, automation opportunities)
- v1.1 adds live guidance (helpful but not critical)
- v1.2-1.3 add strategic planning features (nice-to-have)

**Start with v1.0, collect feedback, then decide if PD integration is high priority.**

---

## Decision Needed

- [ ] **Yes, add to roadmap** - High value for oncall engineers
- [ ] **Yes, but later** - Focus on v1.0 adoption first (recommended)
- [ ] **No, out of scope** - Keep toolkit focused on post-incident analysis
- [ ] **Prototype first** - Test with Temporal team, then decide

---

**Your input?**
