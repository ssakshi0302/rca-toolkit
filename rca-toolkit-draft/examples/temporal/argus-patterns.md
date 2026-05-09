# Temporal Argus Query Patterns

**Source**: Temporal Doctor pd-triage skill (OrcaaS team)  
**URL**: https://git.soma.salesforce.com/orcaas/temporal-doctor/pull/3  
**Purpose**: Production-tested Argus metric query patterns for Temporal incident investigation

**Full Argus documentation**: https://git.soma.salesforce.com/ArgusMonitoring/Argus/wiki/Transforms

---

## Argus Scope Pattern

**Never hardcode scopes** — FI format varies. Always discover first.

### Scope Discovery Query
```
scope: temporal*<service>*<FI_short>*<FD>*
metric: *
discovery_type: scope
```

**FI_short examples**:
- `esvc1` for `aws-esvc1-useast2`
- `fdev1` for `fdev1-uswest2`
- `prod12` for `aws-prod12-apsouth2`

**Example**:
```
scope: temporal*temporalhistory*esvc1*foundation*
metric: *
discovery_type: scope
```

Returns: `temporal.temporalhistory.aws.aws-esvc1-useast2.foundation`

---

## Query Patterns by Metric Type

### Counter (C) → Rate
Temporal counters are **cumulative**. Always convert to rate.

```
RATE(<ms>:<ms>:<scope>:<metric>{service_instance=<SI>}:sum:1m-sum,#1m#,#true#,#true#)
```

**Parameters**:
- `<ms>`: Epoch milliseconds (start and end)
- `<scope>`: Discovered scope (e.g., `temporal.temporalmatching.aws.aws-esvc1-useast2.foundation`)
- `<metric>`: Counter metric name (e.g., `client_errors`)
- `{service_instance=<SI>}`: Filter to specific SI (e.g., `temporalmatching2011`)
- `:sum:1m-sum`: Aggregation (sum across all pods, 1-minute resolution)
- `#1m#`: Rate interval (1-minute rate)
- `#true#`: Handle counter resets (removes negative spikes from pod restarts)
- `#true#`: Interpolate missing data points

**Example** (client_errors counter):
```
RATE(1776735000000:1776778600000:temporal.temporalmatching.aws.aws-esvc1-useast2.foundation:client_errors{service_instance=temporalmatching2011}:sum:1m-sum,#1m#,#true#,#true#)
```

---

### Histogram (H) → Average
Histograms emit `_sum` (total) and `_count` (observations).  
**Average** = rate(sum) / rate(count).

```
DIVIDE_V(
  RATE(<ms>:<ms>:<scope>:<metric>_sum{service_instance=<SI>}:sum:1m-sum,#1m#,#true#,#true#),
  RATE(<ms>:<ms>:<scope>:<metric>_count{service_instance=<SI>}:sum:1m-sum,#1m#,#true#,#true#)
)
```

**Example** (client_latency histogram):
```
DIVIDE_V(
  RATE(1776735000000:1776778600000:temporal.temporalfrontend.aws.aws-esvc1-useast2.foundation:client_latency_sum{service_instance=temporalfrontend2001}:sum:1m-sum,#1m#,#true#,#true#),
  RATE(1776735000000:1776778600000:temporal.temporalfrontend.aws.aws-esvc1-useast2.foundation:client_latency_count{service_instance=temporalfrontend2001}:sum:1m-sum,#1m#,#true#,#true#)
)
```

---

### Gauge (G) → Direct Query
Gauges are instantaneous values. Query directly with avg aggregator.

```
<ms>:<ms>:<scope>:<metric>{service_instance=<SI>}:avg:1m-avg
```

**Example** (num_goroutines gauge):
```
1776735000000:1776778600000:temporal.temporalhistory.aws.aws-esvc1-useast2.foundation:num_goroutines{service_instance=temporalhistory2011}:avg:1m-avg
```

---

## Derived Metrics

### Error Percentage
```
SCALE(DIVIDE_V(
  RATE(<ms>:<ms>:<scope>:service_errors{service_instance=<SI>}:sum:1m-sum,#1m#,#true#,#true#),
  RATE(<ms>:<ms>:<scope>:service_requests{service_instance=<SI>}:sum:1m-sum,#1m#,#true#,#true#)
),#100#)
```

