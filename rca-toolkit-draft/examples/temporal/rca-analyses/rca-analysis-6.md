# RCA Analysis #6: Temporal History Service Availability Issue (prod8-cacentral1/cdp2)

**Analysis Date**: 2026-05-08
**Incident Date**: September 6, 2025
**Environment**: Production (prod8-cacentral1/cdp2)
**Service**: Temporal History Service
**Severity**: Sev 3 → Sev 2

---

## Incident Metadata

| Field | Value |
|-------|-------|
| **PRB** | PRB-0028677 |
| **ICC Channel** | #icc-80240861 |
| **Severity** | Sev 3 (escalated to Sev 2 at 5:24pm) |
| **Environment** | Production (prod8-cacentral1/cdp2) |
| **Service** | Temporal History Service |
| **Date/Time** | Sept 6, 2025, 2:45pm UTC+1 (impact start) |
| **Detected By** | Monitoring (PagerDuty alert at 3:14pm) |
| **Detection Method** | Automated availability alert |

---

## Timeline Analysis

### Key Metrics

| Metric | Time (UTC+1) | Duration | Notes |
|--------|--------------|----------|-------|
| **Impact Start** | 2:45pm | - | Customer impact begins |
| **TTDetect** | 3:14pm | **29 minutes** | First PD alert fires |
| **TTInitiate** | 4:10pm | **56 minutes** | Incident channel created |
| **TTAssemble** | 5:30pm | **1h 20m** | Required teams engaged |
| **TTDiagnose** | 4:00pm | **-10 minutes** | Root cause identified before initiation |
| **TTMD** | 6:30pm | **2h 20m** | Mitigation path decided |
| **TTFix** | 9:29pm | **5h 19m** | Mitigation completed |
| **TTRestore (TTR)** | 9:29pm | **6h 44m** | Total incident duration |

### Detailed Timeline

| Time (UTC+1) | Event | Action Taken |
|--------------|-------|--------------|
| 2:45pm | Impact starts (undetected) | - |
| 3:14pm | PagerDuty alert for history availability drop | Team begins investigation |
| 3:15-3:40pm | Dashboard review | Found OOMKilled events earlier in day; waited for auto-recovery |
| 3:41-4:10pm | Pod investigation | 1/6 history pods in CrashLoopBackOff, 2 pods at 85-95% memory |
| 4:10pm | **Incident initiated** (Sev 3) | Started history pod restart via MO |
| 4:10-4:30pm | Post-restart monitoring | Multiple pods intermittently entering CrashLoopBackOff |
| 4:30-5:30pm | Root cause investigation | Found ~50k workflows in admin_service (8h), DB CPU >99% |
| 5:24pm | **Severity escalation to Sev 2** | Recognizing stress on history service |
| 5:30-5:45pm | Internal discussion | Mitigation path: DB + history resource upgrade |
| 5:45-6:50pm | Customer engagement | Discussed with CDP team about workflow traffic patterns |
| 6:30-7:45pm | Mitigation preparation | Decided on resource sizes, created PRs |
| 9:29pm | **Mitigation complete** | DB upscaled (r6g.large → r6g.2xlarge), history memory increased (2Gi → 8Gi) |

---

## Delay Analysis

### Detection Delay: 29 minutes (MODERATE)

**What happened:**
- Impact started at 2:45pm
- First alert fired at 3:14pm (29 min delay)
- Argus alert timestamp was 9 minutes earlier than PD alert

**Why it matters:**
- 29 minutes of undetected customer impact
- Initial OOMKilled events occurred earlier in the day but didn't trigger alerts
- Alert timing discrepancy (Argus vs PD) suggests alert pipeline issues

**Gaps identified:**
1. **No proactive memory alerts** - Pods were running at 60-80% memory for days before incident
2. **No OOMKilled alerts** - Container restarts weren't triggering notifications
3. **No DB CPU alerts** - Database hitting 99% CPU wasn't immediately detected
4. **Alert delivery delays** - 9-minute gap between Argus detection and PD notification

