# Temporal Splunk Query Patterns

**Source**: Temporal Doctor pd-triage skill (OrcaaS team)  
**URL**: https://git.soma.salesforce.com/orcaas/temporal-doctor/pull/3  
**Purpose**: Production-tested Splunk query patterns for Temporal incident investigation

---

## Splunk API Endpoints

Choose the API endpoint based on the environment:

| Environment | Splunk API Endpoint |
|-------------|---------------------|
| **prod, esvc** | `https://splunk-api-noncore.log-analytics.monitoring.aws-esvc1-useast2.aws.sfdc.cl` |
| **dev, test, preprod** | `https://splunk-api-preprod.log-analytics.monitoring.aws-esvc1-useast2.aws.sfdc.cl` |

**Splunk Web URLs** (for clickable links in reports):

| Environment | Splunk Web Base URL |
|-------------|---------------------|
| **prod, esvc** | `https://splunk-web-noncore.log-analytics.monitoring.aws-esvc1-useast2.aws.sfdc.is` |
| **dev, test, preprod** | `https://splunk-web-preprod.log-analytics.monitoring.aws-esvc1-useast2.aws.sfdc.cl` |

---

## Time Window Strategy

### Recommended Window: 12 Hours Before Incident
Use the same 12-hour window as Argus metrics (baseline comparison).

```
Window: T-12h to T (where T = incident resolution time or now)
```

### Batched Querying (Avoid Timeouts)
Query in **1-hour batches** to avoid timeout and 5,000 event cap:

```
Batch 1:  earliest=T-12h  latest=T-11h
Batch 2:  earliest=T-11h  latest=T-10h
...
Batch 12: earliest=T-1h   latest=T
```

**Run batches in parallel** (3-4 concurrent queries) for faster results.

**Prioritization**: If time is limited, start with **most recent 2 hours** (closest to incident).

---

## Query Structure

### Phase 1: Error Summary (Stats Grouping)
Get count of errors grouped by type/message for each 1-hour batch:

```spl
search index=distapps 
  falcon_instance=*<FI>-* 
  functional_domain=<FD> 
  k8s_namespace=orcaas
  k8s_pod_name=<service><cluster_index>-*
  (level=error OR level=warning OR "*error*")
  earliest=<batch_start> 
  latest=<batch_end>
  | rex field=_raw "\"msg\":\"(?<error_msg>[^\"]+)\""
  | rex field=_raw "\"error\":\"(?<error_detail>[^\"]+)\""
  | rex field=_raw "\"grpc_code\":\"(?<grpc_code>[^\"]+)\""
  | rex field=_raw "\"operation\":\"(?<operation>[^\"]+)\""
  | stats count by error_msg, grpc_code, operation
  | sort -count
```

**Columns**: `["error_msg", "grpc_code", "operation", "count"]`

After all batches, **merge results** by summing counts for each unique (error_msg, grpc_code, operation) combination.

### Phase 2: Sample Logs for Top Errors
After identifying top error types, fetch raw samples for context:

```spl
search index=distapps 
  falcon_instance=*<FI>-* 
  functional_domain=<FD> 
  k8s_namespace=orcaas
  k8s_pod_name=<service><cluster_index>-*
  "*<top_error_msg>*"
  earliest=<epoch_start> 
  latest=<epoch_end>
  | head 5
```

**Columns**: `["_time", "_raw"]`

---

## Field Filtering Best Practices

### 1. Falcon Instance (FI) — Always Use Wildcards
**IMPORTANT**: Use `*<FI>-*` with trailing dash to avoid false matches.

**Why**: Splunk `falcon_instance` values contain region suffixes:
- PagerDuty FI: `esvc1` or `aws-esvc1-useast2`
- Splunk values: `aws-esvc1-useast2-euwest3`, `aws-prod12-apsouth2`, etc.

**Examples**:
```spl
falcon_instance=*esvc1-*      ✅ Correct (matches esvc1 in any region)
falcon_instance=*prod12-*     ✅ Correct
falcon_instance=*prod1*       ❌ WRONG (also matches prod10, prod11, prod12, ...)
```

