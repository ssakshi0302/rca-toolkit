# Runbook: Diagnosis - <pattern_name>

**Pattern ID**: `<pattern_key>`  
**Trigger**: <alert_name>  
**Frequency**: <frequency>  
**Last Occurrence**: <last_occurrence>  
**Average TTD**: <avg_ttd> | **Average TTR**: <avg_ttr>

---

## Symptoms

**User Impact**:
- <user_symptom_1>
- <user_symptom_2>

**System Indicators**:
- **Alert**: <alert_name>
- **Metric**: `<metric_name>` shows <threshold>
- **Logs**: `<log_pattern>`

---

## Diagnosis Steps

### Step 1: Verify Scope
**Action**: Check if issue is isolated to <service> or affects multiple services

**Command**:
```
<scope_check_command>
```

**Expected Result**: <what_to_look_for>

**Decision Point**:
- **If isolated to <service>** → Proceed to Step 2
- **If affects multiple services** → Check sibling services (<sibling_list>)
- **If cluster-wide** → Escalate to <escalation_team>

---

### Step 2: Identify Root Cause
**Action**: <specific_investigation_action>

**Command**:
```
<investigation_command>
```

**Expected Result**: <what_indicates_root_cause>

**Decision Point**:
- **If <condition_A>** → Root cause is <root_cause_A>, proceed to Step 3A
- **If <condition_B>** → Root cause is <root_cause_B>, proceed to Step 3B
- **If unclear** → Proceed to Step 3 (general validation)

---

### Step 3A: Validate Root Cause A - <root_cause_A>
**Action**: Confirm <root_cause_A> via additional evidence

**Command**:
```
<validation_command_A>
```

**Expected Result**: <confirmation_signal_A>

**Next Action**: Refer to **Remediation Runbook**: `runbooks/remediation/<remediation_pattern_A>.md`

---

### Step 3B: Validate Root Cause B - <root_cause_B>
**Action**: Confirm <root_cause_B> via additional evidence

**Command**:
```
<validation_command_B>
```

**Expected Result**: <confirmation_signal_B>

**Next Action**: Refer to **Remediation Runbook**: `runbooks/remediation/<remediation_pattern_B>.md`

---

### Step 3: General Impact Validation
**Action**: Check current error rate, latency, and user-facing impact

**Command**:
```
<impact_check_command>
```

**Expected Result**: 
- Error rate: <threshold>
- Latency (P99): <threshold>
- User reports: <source>

**Decision Point**:
- **If actively impacting users** → Proceed to immediate remediation
- **If auto-resolved** → Skip remediation, proceed to verification only

---

## Cross-Service Correlation

**Check these sibling services** (same cluster):
- <sibling_service_1>: `<sibling_check_command_1>`
- <sibling_service_2>: `<sibling_check_command_2>`

**Why**: <reason_for_checking_siblings>

---

## Common Pitfalls

**Pitfall 1: <common_mistake>**
- **Symptom**: <how_it_looks>
- **Why wrong**: <explanation>
- **Correct approach**: <what_to_do_instead>

**Pitfall 2: <common_mistake>**
- **Symptom**: <how_it_looks>
- **Why wrong**: <explanation>
- **Correct approach**: <what_to_do_instead>

---

## Related Incidents

<related_rca_list>

---

## Metadata

**Generated**: <generation_timestamp>  
**Pattern Confidence**: <confidence_level> (<pattern_count> incidents match)  
**Team**: <team_name>  
**Services**: <service_list>  
**Root Cause Category**: <root_cause_category>