### Diagnosis Delay: 50 minutes (GOOD - but misleading metric)

**What happened:**
- Root cause identified at 4:00pm (DB CPU >99%, workflow surge)
- Incident initiated at 4:10pm
- **Negative TTDiagnose (-10 minutes)** indicates root cause found before formal incident start

**Why this is misleading:**
- Team spent 25-30 minutes (3:15-3:40pm) waiting for auto-recovery
- Another 30 minutes (3:41-4:10pm) investigating pod states
- Effective diagnosis time: **~55 minutes from detection** (3:14pm → 4:00pm)
- Root cause understanding was **incomplete** until 4:30-5:30pm timeframe

**Actual diagnosis phases:**
1. **3:15-3:40pm (25 min)**: Surface-level investigation, waiting for recovery
2. **3:41-4:10pm (30 min)**: Pod-level diagnosis (OOMKilled, CrashLoopBackOff)
3. **4:10-4:30pm (20 min)**: Confirmation that restart didn't fix issue
4. **4:30-5:30pm (60 min)**: **True root cause discovery** - workflow surge + DB saturation

**Total effective diagnosis time: ~2 hours 15 minutes** (3:15pm → 5:30pm)

### Resolution Delay: 5h 19m (LONG)

**What happened:**
- Mitigation path decided at 6:30pm
- Resolution completed at 9:29pm
- **3 hours** spent on mitigation execution after decision

**Breakdown of delays:**

#### Phase 1: Investigation continuation (5:30-6:50pm = 1h 20m)
- **Why necessary**: Needed to confirm workflow traffic patterns with CDP team
- **What they did**: Verified all tenant provisioning routes to prod8-cdp2
- **Result**: Confirmed mitigation approach (resource scaling)

#### Phase 2: Decision-making (5:30-6:30pm = 1h, overlapping with Phase 1)
- **What happened**: Internal discussion on DB + history resource upgrade sizing
- **Dependencies**: Needed CDP team confirmation on traffic expectations
- **Communication overhead**: SR involvement, bridge management discussion

#### Phase 3: Implementation (6:30-9:29pm = 2h 59m)
- **What happened**: PR creation, review, deployment
- **Actions**:
  - DB upscale: r6g.large → r6g.2xlarge
  - History service: 2Gi → 8Gi memory
  - Matching service: 1 CPU/2GB → 2 CPU/4GB
  - Opencensus sidecar: 500mb → 1GB
- **Why 3 hours**: Multiple PRs, review process, staged deployments, verification

---

## Root Cause Deep Dive

### Proximate Cause
History service pods experienced memory exhaustion and OOMKills due to workflow surge, while database CPU saturation (>99%) prevented pod recovery, creating a cascading failure.

### Contributing Factors (5+ Whys Analysis)

1. **Workflow surge from admin_service namespace**
   - ~4.4k workflows at 6:45am PDT (10 mins before alert)
   - ~3.3k workflows shortly after alert
   - Sustained 1-2k workflows every 5-10 minutes post-restart
   - Source: Prod8-cdp2 FD used for Trailhead orgs (tenant provisioning/deprovisioning)

2. **Undersized database instance**
   - Running on db.r6g.large
   - CPU hit >99% during workflow surge
   - Remained saturated throughout incident
   - No read replica offloading configured

3. **Undersized history service resources**
   - 2Gi memory insufficient for workload
   - 6 pods managing 4096 shards (~680 shards per pod)
   - Baseline memory 60-80% for days before incident
   - No horizontal autoscaling configured

4. **Failed pod recovery mechanism**
   - History pods require shard acquisition on restart
   - DB saturation prevented shard claims (5-minute retry with backoff)
   - Cache warm-up generated additional DB reads (GetMutableState calls)
   - Pods stuck in CrashLoopBackOff, couldn't reach "Serving" state

