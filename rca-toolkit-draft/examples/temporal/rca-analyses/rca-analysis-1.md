## RCA Analysis #1: ESVC1 Day2 Clusters Availability Alerts

### Incident Overview
- **PRB ID**: PRB-0028419
- **Severity**: SEV-3 (initially SEV-4, escalated to SEV-3 on Aug 2)
- **Type**: Performance degradation / Resource exhaustion
- **Date**: July 30 - August 3, 2026
- **ICC Channel**: #icc-78705213
- **RCA Document**: https://docs.google.com/document/d/1gIb5mRWSKwWq824lXrET2rYPw1tXgZin1iVGwcC8emI/edit

### Timeline Summary
| Phase | Timestamp | Duration from Start | Notes |
|-------|-----------|---------------------|-------|
| Incident Start | July 30, 00:54 AM IST | 0 | High DB CPU utilization begins (silent failure) |
| Detection | July 30, 08:29 PM IST | **TTD: ~20 hours** | First PD alert fires for UI Availability on cluster 2011 |
| Investigation Started | July 30, 08:52 PM IST | +20h 58m | Team discovers alert was false positive for 2011, actual drop on 2031 |
| Root Cause Identified | July 31, 10:36 PM IST | +45h 42m | **TTD (diagnose): ~25 hours** | OOM kills in opencensus-refinery sidecar discovered |
| Incident Created | July 31, 11:44 PM IST | +46h 50m | ICC spun up as SEV-4 |
| Mitigation Decision | Aug 1, 11:14 AM IST | +58h 20m | Finalized to scale DB from large to 2xlarge + matching resources |
| Resolution Applied | Aug 3, 6:30 AM IST | **TTR: 4d 6h** | DB scaled, matching resources increased |

### Critical Delays Analysis

#### Detection Phase (TTD: ~20 hours)
**What caused delay:**
- **Silent failure mode**: DB CPU utilization reached ~100% starting July 30 00:54 AM, but no alerts existed for DB CPU monitoring. The database quietly saturated for 20 hours before downstream symptoms manifested.
- **Downstream symptom alerting**: Detection relied on availability drops in UI service (which queries frontend/history, which query DB). This multi-hop dependency chain delayed visibility.
- **False positive alerts**: The first alert fired for cluster 2011 (prod), but the actual availability drop was on cluster 2031 (pre-prod). The alert query was "cluster agnostic" and incorrectly routed, causing initial confusion.
- **Intermittent errors**: The 5xx errors on UI and SFMP were intermittent, not sustained, making the issue appear less severe initially.

**Automation opportunity:**
- **Detection**: Direct DB CPU utilization alerting would have detected the issue at 00:54 AM instead of 20 hours later. Alert on: `db.cpu_utilization > 80% for 10 minutes`.
- **Pattern recognition**: Correlation between upstream DB saturation and downstream service errors (frontend timeouts → UI 5xx) could have been flagged immediately when DB CPU spiked.
- **Resource saturation detection**: Container/sidecar memory usage alerts (opencensus-refinery sidecar OOM) would have provided an earlier signal. Alert on: `container_memory_usage > 90% of limit`.
- **Value**: Reduce TTD by ~19.5 hours (from 20h to 30 minutes).

#### Diagnosis Phase (~25 hours to identify root cause)
**What caused delay:**
- **Misleading pod status**: Grafana dashboard showed all 21 matching pods as "Running" even though they were in CrashLoopBackOff state. This incorrect dashboard data led investigators down the wrong path.
- **Metrics lag confusion**: Matching availability metrics appeared to be lagging, causing team to suspect monitoring infrastructure issues rather than actual service problems. Time spent debugging monitoring system (~4 hours).
- **Cluster attribution error**: Alerts fired for cluster 2011, but availability drop was on cluster 2031. Team spent time validating alert expressions and confirming with mesh team that queries were correct (~8 hours).
- **Splunk log gaps**: Initial Splunk queries on the noncore instance returned no results for aws-esvc1-useast2. Team had to discover the correct Splunk index (index=distapps with FI/FD filters).
- **Hidden root cause**: The actual root cause (massive worker polling from publicproxy_day2 and sitebridge_day2 namespaces, each with unique task queues) required deep investigation into namespace-level request patterns.

