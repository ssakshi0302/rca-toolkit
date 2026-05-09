## RCA Analysis #2: temporalfrontend Availability Drop (aws-prod24-apsouth2)

### Incident Overview
- **PRB ID**: [To be created - not yet assigned]
- **Severity**: SEV-1 (Critical)
- **Type**: Service mesh routing misconfiguration / Race condition
- **Date**: April 11-12, 2026
- **ICC Channel**: #temporal-oncall-discussion (thread: Apr 12, 03:49 UTC)
- **PagerDuty**: Q3ZN00MCW0E8D3
- **RCA Document**: https://docs.google.com/document/d/16gNmZynFbncDh3M-v9O_G3tKJ1YPqb2W0y9izj-z9_w/edit

### Timeline Summary
| Phase | Timestamp (UTC) | Duration from Start | Notes |
|-------|-----------------|---------------------|-------|
| Incident Start | Apr 11, 17:30 | 0 | temporalhistory release starts |
| First Errors | Apr 11, 17:37 | +7 min | serviceerror_Unavailable appears (below alert threshold) |
| Major Error Spike | Apr 11, 17:54 | +24 min | 1,226 upstream connect errors/min, PassthroughCluster traffic begins |
| Detection | Apr 11, 18:11 | **TTD: 17 min** (from 17:54) | PagerDuty alert fires for frontend availability < 99.9% |
| Investigation Started | Apr 12, 03:49 | **+10.3 hours after alert** | Abhishek opens investigation thread |
| Root Cause Identified | Apr 12, 17:25 | +23h 55m | **TTD (diagnose): ~14 hours** | PassthroughCluster discovered on Mesh Debug Dashboard |
| Mitigation Started | Apr 12, 17:33 | +24h 3m | Rolling restart of history pods initiated via MO |
| Resolution | Apr 12, 18:31 | **TTR: ~25 hours** | Rolling restart completed, errors eliminated |

### Critical Delays Analysis

#### Detection Phase (TTD: 17 minutes from major spike)
**What caused delay:**
- **Below-threshold initial errors**: First errors appeared at 17:37 UTC but were below the alert threshold (99.9% availability for 6 minutes). Individual error bursts didn't trigger alerts until sustained pattern emerged.
- **Alert inertia settings**: Alert configured with 6-minute inertia window, requiring sustained availability < 99.9% before firing. This is by design to prevent alert fatigue, but adds detection latency.
- **Race condition invisibility**: The 2 pods rescheduled at 17:53 UTC (1 minute before major spike) went undetected. No alerting on unexpected pod rescheduling events during releases.

**Automation opportunity:**
- **Detection**: Monitor PassthroughCluster traffic immediately. Alert on: `istio_passthrough_cluster_requests > 0 for any OrcaaS service`. This would have detected the issue at 17:54 UTC (same time as major spike), but with clear root cause indication.
- **Release-correlated error detection**: When a release starts (17:30 UTC), lower the error rate threshold temporarily and alert on: "Error rate increased 10x within 10 minutes of release start". This would have detected at 17:37 UTC (7 min delay instead of 17 min).
- **Pod rescheduling event alerting**: Alert on unexpected pod replacements during releases: "2 history pods rescheduled at 17:53 UTC (not part of planned rollout)". This is a strong signal of problems.
- **Value**: PassthroughCluster monitoring would have provided immediate root cause visibility at 17:54 UTC, but primary TTD gain would come from investigation phase (see below).

#### Investigation Phase (10.3 hours delay before investigation started)
**What caused delay:**
- **Self-mitigation masked severity**: The acute impact (5 error waves) ended at 18:54 UTC, with availability returning to 99.9-100%. Low-level errors (~200/min) persisted but weren't severe enough to wake up oncall overnight.
- **Weekend/off-hours timing**: Alert fired at 18:11 UTC (likely evening in primary timezone). Investigation didn't start until 03:49 UTC next day (~10 hours later), suggesting overnight delay.
- **No customer escalation**: No direct customer impact was reported. The pm_system_3l2picj4xr namespace (PM AI Workbench) was primary affected, and FIT tests failed, but no customer tickets created urgency.

