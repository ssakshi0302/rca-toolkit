## RCA Analysis #3: Regrello Namespace Archival Backlog

**Document**: RCA for archival backlog incident in aws-test1-uswest2/regrello cluster

### Incident Overview
- **PRB ID**: Not specified
- **Severity**: Not explicitly stated (inferred P1/P2 - service degradation)
- **Type**: Configuration-induced overload / Capacity exhaustion
- **Date**: 2026-04-24 to 2026-04-27 (ongoing at time of summary)
- **Status**: Investigation Summary (root cause identified, resolution ongoing)

### Timeline Summary
| Phase | Timestamp | Duration from Start | Notes |
|-------|-----------|---------------------|-------|
| Archival Enabled | 2026-04-24 13:20 UTC | 0 | 4.5M workflows seeded into archival queue |
| Incident Start | 2026-04-24 ~13:20 UTC | 0 | Backlog begins forming (not draining) |
| Detection | 2026-04-27 17:00 UTC | **TTD: ~3 days** | Investigation launched - backlog at ~15M tasks |
| Root Cause Identified | 2026-04-27 18:00 UTC | **TTD: ~3 days + 1 hour** | "Retry storm" identified - persistence rate limit exhausted |
| Resolution | Ongoing | **TTR: TBD** | Remediation in progress |

**Note on Timeline**: 3-day detection delay is the critical issue. By the time investigation started, the problem had compounded significantly (~5 archivals/min with ~1.08M/min failed retry attempts, 100% throttle rate).

### Critical Delays Analysis

#### Detection Phase (3 days delay)

**Why detection was delayed**:
1. **No monitoring on archival queue drain rate**: System knew queue depth was growing, but no alert on "archival not making progress"
2. **No alerting on archival success/failure ratio**: Failed archivals were retrying silently without raising alarms
3. **No anomaly detection on throttle rate**: 100% persistence throttle rate for namespace went unnoticed
4. **No capacity planning for archival enablement**: When 4.5M workflows were seeded, no validation that namespace could handle the load

**What should have alerted within minutes/hours**:
- Alert: "Archival queue depth increasing for 6+ hours with no drain progress"
- Alert: "Namespace persistence throttle rate >90% for 1+ hour"
- Alert: "Archival failure rate >50% (retry storm detected)"
- Alert: "Archival tasks consuming >X% of namespace persistence quota"

**Impact of delay**: 
- Backlog grew from 4.5M → ~15M tasks over 3 days
- Retry storm self-amplified (each failure triggered more retries)
- Namespace persistence became completely starved (100% throttle)

#### Diagnosis Phase (1 hour after detection)

**What enabled fast diagnosis**:
- Once investigation started, root cause was identified within 1 hour
- Metrics showed clear signal: 100% throttle rate + high retry rate
- Pattern was recognizable: "retry storm" exhausting rate limit

**Key observation**: The diagnosis was fast once attention was focused. The real problem was the 3-day detection gap.

#### Resolution Phase (ongoing)

**Likely resolution approaches**:
1. Increase namespace persistence rate limit (quota bump)
2. Implement backoff/circuit breaker on archival task retries
3. Pause archival to let namespace recover, then re-enable with throttling
4. Batch archival tasks to reduce per-task overhead

**Complexity**: Resolution requires coordination between:
- Quota management (increase rate limit)
- Code changes (improve retry logic)
- Operational changes (phased re-enablement)

### Automation Assessment

#### High-Value Automation Opportunities

1. **Archival Progress Monitoring**
   - **Phase**: Detection
   - **Impact**: Reduces TTD from 3 days to <1 hour
   - **Description**: Monitor archival queue drain rate and alert if:
     - Queue depth increasing for >6 hours
     - Drain rate < expected rate (e.g., <10% of seeded volume per hour)
     - Success rate < 50%
   - **Feasibility**: High - metrics already exist (queue depth, archival rate)
   - **Implementation**: Create derived metric: `(archival_success_rate, archival_drain_rate_per_hour)` with anomaly detection

2. **Namespace Throttle Rate Alerting**
   - **Phase**: Detection
   - **Impact**: Reduces TTD from 3 days to <15 minutes
   - **Description**: Alert on namespace-level persistence throttle rate >75% sustained for >15 minutes
   - **Feasibility**: High - throttle metrics exist per namespace
   - **Implementation**: Alert rule: `namespace_persistence_throttle_rate{namespace="X"} > 0.75 for 15m`