**Automation opportunity:**
- **Diagnosis**: Automated pod health correlation across multiple data sources (Kubernetes API + Grafana + container status) would immediately surface the CrashLoopBackOff vs "Running" discrepancy.
- **Namespace workload analysis**: Automatic detection of anomalous namespace behavior: "namespace X has 2500 workers with 2500 unique task queues, causing 10x normal poll request volume". This pattern should trigger investigation prompts.
- **Resource exhaustion causation chain**: Automated correlation: `DB CPU 100% → matching service errors → opencensus-refinery sidecar OOM → matching pods CrashLoopBackOff → availability drop`. Present this chain immediately.
- **Historical pattern matching**: Similar incident occurred in GIA2H (Dec 2025) with "stale IP" issue resolved by rolling restart. Automation should surface: "Similar pattern to PRB-XXXXX from Dec 2025, consider rolling restart".
- **Value**: Reduce diagnosis time by ~20 hours (from 25h to 5h).

#### Resolution Phase (4 days 6 hours to resolve)
**What caused delay:**
- **Validation requirements**: Team needed to reproduce the issue in lower environments (fdev1/cluster1, dev2/foundation) to ensure no customer impact during remediation. This took ~36 hours:
  - Generated 2500 workers with 2500 unique task queues to replicate load
  - Tested DB upgrade from large to 2xlarge with active workload
  - Validated matching resource increase (CPU: 1→2, Memory: 2Gi→4Gi)
- **Configuration discovery**: DB "apply immediately" flag was set to false by default, causing DB upgrade to be scheduled for maintenance window instead of executing immediately. Required creating new EBF_artifact branch (~3 hours).
- **Moratorium and trigger version confusion**: Terraform changes hadn't been released since July 2024 due to moratorium. Different clusters built with different trigger versions (1, 2, or 3), requiring manual validation to prevent unintended password rotation (~4 hours).
- **Missing pipelines**: Rolling restart pipelines were not present on production for matching, history, worker, and ui services. Required creating MO change cases as backup (~6 hours).
- **EAR process**: Emergency Approval Request (EAR) document preparation, leadership review, and approval took ~12 hours.
- **Stringent code hygiene issues**: Inconsistent Git branching in temporal-helm-charts and temporal-aws-terraform repos made hot-fix deployment more challenging.

**Automation opportunity:**
- **Resolution**: Partial automation possible, but this incident required significant human judgment:
  - **Auto-scaling**: If DB CPU thresholds are exceeded, system could auto-recommend (not auto-execute) scaling plan with estimated impact and cost.
  - **Namespace throttling**: Automated dynamic config updates to rate-limit namespaces exceeding thresholds (e.g., `frontend.namespaceCount` limit). This could have been applied immediately on July 30 to mitigate impact while planning DB scaling.
  - **Rolling restart automation**: For known patterns (mesh routing issues, stale IPs), system could suggest rolling restart with one-click approval workflow.
- **Why full automation not feasible**: This resolution required infrastructure changes (DB scaling), cost implications ($), multi-environment validation, and compliance processes (EAR, leadership approval). These require human judgment.
- **Value**: Reduce resolution time by ~1 day (from 4d 6h to 3d 6h) through faster diagnosis and auto-generated remediation plans.

