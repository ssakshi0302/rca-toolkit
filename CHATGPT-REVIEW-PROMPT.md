# ChatGPT Review Prompt - RCA Toolkit for Executive Presentation

---

## Context: What This Is

I've built an **RCA (Root Cause Analysis) Toolkit** that analyzes incident post-mortems to identify automation opportunities and reduce incident response time. The toolkit has been tested on 6 real Temporal production incidents and has generated:

1. **Individual RCA analyses** (6 incidents analyzed)
2. **Executive summary** (1-2 page action plan)
3. **Runbooks** (2 recurring patterns identified)
4. **ROI calculation** ($264K-529K/year savings potential)

**Repository**: https://github.com/ssakshi0302/rca-toolkit

---

## Purpose of This Exercise

### Problem We're Solving
- **Incidents recur** without detection - same patterns repeat (e.g., mesh routing failure happened twice in 4 months)
- **Manual incident response is slow** - Average 16.5h to detect, 33.7h to resolve
- **No quantified ROI** for automation investments - teams don't know which automations to prioritize
- **Knowledge loss** - RCA insights stay in Google Docs, not actionable

### Our Goal
Build a **data-driven approach** to:
1. **Identify automation opportunities** from existing incident RCAs
2. **Quantify time savings** (TTD/TTX/TTR reduction)
3. **Detect recurring patterns** (≥2 similar incidents = runbook candidate)
4. **Calculate ROI** (manual effort vs automated effort × oncall rate)
5. **Generate actionable recommendations** (specific alerts with thresholds, runbooks, automation candidates)

### Why This is Helpful

**For Engineering Teams**:
- Converts 6 RCAs (spread across Google Docs) into structured data
- Identifies **10 missing alerts** with specific thresholds (e.g., "DB CPU >80% for 10+ min")
- Generates **2 deterministic runbooks** for recurring patterns
- Shows **what worked well** (not just gaps) - positive signals, effective tools

**For Leadership**:
- Quantified ROI: **$264K-529K/year** savings potential
- Clear action plan: immediate (30 days), short-term (60-90 days), long-term (6-12 months)
- Time reduction: **99.5% TTD, 70-90% TTX, 94% TTR**
- Data-driven decisions: all findings reference specific RCAs with links

**For Organization**:
- Team-agnostic approach (any distributed system can use it)
- Post-incident analysis (safe, not live automation)
- Pattern detection prevents recurrence
- Historical RCA corpus becomes searchable/actionable

---

## What I Need from You

I'm about to present this to **executives and engineering leadership** at Salesforce. Before I do, I need an **external review** to ensure:

1. **Clarity**: Is the value proposition clear?
2. **Credibility**: Are the ROI calculations sound and defensible?
3. **Completeness**: Are there obvious gaps or missing considerations?
4. **Actionability**: Are the recommendations specific and implementable?
5. **Executive readiness**: Is the executive summary scannable and compelling?

---

## Documents to Review