5. **Lack of rate limiting on customer side**
   - Admin service tenant provision endpoint had no active rate limiting
   - All requests forwarded directly to Temporal without throttling
   - Background scheduled tasks (TenantScheduledLifecycleTaskService) continued retrying failed workflows
   - Load amplification: real-time requests + scheduled retries

### System Architecture Weaknesses

**Resource Configuration:**
- Services using default CPU/memory configs (not optimized per service)
- No HPA based on memory utilization
- No VPA for vertical autoscaling
- 2 pods at 85-95% memory before incident - no alerts

**Database Architecture:**
- Single writer instance handling all operations
- Reader instances provisioned but unused
- No read query offloading
- db.r6g.large insufficient for peak load

**Monitoring Gaps:**
- No DB CPU utilization alerts
- No container restart alerts
- No CPU/memory utilization alerts (main + sidecars)
- No proactive memory pressure alerts
- Alert delivery delays (Argus → PD)

**Client-Side Issues (admin_service):**
- No rate limiting enforcement on provision endpoint
- Misleading logs ("workflow started" during timeout exceptions)
- No workflow submission failure alerts
- No circuit breaker for Temporal server overload
- Background retries amplified load during incident

---

## Impact Assessment

### Actual Impact
- **15,000 workflows** in admin_service namespace experienced high latency
- **P95 StartWorkflowExecution latency: 8 seconds** (significant increase)
- **No workflow failures** - all eventually completed
- **Client timeout errors** leading to periodic retries
- **Temporal UI**: 5xx errors for customers attempting access

### Potential Impact (if not mitigated)
- Increased workflow completion times across all namespaces
- Complete Temporal UI unavailability
- Workflow submission failures
- Cascading failures to other Temporal services (frontend, matching)

### Business Impact
- Tenant provisioning delays for Trailhead orgs
- Customer-facing timeout errors
- Operational overhead (6h 44m incident duration)

---

## Diagnostic Deep Dive

### Metrics Used

**Primary Signals:**
1. **History availability alert** (PagerDuty) - Initial detection
2. **OOMKilled events** - Found in dashboards (3:15-3:40pm)
3. **Pod states** - CrashLoopBackOff, memory usage 85-95% (3:41-4:10pm)
4. **Temporal UI** - Workflow count surge (~50k in 8h) (4:30-5:30pm)
5. **DB CPU utilization** - >99% saturation (4:30-5:30pm)
6. **Workflow submission rate** - 1-2k every 5-10 minutes post-restart

**Supporting Metrics:**
- Container restart counts
- Memory utilization trends (backdated to days before incident)
- History service error rates (serviceerror_unavailable)
- Persistence errors in history
- Start workflow latency (P95/P70/P50)
- Shard lock latency (should be <50ms, was elevated)
- Multiple alert types: proxy, frontend, matching, history error rates

### Logs Used

**Splunk Errors:**
- History service errors during incident
- serviceerror_Unavailable errors
- PollWorkflowExecutionHistory failures
- Shard acquisition failures

**Application Logs (admin_service):**
- Misleading "workflow started" messages during timeouts
- WorkflowTimeoutException logs (300s default)
- WorkflowExecutionAlreadyStarted conflict handling

### Actions Taken

**Investigation Phase:**
1. Dashboard review (Temporal Namespace, Temporal Dashboard, SLO Dashboard)
2. Temporal UI inspection for workflow activity
3. Grafana metric analysis
4. Pod state inspection (kubectl/K8s)
5. Historical memory trend analysis (backdated 9 hours)
6. Customer engagement (CDP team) for traffic pattern confirmation

**Remediation Phase:**
1. Rolling restart via MO (8:06am-8:17am PDT) - **FAILED** (pods returned to CrashLoopBackOff)
2. Resource upscaling:
   - DB: r6g.large → r6g.2xlarge
   - History: 2 CPU/2Gi → 2 CPU/8Gi (4Gi request, 8Gi limit)
   - Matching: 1 CPU/2GB → 2 CPU/4GB (proactive)
   - Opencensus sidecar: 500mb → 1GB

