# Temporal Team Example

**Purpose**: Complete end-to-end example with real Temporal incidents

**What's included**:
- **Input**: 6 incident RCAs (Google Docs)
- **Knowledge**: Team config + metrics catalog + query patterns
- **Output**: Individual analyses + batch synthesis + runbooks + executive report

**See**: `EXAMPLE-OVERVIEW.md` for complete workflow walkthrough

---

## Quick Links

- **Complete Workflow**: `EXAMPLE-OVERVIEW.md` - Input → Process → Output
- **Knowledge Files**: This README - How to set up team config
- **RCA Analyses**: `rca-analyses/` - 6 real incidents analyzed
- **Generated Runbooks**: `runbook-*.md` - 2 patterns identified
- **Executive Report**: `incident-automation-executive-report.md` - ROI summary

**Use this as a template** when creating your own team config and knowledge files.

---

## What's Included

### 1. Team Config
**File**: `team-config.yaml`

**Shows**:
- Complete service list (frontend/history/matching/worker)
- Infrastructure mapping (RDS, OpenSearch, Istio)
- Knowledge file paths (metrics, queries, architecture)
- Alert → service → root cause mapping
- Runbook pattern definitions
- Environment priorities (HIGH/MEDIUM/LOW)

**How to adapt**: Copy structure, replace with your services/alerts

---

### 2. Metrics Catalog
**File**: `metrics-catalog.md`

**Size**: ~50KB (89+ metrics)

**Shows**:
- Metric types: Counter (C), Histogram (H), Gauge (G)
- What each metric measures
- How to query (Argus transforms)
- When to use (detection/diagnosis scenarios)

**Organized by**:
- Common metrics (all services)
- Frontend metrics
- History metrics
- Matching metrics
- Worker metrics

**How to adapt**: 
- List your services' metrics (check Prometheus/Argus exports)
- Document type (Counter/Histogram/Gauge)
- Add brief description (what it measures)

---

### 3. Argus Query Patterns
**File**: `argus-patterns.md`

**Size**: ~20KB

**Shows**:
- Scope discovery (how to find metric paths)
- Counter → RATE transform
- Histogram → DIVIDE_V (for averages)
- Gauge queries (direct)
- Service instance filtering
- Argus MVP link generation

**How to adapt**:
- Replace scope prefix (`temporal.*` → `yourservice.*`)
- Use same transform patterns (RATE, DIVIDE_V work for any metrics)
- Add your team's common investigation queries

---

### 4. Splunk Query Patterns
**File**: `splunk-patterns.md`

**Size**: ~20KB

**Shows**:
- Index selection (distapps for prod, preprod for dev)
- Field filtering (falcon_instance, k8s_pod_name)
- Error extraction patterns (rex for msg, grpc_code)
- Batched querying (1-hour chunks to avoid timeout)
- URL generation for clickable links

**How to adapt**:
- Replace index name if different
- Replace pod naming convention (`temporalXXX-*` → `yourserviceXXX-*`)
- Keep rex patterns (JSON log extraction is generic)
- Add your team's common error patterns

---

## Example Structure

```
examples/temporal/
├── README.md                  # This file
├── team-config.yaml           # Complete Temporal config
├── metrics-catalog.md         # 89+ metrics (all services)
├── argus-patterns.md          # Query patterns & transforms
└── splunk-patterns.md         # Log query patterns
```

**Total**: ~130KB (comprehensive example)

---

## How to Use This Example

### Step 1: Review Temporal Config
```bash
cat team-config.yaml
```

**Notice**:
- Service definitions (4 services)
- Knowledge file paths (relative to project root)
- Alert mappings (3 alerts with common causes)
- Runbook patterns (diagnosis + remediation)

---

### Step 2: Review Metrics Catalog
```bash
head -100 metrics-catalog.md
```

**Notice**:
- Metric format: `<name>` (Type) - Description
- Organized by service
- Argus transform examples
- When to use each metric

---

### Step 3: Review Query Patterns
```bash
# Argus patterns
grep "RATE" argus-patterns.md

# Splunk patterns
grep "rex field=_raw" splunk-patterns.md
```

**Notice**:
- Transform syntax (reusable)
- Field extraction patterns (JSON parsing)
- Batching strategy (avoid timeouts)

---

### Step 4: Adapt for Your Team

**Copy structure**:
```bash
mkdir -p your-team-repo/.claude/config
cp examples/temporal/team-config.yaml your-team-repo/.claude/config/myteam-config.yaml

mkdir -p your-team-repo/.claude/context/myteam
cp examples/temporal/metrics-catalog.md your-team-repo/.claude/context/myteam/
cp examples/temporal/argus-patterns.md your-team-repo/.claude/context/myteam/
cp examples/temporal/splunk-patterns.md your-team-repo/.claude/context/myteam/
```

**Customize**:
1. Edit `team-config.yaml` → your services, alerts
2. Edit `metrics-catalog.md` → your metrics
3. Edit `argus-patterns.md` → your scope prefix
4. Edit `splunk-patterns.md` → your index, pod naming

---

## Common Questions

**Q: Do I need all 89 metrics in my catalog?**  
A: No. Start with 10-20 key metrics (service health, errors, latency, resource usage). Add more as you analyze RCAs.

**Q: My team doesn't use Argus/Splunk. Can I use Prometheus/Elasticsearch?**  
A: Yes. Replace query patterns with your tools. Same concepts apply (counters, histograms, log extraction).

**Q: How detailed should my metrics catalog be?**  
A: Minimum: metric name + type + brief description. Ideal: + query example + when to use.

**Q: Should I include every alert in team config?**  
A: Start with top 5-10 recurring alerts. Add more as you analyze RCAs and identify patterns.

---

## Metrics Catalog Depth Comparison

**Minimal** (good for starting):
```markdown
# Metrics Catalog

## Service Health
- `api_request_total` (Counter) - Total requests
- `api_request_errors` (Counter) - Error count
- `api_request_duration` (Histogram) - Request latency
```

**Temporal Example** (comprehensive):
```markdown
# Metrics Catalog

## Service Availability & Errors

### service_errors (Counter)
**What**: Total errors returned by service
**Labels**: operation, namespace, cause
**Argus**: `RATE(...:service_errors{...}:sum:1m-sum,#1m#,#true#,#true#)`
**When to use**: Detection (spike = incident), diagnosis (breakdown by operation)
**Related**: service_requests (for error percentage)
```

**Your choice**: Start minimal, add detail as you analyze RCAs.

---

## Notes

- **This is a reference implementation** - don't copy blindly, adapt to your context
- **Temporal-specific**: Some patterns are Temporal-specific (namespace metrics, workflow lifecycle)
- **Generic patterns**: Query transforms, log extraction patterns are reusable
- **Size**: 130KB total (your catalog will likely be smaller)

---

**Next**: See `docs/team-onboarding.md` for step-by-step setup guide