3. **Retry Storm Detection**
   - **Phase**: Detection
   - **Impact**: Reduces TTD from 3 days to <30 minutes
   - **Description**: Pattern detection: high task failure rate + high retry rate + low success rate = retry storm
   - **Feasibility**: High - pattern is recognizable from metrics
   - **Implementation**: Alert when: `(task_retry_rate / task_success_rate) > 10 AND task_success_rate < 0.1`

4. **Archival Capacity Pre-Check**
   - **Phase**: Prevention (before enablement)
   - **Impact**: Prevents incident entirely
   - **Description**: Before enabling archival for namespace:
     - Calculate expected load: `workflows_to_archive * archival_task_cost`
     - Compare to namespace persistence quota
     - If load > 70% of quota, require explicit capacity increase or throttled rollout
   - **Feasibility**: Medium - requires understanding of archival task cost profile
   - **Implementation**: Pre-flight check in archival enablement workflow

#### Medium-Value Opportunities

1. **Archival Health Dashboard**
   - **Phase**: Diagnosis
   - **Impact**: Reduces diagnosis time (already fast - 1 hour)
   - **Description**: Single dashboard showing:
     - Per-namespace archival queue depth trend
     - Archival success/failure/retry rates
     - Namespace persistence quota utilization
     - Estimated time to drain at current rate
   - **Feasibility**: High - data exists, needs aggregation
   - **Benefit**: Speeds up investigation once attention is focused

2. **Automated Capacity Recommendation**
   - **Phase**: Diagnosis → Resolution
   - **Impact**: Reduces time to identify fix (suggest quota increase amount)
   - **Description**: Given observed retry rate and queue depth, calculate:
     - Required persistence quota to drain backlog in target timeframe (e.g., 24 hours)
     - Suggested quota increase value
   - **Feasibility**: Medium - requires modeling archival task cost
   - **Benefit**: Reduces manual calculation during incident

3. **Circuit Breaker for Archival Tasks**
   - **Phase**: Prevention / Auto-mitigation
   - **Impact**: Prevents retry storm amplification
   - **Description**: If archival task failure rate exceeds threshold:
     - Stop retrying immediately
     - Alert operator with "circuit breaker triggered"
     - Require manual intervention to re-enable
   - **Feasibility**: Medium - requires code change to archival task executor
   - **Benefit**: Prevents self-inflicted amplification of the problem

#### Low-Value / Not Automatable

1. **Manual Resolution Decisions**
   - **Why not automatable**: Requires judgment:
     - Should we pause archival entirely or throttle it?
     - How much quota increase is safe/approved?
     - What's the business priority (drain fast vs. minimize cost)?
   - **Human decision needed**: Resolution strategy depends on business context

2. **Quota Approval Workflow**
   - **Why not automatable**: Quota increases may require:
     - Cost approval
     - Capacity planning review
     - Understanding downstream impact (does persistence layer have capacity?)
   - **Human decision needed**: Financial and operational approval gates

#### Special Considerations

**Configuration-Induced Overload Pattern**:
- This incident is caused by enabling a feature (archival) without capacity validation
- Pattern applies to any bulk background task: schema migrations, data backfills, compaction, etc.
- Automation needs to:
  1. **Predict**: "Will this configuration cause overload?"
  2. **Detect**: "Is overload happening right now?"
  3. **React**: "How do we stop the overload?"

**Capacity/Quota Exhaustion Detection**:
- Key insight: The system had the data to detect this (throttle rate, retry rate, queue depth)
- Gap: No alerting logic to combine these signals into "something is wrong"
- Opportunity: Build anomaly detection that learns normal patterns and flags deviations

**Retry Storm as a General Pattern**:
- Retry storms are a common failure mode in distributed systems
- Characteristics: High retry rate + low success rate + resource exhaustion
- Generalize: Any task type (archival, replication, workflow execution) can exhibit this
- Solution: Generic retry storm detector that works across task types

### Key Takeaways

1. **3-day detection gap is the killer**: By the time humans noticed, the problem had compounded 3x (4.5M → 15M). Fast detection (<1 hour) would have prevented retry storm amplification.

2. **All signals existed, but no alerting**: The system knew queue depth, throttle rate, and retry rate. The gap was nobody was watching these together to say "archival is failing."

3. **Capacity validation before config change**: Enabling archival for 4.5M workflows should have triggered a pre-flight check: "Does this namespace have capacity to handle this load?"

4. **Retry storms are self-amplifying**: Without circuit breakers, failed retries consume more resources, causing more failures, causing more retries. Detection + circuit breaker would stop amplification.

5. **Automation opportunity is massive**: This incident could have been detected in <1 hour (vs. 3 days) with basic threshold alerting. More sophisticated anomaly detection could have caught it in <15 minutes.
