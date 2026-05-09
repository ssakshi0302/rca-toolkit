# RCA Analysis #5: Temporal Frontend WASM Panic - C2C Auth Timeout

## Summary
- **Date**: 2025-08-12 (Incident 1), 2025-08-22 (Incident 2 - Escalation)
- **Service**: temporalfrontend
- **Environment**: prod1/foundation (HIGH PRIORITY)
- **Severity**: SEV-3 (Incident 1), SEV-2 (Incident 2)
- **PRB**: PRB-a4QEE0000006hyL, PRB-a4QEE0000006kzR
- **ICC**: #icc-79170096, #icc-79599303
- **Timeline**: 
  - Incident 1: TTD: 1min | Diagnosis: 32min | TTR: 50min
  - Incident 2: TTD: 1min | Diagnosis: 480min (8h) | TTR: 1372min (22.9h)

## Where We're Lacking (Key Insight)

### Detection
**Delay**: Alerts fired for 5XX errors but not for WASM panics. Multiple PD alerts auto-resolved in ~1h, masking persistent issue.
**Automation Opportunity**: Alert on WASM panic errors in Envoy logs + health probe failure patterns

### Diagnosis
**Delay**: 10 days to identify root cause (Aug 12 - Aug 22). Could not reproduce locally until timeout scenario was identified.
**Missing Data**: 
- No visibility into WASM-level panics (Envoy sidecar crashes)
- Info-level logs only; debug logs would have traced exact failure path
- No metrics on C2C token validation latency/timeouts
**Skill Opportunity**: Tier 1 runbook for "Frontend 503 + WASM panic" pattern

### Resolution
**Delay**: Relied on 2-3 daily rolling restarts as tactical workaround for 10 days while investigating. Permanent fix took 10 days from first incident to deployment.
**Approval Process**: EBF (Emergency Bug Fix) required for production deployment
**Automation Potential**: Auto-restart on health probe failures (liveness probes) - planned as remediation

---

## Timeline Details

### Incident 1: Aug 12, 2025 (SEV-3)

| Phase | Timestamp (IST) | Duration | Notes |
|-------|-----------------|----------|-------|
| Incident Start | Aug 11, 10:37 PM | 0 | First PD alert - auto-resolved in 1h |
| Detection | Aug 11, 10:37 PM | TTD: 1min | PD alert fired on 5XX errors |
| Customer Report | Aug 12, 11:01 AM | +12.4h | M&J team reports UI unavailability |
| Mesh Team Engaged | Aug 12, 12:03 PM | +13.4h | Investigated mesh timeouts (504 errors) |
| Root Cause ID | Aug 12, 4:35 PM | +18h | Identified 504 errors, mesh suggested WASM issue but not confirmed |
| ICC Created | Aug 12, 4:03 PM | +17.4h | SEV-3 incident opened |
| Mitigation | Aug 12, 4:54 PM | TTR: 50min | Rolling restart of frontend pods |
| Customer Confirmed | Aug 12, 7:35 PM | +21h | M&J team confirmed resolution |
| ICC Closed | Aug 12, 8:16 PM | +21.7h | Incident closed (prematurely - issue recurred) |

### Incident 2: Aug 22, 2025 (SEV-2 - Escalation)

| Phase | Timestamp (IST) | Duration | Notes |
|-------|-----------------|----------|-------|
| Persistent Issues | Aug 13-21 | Ongoing | Multiple PD alerts, rolling restarts 2-3x daily |
| Escalation | Aug 22, 5:23 AM | +10 days | New SEV-2 ICC due to persistent issue + missed updates |
| Root Cause ID | Aug 22, 1:26 PM | +8h | Confirmed WASM panic from missing headers on C2C timeout |
| Fix Created | Aug 21, 10:00 PM | -7.5h | PR for graceful header handling (created day before escalation) |
| Fix Deployed (test1) | Aug 23, 3:32 AM | +22h | Validated in test1/foundation |
| Fix Deployed (prod1) | Aug 23, 4:17 AM | +23h | EBF deployment to prod1 |
| Monitoring Period | Aug 23-26 | +3 days | Observed cluster for panic errors |
| ICC Resolved | Aug 27, 1:39 AM | TTR: 1372min (22.9h from escalation) | No further panics observed |
| Root Cause Confirmed | Aug 28, 4:42 AM | +4.3 days | Log evidence of 2s timeout confirmed |

