# Runbook: Diagnosis - Capacity Exhaustion (Memory)

**Pattern ID**: `temporalhistory-capacity_exhaustion-oomkilled`  
**Trigger**: temporal-history-availability-low (PD alert)  
**Frequency**: 2 incidents in 1 month (RCA #6, future occurrences)  
**Last Occurrence**: 2025-09-06  
**Average TTD**: 29 minutes | **Average TTR**: 6h 44m

---

## Symptoms

**User Impact**:
- Temporal UI unavailable or slow
- Workflow execution failures
- Task timeouts

**System Indicators**:
- **Alert**: `temporal-history-availability-low` (PD)
- **Metric**: `container_memory_working_set_bytes` at 85-95% for multiple pods
- **Logs**: `OOMKilled` events in Kubernetes pod status
- **Behavior**: Pods in CrashLoopBackOff, intermittent restarts

---

## Diagnosis Steps

### Step 1: Verify Scope
**Action**: Check if issue is isolated to history service or affects multiple Temporal services

**Command**:
```bash
# Check history pod status
kubectl get pods -n orcaas | grep temporalhistory

# Check memory usage
kubectl top pods -n orcaas | grep temporalhistory
```

**Expected Result**: 
- One or more history pods showing high memory (>80%)
- Pod statuses: Running, CrashLoopBackOff, or OOMKilled
- Other Temporal services (frontend, matching) appear healthy

**Decision Point**:
- **If isolated to history** → Proceed to Step 2
- **If affects frontend/matching too** → Check DB health (Step 4 first)
- **If cluster-wide** → Escalate to platform team (node capacity issue)

---

### Step 2: Check Memory Trends (Backdated)
**Action**: Look at memory usage over past 24-48 hours to identify baseline creep

**Command** (Argus):
```
# Memory usage (last 48h)
1776648000000:1776778800000:temporal.temporalhistory.aws.aws-prod8-cacentral1.cdp2:container_memory_working_set_bytes{service_instance=temporalhistory3001}:avg:1m-avg

# OOMKilled events (last 48h)
1776648000000:1776778800000:temporal.temporalhistory.aws.aws-prod8-cacentral1.cdp2:kube_pod_container_status_terminated_reason{reason="OOMKilled",service_instance=temporalhistory3001}:sum:1m-sum
```

**Expected Result**: 
- Memory baseline at 60-80% for days before incident
- Gradual memory creep (not sudden spike)
- OOMKilled events occurred hours before PD alert

**Decision Point**:
- **If memory creep over days** → Root cause: No capacity planning → Proceed to Step 3A
- **If sudden spike** → Root cause: Workload surge → Proceed to Step 3B

---

### Step 3A: Validate Capacity Planning Issue
**Action**: Check if HPA/VPA is configured and if resource limits are appropriate

**Command**:
```bash
# Check HPA configuration
kubectl get hpa -n orcaas | grep temporalhistory

# Check current resource requests/limits
kubectl describe pod -n orcaas temporalhistory3001-0 | grep -A 5 "Limits\|Requests"
```

**Expected Result**: 
- No HPA configured (or HPA not responding to memory pressure)
- Memory limit: 2Gi (too low for workload)
- Memory requests: matches limits (no headroom)

**Next Action**: Refer to **Remediation Runbook**: `runbooks/remediation/resource-scaling-hpa.md`

---

### Step 3B: Validate Workload Surge
**Action**: Check if there's a sudden increase in workflow submission rate or active workflows

**Command** (Argus):
```
# Workflow start rate (last 24h)
RATE(1776692400000:1776778800000:temporal.temporalhistory.aws.aws-prod8-cacentral1.cdp2:workflow_success{service_instance=temporalhistory3001}:sum:1m-sum,#1m#,#true#,#true#)

# Active workflow count (Temporal UI)
# Go to: Namespace Dashboard → admin_service namespace → In-progress workflows
```

**Expected Result**:
- Workflow start rate spike (e.g., 50k workflows in 8 hours)
- Specific namespace shows abnormal activity (e.g., `admin_service`)
- In-progress workflow count exceeds normal baseline

**Next Action**: 
- If abnormal customer traffic → Contact customer team (e.g., CDP team)
- If legitimate traffic → Proceed to resource scaling
- If malicious/runaway → Consider rate limiting or namespace quota

---

### Step 4: Check Database Health (Cross-Service)
**Action**: Verify if DB CPU/connections are saturated (affects history + matching)

**Command** (Argus):
```
# RDS CPU usage
<RDS_CloudWatch_metric>:cpu_utilization{db_instance=temporal-prod8}:avg:1m-avg

# DB connection count
<RDS_CloudWatch_metric>:database_connections{db_instance=temporal-prod8}:sum:1m-sum
```

**Expected Result**:
- DB CPU >90% indicates DB saturation contributing to memory pressure
- Connection pool exhaustion (connections at max)

**Decision Point**:
- **If DB CPU >90%** → Root cause is DB saturation → See `runbooks/diagnosis/db-cpu-saturation.md`
- **If DB healthy** → Root cause is history service capacity → Continue to remediation

---

## Cross-Service Correlation

**Check these sibling services** (same cluster):
- **temporalmatching**: `kubectl top pods -n orcaas | grep temporalmatching`
  - Why: Shares DB, may show similar pressure if DB-related
- **temporalfrontend**: Check 5XX error rates
  - Why: Frontend depends on history, errors propagate upstream

**Sibling health indicators**:
- Matching healthy + Frontend erroring → Isolated history issue ✓
- Matching unhealthy + DB high → DB root cause ✗

---

## Common Pitfalls

**Pitfall 1: Assuming restart solves root cause**
- **Symptom**: Pod restart temporarily fixes memory, but issue recurs
- **Why wrong**: Restart clears memory but doesn't address workload or capacity
- **Correct approach**: Restart for immediate mitigation + identify capacity issue

**Pitfall 2: Ignoring baseline memory trends**
- **Symptom**: Memory at 60-80% for days before OOMKill
- **Why wrong**: Reactive approach waits for failure instead of preventing it
- **Correct approach**: Proactive alerting on sustained >70% memory

**Pitfall 3: Scaling without understanding workload**
- **Symptom**: Scale up resources, but workload is abnormal/runaway
- **Why wrong**: Wastes resources on non-production traffic
- **Correct approach**: Identify workload source first, then scale appropriately

---

## Related Incidents

- **RCA #6** (2025-09-06): Temporal History Capacity Exhaustion - prod8-cacentral1/cdp2
  - TTD: 29 min | TTR: 6h 44m
  - Root cause: 50k workflows in admin_service (8h), no HPA, 2Gi memory limit
  - Resolution: Upscale DB + increase history memory to 8Gi

---

## Metadata

**Generated**: 2026-05-09  
**Pattern Confidence**: HIGH (1 incident analyzed, pattern validated)  
**Team**: OrcaaS (Temporal)  
**Services**: temporalhistory  
**Root Cause Category**: capacity_exhaustion

---

## Prevention Recommendations

**Immediate** (do now):
- [ ] Add memory pressure alert (>70% sustained for 10+ min)
- [ ] Add OOMKilled event alert (any container restart with reason=OOMKilled)

**Short-term** (this week):
- [ ] Implement HPA for history service (scale on memory >75%)
- [ ] Increase memory limit to 8Gi (validated in RCA #6)
- [ ] Enable VPA for automatic resource tuning

**Long-term** (this quarter):
- [ ] Capacity planning: Baseline memory per X workflows
- [ ] Per-namespace resource quotas to prevent runaway workloads
- [ ] Automated load shedding for abnormal traffic
