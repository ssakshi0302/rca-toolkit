# Post-Incident Analysis Framework - Executive Navigation Guide

**Purpose**: Quick links to key documents for different audiences

---

## For Executives & Leadership

### 1. Executive Action Plan (1-page read) ⭐ **PRIMARY DOCUMENT**
**File**: `EXECUTIVE-SUMMARY.md`

**This is the recommended document for leadership review, Slack sharing, and architectural discussions.**

**What it contains**:
- Problem (clear, concise)
- 6 incidents analyzed (table format)
- Framework observations (10 missing alerts, 2 recurring patterns)
- Engineering recommendations (with implementation timeline)
- Time savings potential: ~1,240 hours/year estimated
- Phased roadmap (immediate/short/long)

**Time**: 3-5 minutes

---

### 2. Detailed Analysis (4-page read)
**File**: `TEMPORAL-RESULTS-EXECUTIVE-SUMMARY.md`

**What it contains**:
- All findings from 1-page summary
- Detailed incident timelines
- Extended analysis and context
- Automation capabilities required

**Time**: 10-15 minutes (use if 1-page raises questions)

---

### 2. Purpose & Scope (detailed context)
**File**: `PURPOSE-AND-SCOPE.md`

**What it contains**:
- What framework does (build operational maturity through incident analysis)
- Detection, Diagnosis, Remediation focus
- Use cases (post-incident, quarterly planning, runbook library)
- Success metrics (Temporal team results)
- Design principles (operational maturity first, signal-aware, data-driven)

**Time**: 10-15 minutes

---

## For Technical Leadership & Engineering Managers

### 3. Complete Example Walkthrough
**File**: `examples/temporal/EXAMPLE-OVERVIEW.md`

**What it contains**:
- 6 real incidents analyzed (with timelines)
- Input (Google Docs) → Process (skill) → Output (runbooks + report)
- Pattern detection (2 recurring patterns from 6 RCAs)
- Runbook examples (capacity exhaustion, WASM panic)
- Key insights (detection gaps, diagnosis patterns, safe automation)

**Time**: 15-20 minutes

---

### 4. Framework Overview (main README)
**File**: `README.md`

**What it contains**:
- What framework does (operational maturity through incident analysis)
- Vision and key learnings from Temporal pilot
- Phased maturity model (observational → guided → automated)
- Quick start (3 steps, 5-10 minutes)
- Example outputs (what teams get)

**Time**: 10 minutes

---

## For Teams Adopting Framework

### 5. Quick Start Guide
**File**: `docs/quick-start.md`

**What it contains**:
- 5-10 minute setup
- Prerequisites (Claude Code, MCP access)
- Create team config (3 min)
- Analyze first RCA (2 min)
- Next steps (batch analysis, runbook generation)

**Time**: 10-15 minutes (hands-on)

---

### 6. Team Onboarding (comprehensive)
**File**: `docs/team-onboarding.md`

**What it contains**:
- Gather knowledge files (metrics, queries, architecture)
- Create team config (full example)
- Customize templates (optional)
- Batch analysis workflow
- Integrate with incident response

**Time**: 1-2 hours (first-time setup)

---

## For Demonstrations

### Best Path for Different Audiences

**For Executives (10 minutes)**:
1. Start with `EXECUTIVE-SUMMARY.md` (1-page action plan)
2. Highlight: 1,240 hours/year potential savings (71% reduction)
3. Show operational gaps: 10 missing alerts, 2 recurring patterns
4. Show 6 real incidents analyzed

**For EMs / Tech Leads (20 minutes)**:
1. Start with `examples/temporal/EXAMPLE-OVERVIEW.md`
2. Show complete workflow (input → output)
3. Show 2 generated runbooks (real patterns)
4. Show batch synthesis (aggregate metrics)
5. Discuss adoption for their team

**For Engineers / SREs (30 minutes)**:
1. Start with `README.md` (framework overview)
2. Walk through `docs/quick-start.md` (hands-on)
3. Show Temporal team config (how to adapt)
4. Show generated runbooks (actionable steps)
5. Discuss customization (templates, patterns)

---

## File Structure Summary

