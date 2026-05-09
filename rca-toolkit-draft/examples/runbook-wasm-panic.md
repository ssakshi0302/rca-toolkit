# Runbook: Diagnosis - Frontend WASM Panic (C2C Auth Timeout)

**Pattern ID**: `temporalfrontend-wasm_panic-c2c_timeout`  
**Trigger**: temporal-frontend-availability-low (5XX errors) + WASM panic logs  
**Frequency**: 1 incident (10-day duration with 2-3 daily occurrences) - RCA #5  
**Last Occurrence**: 2025-08-22  
**Average TTD**: 1 minute (alert) | **Average TTX**: 8 hours (root cause) | **Average TTR**: 22.9 hours

---

## Symptoms

**User Impact**:
- Temporal UI unavailable (503 errors)
- Intermittent access (recovers after pod restart)
- Multiple customer teams reporting issues

**System Indicators**:
- **Alert**: `temporal-frontend-availability-low` (5XX errors >threshold)
- **Metric**: Frontend 5XX error rate spike, then auto-resolves in ~1h
- **Logs**: `wasm` panic errors in Envoy sidecar logs
- **Behavior**: Frontend pods appear healthy but Envoy sidecar crashes intermittently

---

## Diagnosis Steps

### Step 1: Verify Scope
**Action**: Check if issue is isolated to frontend or affects backend services

**Command**:
```bash
# Check frontend pod status
kubectl get pods -n orcaas | grep temporalfrontend

# Check for CrashLoopBackOff or restarts
kubectl get pods -n orcaas -o wide | grep temporalfrontend | awk '{print $4}'
```

**Expected Result**:
- Frontend pods show high restart count (not immediately obvious)
- Pods appear "Running" but Envoy sidecar is crashing
- Backend services (history, matching) are healthy

**Decision Point**:
- **If isolated to frontend + pods restarting** → Proceed to Step 2
- **If backend services also affected** → Check DB/mesh health first
- **If no restarts visible** → Check sidecar logs (Step 3)

---

### Step 2: Check for WASM Panic Errors
**Action**: Search Splunk for WASM-specific panic patterns

**Command** (Splunk):
```splunk
index=distapps 
  falcon_instance=*prod1-* 
  functional_domain=foundation 
  k8s_namespace=orcaas
  k8s_pod_name=temporalfrontend1-*
  ("wasm" AND ("panic" OR "RuntimeError: unreachable" OR "unexpected status"))
  earliest=<incident_start_epoch> 
  latest=<incident_end_epoch>
  | stats count by _time span=5m
  | sort -_time
```

**Expected Result**:
- WASM panic errors correlated with 5XX error spikes
- Panic message: `RuntimeError: unreachable` or `unexpected status`
- Errors occur intermittently (not continuous)

**Decision Point**:
- **If WASM panics present** → Proceed to Step 3 (identify panic trigger)
- **If no WASM panics** → Different root cause (check mesh config, deployment timing)

---

### Step 3: Identify Panic Trigger (C2C Auth Timeout)
**Action**: Check for missing headers or timeout-related errors preceding WASM panic

**Command** (Splunk):
```splunk
index=distapps 
  falcon_instance=*prod1-* 
  functional_domain=foundation 
  k8s_namespace=orcaas
  k8s_pod_name=temporalfrontend1-*
  ("Dispatched call for token validation" OR "No headers present in HTTP call response")
  earliest=<incident_start_epoch> 
  latest=<incident_end_epoch>
  | transaction call_id maxpause=15s
  | eval gap = _time_diff
  | where gap >= 2
  | table _time, call_id, gap, log_message
```

**Expected Result**:
- Token validation call → No response for 2+ seconds
- Log message: `No headers present in HTTP call response`
- Gap between "Dispatched call" and response ≥2 seconds (C2C timeout)

**Root Cause**: C2C auth service timeout → WASM extension expects headers → panic on missing headers

---

### Step 4: Check C2C Auth Latency
**Action**: Validate C2C upstream latency approaching or exceeding timeout threshold

**Command** (Grafana - Mesh Service-to-Service Dashboard):
```
Dashboard: https://moncloud-grafana.sfproxy.monitoring.aws-esvc1-useast2.aws.sfdc.cl/d/RqisbMqWk/mesh-service-to-service

Filters:
- originating_service: temporalfrontend
- destination_service: c2c-public
- Metric: P95/P99 latency

Time range: Last 48 hours (compare before/during incident)
```

**Expected Result**:
- P95 latency: 1.5-1.9 seconds (approaching 2s timeout)
- P99 latency: >2 seconds (exceeding timeout)
- Latency spike correlated with WASM panic timing

**Decision Point**:
- **If C2C latency >2s** → Root cause: Upstream timeout → Proceed to remediation
- **If C2C latency normal** → Check for config change or WASM version upgrade

