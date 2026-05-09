# Agent Architecture

**Parallel Processing for Efficient RCA Analysis**

---

## Overview

The framework uses Claude's Agent tool for parallel processing of multiple RCAs. When analyzing incidents in batch mode, each RCA is processed by an independent subagent, enabling concurrent analysis and faster results.

---

## Why Agents?

### Single RCA Analysis
For a single RCA, direct processing is efficient:
- Read Google Doc
- Extract timeline and gaps
- Generate analysis
- Write output file

**Time**: ~2-5 minutes per RCA

### Batch Analysis (Without Agents)
Processing 6 RCAs sequentially:
- RCA #1: 3 minutes
- RCA #2: 3 minutes  
- RCA #3: 3 minutes
- RCA #4: 3 minutes
- RCA #5: 3 minutes
- RCA #6: 3 minutes

**Total**: ~18 minutes (sequential)

### Batch Analysis (With Agents)
Processing 6 RCAs in parallel:
- All 6 RCAs spawn simultaneously
- Each runs in independent agent
- Results collected when all complete

**Total**: ~4-6 minutes (parallel)

**Speedup**: 3-4x faster for batch analysis

---

## Agent Workflow

### Batch Mode Invocation

```bash
/rca-analyzer --batch <url1>, <url2>, <url3>, <url4>, <url5>, <url6>
```

### Framework Orchestration

```
Main Skill
    ↓
Parse URLs (6 RCAs)
    ↓
Spawn 6 Agents in Parallel
    ├─ Agent 1: Extract RCA #1
    ├─ Agent 2: Extract RCA #2
    ├─ Agent 3: Extract RCA #3
    ├─ Agent 4: Extract RCA #4
    ├─ Agent 5: Extract RCA #5
    └─ Agent 6: Extract RCA #6
    ↓
Collect Results
    ↓
Pattern Detection (≥2 similar)
    ↓
Generate Synthesis
    ↓
Generate Runbooks (if patterns found)
    ↓
Generate Operational Findings (if --generate-summary)
```

---

## Agent Configuration

### Subagent Prompt Template

Each agent receives:

**Task**:
```
Extract incident data from this RCA document and identify operational gaps.

RCA URL: [url]
Team Config: [config-data]

Extract:
1. Timeline (incident start, detection, diagnosis, resolution)
2. Services affected
3. Root cause and symptoms
4. Observable signals (logs, metrics, dashboards)
5. What worked well (positive signals, effective tools)
6. Gaps (detection, diagnosis, remediation)

Output format: Structured markdown following output-format-spec.md
```

**Context Provided**:
- Team configuration (services, metrics, query patterns)
- Output format specification
- Team-specific alerting context

**Independent Execution**:
- Each agent reads its own Google Doc (via Google Workspace MCP)
- Each agent generates its own analysis file
- No inter-agent communication needed (fully parallel)

---

## Implementation Details

### Spawning Agents

```python
# Pseudocode - actual implementation in skill
for rca_url in batch_urls:
    agent = spawn_agent(
        description=f"Extract RCA from {rca_url}",
        prompt=build_extraction_prompt(rca_url, team_config),
        isolation="worktree"  # Optional: isolated git worktree
    )
    agents.append(agent)

# Wait for all agents to complete
results = await_all(agents)
```

### Collecting Results

After all agents complete:
1. Read each generated analysis file
2. Extract patterns (service + symptom + root cause)
3. Count occurrences (≥2 = recurring pattern)
4. Generate batch synthesis
5. Generate runbooks for patterns
6. Generate operational findings (if requested)

---

## Benefits of Agent Architecture

### 1. Speed
**Parallel processing**: 6 RCAs in ~5 minutes vs ~18 minutes sequential

### 2. Isolation
**Independent execution**: Failure in one agent doesn't block others

### 3. Scalability
**Works with any batch size**: 2 RCAs or 20 RCAs (up to system limits)

### 4. Resource Efficiency
**Concurrent API calls**: Multiple Google Docs read simultaneously

### 5. Clean Code
**Separation of concerns**: Main skill orchestrates, agents execute

---

## Error Handling

### Agent Failure Scenarios

**Scenario 1: Google Doc access denied**
- Agent reports: "Cannot read document (permissions)"
- Framework: Logs error, continues with remaining RCAs
- Result: Partial batch analysis (5/6 RCAs processed)

**Scenario 2: Malformed RCA (missing timeline)**
- Agent reports: "Timeline not found in document"
- Framework: Marks as incomplete, continues
- Result: Analysis marked "incomplete - timeline missing"

**Scenario 3: Network timeout**
- Agent: Retries (configurable)
- Framework: Waits up to timeout limit
- Result: Either succeeds on retry or marks as failed

### Graceful Degradation

```
Batch of 6 RCAs:
- RCA #1: ✅ Success
- RCA #2: ✅ Success
- RCA #3: ❌ Access denied
- RCA #4: ✅ Success
- RCA #5: ⚠️  Incomplete (missing timeline)
- RCA #6: ✅ Success

Result: Generate synthesis from 4 complete RCAs
        Log warnings for #3 (access) and #5 (incomplete)
```