This calculates: (errors / requests) × 100

---

### Removing Counter Reset Artifacts
RATE with `#true#` handles negative spikes, but first value after reset can be abnormally high.

**Use CULL_ABOVE to remove outliers**:
```
CULL_ABOVE(
  RATE(<ms>:<ms>:<scope>:<metric>{service_instance=<SI>}:sum:1m-sum,#1m#,#true#,#true#),
  #<threshold>#,
  #value#
)
```

**Example** (remove spikes >10,000):
```
CULL_ABOVE(
  RATE(...),
  #10000#,
  #0#
)
```

---

## Time Range Best Practices

### Always Use Epoch Milliseconds
**Never use relative time** (`-2h`) in production queries — it shifts with each execution.

Convert to epoch milliseconds:
```
Python: int(datetime.timestamp() * 1000)
Bash: date +%s%3N
```

### Recommended Window for Incident Analysis
**12 hours before incident** to establish baseline + incident window.

Example:
- Incident detected: 2025-09-06 14:00 UTC → `1776778800000`
- Start baseline: 2025-09-06 02:00 UTC → `1776735600000`
- Query: `1776735600000:1776778800000:<scope>:<metric>...`

---

## Service Instance Filtering

Always filter by `service_instance` tag to scope to specific cluster:

```
{service_instance=temporalfrontend2001}
```

**Format**: `<service><cluster_id><trailing_digits>`
- `temporalfrontend2001` → service=temporalfrontend, cluster=2
- `temporalhistory1001` → service=temporalhistory, cluster=1
- `temporalmatching3001` → service=temporalmatching, cluster=3

---

## Aggregation Patterns

### Across All Pods (Sum)
```
:sum:1m-sum
```
Use for counters/histograms (total error count, total request count).

### Across All Pods (Average)
```
:avg:1m-avg
```
Use for gauges (average goroutines, average memory).

### Per-Pod Breakdown
```
:avg:1m-avg-1m-avg
```
Returns per-pod time series (useful for identifying which pod is misbehaving).

---

## Argus MVP Links

Always include Argus MVP link in reports for interactive exploration:

```
https://monitoring.internal.salesforce.com/argusmvp/#/editexpression?expression=<URL_ENCODED_QUERY>
```

**URL encoding**:
- Space → `%20`
- Colon → `%3A`
- Hash → `%23`
- Curly brace → `%7B` / `%7D`
- Comma → `%2C`

**Example**:
```
Query: RATE(-2h:temporal.temporalfrontend.aws.aws-esvc1-useast2.foundation:client_errors{service_instance=temporalfrontend2001}:sum:1m-sum,#1m#,#true#,#true#)

URL-encoded: RATE(-2h%3Atemporal.temporalfrontend.aws.aws-esvc1-useast2.foundation%3Aclient_errors%7Bservice_instance%3Dtemporalfrontend2001%7D%3Asum%3A1m-sum%2C%231m%23%2C%23true%23%2C%23true%23)

Full link: https://monitoring.internal.salesforce.com/argusmvp/#/editexpression?expression=RATE(-2h%3Atemporal.temporalfrontend.aws.aws-esvc1-useast2.foundation%3Aclient_errors%7Bservice_instance%3Dtemporalfrontend2001%7D%3Asum%3A1m-sum%2C%231m%23%2C%23true%23%2C%23true%23)
```

---

## Transform Quick Reference

| Transform | Purpose | Example |
|-----------|---------|---------|
| **RATE** | Convert counter to rate | `RATE(...,#1m#,#true#,#true#)` |
| **DIVIDE_V** | Divide two series | `DIVIDE_V(RATE(...), RATE(...))` |
| **SCALE** | Multiply by constant | `SCALE(...,#100#)` |
| **CULL_ABOVE** | Remove values >threshold | `CULL_ABOVE(...,#10000#,#0#)` |
| **CULL_BELOW** | Remove values <threshold | `CULL_BELOW(...,#0.01#,#0#)` |
| **DERIVATIVE** | Simple delta (NO reset handling) | Avoid — use RATE instead |