---

### Step 5: Verify WASM Extension Configuration
**Action**: Check if WASM extension has graceful timeout handling

**Command**:
```bash
# Check Envoy config for WASM extension
kubectl exec -it temporalfrontend1-0 -c istio-proxy -n orcaas -- cat /etc/istio/proxy/envoy-rev.json | jq '.static_resources.listeners[].filter_chains[].filters[] | select(.name=="envoy.filters.http.wasm")'
```

**Expected Result**:
- WASM extension configured for C2C auth
- No graceful timeout handling for missing headers (bug)
- Extension version: <check_version>

**Next Action**: Refer to **Remediation Runbook**: `runbooks/remediation/wasm-graceful-timeout.md`

---

## Cross-Service Correlation

**Check these services** (related to C2C auth):
- **C2C Auth Service** (c2c-public):
  - Dashboard: Mesh Service-to-Service → c2c-public latency
  - Why: Upstream timeout is root cause
- **Mesh Service (Istio)**:
  - Check for recent mesh upgrades or config changes
  - Why: WASM extension runs in Envoy sidecar

**Correlation indicators**:
- C2C latency spike + WASM panics → C2C timeout root cause ✓
- Mesh config change + WASM panics → Mesh misconfiguration ✗

---

## Common Pitfalls

**Pitfall 1: Assuming rolling restart solves root cause**
- **Symptom**: Restart fixes issue temporarily, but panics recur within hours
- **Why wrong**: Restart clears panic state but doesn't fix timeout handling
- **Correct approach**: Restart for mitigation + deploy graceful timeout fix

**Pitfall 2: Missing WASM panic correlation**
- **Symptom**: 5XX errors visible, but WASM panics not checked
- **Why wrong**: Envoy sidecar logs aren't checked by default in pod health
- **Correct approach**: Always check istio-proxy container logs for WASM errors

**Pitfall 3: Focusing on frontend app, not sidecar**
- **Symptom**: Frontend application logs show no errors
- **Why wrong**: Panic occurs in Envoy sidecar (istio-proxy), not app container
- **Correct approach**: Check Envoy sidecar logs first for mesh-related issues

**Pitfall 4: Auto-resolved alerts masking persistent issue**
- **Symptom**: PD alerts fire and auto-resolve within 1 hour
- **Why wrong**: Temporary recovery (pod restart) masks recurring problem
- **Correct approach**: Track alert recurrence (>2x in 24h = persistent issue)

---

## Related Incidents

- **RCA #5** (2025-08-12 to 2025-08-22): Temporal Frontend WASM Panic - prod1/foundation
  - Duration: 10 days (with 2-3 daily restarts)
  - TTD: 1 min (alert fires quickly)
  - TTX: 8 hours (10 days to root cause)
  - TTR: 22.9 hours (from escalation to fix deployment)
  - Root cause: C2C timeout (2s) → WASM panic on missing headers
  - Resolution: Deploy graceful header handling in WASM extension

---

## Metadata

**Generated**: 2026-05-09  
**Pattern Confidence**: HIGH (1 incident analyzed, pattern validated)  
**Team**: OrcaaS (Temporal)  
**Services**: temporalfrontend, c2c-public (upstream)  
**Root Cause Category**: wasm_panic (upstream_timeout)

---

## Prevention Recommendations

**Immediate** (do now):
- [ ] Add alert for WASM panic errors in Envoy logs
- [ ] Add alert for recurring PD alerts (>2x in 24h same cluster)
- [ ] Add alert for C2C auth latency (P95 >1.5s, P99 >2s)

**Short-term** (this week):
- [ ] Deploy graceful timeout handling in WASM extension (validate in test first)
- [ ] Enable debug-level logs for C2C token validation (temporary for diagnosis)
- [ ] Create Tier 1 runbook for "Frontend 503 + WASM panic" pattern

**Long-term** (this quarter):
- [ ] Implement health probe failure detection (liveness probes for sidecar)
- [ ] Automated restart on WASM panic (with alerting)
- [ ] SLO for C2C auth latency (<1s P95, <1.5s P99)
- [ ] Upstream timeout configuration tuning (increase to 5s with graceful handling)

---

## Remediation Path

**Immediate Mitigation**:
1. Rolling restart of affected frontend pods (temporary fix)
2. Monitor for recurrence within 1-2 hours

**Permanent Fix**:
1. Deploy WASM extension patch with graceful timeout handling
2. Test in test1/foundation first
3. Deploy to production via EBF (Emergency Bug Fix)
4. Monitor for 3-4 days to confirm no recurrence

**Rollback Plan**:
- If patch causes new issues, revert WASM extension to previous version
- Continue tactical restarts until proper fix is identified