### 2. Service Instance (SI) — Extract Cluster Index
**Pattern**: `k8s_pod_name=<service><cluster_index>-*`

**How to extract cluster index**:
- SI format: `temporalfrontend2001` → service=`temporalfrontend`, cluster=`2`
- SI format: `temporalhistory1001` → service=`temporalhistory`, cluster=`1`
- SI format: `temporalmatching3001` → service=`temporalmatching`, cluster=`3`

**Extraction logic**:
```
SI: temporalfrontend2001
service = temporalfrontend
cluster_index = 2 (digit between service name and trailing digits)
k8s_pod_name filter = temporalfrontend2-*
```

**Examples**:
```spl
k8s_pod_name=temporalfrontend2-*     ✅ Cluster 2 only
k8s_pod_name=temporalhistory1-*      ✅ Cluster 1 only
k8s_pod_name=temporalmatching3-*     ✅ Cluster 3 only
```

### 3. Functional Domain (FD)
**Standard value**: `foundation`

Other FDs may exist (e.g., `cdp2`), use value from PagerDuty alert.

---

## Common Error Extraction Patterns

### Extract Error Message
```spl
| rex field=_raw "\"msg\":\"(?<error_msg>[^\"]+)\""
```

### Extract Error Detail
```spl
| rex field=_raw "\"error\":\"(?<error_detail>[^\"]+)\""
```

### Extract gRPC Code
```spl
| rex field=_raw "\"grpc_code\":\"(?<grpc_code>[^\"]+)\""
```

### Extract Operation
```spl
| rex field=_raw "\"operation\":\"(?<operation>[^\"]+)\""
```

### Extract Service Name (from log)
```spl
| rex field=_raw "\"service\":\"(?<service_name>[^\"]+)\""
```

### Extract Namespace
```spl
| rex field=_raw "\"wf-namespace\":\"(?<wf_namespace>[^\"]+)\""
```

---

## Example Queries

### Error Summary for temporalfrontend (cluster 1) on esvc1/foundation
```spl
search index=distapps 
  falcon_instance=*esvc1-* 
  functional_domain=foundation 
  k8s_namespace=orcaas
  k8s_pod_name=temporalfrontend1-*
  (level=error OR "*error*") 
  earliest=1776735000 
  latest=1776738600
  | rex field=_raw "\"msg\":\"(?<error_msg>[^\"]+)\""
  | rex field=_raw "\"grpc_code\":\"(?<grpc_code>[^\"]+)\""
  | rex field=_raw "\"operation\":\"(?<operation>[^\"]+)\""
  | stats count by error_msg, grpc_code, operation
  | sort -count
```

### Error Summary for temporalhistory (cluster 2) on dev2/foundation
```spl
search index=distapps 
  falcon_instance=*dev2-* 
  functional_domain=foundation 
  k8s_namespace=orcaas
  k8s_pod_name=temporalhistory2-*
  (level=error OR "*error*") 
  earliest=1776735000 
  latest=1776738600
  | rex field=_raw "\"msg\":\"(?<error_msg>[^\"]+)\""
  | rex field=_raw "\"grpc_code\":\"(?<grpc_code>[^\"]+)\""
  | stats count by error_msg, grpc_code
  | sort -count
```

### Sample Logs for Specific Error
```spl
search index=distapps 
  falcon_instance=*esvc1-* 
  functional_domain=foundation 
  k8s_namespace=orcaas
  k8s_pod_name=temporalfrontend1-*
  "*service failures*"
  earliest=1776735000 
  latest=1776738600
  | head 5
```

---

## Timeout Fallback Strategy

If a 1-hour batch query times out (rare), try:

### Fallback 1: Simplified Grouping
```spl
| top limit=20 error_msg
```

### Fallback 2: Raw Head
```spl
| head 50
```

Note the failed batch in output and continue with other batches.

---