```
rca-toolkit-draft/
├── EXECUTIVE-NAVIGATION.md              # This file (navigation guide)
├── EXECUTIVE-SUMMARY.md                 # 1-page action plan ✅
├── TEMPORAL-RESULTS-EXECUTIVE-SUMMARY.md  # 4-page detailed analysis ✅
├── README.md                            # Framework overview
├── PURPOSE-AND-SCOPE.md                 # Detailed purpose
│
├── Examples
│   └── temporal/
│       ├── EXAMPLE-OVERVIEW.md          # Complete workflow ✅
│       ├── incident-automation-executive-report.md  # Same as root
│       ├── rca-analyses/                # 6 RCAs + synthesis
│       └── ... (knowledge files)
│
└── Documentation
    ├── docs/quick-start.md              # 5-10 min setup
    ├── docs/team-onboarding.md          # 1-2 hour guide
    └── docs/runbook-spec.md             # Full specification
```

---

## Key Messages by Audience

### For Executives
**Message**: "Build operational maturity before scaling automation - understand the signals first"

**Data points**:
- 1,240 hours/year potential savings (Temporal team, 6 incidents)
- 100% of incidents had observable signals - delays caused by missing alerting logic
- Detection delay is the largest bottleneck (75% had TTD >10 hours)
- 10 specific missing alerts identified
- 2 patterns identified from just 6 RCAs
- Team-agnostic (works for any distributed system)

---

### For Engineering Managers
**Message**: "Framework identifies operational maturity gaps from existing RCAs"

**Data points**:
- 6 RCAs analyzed → 10 missing alerts found
- Pattern detection (≥2 similar incidents)
- Deterministic runbooks (step-by-step guidance)
- Post-incident analysis (not live automation)
- Phased approach: Observational insights → Guided assistance → Automated reasoning
- 1-2 hour setup for teams

---

### For Engineers / SREs
**Message**: "Turn RCAs into actionable runbooks with quantified time savings"

**Data points**:
- Batch analysis (parallel processing)
- Config-driven (your metrics, your queries)
- Generic templates (adapt for your services)
- Real examples (Temporal: 89+ metrics, 6 RCAs)
- Runbook output (diagnosis + remediation with rollback)

---

## Usage Recommendations

### Before Team Meeting
**Send ahead**:
1. `EXECUTIVE-SUMMARY.md` (3-min read - 1-page action plan)
2. Context: "Analyzed 6 Temporal incidents, found operational maturity gaps with 1,240 hours/year potential savings"

### During Demo (20-30 min)
**Agenda**:
1. Problem (5 min): Incidents recur, operational gaps not systematically identified
2. Solution (5 min): Framework analyzes RCAs, identifies maturity gaps
3. Temporal Results (10 min): Show `EXAMPLE-OVERVIEW.md`, runbooks, key learnings
4. Next Steps (5 min): Phased approach, pilot with 1-2 teams
5. Q&A (5 min)

### After Meeting
**Share**:
1. Repository link (once created)
2. Quick start guide for interested teams
3. Slack channel for questions (#rca-automation)

---

## Common Questions

**Q: Is this live automation?**
A: No. Post-incident analysis only. Identifies opportunities, doesn't execute.

**Q: How long does setup take?**
A: 5-10 minutes for quick start, 1-2 hours for comprehensive setup.

**Q: Do we need 89 metrics like Temporal?**
A: No. Start with 10-20 key metrics. Add more as you analyze RCAs.

**Q: What if we don't use Argus/Splunk?**
A: Framework is tool-agnostic. Use Prometheus, Elasticsearch, whatever you have.

**Q: How many RCAs needed to see patterns?**
A: Temporal found 2 patterns from 6 RCAs. Start with 5-10 for meaningful analysis.

**Q: Is remediation automation safe?**
A: Framework identifies candidates with safety checks. Teams decide what to automate. Phase 1 focuses on detection and diagnosis only.

---

## Next Steps

### For Approval
1. Review `EXECUTIVE-SUMMARY.md` (1-page action plan)
2. Review `TEMPORAL-RESULTS-EXECUTIVE-SUMMARY.md` (detailed 4-page analysis)
3. Validate time savings methodology
4. Approve repository creation + pilot teams

### For Pilot
1. Identify 1-2 pilot teams (Kafka? Heroku?)
2. Onboard using `docs/quick-start.md`
3. Run against their RCAs (5-10 incidents)
4. Collect feedback (2-4 weeks)

### For Org Rollout
1. Announce in #platform-reliability
2. Share Temporal results + pilot results
3. Provide quick-start guide
4. Onboard interested teams (office hours, Slack channel)

---

**Repository Status**: Ready for creation  
**Recommendation**: Create `git.soma.salesforce.com/orcaas/post-incident-analysis-framework`  
**Access**: Private, org-level (all Salesforce teams)