**Validation:**
- Error rate dropped to 0
- ~15k "In progress" workflows completed within minutes
- Service availability restored

---

## Remediation Analysis

### What Fixed It

**Immediate Fix (Emergency Bug Fix):**
1. **Database upscaling** - r6g.large → r6g.2xlarge
   - Reduced CPU saturation
   - Enabled history pods to acquire shards successfully
   - Allowed cache warm-up to complete

2. **History service memory increase** - 2Gi → 8Gi
   - Eliminated OOMKills
   - Provided headroom for shard ownership and cache
   - Enabled stable pod operation

3. **Proactive scaling of related services**
   - Matching service: 1 CPU/2GB → 2 CPU/4GB
   - Opencensus sidecar: 500mb → 1GB
   - Prevented secondary failures

### Long-Term Remediation (Work Items Created)

**Scalability & Resiliency:**
1. **W-19692412**: Optimize CPU/Memory configs per service (not defaults)
2. **W-19356420**: Implement rate limiting and throttling
3. **W-19361451**: HPA based on memory utilization
4. **W-19747923**: Explore VPA for vertical autoscaling
5. **W-19239873**: Offload read-only queries to DB reader instances

**Monitoring & Alerting:**
1. **W-19569289**: DB CPU utilization alerts (Ready for Review)
2. **W-19747878**: Container restart alerts
3. **W-19338432**: CPU/Memory utilization alerts (main + sidecars)
4. **W-19747831**: Fine-tune K8s probes (liveness/readiness)
5. **W-19747949**: Analyze error types for workflow submission failures

**TTD/TTR Improvements:**
- Section marked "To be added" in RCA

**Client-Side (admin_service) Recommendations:**
1. Enable rate limiting enforcement on provision endpoint
2. Add workflow submission failure alerts
3. Fix misleading logs (workflow started vs timeout)
4. Implement circuit breaker for Temporal overload
5. Coordinate real-time requests with scheduled retries

### Commands/Scripts Used