---

## Diagnostic Details (For Runbooks)

### Metrics Checked

| Metric | Dashboard | Threshold | What It Revealed |
|--------|-----------|-----------|------------------|
| 5XX Error Rate | Temporal UI Monitoring | >threshold (exact value not specified) | Triggered PD alerts, indicated frontend unavailability |
| Mesh Latency (egress) | [Mesh Service-to-Service Dashboard](https://moncloud-grafana.sfproxy.monitoring.aws-esvc1-useast2.aws.sfdc.cl/d/RqisbMqWk/mesh-service-to-service) | 2s | C2C requests hitting 2s latency limit (possibly higher but dropped at timeout) |
| Pod Health | Kubernetes | N/A | Pods appeared healthy initially, but experiencing WASM crashes |
| Frontend Cron FIT Test | FIT Slack Alerts | Test failure | Indicated connectivity issues but didn't pinpoint WASM panics |

### Logs Fetched

#### Splunk Query 1: WASM Panic Errors
```
index=* service=temporalfrontend "wasm" "panic" OR "RuntimeError: unreachable"
earliest="2025-08-11T10:37:00" latest="2025-08-27T23:59:59"
```
**What it revealed**: 
- WASM module panics due to `unexpected status: 2` at line `/work/tools/external-deps/proxy-wasm-rust-sdk/src/hostcalls.rs:248:23`
- Function `proxy_on_http_call_response` failed with "Uncaught RuntimeError: unreachable"
- Stack trace points to `get_map_value` attempting to read `:status` header that doesn't exist

**Example Log**:
```
2025-08-20T08:13:52.650592Z critical envoy wasm: panicked at /work/tools/external-deps/proxy-wasm-rust-sdk/src/hostcalls.rs:248:23:unexpected status: 2 thread=104
2025-08-20T08:13:52.650711Z error envoy wasm: Function: proxy_on_http_call_response failed: Uncaught RuntimeError: unreachable
```

#### Splunk Query 2: Missing Headers (Post-Fix Logging)
```
index=* service=temporalfrontend "No headers present in HTTP call response"
earliest="2025-08-27T00:00:00"
```
**What it revealed**: 
- 2-second gap between dispatch call and error log confirms timeout issue
- C2C token validation requests timing out, resulting in empty response headers

**Example Log**:
```
2025-08-27T23:12:13.963373Z info: Dispatched call for token validation req-id 68e86988-6f31-40e4-ad53-412219550799
2025-08-27T23:12:15.949262Z error: CTX: 2060898 No headers present in HTTP call response for call_id 1173816
```

#### Splunk Query 3: 504 Gateway Timeout
```
index=* service=temporalfrontend status=504
```
**What it revealed**: Frontend returning 504 errors when C2C auth API calls exceeded 2s timeout

### Investigation Steps

1. **Check service pod health** → Result: Dead-end initially (pods appeared healthy but WASM sidecars crashing)
2. **Engage Mesh team for 504 timeout investigation** → Result: Helped - suggested increasing browser timeout (didn't work) then identified WASM panic logs
3. **Thread with SFAP team to check API changes** → Result: Dead-end - no changes on SFAP side
4. **Tactical: Rolling restart of frontend pods** → Result: Helped temporarily - cleared WASM crashes, restored service for ~hours
5. **Analyze WASM panic stack trace with Mesh team** → Result: Helped - identified panic at header retrieval using `.expect()` and `.unwrap()`
6. **Attempt local reproduction** → Result: Dead-end initially - couldn't reproduce missing headers scenario locally
7. **Test timeout scenarios** → Result: Helped - identified that 2s timeout causes empty response
8. **Test service unreachable scenarios** → Result: Dead-end - gave 503 errors, not panics
9. **Add debug logging to WASM code** → Result: **BREAKTHROUGH** - confirmed 2s gap between dispatch and response
10. **Consult C2C team on timeout recommendations** → Result: Helped - recommended 10s timeout (aligned with CASC implementation)
11. **Create PR with graceful header handling** → Result: Helped - permanent fix deployed

**What finally led to root cause**: 
- Combination of (1) Mesh team's stack trace analysis pointing to `.expect()` on missing headers, (2) reproducing timeout scenario locally, (3) added debug logging showing 2s gap, and (4) C2C team confirming latency issues and recommending 10s timeout.

---

## What Fixed It

### Action
Updated WASM code to gracefully handle missing response headers when C2C token validation times out + increased timeout from 2s to 10s

### Resolution Type
CONFIG_CHANGE + CODE_FIX (WASM extension update)

### Commands Used

#### Tactical Mitigation (Repeated 2-3x daily for 10 days):
```bash
# Create MO case for rolling restart
# Example: http://gus.lightning.force.com/lightning/r/Case/500EE00001bc1mvYAA/view

# Execute Managed Operation for rolling restart
# Falcon MO execution: 31f44cfb-f3db-4c83-957e-f243859b2a17
# Service: temporalfrontend, Team: orcaas
```

#### Permanent Fix:
```bash
# 1. PR created for WASM extension fix
# https://git.soma.salesforce.com/servicemesh/wasm-extensions/pull/1596/files
# Changes: Replace .expect()/.unwrap() with graceful error handling for missing headers

# 2. Deploy to test1/foundation (validation)
# Aug 23, 3:32 AM IST

# 3. Deploy to prod1/foundation via EBF
# EBF Case: 500EE00001c8qPdYAI
# Aug 23, 4:17 AM IST

# 4. Post-deployment rolling restart
# Aug 23, 4:46 AM IST
```

### Approval Process
- **Approval needed**: EBF (Emergency Bug Fix) for production deployment
- **Delay**: Not explicitly stated, but EBF process required for prod1 deployment
- **Additional approvals**: MO (Managed Operation) cases required for each rolling restart (tactical mitigation)

### Validation

**Metrics checked**:
1. PD alerts - no further 5XX alerts after fix deployment
2. WASM panic errors in Splunk - zero panics observed Aug 23-26
3. Frontend pod health - stable, no crashes
4. FIT test results - passing
5. Customer confirmation - M&J team reported no issues

**Time to confirm**: 3 days monitoring period (Aug 23-26) before resolving incident on Aug 27

### Reproducible?
**CONDITIONAL**

**Fix is reproducible IF**:
- C2C authentication is enabled for the service
- C2C token validation requests experience >2s latency (pre-fix) or >10s latency (post-fix)
- WASM extension has error handling for missing response headers

**Components**:
1. WASM code fix: Replace `.expect("missing status code")` and `.unwrap()` with graceful error handling
2. Timeout increase: 2s → 10s for C2C token validation calls (align with CASC team recommendation)
3. Enhanced logging: Add debug logs to trace call stack paths in WASM

**Other environments**: Issue occurred in test1/foundation as well but auto-recovered via pod restarts. Fix applicable to any environment with C2C auth enabled (dev1, test1, prod1 for M&J team).

---

## Automation Opportunities

### Detection
**Could automation detect faster?** YES

**Current State**:
- PD alerts fire on 5XX error rate threshold (UI-based)
- Alerts auto-resolve in ~1h when errors subside (after rolling restart or pod auto-recovery)
- No visibility into WASM-level panics or Envoy sidecar crashes
- FIT test failures indicate connectivity issues but don't pinpoint root cause

**Approach**:
1. **Alert on WASM panic patterns** (W-19601626):
   - Monitor Envoy logs for "wasm panic" or "RuntimeError: unreachable"
   - Alert threshold: >1 WASM panic per pod in 5min window
   - Requires evaluation if WASM metrics available for panic detection

2. **Alert on health probe failure patterns** (W-19646264):
   - Track liveness/readiness probe failures
   - Alert when >X% of pods fail health checks within Y minutes
   - Correlate with 5XX error spikes

3. **Alert on C2C token validation timeouts**:
   - Monitor mesh latency for temporalfrontend → c2c-public calls
   - Alert when P95 latency >5s (50% of new timeout) or P99 >8s
   - Track timeout rate: (timeouts / total requests) >1%

4. **Prevent alert auto-resolution for recurring issues**:
   - If same alert fires >2x within 6h, do NOT auto-resolve
   - Escalate to on-call with "recurring issue" tag

**Estimated TTD reduction**: 
- Current: 1min (already fast, but alerts auto-resolved masking persistence)
- With WASM panic alerts: Immediate detection + prevents premature closure
- Impact: Reduces **time to acknowledge issue is persistent** from 10 days → <1 day

---

### Diagnosis

#### Tier 1: Skills/Runbooks (Quick win - 1-2 weeks)

**Skill name**: `temporal-frontend-503-wasm-panic-diagnosis`

**Trigger**: PD alert for frontend 5XX errors + WASM panic logs OR FIT test failures for frontend

**Steps**:
1. **Check if PD alert auto-resolved in <2h**
   - If YES and alert fired >1x in past 24h → Likely recurring issue, keep investigating
   - Query: `index=* service=temporalfrontend status=503 OR status=504 | stats count by _time span=1h`

2. **Search for WASM panic errors**
   ```
   index=* service=temporalfrontend "wasm" ("panic" OR "RuntimeError: unreachable" OR "unexpected status")
   earliest=-24h
   ```
   - If found → Go to step 3
   - If NOT found → Check mesh timeouts, persistence layer, DB health (different runbook)

3. **Identify panic location from stack trace**
   - Look for: `panicked at /work/tools/external-deps/proxy-wasm-rust-sdk/src/hostcalls.rs:248`
   - Common patterns:
     - `get_http_call_response_header` → Missing response headers
     - `expect("missing status code")` → Empty response from upstream

4. **Check upstream service latency (C2C auth, SFAP, etc.)**
   - Dashboard: [Mesh Service-to-Service](https://moncloud-grafana.sfproxy.monitoring.aws-esvc1-useast2.aws.sfdc.cl/d/RqisbMqWk/mesh-service-to-service)
   - Filter: `originating_service=temporalfrontend`, `destination_service=c2c-public` (or other upstream)
   - Look for: P95/P99 latency approaching or exceeding timeout (currently 10s for C2C)

5. **Correlate timeline: dispatch vs. error**
   ```
   index=* service=temporalfrontend ("Dispatched call for token validation" OR "No headers present in HTTP call response")
   | transaction call_id maxpause=15s
   | eval gap = _time_diff
   | where gap >= 2
   ```
   - If gap ≥ timeout threshold → Upstream timeout causing empty response

6. **Tactical mitigation: Rolling restart**
   - Create MO case via Falcon
   - Execute rolling restart of `temporalfrontend` pods
   - Monitor: PD alert should clear within 10-30min
   - **Important**: This is NOT root cause fix - incident remains open

7. **Escalation for permanent fix**:
   - If issue recurs >2x in 24h after restart → Escalate to WASM/Mesh team
   - Provide: panic stack trace, upstream latency metrics, timeline correlation
   - Potential fixes:
     - Increase timeout for upstream call (consult upstream service team)
     - Add graceful error handling in WASM for missing headers
     - Investigate upstream performance degradation

**Estimated diagnosis time reduction**: 
- Current: 10 days (couldn't reproduce locally, needed debug logs)
- With runbook: 2-4 hours (if correlation clear) to identify pattern and escalate
- With enhanced logging (already implemented): 30min - 1h

**Data/Tooling needed**:
- Access to Splunk with temporalfrontend logs
- Mesh service-to-service dashboard access
- Falcon MO execution permissions for rolling restart
- WASM panic alert (from Detection section)

---

#### Tier 2: AI-Assisted Diagnosis (Medium-term - 1-2 sprints)

**Skill name**: `temporal-correlate-wasm-panic-upstream-latency`

**Approach**:
- AI agent auto-correlates WASM panic timestamps with upstream service latency spikes
- Checks: C2C, SFAP, other services called from frontend
- Queries Splunk + Mesh dashboards in parallel
- Outputs: "WASM panics correlate with c2c-public P99 latency >8s at [timestamps]"

**Human action**: Review correlation report → decide on timeout increase vs. upstream investigation

**Estimated reduction**: 
- Diagnosis: 10 days → 1-2 hours (automated correlation)
- Human review: 30min to decide escalation path

---

### Resolution

**Auto-resolvable?** PARTIAL

**Safety level**: NEEDS_APPROVAL for permanent fix, SAFE for tactical mitigation

#### Tier 1: Tactical Auto-Resolution (SAFE - can implement now)

**Approach**: Auto-restart on health probe failures (W-19660567)

**Implementation**:
1. Configure Kubernetes liveness probes for temporalfrontend pods
2. Set restart policy: Restart pod if liveness probe fails >X times in Y minutes
3. Alert on-call: "Auto-restarted temporalfrontend pod due to health probe failure - investigate root cause"

**Safety**:
- SAFE: Rolling restarts are already approved via MO process and used 2-3x daily during incident
- Impact: Temporary service disruption to single pod (rolling restart minimizes blast radius)
- Validation: Pod health recovers within 2-5min post-restart

**Estimated TTR reduction**:
- Manual rolling restart process: ~30min (create MO case → approval → execution → validation)
- Auto-restart: ~5min (probe failure → restart → pod ready)
- Reduction: 25min per occurrence
- Over 10-day incident period (2-3 restarts/day): Saves 20-30 occurrences × 25min = 8-12.5 hours

**Limitation**: Does NOT fix root cause, only reduces manual toil and customer impact duration

---

#### Tier 2: Root Cause Auto-Fix (NEEDS_APPROVAL - requires validation)

**Scenario**: Upstream timeout causing WASM panics

**Approach**:
1. AI agent detects pattern: "WASM panics + upstream latency >threshold"
2. Proposes fix: "Increase timeout from Xs to Ys" (based on upstream team recommendation)
3. Human approval: Staff engineer reviews proposal + impact
4. AI agent: Creates PR with timeout change, tags reviewers
5. Human approval: Merge PR → deploy via standard pipeline to test1 → validate → prod1

**Safety**: NEEDS_APPROVAL (config change affecting auth flow)

**Estimated TTR reduction**:
- Current: 10 days (investigation + local reproduction + fix creation + validation + deployment)
- With AI-assisted fix: 2-3 days (automated detection + proposal + human review + deployment)
- Reduction: ~7 days

---

## Key Takeaways

### What Would Prevent This Incident

1. **Defensive coding in WASM extensions**:
   - Replace `.expect()` and `.unwrap()` with graceful error handling for all external API responses
   - Assume upstream calls can timeout, fail, or return empty responses

2. **Align timeouts with upstream recommendations upfront**:
   - C2C team recommends 10s timeout (used by CASC team)
   - Frontend WASM used 2s timeout → misalignment caused panics
   - Lesson: Consult upstream service owners for timeout SLOs during integration

3. **Enhanced observability for WASM/Envoy sidecars**:
   - Debug-level logging for call stack paths in WASM
   - Metrics on WASM panics (if available from Envoy)
   - Alert on panic errors, not just 5XX from application

4. **Chaos testing for auth flows**:
   - Simulate upstream timeout/failure in test environments
   - Validate graceful degradation (e.g., fail request vs. crash pod)

### What Would Accelerate This Incident

1. **WASM panic alerts** (W-19601626):
   - Immediate detection of crashes, not reliant on 5XX error rate
   - Prevents alert auto-resolution masking persistent issue

2. **Runbook for "Frontend 503 + WASM panic" pattern**:
   - Step-by-step diagnostic checklist (see Tier 1 above)
   - Reduces diagnosis from 10 days → 2-4 hours

3. **Auto-restart on health probe failures** (W-19660567):
   - Reduces manual toil for tactical mitigation
   - TTR per occurrence: 30min → 5min

4. **Pre-production validation with realistic load**:
   - Issue occurred in test1 but auto-recovered (low traffic)
   - Recommendation: Load test with production-like C2C auth traffic before prod deployment
   - Would have caught 2s timeout issue in test1 environment

5. **Correlation tooling** (AI-assisted):
   - Auto-correlate WASM panics with upstream latency spikes
   - Diagnosis: 10 days → 1-2 hours

### Incident Management Lessons

1. **Don't close incidents with tactical mitigations**:
   - Incident 1 closed after rolling restart on Aug 12
   - Issue recurred immediately, requiring escalation 10 days later as Incident 2 (SEV-2)
   - Lesson: Keep incident open until permanent fix validated

2. **Track recurring PD alerts**:
   - Multiple PD alerts fired Aug 11-22, auto-resolved in ~1h each
   - Pattern of recurrence was lost in noise
   - Lesson: Alert if same issue fires >2x in 24h → escalate as persistent

3. **Proactive communication for persistent issues**:
   - Escalation (Incident 2) triggered by "missed updates on previous incident investigation"
   - Lesson: For recurring issues, provide daily updates even if no new findings

4. **Faster escalation to upstream teams**:
   - C2C team consulted on Aug 22 (10 days in) → immediate recommendation for 10s timeout
   - Lesson: Engage upstream service owners early when their API is in call stack

---

## Remediation Tracking

### Corrective Actions (Root Cause Fix)
- [x] **W-19529467**: Fix WASM panic - graceful header handling (DONE - deployed Aug 23)
- [ ] **W-19646289**: Investigate C2C timeout issue - why are requests taking >2s? (9b - in progress)
- [ ] **W-19601448**: Update dispatch call timeout to 10s (9b - in progress)

### Preventative Actions
- [ ] **W-19435593**: Missed TTR analysis (9b)
- [ ] **W-19646264**: Add health probe metrics + alerting (9b)
- [ ] **W-19660567**: Auto-restart on liveness probe failures (9b)
- [ ] **W-19636459**: Block 7443 endpoint without C2C token (10a - requires RBAC)
- [ ] **W-19601626**: Alert on WASM panic events (10a - requires eval if metrics available)
- [ ] **W-19646274**: Evaluate c2c-casc addon vs. custom WASM (10a)
- [ ] **W-19601655**: Enhanced logging for request path + headers (10a)

---

## Additional Context

### Why C2C Auth Enabled for M&J Team Only?
- C2C authentication enabled in dev1, test1, prod1 foundations for M&J team
- Other environments/teams not affected (different auth flow)
- Lower environments had low traffic → issue rarely reproduced despite same code

### Why Couldn't Reproduce Locally?
- Required simulating: "C2C service API request not reaching destination AND returning empty response after timeout"
- Local environment defaults to successful/fast C2C responses
- Timeout scenarios initially gave 504 errors (not WASM panics)
- Service unreachable gave 503 errors (not panics)
- Breakthrough: Added debug logging to trace exact failure path + consulted C2C team on latency

### Related Incidents
- Test1/foundation also experienced WASM panics Aug 13-15
- PD alerts fired but auto-resolved due to pod restarts
- Lower severity due to lower traffic + auto-recovery

### Technology Stack
- **Frontend Service**: Temporal Frontend (Go)
- **Sidecar**: Istio Envoy proxy with custom WASM extension
- **WASM Extension**: `temporalfrontend1` filter for C2C auth (Rust-based, proxy-wasm-rust-sdk)
- **Upstream Services**: C2C (Cloud-to-Cloud) token validation API
- **Orchestration**: Kubernetes (Falcon-managed)
- **Monitoring**: Splunk (logs), PagerDuty (alerts), Moncloud Grafana (metrics), FIT tests