---

## Performance Characteristics

### Batch Size vs Time

| Batch Size | Sequential | Parallel (Agents) | Speedup |
|------------|------------|-------------------|---------|
| 2 RCAs     | ~6 min     | ~4 min            | 1.5x    |
| 4 RCAs     | ~12 min    | ~5 min            | 2.4x    |
| 6 RCAs     | ~18 min    | ~6 min            | 3.0x    |
| 10 RCAs    | ~30 min    | ~8 min            | 3.75x   |

**Note**: Actual times depend on RCA document size and network latency

### Optimal Batch Size

**Recommended**: 4-8 RCAs per batch
- Small enough for manageable review
- Large enough to benefit from parallelization
- Pattern detection works well (≥2 occurrences likely)

**Too small** (<3 RCAs): Limited pattern detection
**Too large** (>12 RCAs): Overwhelming review, diminishing speedup returns

---

## Configuration

### Agent Timeout

Default: 5 minutes per agent

```yaml
agent:
  timeout: 300  # seconds
  retry: 2      # retry attempts
```

### Parallel Limit

Default: System-dependent (typically 6-10 concurrent agents)

```yaml
agent:
  max_parallel: 8  # max concurrent agents
```

### Isolation Mode

**Default**: No isolation (agents share context)
**Optional**: Worktree isolation (each agent in separate git worktree)

```python
agent = spawn_agent(
    isolation="worktree"  # Optional: isolated git environment
)
```

---

## Monitoring Agent Progress

### Real-Time Status

During batch processing:
```
Processing 6 RCAs in parallel...
├─ Agent 1 (RCA #1): Reading Google Doc...
├─ Agent 2 (RCA #2): Extracting timeline...
├─ Agent 3 (RCA #3): ✅ Complete
├─ Agent 4 (RCA #4): Identifying gaps...
├─ Agent 5 (RCA #5): Reading Google Doc...
└─ Agent 6 (RCA #6): ✅ Complete

Waiting for remaining agents...
```

### Completion Summary

```
✅ Batch Analysis Complete
├─ Processed: 6 RCAs
├─ Success: 5 RCAs
├─ Incomplete: 1 RCA (missing timeline)
├─ Patterns: 2 recurring patterns found
├─ Runbooks: 2 runbooks generated
├─ Time: 5 minutes 23 seconds
└─ Files:
    ├─ rca-analysis-1.md
    ├─ rca-analysis-2.md
    ├─ rca-analysis-3.md
    ├─ rca-analysis-4.md (incomplete)
    ├─ rca-analysis-5.md
    ├─ rca-analysis-6.md
    ├─ batch-synthesis-6-rcas.md
    └─ runbooks/ (2 generated)
```

---

## Testing Agent Behavior

### Test Single RCA First

```bash
/rca-analyzer https://docs.google.com/document/d/YOUR_RCA
```

**Verify**:
- Google Doc access works
- Timeline extraction accurate
- Gaps identified correctly
- Output format matches spec

### Test Small Batch

```bash
/rca-analyzer --batch <url1>, <url2>
```

**Verify**:
- Both agents spawn successfully
- Parallel processing works
- Results collected correctly
- No inter-agent interference

### Test Full Batch

```bash
/rca-analyzer --batch <url1>, <url2>, <url3>, <url4>, <url5>, <url6>
```

**Verify**:
- Pattern detection works (≥2 similar)
- Synthesis generated correctly
- Runbooks generated if patterns found
- Performance improved vs sequential

---

## Troubleshooting

### Problem: Agents not spawning in parallel

**Symptom**: RCAs processed sequentially (slow)

**Cause**: Agent tool not available or parallelization disabled

**Fix**: Verify Claude Code supports Agent tool, check configuration

---

### Problem: Agent timeouts

**Symptom**: "Agent timed out after 5 minutes"

**Cause**: Large RCA document or slow network

**Fix**: Increase timeout in configuration or process RCA separately

---

### Problem: Incomplete results

**Symptom**: Some RCAs marked "incomplete"

**Cause**: Missing timeline data in RCA document

**Fix**: Ensure RCA documents follow standard format (timeline section required)

---

## Future Improvements

### Potential Enhancements

1. **Dynamic parallelization**: Adjust based on system load
2. **Progress streaming**: Real-time updates from each agent
3. **Incremental results**: Show completed RCAs immediately
4. **Agent pooling**: Reuse agents for subsequent batches
5. **Priority queuing**: Process high-severity RCAs first

---

## Summary

**Agent architecture enables**:
- ✅ Parallel processing (3-4x faster)
- ✅ Independent execution (isolation)
- ✅ Scalable batch analysis
- ✅ Graceful error handling
- ✅ Clean separation of concerns

**Framework orchestrates**:
- Spawning agents for each RCA
- Collecting results
- Pattern detection across RCAs
- Synthesis and runbook generation

**Result**: Efficient, scalable incident analysis framework

---

**See also**:
- `skills/rca-analyzer/skill.md` - Complete skill specification
- `docs/output-format-spec.md` - Output format for agents
- `examples/temporal/` - Real batch analysis example (6 RCAs)