**Managed Operation (MO):**
- Rolling restart of history pods: [MO execution](https://falcon.devhub.internal.salesforce.com/managed-operations/execution/29081202-7da4-4734-9cf3-f8e937053fa1/)
- Case: 500EE00001cpJYUYA2

**Resource Changes (PRs created):**
- DB upscaling configuration
- History service resource limits
- Matching service resource limits
- Opencensus sidecar memory limits

### Reproducibility

**High reproducibility** under similar conditions:
1. Workflow surge (4k+ workflows in short period)
2. DB instance undersized for peak load
3. History service memory insufficient (baseline 60-80%)
4. No rate limiting on customer endpoints
5. Background retry jobs amplifying load

**Likelihood of recurrence: HIGH** without remediation work items completed.

**Key risk factors:**
- Prod8-cdp2 FD continues serving Trailhead orgs
- Tenant provisioning patterns unchanged
- Resource configs remain at emergency levels (not permanent)
- Rate limiting still not enforced
- No autoscaling configured

---

## Automation Opportunities (CRITICAL)

### Detection Automation (High Priority)

**1. Proactive Memory Pressure Alerts**
- **Gap**: Pods running at 60-80% memory for days without alerting
- **Opportunity**: Alert at 70% memory sustained for 10+ minutes
- **Impact**: Could have detected issue **days before incident**
- **Complexity**: Low (Argus metric already exists)
- **GUS**: W-19338432

**2. DB CPU Utilization Alerts**
- **Gap**: DB hit 99% CPU with no alert
- **Opportunity**: Alert at 80% CPU sustained for 5+ minutes
- **Impact**: **29 minutes faster detection** (would have alerted at 2:45pm vs 3:14pm)
- **Complexity**: Low (RDS metrics available)
- **GUS**: W-19569289 (Ready for Review)

**3. OOMKilled/Container Restart Alerts**
- **Gap**: Early morning OOMKills didn't trigger alerts
- **Opportunity**: Alert on any container restart (immediate)
- **Impact**: **Hours earlier detection** (6:07am vs 3:14pm)
- **Complexity**: Low (K8s events available)
- **GUS**: W-19747878

**4. Alert Delivery Pipeline Fix**
- **Gap**: 9-minute delay between Argus detection and PagerDuty alert
- **Opportunity**: Investigate and fix alert routing delays
- **Impact**: **9 minutes faster detection**
- **Complexity**: Medium (requires platform investigation)

**Combined detection improvement: 29 minutes → near real-time** (hours earlier with OOMKilled alerts)

### Diagnosis Automation (Medium Priority)

**5. Auto-Correlation Dashboard**
- **Gap**: Team manually correlated metrics across multiple dashboards
- **Opportunity**: Single view correlating:
  - Pod health (restarts, memory, CPU)
  - DB metrics (CPU, connections, latency)
  - Workflow submission rates per namespace
  - Error rates across services
- **Impact**: **30-60 minutes faster diagnosis** (reduce 2h 15m → 1h 15m)
- **Complexity**: Medium (requires dashboard development)

**6. Workflow Surge Detection**
- **Gap**: Workflow surge identified manually through UI inspection
- **Opportunity**: Auto-detect workflow rate anomalies (statistical)
- **Impact**: **20-30 minutes faster diagnosis**
- **Complexity**: Medium (requires ML/statistical model)

**7. Root Cause Suggestion Engine**
- **Gap**: Manual 5-whys analysis required
- **Opportunity**: AI/rule-based suggestions:
  - "DB CPU >99% + History OOMKills → likely resource exhaustion"
  - "Workflow surge in namespace X → check customer traffic patterns"
- **Impact**: **30-45 minutes faster diagnosis**
- **Complexity**: High (requires AI/knowledge base)

**Combined diagnosis improvement: 2h 15m → 45m-1h** (~60-70% reduction)

### Resolution Automation (High Priority)

**8. Auto-Scaling (HPA + VPA)**
- **Gap**: Manual resource sizing decision + PR + deployment (3 hours)
- **Opportunity**: 
  - HPA: Auto-scale history pods based on memory (4-12 pods)
  - VPA: Auto-recommend memory limits based on usage
- **Impact**: **~2 hours faster resolution** (6:30pm → 4:30pm mitigation complete)
- **Complexity**: Medium (HPA config), High (VPA + safe guardrails)
- **GUS**: W-19361451, W-19747923

**9. Automated DB Read Replica Routing**
- **Gap**: All queries hit writer instance
- **Opportunity**: Route read-only queries (GetMutableState, etc.) to readers
- **Impact**: **Prevents incident entirely** (DB CPU stays <80%)
- **Complexity**: Medium (requires query classification)
- **GUS**: W-19239873

**10. Rate Limiting Auto-Enforcement**
- **Gap**: Customer endpoints have no rate limiting
- **Opportunity**: Auto-throttle at service/namespace level
- **Impact**: **Prevents incident entirely** (caps workflow submission rate)
- **Complexity**: Medium (requires policy engine config)
- **GUS**: W-19356420

**11. Circuit Breaker for DB Saturation**
- **Gap**: History pods repeatedly tried to acquire shards despite DB saturation
- **Opportunity**: Auto-detect DB unavailability, back off aggressively
- **Impact**: **Reduces cascading failures**, faster recovery
- **Complexity**: Medium (requires service code changes)

**12. One-Click Mitigation Playbook**
- **Gap**: 3-hour manual process (sizing decision → PR → deploy)
- **Opportunity**: Pre-approved runbook:
  - "DB CPU >90% for 10+ min → auto-upscale one tier"
  - "History memory >85% → auto-scale pods or increase limits"
- **Impact**: **~1.5 hours faster resolution** (6:30pm → 5:00pm)
- **Complexity**: Medium (requires approval workflow + automation)

**Combined resolution improvement: 5h 19m → 1-2h** (~60-70% reduction)

### Prevention Automation (Highest Impact)

**13. Capacity Planning Automation**
- **Gap**: Baseline memory 60-80% for days without action
- **Opportunity**: Weekly capacity reports with recommendations
- **Impact**: **Prevents incident entirely**
- **Complexity**: Low (simple reporting + thresholds)

**14. Load Testing per FD**
- **Gap**: Unknown capacity limits for prod8-cdp2 FD
- **Opportunity**: Periodic load tests to establish thresholds
- **Impact**: **Prevents incident + informs rate limits**
- **Complexity**: Medium (requires test framework)

**15. Proactive Resource Right-Sizing**
- **Gap**: Services using default configs
- **Opportunity**: Monthly review of resource usage → adjust configs
- **Impact**: **Prevents incident entirely**
- **Complexity**: Low (manual review) → Medium (automated recommendations)
- **GUS**: W-19692412

---

## Automation ROI Summary

| Priority | Opportunity | Detection | Diagnosis | Resolution | Prevention | Complexity |
|----------|-------------|-----------|-----------|------------|------------|------------|
| **P0** | DB CPU alerts | ✓ (29 min) | - | - | - | Low |
| **P0** | OOMKilled alerts | ✓ (hours) | - | - | - | Low |
| **P0** | Memory pressure alerts | ✓ (days) | - | - | ✓ | Low |
| **P0** | HPA on memory | - | - | ✓ (2h) | ✓ | Medium |
| **P0** | DB read replica routing | - | - | - | ✓ | Medium |
| **P1** | Rate limiting | - | - | - | ✓ | Medium |
| **P1** | Auto-correlation dashboard | - | ✓ (30-60m) | - | - | Medium |
| **P1** | One-click mitigation | - | - | ✓ (1.5h) | - | Medium |
| **P2** | Workflow surge detection | - | ✓ (20-30m) | - | - | Medium |
| **P2** | VPA recommendations | - | - | ✓ (included) | ✓ | High |
| **P2** | Circuit breaker | - | - | ✓ (recovery) | - | Medium |

**Total potential impact:**
- **TTD: 29 min → <5 min** (83% improvement)
- **TTDiagnose: 2h 15m → 45m-1h** (60-70% improvement)
- **TTR: 6h 44m → 1-2h** (70-80% improvement)
- **Prevention: HIGH** (3+ mechanisms to prevent entirely)

---

## Key Learnings

### Where We're Lacking

**1. DETECTION (Critical Gap)**
- No proactive resource monitoring (memory, CPU, DB)
- Alert delivery delays (9 minutes Argus → PD)
- No early warning system for capacity exhaustion
- OOMKilled events not triggering alerts

**2. PROACTIVE CAPACITY MANAGEMENT (Critical Gap)**
- Services running at 60-80% memory baseline for days
- No capacity planning or trending analysis
- Using default resource configs (not optimized per service)
- No autoscaling configured (HPA or VPA)

**3. CLIENT-SIDE PROTECTION (Critical Gap)**
- No rate limiting enforcement
- No circuit breakers for downstream overload
- Background retries amplifying load during incidents
- Poor observability (misleading logs)

### What Went Well

**1. DIAGNOSIS (Strength)**
- Comprehensive metric correlation (pods, DB, workflows, namespaces)
- Effective use of multiple data sources (Grafana, Temporal UI, Splunk)
- Historical trend analysis (backdated memory usage)
- Customer engagement to confirm traffic patterns

**2. CROSS-TEAM COLLABORATION (Strength)**
- Engaged CDP team to understand workflow patterns
- SR coordination for impact assessment
- Clear communication throughout incident

**3. COMPREHENSIVE REMEDIATION PLANNING (Strength)**
- Not just immediate fix - created 10+ follow-up work items
- Addressed root causes at multiple layers (DB, service, client)
- Both proactive (matching service) and reactive (history) scaling
- Detailed client-side RCA section (admin_service perspective)

### Specific Bottlenecks

**Bottleneck #1: Detection Delay (29 minutes)**
- **Time**: 2:45pm (impact) → 3:14pm (alert)
- **Cost**: 29 minutes of undetected customer impact
- **Root cause**: No DB CPU alerts, no memory pressure alerts
- **Fix**: W-19569289, W-19338432, W-19747878

**Bottleneck #2: False Recovery Assumption (25 minutes)**
- **Time**: 3:15pm → 3:40pm
- **Cost**: 25 minutes waiting for auto-recovery that never came
- **Root cause**: Incomplete understanding of pod recovery requirements
- **Fix**: Runbook documentation, K8s probe tuning (W-19747831)

**Bottleneck #3: Incomplete Root Cause (1h 20m)**
- **Time**: 4:00pm (thought diagnosed) → 5:30pm (actually diagnosed)
- **Cost**: 1h 20m of incomplete understanding
- **Root cause**: Initial focus on pod symptoms, not underlying DB saturation
- **Fix**: Auto-correlation dashboard, root cause suggestion engine

**Bottleneck #4: Customer Coordination (1h 20m)**
- **Time**: 5:30pm → 6:50pm
- **Cost**: 1h 20m to confirm traffic patterns with CDP team
- **Root cause**: No shared understanding of traffic routing or capacity limits
- **Fix**: Capacity planning automation, load testing per FD

**Bottleneck #5: Manual Mitigation Execution (3 hours)**
- **Time**: 6:30pm → 9:29pm
- **Cost**: 3 hours for PR + review + deployment
- **Root cause**: No pre-approved scaling playbooks, no autoscaling
- **Fix**: HPA/VPA (W-19361451, W-19747923), one-click mitigation

---

### Detection Capabilities Required

**Must-Have:**
1. **Multi-layer resource monitoring** with proactive alerts:
   - Application layer (pod memory, CPU)
   - Database layer (CPU, connections, query latency)
   - Workflow layer (submission rates, latency)
2. **Alert delivery SLAs** (<1 minute end-to-end)
3. **Anomaly detection** for workflow surge patterns
4. **Historical trending** to identify slow degradation

**Nice-to-Have:**
- Cross-service alert correlation
- Predictive alerting (ML-based capacity forecasting)

### Diagnosis Capabilities Required

**Must-Have:**
1. **Unified observability** across:
   - Kubernetes (pod states, events, resource usage)
   - Application metrics (error rates, latency, throughput)
   - Database metrics (CPU, memory, query performance)
   - Workflow metrics (submission rates, completion rates, queue depths)
2. **Namespace-level isolation** for multi-tenant analysis
3. **Historical playback** to analyze backdated trends
4. **Log correlation** with metrics

**Nice-to-Have:**
- AI-powered root cause suggestions
- Automated runbook recommendations
- Similar incident pattern matching

### Resolution Capabilities Required

**Must-Have:**
1. **Autoscaling support**:
   - HPA based on memory (not just CPU)
   - Safe guardrails for resource limits
2. **Database capacity management**:
   - Read replica routing automation
   - Auto-upscaling with rollback
3. **Rate limiting/throttling** at service and namespace levels
4. **One-click rollback** for failed mitigations

**Nice-to-Have:**
- VPA with safe recommendations
- Circuit breaker auto-configuration
- Progressive rollout automation

### Client-Side Requirements

**Must-Have (for admin_service and similar):**
1. **Rate limiting enforcement** with 429 retry logic
2. **Circuit breaker** for downstream service overload
3. **Workflow submission observability** (success/failure/timeout metrics)
4. **Retry coordination** (don't amplify load during incidents)

---

## Data Quality Assessment

**RCA Quality: EXCELLENT**

**Strengths:**
- Comprehensive timeline with all key metrics (TTD, TTDiagnose, TTR, etc.)
- Detailed 5+ whys analysis
- Both Temporal and client (admin_service) perspectives included
- Extensive appendix with graphs, metrics, alerts, and reference links
- 10+ follow-up work items created with GUS links
- Clear distinction between immediate fix and long-term remediation

**Completeness:**
- ✓ Metadata (PRB, ICC, severity, dates)
- ✓ Timeline (detailed with UTC+1 timestamps)
- ✓ Root cause (multi-layer: workflow surge, DB saturation, memory exhaustion)
- ✓ Delays (TTD, TTDiagnose, TTR all documented)
- ✓ Diagnostic details (metrics, logs, dashboards)
- ✓ Remediation (immediate + long-term work items)
- ✓ Impact assessment (actual + potential)
- ✓ Client perspective (admin_service RCA section)
- ⚠ TTD/TTR improvement section marked "To be added"

**Areas for Improvement:**
- TTD/TTR improvement recommendations incomplete
- Some GUS work items lack status updates
- No explicit cost analysis (business impact quantification)
- Could benefit from clearer "automation opportunities" section

---

## Incident Classification

**Type**: Capacity/Resource Exhaustion + Cascading Failure
**Complexity**: High (multi-layer: application, database, client interactions)
**Preventability**: HIGH (multiple prevention mechanisms available)
**Automation Potential**: VERY HIGH (detection, diagnosis, resolution all automatable)

---

### Test Scenarios to Run

1. **Simulate workflow surge**: 4k+ workflows in 10 minutes
2. **Simulate DB saturation**: Drive CPU to >90%
3. **Simulate OOMKill**: Force pod restarts under load
4. **Measure**: TTD, TTDiagnose, TTR with each platform
5. **Evaluate**: Which platform provides actionable insights fastest?

---

## Appendix: Reference Links

- [PRB-0028677](https://gus.lightning.force.com/lightning/r/SM_Problem__c/a4QEE0000006mun2AA/view)
- [ICC #icc-80240861](https://salesforce.enterprise.slack.com/archives/C09DWTV7ZQW)
- [PagerDuty Alert](https://salesforce-internal.slack.com/archives/C02Q5UAUG6N/p1757168060977029)
- [Slack Thread](https://salesforce-internal.slack.com/archives/C071FM827R9/p1757168508382159)
- [Rolling Restart MO](https://falcon.devhub.internal.salesforce.com/managed-operations/execution/29081202-7da4-4734-9cf3-f8e937053fa1/)
- [Case 500EE00001cpJYUYA2](https://gus.lightning.force.com/lightning/r/Case/500EE00001cpJYUYA2/view)
- [Temporal Namespace Dashboard](https://moncloud-grafana.sfproxy.monitoring.aws-esvc1-useast2.aws.sfdc.cl/d/RwVg-cRSz/temporal-namespace-dashboard)
- [Temporal Dashboard](https://moncloud-grafana.sfproxy.monitoring.aws-esvc1-useast2.aws.sfdc.cl/d/2lZ-4aiMa/temporal-dashboard)
- [Temporal SLO Dashboard](https://moncloud-grafana.sfproxy.monitoring.aws-esvc1-useast2.aws.sfdc.cl/d/Yqb45reGk/falcon-service-owner-dashboard)

**GUS Work Items:**
- W-19692412: Optimize CPU/Memory configs
- W-19356420: Rate limiting and throttling
- W-19361451: HPA based on memory
- W-19747923: VPA exploration
- W-19239873: DB read replica routing
- W-19569289: DB CPU alerts (Ready for Review)
- W-19747878: Container restart alerts
- W-19338432: CPU/Memory alerts
- W-19747831: K8s probe tuning
- W-19747949: Workflow submission error analysis
