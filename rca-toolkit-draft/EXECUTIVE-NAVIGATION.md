# RCA Toolkit - Executive Navigation Guide

**Purpose**: Quick links to key documents for different audiences

---

## For Executives & Leadership

### 1. Executive Summary (1-page read) ⭐ START HERE
**File**: `EXECUTIVE-SUMMARY.md`

**What it contains**:
- Problem (clear, concise)
- 6 incidents analyzed (table format)
- 10 missing alerts (specific: DB CPU, memory pressure, OOMKilled, etc.)
- 2 patterns identified (capacity exhaustion, WASM panic)
- Time reduction: 99.5% TTD, 70-90% TTX, 94% TTR
- ROI: **$264K-529K/year**
- Automation roadmap (immediate/short/long)
- Recommendation (approve what)

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
- What toolkit does (identify automation opportunities)
- TTD/TTX/TTR focus (reduce time to detect, diagnose, remediate)
- Use cases (post-incident, quarterly planning, runbook library)
- Success metrics (Temporal team results)
- Design principles (data-driven, team-agnostic, safety-first)

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

### 4. Toolkit Overview (main README)
**File**: `README.md`

**What it contains**:
- What toolkit does (features, examples, use cases)
- Quick start (3 steps, 5-10 minutes)
- Example outputs (what teams get)
- ROI examples (Temporal: $264K-529K/year)

**Time**: 10 minutes

---

## For Teams Adopting Toolkit

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
1. Start with `TEMPORAL-RESULTS-EXECUTIVE-SUMMARY.md`
2. Highlight ROI: **$264K-529K/year**
3. Show time reduction: 99.5% TTD, 94% TTR
4. Show 6 real incidents analyzed

**For EMs / Tech Leads (20 minutes)**:
1. Start with `examples/temporal/EXAMPLE-OVERVIEW.md`
2. Show complete workflow (input → output)
3. Show 2 generated runbooks (real patterns)
4. Show batch synthesis (aggregate metrics)
5. Discuss adoption for their team

**For Engineers / SREs (30 minutes)**:
1. Start with `README.md` (toolkit overview)
2. Walk through `docs/quick-start.md` (hands-on)
3. Show Temporal team config (how to adapt)
4. Show generated runbooks (actionable steps)
5. Discuss customization (templates, patterns)

---

## File Structure Summary

```
rca-toolkit-draft/
├── EXECUTIVE-NAVIGATION.md              # This file (navigation guide)
├── TEMPORAL-RESULTS-EXECUTIVE-SUMMARY.md  # 2-page exec summary ✅
├── README.md                            # Toolkit overview
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
**Message**: "Automation can reduce incident response time by 90%+ with quantified ROI"

**Data points**:
- $264K-529K/year savings (Temporal team, 6 incidents)
- 99.5% TTD reduction (16.5h → 5 min with 10 alerts)
- 94% TTR reduction (33.7h → 2h with automation)
- 2 patterns identified from just 6 RCAs
- Team-agnostic (works for any distributed system)

---

### For Engineering Managers
**Message**: "Toolkit identifies automation opportunities from existing RCAs"

**Data points**:
- 6 RCAs analyzed → 10 missing alerts found
- Pattern detection (≥2 similar incidents)
- Deterministic runbooks (step-by-step guidance)
- Post-incident analysis (safe, not live automation)
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
1. `TEMPORAL-RESULTS-EXECUTIVE-SUMMARY.md` (5-min read)
2. Context: "We analyzed 6 Temporal incidents, found $264K-529K/year automation ROI"

### During Demo (20-30 min)
**Agenda**:
1. Problem (5 min): Incidents recur, no pattern detection
2. Solution (5 min): Toolkit analyzes RCAs, identifies automation
3. Temporal Results (10 min): Show `EXAMPLE-OVERVIEW.md`, runbooks
4. Next Steps (5 min): Pilot with 1-2 teams, then org-wide
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
A: Toolkit is tool-agnostic. Use Prometheus, Elasticsearch, whatever you have.

**Q: How many RCAs needed to see patterns?**
A: Temporal found 2 patterns from 6 RCAs. Start with 5-10 for meaningful analysis.

**Q: Is remediation automation safe?**
A: Toolkit identifies candidates with safety checks. Teams decide what to automate.

---

## Next Steps

### For Approval
1. Review `TEMPORAL-RESULTS-EXECUTIVE-SUMMARY.md`
2. Validate ROI calculation methodology
3. Approve repository creation + pilot teams

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
**Recommendation**: Create `git.soma.salesforce.com/orcaas/rca-toolkit`  
**Access**: Private, org-level (all Salesforce teams)