**Automation opportunity:**
- **Severity auto-escalation**: When availability drops to 86.7% (as observed in this incident), system should auto-escalate to SEV-1 and page oncall immediately, even if errors subsequently decrease. Pattern: "5 distinct availability dip waves (86.7%, 92.6%, 90.7%, 95.1%, 88.8%) in 60 minutes" is critical.
- **Release-incident correlation alert**: System should flag: "Availability drop started 7 minutes after temporalhistory release to aws-prod24-apsouth2. Rolling back or investigating release is high priority."
- **FIT failure correlation**: When FIT tests fail coincident with availability drops and PassthroughCluster traffic, auto-escalate severity.
- **Value**: Would not reduce TTD from 17 min (alert already fired), but would reduce investigation start delay from 10.3 hours to immediate response (save ~10 hours).

#### Diagnosis Phase (~14 hours to identify PassthroughCluster root cause)
**What caused delay:**
- **Healthy pod assumption**: All 6 history pods showed "Running" status with 0 restarts. Kubernetes health checks passed. This gave false confidence that pods were healthy, delaying investigation into mesh routing.
- **Mesh routing blind spot**: PassthroughCluster traffic was not in standard Temporal dashboard or triage runbook. Andrew discovered it at 17:25 UTC (Apr 12) on the Mesh Debug Dashboard after systematic elimination of other hypotheses.
- **Time spent on dead ends**: Andrew systematically checked:
  - Envoy warnings in logs (no smoking gun)
  - istio-proxy logs (nothing unusual)
  - Circuit breaker dashboard (no trips)
  - This methodical investigation took ~13 hours before discovering PassthroughCluster.
- **Unknown rescheduling event**: The fact that 2 pods were rescheduled at 17:53 UTC (vs 17:31 for the other 4) was discovered at 16:51 UTC Apr 12 - nearly 24 hours later. This event correlation was critical to understanding the race condition.
- **Known issue not prioritized**: Similar "stale IP" issue occurred in GIA2H (Dec 2025) resolved by rolling restart. A work item existed but root cause investigation hadn't progressed, leaving the team vulnerable to recurrence.

**Automation opportunity:**
- **Diagnosis**: Immediate PassthroughCluster detection would have eliminated 13 hours of dead-end investigation:
  - At 17:54 UTC when PassthroughCluster traffic started, system should alert: "2 of 6 history pods routing through PassthroughCluster. This indicates Istio mesh misconfiguration. Known issue similar to GIA2H (Dec 2025). Recommend rolling restart."
- **Pod rescheduling timeline correlation**: Automatically present timeline: "17:30: Release starts → 17:31: 4 pods created → 17:53: 2 pods rescheduled → 17:54: PassthroughCluster traffic + major errors". This timeline makes the causation obvious.
- **Mesh health in standard dashboard**: Add PassthroughCluster traffic panel to standard Temporal Grafana dashboard (not buried in separate Mesh Debug Dashboard). Make mesh routing health as visible as pod health.
- **Historical pattern matching**: Surface similar incident (GIA2H Dec 2025) immediately: "This error pattern (upstream connect error, healthy pods, mesh routing issue) matches PRB-XXXXX. That was resolved by rolling restart."
- **Value**: Reduce diagnosis time from ~14 hours to ~1 hour (immediate PassthroughCluster detection + historical pattern match + auto-suggested rolling restart).

#### Resolution Phase (~1 hour from diagnosis to resolution)
**What caused delay:**
- **MO approval process**: Change Case 89607662 required peer approval (Shivangi). This took ~8 minutes (17:33 to 17:40 UTC).
- **Rolling restart execution time**: Restart took 51 minutes (17:40 to 18:31 UTC) to cycle through all history pods safely.
- **Validation time**: After restart completed, team monitored to confirm ServiceUnavailable errors gone and PassthroughCluster traffic stopped.

**Automation opportunity:**
- **Resolution**: Rolling restart could be automated for known patterns:
  - When PassthroughCluster traffic detected + healthy pods + no recent pod changes except rescheduling → auto-suggest rolling restart with one-click approval.
  - System could pre-generate MO with justification: "PassthroughCluster traffic detected. Historical pattern (GIA2H Dec 2025) resolved by rolling restart. No release rollback needed."
  - For lower-severity incidents or non-production environments, consider auto-executing rolling restart without approval (with audit logging).