### Primary Document (START HERE)
**Executive Summary**: [rca-toolkit-draft/EXECUTIVE-SUMMARY.md](https://github.com/ssakshi0302/rca-toolkit/blob/main/rca-toolkit-draft/EXECUTIVE-SUMMARY.md)
- 1-2 pages, action-oriented
- Contains: Problem, findings, ROI, immediate actions, recommendations
- **This is what execs will see first**

### Supporting Documents (if needed for context)

**Complete Workflow Example**:
- [examples/temporal/EXAMPLE-OVERVIEW.md](https://github.com/ssakshi0302/rca-toolkit/blob/main/rca-toolkit-draft/examples/temporal/EXAMPLE-OVERVIEW.md)
- Shows input (6 RCA Google Docs) → output (analyses, runbooks, report)

**Individual RCA Analyses** (sample one):
- [rca-analysis-1.md](https://github.com/ssakshi0302/rca-toolkit/blob/main/rca-toolkit-draft/examples/temporal/rca-analyses/rca-analysis-1.md) - DB CPU Saturation incident
- Shows timeline, gaps, automation opportunities, ROI

**Batch Synthesis**:
- [batch-synthesis-6-rcas.md](https://github.com/ssakshi0302/rca-toolkit/blob/main/rca-toolkit-draft/examples/temporal/rca-analyses/batch-synthesis-6-rcas.md)
- Aggregate analysis across all 6 RCAs

**Runbook Example**:
- [runbook-capacity-exhaustion.md](https://github.com/ssakshi0302/rca-toolkit/blob/main/rca-toolkit-draft/examples/runbook-capacity-exhaustion.md)
- Shows deterministic diagnosis steps for recurring pattern

---

## Specific Questions for Review

### 1. Executive Summary Quality
**Question**: Does the executive summary effectively communicate value in 3-5 minutes?

**Look for**:
- Is the 3-line summary (bottleneck, impact, ROI) compelling?
- Are the critical findings clear and data-backed?
- Is the incident summary table easy to scan?
- Are immediate actions specific enough? (e.g., "DB CPU >80% for 10+ min")
- Are recommendations actionable with clear approval items?

**Red flags**:
- Too much text, not scannable
- Vague recommendations ("consider automation")
- Missing context on RCA references
- Unclear ROI calculation

---

### 2. ROI Calculation Soundness
**Question**: Is the ROI calculation methodology defensible?

**Our approach**:
```
Manual effort per incident:
- Detection: 16.5h avg (range: 29 min - 3 days)
- Diagnosis: 13.3h avg
- Resolution: 33.7h avg
- Total: 63.5h/incident

With automation:
- Detection: 2h (with 10 missing alerts)
- Diagnosis: 4h (with runbooks)
- Resolution: 20h (with guided remediation)
- Total: 26h/incident

Savings: 37.5h/incident × 25 incidents/year × $200-400/hour = $264K-529K/year
```

**Look for**:
- Are assumptions reasonable? (25 incidents/year, $200-400/hour oncall rate)
- Are time reductions realistic? (16.5h → 2h detection)
- Is the calculation methodology clear?
- Are there missing costs (implementation, maintenance)?

---

### 3. Gap Analysis Completeness
**Question**: Are we missing obvious gaps or considerations?

**What we identified**:
- **Detection gaps**: 10 missing alerts (DB CPU, memory pressure, OOMKilled, PassthroughCluster, etc.)
- **Diagnosis gaps**: Manual correlation, noisy logs, missing runbooks
- **Remediation gaps**: Manual processes, approval delays, no pre-flight validation

**Check**:
- Are there other common incident response bottlenecks?
- Should we address alert fatigue? (20+ alerts in 3-day sample)
- Should we mention false positive risks?
- Are safety considerations for automation adequate?

---

### 4. Data-Driven vs. Subjective
**Question**: Are findings factual or do they sound like opinions/marketing?

**Our style guidelines**:
- ✅ Data-driven: "RCA #1: 20h TTD (DB CPU 100%, no alert)"
- ✅ Specific: "DB CPU >80% for 10+ min | 20h → 2 min TTD"
- ❌ Marketing: "game-changer", "critical", "exciting"
- ❌ Vague: "Add availability alert" (need thresholds)

**Check**:
- Are all findings backed by specific RCA data?
- Are all metrics quantified?
- Is language factual and professional?

---

### 5. Executive Concerns Addressed
**Question**: Will executives ask questions we haven't addressed?

**Common executive concerns**:
- **Cost**: Implementation cost vs. savings (we show ROI, but not implementation cost)
- **Risk**: What if automation fails? (we mention safety checks, rollback plans)
- **Timeline**: How long to implement? (we show immediate/short/long-term)
- **Proof**: How do we know this works? (we show 6 real incidents analyzed)
- **Scale**: Does this work for other teams? (we mention team-agnostic approach)

**Check**:
- Should we add implementation cost estimate?
- Should we add pilot success criteria?
- Should we mention other teams who could benefit?
- Should we add risk mitigation strategies?

---

### 6. Presentation Flow
**Question**: Does the executive summary tell a clear story?

**Current flow**:
1. Executive Summary (3 lines: bottleneck, impact, ROI)
2. Data Overview (6 RCAs, averages, ranges)
3. Critical Findings (4 key issues)
4. Incident Summary (table with all RCAs)
5. Immediate Actions (alerts, runbooks, AI correlation)
6. Short/Long-Term Actions
7. ROI Estimate (table)
8. Recommendations (3-4 approvals)

**Check**:
- Does the flow make sense?
- Should we start with ROI instead of problem?
- Is the structure logical for decision-making?
- Are there too many sections? (should we consolidate?)

---

### 7. Missing Evidence or Context
**Question**: What additional data would strengthen the case?

**What we have**:
- 6 RCA analyses (prod incidents)
- Timeline data (TTD/TTX/TTR)
- Missing alerts identified (10 specific)
- Recurring patterns detected (2 out of 6)
- ROI calculation

**What we might be missing**:
- Customer impact data (support tickets, SLAs)
- PagerDuty alert volume over time
- Comparison to industry benchmarks
- Success stories from other teams
- Platform comparison data (Matrix vs AI Exchange vs build-custom)

**Check**:
- Would executives want to see customer impact?
- Should we include platform evaluation results?
- Should we mention prior art or similar initiatives?

---

## Your Task

Please review the **Executive Summary** (primary document) and provide:

### 1. Overall Assessment
- **Clarity**: Is the value proposition clear? (1-5 rating + explanation)
- **Credibility**: Are the findings/ROI believable? (1-5 rating + explanation)
- **Completeness**: Are there obvious gaps? (1-5 rating + list gaps)
- **Actionability**: Are recommendations specific? (1-5 rating + examples)

### 2. Specific Suggestions

**For Executive Summary**:
- What's working well? (keep doing)
- What's confusing? (clarify or remove)
- What's missing? (add before presenting)
- What would you change? (structure, content, emphasis)

**For ROI Calculation**:
- Are assumptions reasonable?
- Are time reductions realistic?
- What questions would a CFO ask?
- How would you strengthen the ROI case?

**For Recommendations Section**:
- Are the 3-4 approval items clear?
- Would you approve based on this? Why/why not?
- What additional info would you need to approve?

### 3. Red Flags
- **Credibility risks**: Claims that sound exaggerated or unsubstantiated
- **Missing considerations**: Costs, risks, or dependencies not addressed
- **Presentation issues**: Structure, flow, or formatting problems
- **Decision blockers**: Reasons an exec might say "no" or "not yet"

### 4. Quick Wins
- **Low-hanging fruit**: Easy changes that improve impact
- **Reordering**: Should we lead with different data?
- **Emphasis**: What deserves more/less attention?
- **Formatting**: Tables, bullets, headers that need adjustment

---

## Format Your Response

Please structure your review as:

```markdown
## Overall Assessment
- Clarity: [1-5] - [explanation]
- Credibility: [1-5] - [explanation]
- Completeness: [1-5] - [explanation]
- Actionability: [1-5] - [explanation]

## What's Working Well
- [bullet list of 3-5 strengths]

## What Needs Improvement
- [bullet list of 3-5 issues with specific suggestions]

## ROI Calculation Review
- Assumptions: [reasonable/questionable - explanation]
- Time reductions: [realistic/aggressive - explanation]
- Missing costs: [list any]
- CFO questions: [list 2-3 likely questions]

## Red Flags (if any)
- [bullet list of credibility risks or decision blockers]

## Quick Wins (1-2 hour fixes)
- [bullet list of easy improvements]

## Before You Present
- [checklist of 3-5 must-do items]
```

---

## Additional Context

**My role**: Staff Engineer at Salesforce, OrcaaS team (Temporal service owners)

**Audience**: 
- Engineering VPs and Directors
- Platform Engineering leadership
- Potentially: CTO office (if this goes well)

**Goal**: Get approval to:
1. Implement 10 missing alerts (detection)
2. Create 4 runbooks (diagnosis)
3. Pilot 3 automation candidates (remediation)
4. Expand toolkit to other teams (Kafka, Heroku)

**Timeline**: Want to present within 1-2 weeks

**Success criteria**: 
- Approval to implement immediate actions (30 days)
- Budget/headcount for pilot (if needed)
- Buy-in from 1-2 other teams to adopt toolkit

---

## Thank You

Your external perspective will help ensure this presentation is credible, compelling, and addresses executive concerns. I've been deep in the details—I need someone to tell me what I'm missing or what needs clarification.

Please be direct and critical. I'd rather find issues now than in the exec review meeting.
