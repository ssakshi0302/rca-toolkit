# Temporal RCA Analyzer - Demo Guide

**Prepared for**: Executive/Stakeholder Demo  
**Date**: 2026-05-08  
**Duration**: 10-15 minutes

---

## Demo Overview

**Problem**: Temporal incidents take 26+ hours to detect, 10+ hours to diagnose, and require manual approval delays for resolution.

**Solution**: Automated RCA analysis system that identifies where we're lacking and quantifies automation opportunities.

**Business Impact**: $186K/year time savings (from 4 RCAs analyzed, scales with more data)

---

## Demo Flow (10 minutes)

### 1. The Problem (2 min)

**Show**: Current executive report
```bash
cat research/incident-automation-executive-report.md | head -50
```

**Highlight**:
- Average TTD: 26.4 hours (detection bottleneck)
- 100% had observable signals (not a visibility problem, alerting problem)
- ROI: 1,240 hours/year saved with automation

**Key message**: "We're bleeding time in detection, not resolution."

---

### 2. The System (3 min)

**Show**: File structure
```bash
tree -L 3 -I '.git|backups'
```

**Walk through**:
```
incident-triaging-evaluation/
├── .claude/skills/temporal-rca-analyzer.md   # ← The automation
├── research/
│   ├── incident-automation-executive-report.md  # ← Executive summary
│   └── past rca/
│       ├── rca-analysis-1.md (ESVC1 DB CPU)
│       ├── rca-analysis-2.md (Mesh routing)
│       ├── rca-analysis-3.md (Archival storm)
│       └── rca-analysis-4.md (Karpenter)
└── runbooks/
    ├── metrics-catalog.md                     # ← Knowledge base
    ├── diagnosis/                             # ← Reusable guides
    └── remediation/                           # ← Fix actions
```

**Key message**: "Automated RCA analysis → Reusable runbooks → Foundation for automation"

---

### 3. Live Demo: Single RCA Analysis (3 min)

**Option A: Pre-recorded (Safe)**
```bash
# Show existing analysis
cat "research/past rca/rca-analysis-2.md" | head -100
```

**Highlight sections**:
- Timeline: TTD 17min, but 10h investigation delay
- What delayed: PassthroughCluster metric not in standard dashboard
- Automation opportunity: Tier 1 skill (2 weeks), Tier 2 AI (3-6 months)
- What fixed it: Rolling restart (5 min execution, 12h approval wait)

**Key message**: "Specific, actionable insights from each RCA."

---

**Option B: Live Run (If confident)**
```bash
# Analyze new RCA live
/temporal-rca-analyzer <new-rca-google-doc-url>

# Wait ~2-3 minutes
# Show output file
```

**Risk**: Timing, network issues

---

### 4. Batch Processing Demo (2 min)

**Show**: How we scale
```bash
# Show batch input
cat rca-urls-example.txt

# Explain command (don't run live)
/temporal-rca-analyzer --batch rca-urls-example.txt --env=prod,esvc
```

**Explain**:
- Parallel processing: 10 RCAs in ~20 min (vs. 30 min sequential)
- Background execution: Engineers continue other work
- Auto-backup: No risk of losing current analysis
- Auto-synthesis: Executive report updated automatically

**Key message**: "Scales from 4 RCAs to 50+ RCAs with no manual effort."

---

### 5. PagerDuty Integration (Bonus - 2 min)

**Show**: Alert analysis feature
```bash
# Explain command (don't run live unless pre-fetched)
/temporal-rca-analyzer --pd-alerts --days=30 --channel=#temporal-notifications
```

**What it provides**:
- Alert frequency by service
- Noise analysis (false positives)
- Alert → incident correlation
- Top alert types

**Key message**: "Understand alert patterns, reduce noise."

---

### 6. Business Value (2 min)

**Show**: ROI calculation (from executive report)

**Current state (4 RCAs)**:
- Detection: 26.4h avg (50-95% reduction possible) → 400h/year saved
- Diagnosis: 13.3h avg (40-70% reduction) → 500h/year saved
- Resolution: Approval delays (50% reduction) → 340h/year saved
- **Total**: 1,240h/year = $186K (at $150/hour loaded cost)

**With 15 RCAs**: More accurate estimates, higher confidence

**Path to automation**:
1. **Tier 1 (2-4 weeks)**: Deploy skills/runbooks (40-60% faster diagnosis)
2. **Tier 2 (3-6 months)**: AI-assisted correlation (60-80% faster diagnosis)
3. **Tier 3 (6-12 months)**: Autonomous detection/remediation (80-95% faster)

**Key message**: "Quick wins in weeks, long-term automation in quarters."

---

## Demo Preparation Checklist

### Before Demo (Do Once)

- [ ] Verify file structure exists
```bash
tree -L 2 -I '.git|backups'
```

- [ ] Verify backups working
```bash
ls -lh backups/latest/
```

- [ ] Review executive report (know the numbers)
```bash
cat research/incident-automation-executive-report.md | less
```

- [ ] Review one sample RCA analysis
```bash
cat "research/past rca/rca-analysis-2.md" | less
```

