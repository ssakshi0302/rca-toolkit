# Team Onboarding Guide

**Purpose**: Onboard your team to RCA Toolkit to identify automation opportunities and reduce incident response time (TTD, TTX, TTR)

---

## Overview

**What you'll create**:
1. Team config (points to your metrics/queries/architecture)
2. Knowledge files (your team's operational knowledge)
3. Custom runbook templates (optional, if default patterns don't fit)

**Time**: 1-2 hours for initial setup

---

## Phase 1: Gather Your Knowledge Files

**You need**:
- Metrics catalog (what metrics your services expose)
- Query patterns (Argus/Prometheus/Splunk queries)
- Service architecture (optional, but helpful for diagnosis)

**Example structure**:
```
your-team-repo/
└── .claude/context/
    ├── metrics-catalog.md        # Your metrics
    ├── query-patterns.md         # Your queries
    └── service-architecture.md   # Your architecture
```

### Metrics Catalog Format

**Minimum**: List your key metrics

```markdown
# Team Metrics Catalog

## Service Health
- `api_request_total` (Counter) - Total requests
- `api_request_errors` (Counter) - Error count
- `api_request_duration` (Histogram) - Request latency

## Resource Utilization
- `cpu_usage_percent` (Gauge) - CPU usage
- `memory_usage_bytes` (Gauge) - Memory usage
```

### Query Patterns Format

**Minimum**: Basic query examples for common investigations

```markdown
# Query Patterns

## Check Error Rate
Prometheus:
`rate(api_request_errors[5m]) / rate(api_request_total[5m])`

Splunk:
`index=myservice level=error | stats count by service`

## Check Latency (P99)
Prometheus:
`histogram_quantile(0.99, rate(api_request_duration_bucket[5m]))`
```

---

## Phase 2: Create Team Config

**Location**: `.claude/config/<team>-config.yaml`

**Template**: Start with `rca-toolkit/templates/config/team-config-example.yaml`

### Minimal Config

```yaml
team:
  name: MyTeam
  service_prefix: myservice

services:
  - name: myservice-api
  - name: myservice-worker

knowledge:
  metrics_catalog: .claude/context/metrics-catalog.md
  query_patterns: .claude/context/query-patterns.md

environments:
  HIGH: [prod]
  MEDIUM: [staging]
  LOW: [dev]
```

### Full Config (with alerts and patterns)

```yaml
team:
  name: MyTeam
  service_prefix: myservice
  pagerduty_service: MyService Production

services:
  - name: myservice-api
    port: 8080
    type: API Gateway
  - name: myservice-worker
    port: 8081
    type: Background Worker
  - name: myservice-db
    port: 5432
    type: Database

infrastructure:
  database: PostgreSQL
  search: Elasticsearch
  mesh: Istio
  monitoring: Prometheus
  logging: Splunk

knowledge:
  metrics_catalog: .claude/context/myteam/metrics-catalog.md
  query_patterns: .claude/context/myteam/query-patterns.md
  architecture: .claude/context/myteam/service-architecture.md
  incidents: .claude/context/myteam/incident-patterns.md

# Map PagerDuty alerts to services and common root causes
alerts:
  myservice-api-errors-high:
    services: [myservice-api]
    common_causes:
      - database_saturation
      - rate_limiting
      - deployment_issue
      - mesh_failure
    runbook_pattern: diagnosis/api-errors

  myservice-db-cpu-high:
    services: [myservice-api, myservice-worker, myservice-db]
    common_causes:
      - workload_spike
      - slow_query
      - missing_index
      - archival_load
    runbook_pattern: diagnosis/db-saturation

  myservice-worker-queue-backlog:
    services: [myservice-worker]
    common_causes:
      - worker_capacity
      - downstream_failure
      - poison_message
    runbook_pattern: diagnosis/queue-backlog

# Define diagnosis patterns (maps symptoms → templates)
runbook_patterns:
  diagnosis:
    api_errors:
      symptoms: [high_error_rate, slow_response, upstream_timeout]
      template: templates/runbook/diagnosis-api-errors.md
    db_saturation:
      symptoms: [high_cpu, slow_queries, connection_pool_exhaustion]
      template: templates/runbook/diagnosis-db-saturation.md
    queue_backlog:
      symptoms: [queue_depth_growing, processing_slow]
      template: templates/runbook/diagnosis-queue-backlog.md

  # Define remediation patterns (maps root causes → templates)
  remediation:
    rolling_restart:
      when: [memory_leak, stale_connections, goroutine_leak]
      template: templates/runbook/remediation-rolling-restart.md
    resource_scaling:
      when: [cpu_saturation, memory_pressure, workload_surge]
      template: templates/runbook/remediation-resource-scaling.md
    config_rollback:
      when: [deployment_issue, config_change_correlation]
      template: templates/runbook/remediation-config-rollback.md

# Environment priority (for batch processing order)
environments:
  HIGH:
    - prod
    - production
    - esvc
  MEDIUM:
    - staging
    - preprod
    - uat
  LOW:
    - dev
    - test
    - local
```

---

## Phase 3: Test Configuration

### Validate Config

```bash
# Run validation script
./scripts/validate-config.sh .claude/config/myteam-config.yaml

# Expected output:
✅ Team config valid
✅ Knowledge files found
✅ Services defined: 3
✅ Alerts mapped: 3
✅ Patterns defined: 6
```

### Test with One RCA

```bash
# Analyze single RCA
/rca-analyzer https://docs.google.com/.../your-rca --config=.claude/config/myteam-config.yaml
```

**Review output**:
- Does it correctly identify your service?
- Does it extract timeline (TTD/TTX/TTR)?
- Does it match root cause to your patterns?

---

## Phase 4: Customize Runbook Templates (Optional)

**When to customize**:
- Default templates don't match your workflow
- You have team-specific commands/tools
- You want different structure

### Create Custom Template

1. **Copy default template**:
```bash
cp rca-toolkit/templates/runbook/diagnosis-template.md \
   your-repo/templates/runbook/diagnosis-mypattern.md
```

2. **Edit placeholders**:
```markdown
# Runbook: Diagnosis - Database Saturation

**Pattern ID**: `myservice-db-saturation`

## Diagnosis Steps

### Step 1: Check DB CPU
**Command**:
```
# YOUR team-specific command
kubectl exec -it myservice-db-primary -- psql -c "SELECT * FROM pg_stat_activity WHERE state = 'active';"
```
```

3. **Reference in config**:
```yaml
runbook_patterns:
  diagnosis:
    db_saturation:
      symptoms: [high_cpu, slow_queries]
      template: your-repo/templates/runbook/diagnosis-mypattern.md
```

---

## Phase 5: Run Batch Analysis

**After 2+ RCAs are documented**:

```bash
# Batch analyze
/rca-analyzer --batch \
  https://docs.google.com/.../rca1, \
  https://docs.google.com/.../rca2, \
  https://docs.google.com/.../rca3 \
  --generate-runbook \
  --config=.claude/config/myteam-config.yaml
```

**Expected output**:
```
✅ Batch Analysis Complete
├─ RCAs Analyzed: 3
├─ Patterns Identified: 2
│  ├─ myservice-api-db_saturation (2 occurrences)
│  └─ myservice-worker-queue_backlog (1 occurrence)
├─ Runbooks Generated: 1
│  └─ runbooks/diagnosis/myservice-api-db_saturation.md
└─ Estimated ROI: $15K-30K/year
```

---

## Phase 6: Integrate with Team Workflow

### Add to Incident Response

**Update incident runbooks** to reference RCA toolkit:

```markdown
## Post-Incident

1. Write RCA in Google Doc (use team template)
2. Run RCA analysis:
   `/rca-analyzer https://docs.google.com/.../your-rca`
3. Review generated analysis for gaps
4. Update knowledge files if new patterns found
```

### Automate (Optional)

**Trigger analysis automatically** when RCA doc is completed:

```bash
# Google Apps Script webhook
# Triggered when RCA doc moves to "Complete" folder
# POST to Claude Code API with doc URL
```

---

## Maintenance

### When to Update Config

**Add new service**:
```yaml
services:
  - name: myservice-newapi
    port: 9090
```

**Add new alert**:
```yaml
alerts:
  myservice-newapi-errors:
    services: [myservice-newapi]
    common_causes: [deployment_issue]
```

**Add new pattern**:
```yaml
runbook_patterns:
  diagnosis:
    newpattern:
      symptoms: [new_symptom]
```

### When to Update Knowledge Files

**New metrics added** → Update `metrics-catalog.md`  
**New query patterns** → Update `query-patterns.md`  
**Architecture changes** → Update `service-architecture.md`

---

## Examples from Other Teams

### Temporal Team Config

**See**: `.claude/config/temporal-config.yaml` (in your private workspace)

**Highlights**:
- 4 services (frontend/history/matching/worker)
- Complex alert → service → root cause mapping
- Pattern → template mapping for 6 patterns

### Kafka Team Config (Hypothetical)

```yaml
team:
  name: Kafka Platform
  service_prefix: kafka

services:
  - name: kafka-broker
  - name: kafka-zookeeper
  - name: kafka-schema-registry

alerts:
  kafka-broker-underreplicated:
    services: [kafka-broker]
    common_causes: [broker_failure, network_partition]

runbook_patterns:
  diagnosis:
    underreplication:
      symptoms: [underreplicated_partitions, isr_shrink]
```

---

## Getting Help

**Issues**:
- Config validation fails → Check YAML syntax
- Patterns not detected → Need ≥2 RCAs with same symptom/root cause
- Runbooks incomplete → Customize templates for your workflow

**Questions**:
- Slack: #rca-automation (internal)
- GitHub Issues: git.soma.salesforce.com/orcaas/rca-toolkit/issues

---

**Next**: Run your first batch analysis with `--generate-runbook`