### Slack/Communication Analysis
**ICC Channel observations** (#icc-78705213):
- Thread opened July 31, 11:44 PM IST (46 hours after incident start)
- Investigation began July 30, 8:52 PM IST in #temporal-oncall-discussion, but formal ICC not created until 26 hours later
- Pattern of confusion about:
  - Which cluster was actually affected (2011 vs 2031)
  - Whether monitoring system was broken (metrics lag confusion)
  - Where to find logs (Splunk noncore vs distapps index)
- Customer reports came through support threads, not directly to oncall:
  - PublicProxy team: 503 errors on SFMP (reported Aug 4, but occurred Aug 1)
  - Perforce team: 504 Gateway Timeout and UI loading issues (July 31, 2:58 PM)
- Repeated manual checks of dashboards, pod status, and metrics across multiple team members

### Automation Assessment

#### High-Value Automation Opportunities
1. **DB CPU Utilization Alerting**: [GUS W-17125000]
   - Phase: Detection
   - Impact: Reduces TTD by ~19.5 hours (from 20h to 30m)
   - Description: Alert when DB CPU > 80% for 10 minutes. Include metadata: which namespaces are generating the most queries, query types (poll, read, write), and trend (increasing/stable).
   - Feasibility: High (standard Argus alert on Aurora CloudWatch metrics)

2. **Container/Sidecar Resource Monitoring**: [GUS W-19338432]
   - Phase: Detection
   - Impact: Reduces TTD by ~15 hours (earlier signal from sidecar OOM)
   - Description: Alert on container memory/CPU usage > 90% of limits, especially for opencensus-refinery sidecars. Include OOMKill detection.
   - Feasibility: High (Kubernetes metrics already available)

3. **Pod Health Correlation Dashboard**: [GUS W-19338313]
   - Phase: Diagnosis
   - Impact: Reduces diagnosis time by ~4 hours (eliminate "misleading pod status" confusion)
   - Description: Cross-reference Kubernetes pod status with Grafana metrics. Flag discrepancies: "Grafana shows 21 Running, but kubectl shows 18 CrashLoopBackOff".
   - Feasibility: High (query both K8s API and Grafana, present in single panel)

4. **Namespace Workload Anomaly Detection**:
   - Phase: Diagnosis
   - Impact: Reduces diagnosis time by ~10 hours (immediately identify publicproxy/sitebridge as outliers)
   - Description: Baseline normal poll request rates per namespace. Alert when namespace exceeds 3x baseline or creates excessive unique task queues (>500). Show: "publicproxy_day2 has 5000 workers with 5000 unique task queues, generating 300K poll requests/min (10x normal)".
   - Feasibility: Medium (requires baseline learning period, metric collection per namespace)

5. **Auto-Suggested Remediation Runbook**:
   - Phase: Resolution
   - Impact: Reduces resolution planning time by ~6 hours (auto-generate scaling recommendation)
   - Description: When DB CPU saturated due to high poll volume, system suggests: "Option 1: Scale DB from large to 2xlarge (cost: $X/month, estimated time: 30min, downtime: none). Option 2: Apply namespace rate limiting via dynamic config. Option 3: Rolling restart (only if mesh routing suspected)."
   - Feasibility: Medium (requires decision tree logic, cost estimation, runbook integration)

#### Medium-Value Opportunities
1. **Alert Query Correctness Validation**: [GUS W-19338274]
   - Phase: Detection
   - Impact: Prevents false positives, saves ~2 hours of investigation time
   - Description: Ensure alerts fire for correct cluster/service instance. Run monthly audit: "For each alert, verify last 10 fires matched correct instance".
   - Feasibility: High (alert configuration validation script)

2. **Historical Incident Pattern Matching**:
   - Phase: Diagnosis
   - Impact: Saves ~3 hours by surfacing similar past incidents
   - Description: When symptoms match prior incidents (GIA2H stale-IP Dec 2025), surface in triage dashboard: "This pattern matches PRB-XXXXX (Dec 2025). That incident was resolved by rolling restart. Consider checking PassthroughCluster traffic."
   - Feasibility: Medium (requires RCA corpus embedding and similarity matching)

3. **Splunk Index Auto-Routing**:
   - Phase: Diagnosis
   - Impact: Saves ~1 hour (eliminate Splunk instance confusion)
   - Description: Provide triage command: `/splunk-search cluster=2031` → automatically routes to correct index (distapps vs noncore) with pre-filled FI/FD filters.
   - Feasibility: High (mapping table of FI → Splunk instance + query template)

#### Low-Value / Not Automatable
1. **Multi-Environment Validation**:
   - Phase: Resolution
   - Why not automatable: Requires judgment about acceptable risk, cost trade-offs, and impact tolerance. Testing DB scaling under production-like load requires human-designed test scenarios and observation.

2. **EAR Process and Leadership Approval**:
   - Phase: Resolution
   - Why not automatable: Organizational process requiring human review of business risk, cost, and change impact. Automation could speed up document generation (template pre-fill), but approval decisions remain human.

3. **Git Hygiene and Pipeline Hydration**:
   - Phase: Resolution
   - Why low value for automation: These are process/hygiene issues that slow down resolution but don't directly increase TTR. Automation would be build-time CI/CD checks, not incident-time.

### Key Takeaways
1. **Infrastructure monitoring gaps are critical**: 20-hour detection delay was entirely due to lack of DB CPU alerting. Downstream service availability alerts are insufficient for detecting infrastructure bottlenecks.
2. **Monitoring correctness matters as much as coverage**: Misleading pod status and cluster-agnostic alert queries cost 8+ hours of diagnosis time. Automation must validate monitoring data consistency.
3. **Namespace workload visibility is a blind spot**: The root cause (2500 workers with unique task queues from 2 namespaces) was hidden until deep manual investigation. Namespace-level workload anomaly detection would have surfaced this immediately.
4. **Resolution automation is limited by compliance**: Even with perfect detection and diagnosis, this incident required 4+ days due to validation, EAR processes, and infrastructure changes. Automation can accelerate diagnosis and planning, but resolution still requires human judgment for high-risk changes.