## How to Present Error Summary

Group errors by severity/frequency in triage output:

```
**Error Logs (Splunk, last 12h):**
• `service failures` (grpc_code=Internal, op=StreamWorkflowReplicationMessages) — *1,247 occurrences*
• `persistence error` (grpc_code=Unavailable, op=UpdateShard) — *89 occurrences*
• `context deadline exceeded` (grpc_code=DeadlineExceeded) — *23 occurrences*

Top error sample:
`rpc error: code = Internal desc = unknown cluster ID: 397938`
Source: `common/rpc/interceptor/request_error_handler.go:96`

Splunk Web: <clickable link>
```

---

## Generating Splunk Web Links

Format for clickable link in reports:

```
<base_url>/en-US/app/search/search?q=search%20index%3Ddistapps%20falcon_instance%3D*<FI>-*%20functional_domain%3D<FD>%20k8s_namespace%3Dorcaas%20%22*error*%22&earliest=<epoch>&latest=<epoch>
```

**URL encoding**:
- Space → `%20`
- Colon → `%3D`
- Asterisk → `*` (no encoding needed)
- Quote → `%22`

---

## Data Constraints

| Constraint | Value | Workaround |
|------------|-------|------------|
| **Retention** | 15 days | If incident >15 days old, skip Splunk, note in summary |
| **Results cap (raw)** | 500 lines | Use `\| head 500` for raw queries |
| **Event cap (stats)** | 5,000 events | Batch into 1-hour chunks |
| **Query timeout** | 60 seconds | Use fallback (top limit=20 or head 50) |

---

## Common Patterns by Service

### Frontend Errors
- `upstream connect error` → Check history/matching service health
- `context deadline exceeded` → Check downstream latency
- `resource exhausted` → Check rate limiting or DB throttling
- `nondeterministic workflow` → Workflow code issue

### History Errors
- `persistence error` → Check DB health (RDS CPU, connections)
- `shard acquisition failed` → Check cluster membership, DB locks
- `visibility error` → Check Elasticsearch health

### Matching Errors
- `no poller available` → Workers down or insufficient capacity
- `task queue lease failed` → Check DB health, optimistic lock contention
- `approximate backlog high` → Workers can't keep up with task rate

### Worker Errors
- `workflow task execution failed` → Workflow code error
- `activity task timeout` → Activity took too long or heartbeat missing
- `schedule missed catchup window` → Scheduler backlog

---

## Cross-Service Correlation

When investigating errors on one service, check sibling services in the same FI/FD:

```spl
# Check all Temporal services in same FI/FD
search index=distapps 
  falcon_instance=*<FI>-* 
  functional_domain=<FD> 
  k8s_namespace=orcaas
  (k8s_pod_name=temporalfrontend*-* OR 
   k8s_pod_name=temporalhistory*-* OR 
   k8s_pod_name=temporalmatching*-* OR 
   k8s_pod_name=temporalworker*-*)
  (level=error OR "*error*")
  earliest=<start> 
  latest=<end>
  | rex field=_raw "\"service\":\"(?<service>[^\"]+)\""
  | rex field=_raw "\"msg\":\"(?<error_msg>[^\"]+)\""
  | stats count by service, error_msg
  | sort -count
```

This helps identify if the alerting service is the root cause or a symptom of a sibling service failure.

---

## Notes

- **Always query distapps index** (`index=distapps` — never use `index=*`)
- **Use epoch seconds** for `earliest` and `latest` (not relative time)
- **Batch large time windows** (1-hour chunks) to avoid timeouts
- **Prioritize recent data** (last 2 hours) for active incidents
- **Include service instance filter** (`k8s_pod_name`) to scope to specific cluster
- **Merge batch results** by summing counts across all batches

---

**Last updated**: 2026-05-09  
**Source**: Temporal Doctor pd-triage skill, OrcaaS team operational knowledge  
**Cross-reference**: See `runbooks/metrics-catalog.md` for Splunk queries used in RCA analyses
