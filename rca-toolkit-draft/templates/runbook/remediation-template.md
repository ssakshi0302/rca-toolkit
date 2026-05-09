# Runbook: Remediation - <remediation_action>

**Pattern ID**: `<pattern_key>`  
**Trigger Condition**: <when_to_use>  
**Frequency**: <frequency>  
**Last Used**: <last_occurrence>  
**Average TTR**: <avg_ttr>

---

## Prerequisites

**Before proceeding, verify**:
- [ ] Root cause diagnosed (see `runbooks/diagnosis/<diagnosis_pattern>.md`)
- [ ] User impact confirmed (<impact_level>)
- [ ] Rollback plan reviewed (see section below)
- [ ] <team_name> oncall notified

---

## Safety Checks

**Critical preconditions** (MUST verify before execution):
- [ ] **Check 1**: <safety_check_1>
  - Command: `<safety_command_1>`
  - Expected: <expected_result_1>
- [ ] **Check 2**: <safety_check_2>
  - Command: `<safety_command_2>`
  - Expected: <expected_result_2>
- [ ] **Check 3**: <safety_check_3>
  - Command: `<safety_command_3>`
  - Expected: <expected_result_3>

**STOP if any check fails** → Escalate to <escalation_team>

---

## Remediation Steps

### Step 1: <action_step_1>
**Action**: <detailed_action_description>

**Command**:
```bash
<command_1>
```

**Expected Output**:
```
<expected_output_1>
```

**Validation**:
```bash
<validation_command_1>
```

**Wait time**: <wait_duration> for propagation

**If command fails**:
- **Error**: `<common_error_message>`
- **Cause**: <error_cause>
- **Fix**: <how_to_fix>

---

### Step 2: <action_step_2>
**Action**: <detailed_action_description>

**Command**:
```bash
<command_2>
```

**Expected Output**:
```
<expected_output_2>
```

**Validation**:
```bash
<validation_command_2>
```

**Wait time**: <wait_duration> for propagation

---

### Step 3: <action_step_3>
**Action**: <detailed_action_description>

**Command**:
```bash
<command_3>
```

**Expected Output**:
```
<expected_output_3>
```

**Validation**:
```bash
<validation_command_3>
```

---

## Verification

**After remediation, verify** (in order):

### Verification 1: Service Health
**Check**: <service> is healthy and responding

**Command**:
```bash
<service_health_check>
```

**Success Criteria**: 
- <metric_1> back to baseline (<threshold_1>)
- <metric_2> stable for 5 minutes (<threshold_2>)

---

### Verification 2: Error Rate
**Check**: Error rate returned to normal

**Command**:
```bash
<error_rate_check>
```

**Success Criteria**: 
- Error rate <1% for 5 minutes
- No new errors in last 5 minutes

---

### Verification 3: User Impact
**Check**: No ongoing customer reports

**Command**:
```bash
<user_impact_check>
```

**Success Criteria**: 
- No reports in <incident_channel>
- Support tickets: none related in last 10 minutes

---

## Rollback Plan

**If remediation fails or makes things worse**:

### Rollback Step 1: <rollback_action_1>
**Command**:
```bash
<rollback_command_1>
```

**Expected**: <rollback_result_1>

---

### Rollback Step 2: <rollback_action_2>
**Command**:
```bash
<rollback_command_2>
```

**Expected**: <rollback_result_2>

---

### Rollback Verification
**Command**:
```bash
<rollback_verification_command>
```

**Expected**: System returns to pre-remediation state

---

## Post-Incident Actions

**Immediate** (within 1 hour):
- [ ] Update incident ticket: <ticket_url>
- [ ] Notify <team_name> of resolution
- [ ] Document any deviations from this runbook

**Short-term** (this week):
- [ ] <short_term_action_1>
- [ ] <short_term_action_2>

**Long-term** (this quarter):
- [ ] <long_term_action_1>
- [ ] <long_term_action_2>

---

## Related Runbooks

**Diagnosis**: `runbooks/diagnosis/<diagnosis_pattern>.md`  
**Alternative Remediations**: 
- `runbooks/remediation/<alternative_1>.md` (if <condition>)
- `runbooks/remediation/<alternative_2>.md` (if <condition>)

---

## Related Incidents

<related_rca_list>

---

## Metadata

**Generated**: <generation_timestamp>  
**Pattern Confidence**: <confidence_level> (<pattern_count> incidents match)  
**Team**: <team_name>  
**Services**: <service_list>  
**Remediation Category**: <remediation_category>