**Important**: Always use RATE (not DERIVATIVE) for Temporal counters — RATE handles pod restarts correctly.

---

## Common Query Patterns

### Check Service Health (`up` gauge)
```
<ms>:<ms>:temporal.<service>.aws.<FI>.<FD>:up{service_instance=<SI>}:avg:1m-avg
```

**Interpretation**:
- `up=1` → Service is healthy
- `up=0` → Service is down
- Data gap (no points) → Service stopped emitting metrics (crash/OOM)

---

### Check Request Rate (counter)
```
RATE(<ms>:<ms>:temporal.<service>.aws.<FI>.<FD>:service_requests{service_instance=<SI>}:sum:1m-sum,#1m#,#true#,#true#)
```

**Interpretation**:
- Steady rate → Normal load
- Drop to zero → Service stopped processing
- Spike → Load increase

---

### Check Error Rate (counter)
```
RATE(<ms>:<ms>:temporal.<service>.aws.<FI>.<FD>:service_errors{service_instance=<SI>}:sum:1m-sum,#1m#,#true#,#true#)
```

**Interpretation**:
- Low baseline → Normal error rate
- Spike → Incident window
- High sustained → Systemic issue

---

### Check Latency (histogram)
```
DIVIDE_V(
  RATE(<ms>:<ms>:temporal.<service>.aws.<FI>.<FD>:service_latency_sum{service_instance=<SI>}:sum:1m-sum,#1m#,#true#,#true#),
  RATE(<ms>:<ms>:temporal.<service>.aws.<FI>.<FD>:service_latency_count{service_instance=<SI>}:sum:1m-sum,#1m#,#true#,#true#)
)
```

**Interpretation**:
- Baseline latency → Normal performance
- Spike → Downstream slowness or resource contention
- Sustained increase → Capacity issue

---

### Check Error Percentage (derived)
```
SCALE(DIVIDE_V(
  RATE(<ms>:<ms>:temporal.<service>.aws.<FI>.<FD>:service_errors{service_instance=<SI>}:sum:1m-sum,#1m#,#true#,#true#),
  RATE(<ms>:<ms>:temporal.<service>.aws.<FI>.<FD>:service_requests{service_instance=<SI>}:sum:1m-sum,#1m#,#true#,#true#)
),#100#)
```

**Interpretation**:
- <1% → Healthy
- 1-5% → Elevated but possibly acceptable
- >5% → Incident-level error rate

---

## Sibling Service Checks

Always check sibling services in same FI/FD for root cause attribution:

### Mandatory Sibling Metrics (3 checks per sibling)
1. **`up`** (gauge) — Is service running?
2. **`service_requests`** (counter→RATE) — Is service processing?
3. **`service_errors`** (counter→RATE) — Is service healthy?

**Why all 3**:
- `up=0` or `service_requests=0` → Service is DOWN (likely root cause)
- `up=1` + `service_requests` normal + low `service_errors` → Service is healthy
- Data gap in `up` or `service_requests` → Service was unavailable (treat as down)

**Siblings to check** (same FI/FD):
- temporalfrontend
- temporalhistory
- temporalmatching
- temporalworker
- temporalui
- temporalresourcemanager

---

## Argus Timeout Fallback

If 12h query times out, try shorter windows:
1. 12h → 6h
2. 6h → 1h
3. 1h → 15min

Note reduced window in summary.

---

## Notes

- **Always use epoch milliseconds** (not relative time)
- **Always discover scopes** (don't hardcode FI format)
- **Always filter by service_instance** (scope to specific cluster)
- **Always use RATE for counters** (not DERIVATIVE)
- **Always check sibling `up` + `service_requests`** before attributing root cause
- **Always provide Argus MVP links** for interactive exploration

---

**Last updated**: 2026-05-09  
**Source**: Temporal Doctor pd-triage skill, OrcaaS team operational knowledge  
**Cross-reference**: 
- Complete metric catalog: `.claude/context/temporal-metrics-complete-catalog.md`
- RCA-specific findings: `runbooks/metrics-catalog.md`