- [ ] **Optional**: Pre-run PD alert analysis (so it's ready to show)
```bash
/temporal-rca-analyzer --pd-alerts --days=30
```

---

### During Demo

**Terminal setup**:
1. Increase font size (readability)
2. Clear screen before each command
3. Use `cat | head -N` to show snippets (not full files)

**Pacing**:
- Don't rush through terminal output
- Pause after showing data (let it sink in)
- Highlight specific lines (point or read aloud)

**Backup plan**:
- If live demo fails → show pre-generated files
- If timing tight → skip PD alert demo, focus on RCA analysis

---

## Key Talking Points

### Problem Statement
- "Detection takes 26 hours on average - that's more than a full day."
- "100% of incidents had observable signals. We're not blind, we're just not alerting."
- "Approval delays add 12+ hours even when the fix is known."

### Solution Value
- "This system processes 10 RCAs in 20 minutes. Manually would take days."
- "We get specific, actionable insights - not generic 'improve monitoring'."
- "Runbooks encode tribal knowledge - new engineers can diagnose like senior engineers."

### Business Impact
- "$186K/year time savings from just 4 RCAs. With 15 RCAs, estimate becomes more accurate."
- "Quick wins in weeks: Deploy diagnosis skills, 40-60% faster."
- "Long-term automation: AI-assisted diagnosis in 6 months, autonomous in 12 months."

### Risk Mitigation
- "Tier 1 skills preserve human judgment - low risk, fast deployment."
- "Automatic backups before any update - no data loss."
- "Production-first prioritization - focus where business impact is highest."

---

## Demo Variants

### 5-Minute Lightning Demo
1. Problem: Show TTD numbers (1 min)
2. Solution: Show RCA analysis file (2 min)
3. Value: Show ROI estimate (1 min)
4. Next steps: "Analyze 10 more RCAs this week" (1 min)

### 15-Minute Deep Dive
1. Problem context (3 min)
2. System overview (3 min)
3. RCA analysis walkthrough (4 min)
4. Batch processing demo (3 min)
5. ROI and next steps (2 min)

### 30-Minute Technical Review
- Include: Architecture (subagents, MCP integration)
- Include: Runbook creation logic
- Include: Platform evaluation connection
- Include: Q&A

---

## Common Questions & Answers

### Q: "How accurate is the 26.4h TTD average?"
**A**: "Based on 4 RCAs. Range is 17 minutes to 3 days. We need 10-15 RCAs for statistical confidence, but the pattern is clear: detection is the bottleneck."

### Q: "Can this replace manual RCA reviews?"
**A**: "No - it accelerates analysis and extracts patterns. Engineers still validate findings and make decisions. Think of it as 'RCA assistant', not 'RCA replacement'."

### Q: "What's the implementation timeline?"
**A**: 
- **Week 1-2**: Analyze 10 more RCAs, validate patterns
- **Week 3-4**: Create Tier 1 skills for top 3 incident types
- **Month 2-3**: Deploy skills, measure impact
- **Month 4-6**: Platform evaluation (Matrix, Warden, ICD, Resolve.ai)
- **Month 6-12**: AI-assisted automation (Tier 2/3)

### Q: "How does this relate to platform evaluation?"
**A**: "This analysis defines platform requirements. We need a platform that can:
1. Detect DB CPU saturation at namespace level (from RCA #1)
2. Monitor PassthroughCluster traffic (from RCA #2)
3. Track archival queue depth (from RCA #3)
4. Correlate deployment timing with errors

We'll score platforms (Matrix, Warden, ICD, Resolve.ai) against these requirements."

### Q: "What if we get more RCAs later?"
**A**: "Re-run the batch analyzer. It updates the executive report incrementally. Backup system preserves history."

### Q: "Can other teams use this?"
**A**: "Yes - the skill is reusable. Any team with RCA Google Docs can run it. We'd need to adjust environment priorities and metric names, but the core logic is the same."

---

## Post-Demo Actions

### If Well-Received
1. **Immediate**: Get 10-15 more RCA URLs
2. **This week**: Run batch analysis
3. **Next week**: Present updated findings

### If Questions/Concerns
1. Address specific concerns (capture in notes)
2. Offer pilot: "Let's analyze 5 more RCAs, review together"
3. Provide written summary (email executive report)

### If Neutral
1. Focus on quick wins: "We can deploy Tier 1 skills in 2 weeks"
2. Offer PoC: "Pick one incident type, we'll build a diagnostic skill"
3. Quantify further: "Need more RCAs for confidence? We can analyze 20 this week."

---

## Files to Have Open During Demo

**Terminal 1**: Project root
```bash
cd /Users/sakshi.mehrotra/Documents/repos/incident-triaging-evaluation
```

**Terminal 2**: Executive report
```bash
less research/incident-automation-executive-report.md
```

**Terminal 3**: Sample RCA analysis
```bash
less "research/past rca/rca-analysis-2.md"
```

**Browser**: Google Doc of one RCA (show source material)

---

## Demo Success Criteria

**Minimum**:
- [ ] Audience understands the problem (detection bottleneck)
- [ ] Audience sees the system works (file outputs exist)
- [ ] Audience knows next steps (analyze more RCAs)

**Target**:
- [ ] Audience excited about ROI ($186K/year)
- [ ] Audience agrees to provide more RCA URLs
- [ ] Stakeholder approval to proceed with Tier 1 skills

**Stretch**:
- [ ] Live batch processing approved
- [ ] Platform evaluation budget/timeline approved
- [ ] Team commitment for PoC (pick incident type, deploy skill)

---

**Demo prepared by**: Sakshi Mehrotra  
**System ready**: 2026-05-08  
**Current data**: 4 RCAs, $186K ROI estimate  
**Next milestone**: 15 RCAs analyzed, refined estimates