- **Why partial automation**: Rolling restart is low-risk (graceful pod replacement), but in production SEV-1, human approval is appropriate. Automation value is in pre-generating the MO and providing one-click execution.
- **Value**: Reduce resolution time from ~1 hour to ~30 minutes (eliminate MO creation overhead, maintain approval + execution time).

### Slack/Communication Analysis
**ICC Channel observations** (#temporal-oncall-discussion):
- Thread opened Apr 12, 03:49 UTC (~10 hours after alert)
- Key contributors: Abhishek (thread initiation), Shivangi (initial investigation), Andrew (root cause discovery), Vijaysenthil (guidance)
- Investigation patterns observed:
  - Shivangi: Identified "Upstream Connect error for temporalhistory service" in Splunk, discovered 1 pod pending, noted 99.7% of errors were upstream connect error or disconnect/reset before headers (12:18 UTC Apr 12)
  - Andrew: Correlated error start (17:37) with release start (17:30), identified 6-minute gap (15:00 UTC Apr 12)
  - Andrew: Confirmed issue isolated to aws-prod24-apsouth2 foundation, no other prod FIs affected (16:05 UTC Apr 12)
  - Andrew: Discovered 2 pods rescheduled at 17:53 (not 17:31 like others) - rescheduling signal (16:51 UTC Apr 12)
  - Andrew: Discovered PassthroughCluster traffic on Mesh Debug Dashboard starting exactly at 17:54 UTC (17:25 UTC Apr 12) - **BREAKTHROUGH**
- Communication effectiveness:
  - Systematic investigation: Team eliminated hypotheses methodically (envoy warnings, istio-proxy logs, circuit breakers) before finding root cause
  - Cross-correlation: Good timeline correlation work (release start → first errors → rescheduling → major spike)
  - Persistence: ~13 hours of investigation before breakthrough
- Information that was hard to find:
  - PassthroughCluster traffic (required manually checking Mesh Debug Dashboard, not in standard runbook)
  - Pod rescheduling event (discovered 24 hours later by examining pod creation timestamps)
  - Historical similar incident (GIA2H Dec 2025 "stale IP" issue mentioned but not prioritized)

### Automation Assessment

#### High-Value Automation Opportunities
1. **PassthroughCluster Traffic Monitoring**: [Action Item #3 in RCA]
   - Phase: Detection + Diagnosis
   - Impact: Reduces diagnosis time by ~13 hours (from 14h to 1h)
   - Description: Add Argus alert on non-zero PassthroughCluster traffic for any OrcaaS Temporal service. Add PassthroughCluster panel to standard Temporal Grafana dashboard (not buried in Mesh Debug Dashboard). Alert message: "2 of 6 history pods routing through PassthroughCluster. Istio mesh misconfiguration detected."
   - Feasibility: High (Istio Envoy metrics already available, just needs alert + dashboard panel)

2. **Pod Event Tracking and Correlation**: [Action Item #7 in RCA]
   - Phase: Diagnosis
   - Impact: Reduces diagnosis time by ~6 hours (immediately surface rescheduling event)
   - Description: Log and alert on pod rescheduling events during releases. Present timeline: "Release started 17:30 → 4 pods created 17:31 → 2 pods rescheduled 17:53 (unexpected) → errors spiked 17:54". Sub-minute pod event tracking.
   - Feasibility: High (Kubernetes API events + timeline visualization)

3. **Release-Incident Correlation**: [Action Item #4 in RCA - update runbook]
   - Phase: Detection + Diagnosis
   - Impact: Reduces diagnosis time by ~4 hours (immediate release correlation)
   - Description: When errors spike within 10 minutes of release start, auto-flag: "Error rate increased 10x starting 7 minutes after temporalhistory release. Release correlation: HIGH. Consider rollback or investigate release-induced issues."
   - Feasibility: High (correlate Falcon release events with error rate changes)

4. **Historical Incident Pattern Matching**:
   - Phase: Diagnosis
   - Impact: Reduces diagnosis time by ~8 hours (immediately surface GIA2H similarity)
   - Description: When symptoms match prior incidents, surface in triage dashboard: "This pattern matches GIA2H incident (Dec 2025): healthy pods + upstream connect errors + mesh routing issue. That incident was resolved by rolling restart. Check PassthroughCluster traffic."
   - Feasibility: Medium (requires RCA corpus embedding and symptom matching)

5. **Mesh Health Dashboard Integration**: [Action Item #2 in RCA]
   - Phase: Diagnosis
   - Impact: Reduces diagnosis time by ~10 hours (eliminate manual Mesh Debug Dashboard discovery)
   - Description: Duplicate Mesh Debug Dashboard panels into standard service-owner dashboard. Make mesh routing health as visible as pod health. Include: PassthroughCluster traffic, Envoy upstream cluster distribution, mesh endpoint staleness.
   - Feasibility: High (Grafana dashboard update)

#### Medium-Value Opportunities
1. **Auto-Generated Rolling Restart MO**: [Action Item #4 in RCA]
   - Phase: Resolution
   - Impact: Reduces resolution time by ~30 minutes (eliminate MO creation overhead)
   - Description: When PassthroughCluster traffic detected + healthy pods + release-induced, pre-generate MO with justification: "PassthroughCluster routing detected. Rolling restart recommended (low risk, fixes mesh re-registration). Click to approve and execute."
   - Feasibility: Medium (requires MO template API integration)

2. **Boot-Order Race Condition Detection**: [Action Item #5 in RCA]
   - Phase: Detection
   - Impact: Prevents recurrence, reduces TTD by catching race conditions earlier
   - Description: During simultaneous multi-service releases (frontend, history, matching), detect if Temporal ringpop membership or gRPC server startup completes before Envoy sidecar xDS sync. Alert if race condition detected: "History pod registered with Temporal cluster before Istio Envoy sidecar fully synced. Mesh routing may be incorrect."
   - Feasibility: Low (requires deep instrumentation into Temporal startup sequence and Envoy sidecar sync timing)

3. **Staggered Service Release Ordering**: [Action Item #6 in RCA]
   - Phase: Prevention
   - Impact: Reduces recurrence likelihood, not directly applicable to detection/diagnosis
   - Description: Enforce startup sequence during Falcon green windows (e.g., history before frontend) to reduce simultaneous-start race conditions. Automation monitors release order and alerts if out-of-sequence.
   - Feasibility: Medium (Falcon pipeline integration)

#### Low-Value / Not Automatable
1. **Root Cause Investigation (Boot-Order Race)**:
   - Phase: Diagnosis
   - Why not automatable: Reproducing the race condition (Envoy sidecar xDS sync vs Temporal service startup) requires deep distributed systems debugging, controlled environment reproduction, and code-level understanding. This is a research task, not automatable.

2. **Release Rollback Decision**:
   - Phase: Resolution
   - Why not automatable: Team correctly chose NOT to rollback the temporalhistory release because the release content (PM AI Workbench scale-up dynamic configs) was unrelated to the mesh routing issue. This judgment (is the release the cause?) requires human analysis. Automation can suggest rollback, but decision must be human.

3. **Mesh Configuration Root Cause**:
   - Phase: Diagnosis
   - Why not automatable: Understanding WHY pods registered with stale/incorrect mesh endpoints requires reproducing the race condition in lower environments and examining Istio control plane logs. This is deep troubleshooting, not automatable triage.

### Key Takeaways
1. **"Healthy pods" are insufficient**: Kubernetes health checks (Running, 0 restarts) passed while Istio mesh routing was broken. Automation must monitor mesh-level health independently of pod-level health.
2. **Mesh routing is a blind spot**: PassthroughCluster traffic went undetected for 24 hours because it wasn't in standard monitoring. Adding mesh health panels to service-owner dashboards would have saved ~13 hours of diagnosis time.
3. **Historical pattern matching is high-value**: This incident was similar to GIA2H (Dec 2025). If automation had surfaced that similarity immediately ("this looks like PRB-XXXXX, which was fixed by rolling restart"), diagnosis would have been ~10 hours faster.
4. **Race conditions are hard to prevent, easy to detect**: The boot-order race condition (Temporal service starting before Envoy sidecar fully synced) is difficult to eliminate, but its symptom (PassthroughCluster traffic) is trivial to monitor. Focus automation on detection/remediation, not prevention.
5. **Off-hours investigation delay is significant**: 10.3-hour delay before investigation started suggests overnight/weekend timing. Auto-escalation to SEV-1 when availability drops to 86.7% would have paged oncall immediately, reducing total TTR by ~10 hours.
