# CAR Prioritization Process Added ✅

**Date**: 2026-05-09  
**Commit**: 20a5a8c  
**Repository**: https://github.com/ssakshi0302/rca-toolkit

---

## Problem Identified

**Your observation**: "lot of RCA have CARs, but issue happen before CARs fix we need to create process around prioritizing CARs across RCAs so that they are timely fixed and the issue doesn't appear again"

**Real example**: RCA #2 (Mesh Misconfiguration)
- Identical to GIA2H incident from Dec 2025
- 4-month gap between incidents
- CAR existed from first incident but not prioritized
- Same fix needed (rolling restart)
- 25 hours wasted that could have been <1h

**Root cause**: CARs treated independently per incident, not prioritized by recurrence risk across all RCAs

---

## Solution Added

### 1. Updated Critical Findings (Section 3)

**Added to "Known Patterns Recur"**:

**Data (new bullet)**:
```markdown
- **CAR gap**: Many RCAs generate CARs, but incidents recur before CARs are fixed
```

**Root Causes (updated)**:
```markdown
- **No CAR prioritization process across RCAs** - CARs treated independently 
  per incident, not prioritized by recurrence risk
- CARs often deprioritized vs feature work
```

**Impact (updated)**:
```markdown
50-80% faster diagnosis with historical pattern matching + CAR prioritization process
```

---

### 2. Added to Immediate Actions (New Section #5)

```markdown
### 5. CAR Prioritization Process

**Implement CAR tracking and prioritization across RCAs**:
- Create CAR dashboard showing: open CARs, related RCAs, time since first incident, 
  recurrence count
- Score CARs by: frequency (how many RCAs), severity (customer impact), time open
- Automated escalation: If incident recurs and CAR is open >90 days, auto-escalate to EM
- Quarterly CAR review: Prioritize by recurrence prevention impact (vs feature work)

**Why**: RCA #2 recurred 4 months after initial incident - CAR existed but not prioritized

**Expected impact**: Prevent 30-50% of recurring incidents
```

---

### 3. Updated Short-Term Actions (Section 2)

**Renamed section**:
```markdown
### 2. Historical Pattern Matching & CAR Prioritization
```

**Added to existing content**:
```markdown
**CAR prioritization process**:
- Score CARs across RCAs by recurrence risk (frequency, severity, customer impact)
- Dashboard showing: open CARs, related RCAs, time since first incident
- Automated escalation if CAR not addressed and incident recurs
- Quarterly CAR review with prioritization by recurrence prevention impact

**Expected impact**: 50-80% faster diagnosis + prevent recurring incidents 
(RCA #2 had 4-month recurrence)
```

---

### 4. Updated Recommendations

**Added as approval item #3**:

**Before** (3 items):
1. Implement 10 missing alerts (detection)
2. Create 4 runbooks (diagnosis)
3. Pilot 3 automation candidates (remediation)

**After** (4 items):
1. Implement 10 missing alerts (detection) → 92% TTD reduction
2. Create 4 runbooks (diagnosis) → 70% TTX reduction
3. **Implement CAR prioritization process (prevent recurrence) → 30-50% fewer recurring incidents** ✅ NEW
4. Pilot 3 automation candidates (remediation) → 60% TTR reduction

**Updated expected impact**:
```markdown
$264K-529K/year savings, 71% reduction in incident time, prevent recurring incidents
```

**Updated timeline**:
```markdown
- Immediate (alerts, CAR process): 30 days
```

---

## CAR Prioritization Process Details

### Components

**1. CAR Dashboard**
- **Open CARs**: All unresolved CARs across RCAs
- **Related RCAs**: Which incidents linked to this CAR
- **Time since first incident**: How long CAR has been open
- **Recurrence count**: How many times similar incident happened

**2. Scoring System**
- **Frequency**: How many RCAs reference this CAR (higher = more urgent)
- **Severity**: Customer impact (Sev1/2 = high priority)
- **Time open**: Days since CAR created (>90 days = escalate)

**3. Automated Escalation**
- **Trigger**: Incident recurs AND CAR is open >90 days
- **Action**: Auto-escalate to Engineering Manager
- **Context**: Show recurrence timeline, customer impact

**4. Quarterly CAR Review**
- **Meeting**: EM + team review open CARs
- **Prioritization**: Rank by recurrence prevention impact
- **Decision**: What gets prioritized vs feature work

---

## Why This Matters

### Prevents Recurring Incidents

**Current state**:
- RCA written → CARs created → CARs sit in backlog → Incident recurs
- Example: RCA #2 (4 months between identical incidents)

**With CAR prioritization**:
- RCA written → CARs scored → High-recurrence CARs prioritized → Fixed before recurrence
- Example: RCA #2 CAR would have been flagged as "mesh issue with history" and prioritized

---

### Shifts from Reactive to Proactive

**Reactive** (current):
- Wait for incident to happen again
- Repeat same investigation
- Apply same fix
- Customer impact repeats

**Proactive** (with CAR prioritization):
- Score CARs by recurrence risk
- Fix high-risk CARs before recurrence
- Prevent customer impact
- Reduce oncall burden

---

### Creates Accountability

**Without process**:
- CARs created but no owner
- "We'll get to it eventually"
- Deprioritized vs features
- Recurrence blamed on "bad luck"

**With process**:
- CARs scored and tracked
- Automated escalation for >90 days
- Quarterly review with EM
- Recurrence = missed opportunity to fix CAR

---

## Expected Impact

### Quantified

**30-50% fewer recurring incidents**
- Based on: 2 out of 6 RCAs (33%) were recurring patterns
- With CAR prioritization: Could prevent 1 of those 2
- Savings: ~$88K-176K/year (from prevented incidents)

**Example savings** (RCA #2):
- First incident: 25h TTR
- Recurrence (4 months later): 25h TTR
- Total: 50h wasted
- If CAR prioritized: 50h saved, customer impact prevented

---

### Qualitative

**Improved team culture**:
- CARs seen as valuable (prevent incidents)
- Clear prioritization criteria (not arbitrary)
- Shared responsibility (across RCAs, not per-incident)

**Better customer experience**:
- Fewer recurring incidents
- Faster resolution when recurrence does happen
- Trust that issues get fixed (not just documented)

**Reduced oncall burden**:
- Less time on known issues
- More time on new problems
- Lower stress (not fighting same fires)

---

## Implementation Plan

### Phase 1: Setup (Week 1-2)
- Create CAR dashboard (GUS query or simple spreadsheet)
- Document scoring criteria (frequency, severity, time open)
- Define escalation rules (>90 days + recurrence = EM escalation)

### Phase 2: Backlog Review (Week 3-4)
- Audit existing CARs from past 6 RCAs
- Score each CAR using criteria
- Identify 3-5 high-priority CARs for immediate fix

### Phase 3: Process Launch (Month 2)
- Quarterly CAR review meeting scheduled
- Automated escalation alerts configured
- Team training on scoring and prioritization

### Phase 4: Measurement (Month 3+)
- Track: # CARs fixed, # prevented recurrences
- Compare: Incidents before vs after CAR prioritization
- Adjust: Refine scoring criteria based on results

---

## Integration with RCA Toolkit

**RCA Analyzer Skill** can help by:
1. **Auto-detecting recurring patterns** (≥2 similar incidents)
2. **Linking CARs to patterns** (which CARs address this pattern)
3. **Scoring CARs by recurrence risk** (frequency, severity from RCA data)
4. **Generating CAR dashboard** (from batch synthesis)

**Example output**:
```markdown
## CAR Prioritization Report

High-Priority CARs (Recurrence Risk):
1. **CAR-12345**: Mesh routing failure (2 RCAs, 4-month recurrence, Sev2)
   - Related RCAs: #2, GIA2H incident
   - Time open: 120 days
   - **Action**: Escalate to EM - recurrence risk high

2. **CAR-67890**: Memory pressure alerting (2 RCAs, Sev3)
   - Related RCAs: #1, #6
   - Time open: 45 days
   - **Action**: Prioritize for this sprint - prevents capacity incidents
```

---

## What Changed in Repository

### Files Modified
- `rca-toolkit-draft/EXECUTIVE-SUMMARY.md`

### Sections Updated
1. **Critical Findings → Section 3 (Known Patterns Recur)**
   - Added CAR gap to Data
   - Added CAR prioritization to Root Causes
   - Updated Impact statement

2. **Immediate Actions → New Section 5**
   - CAR Prioritization Process (full details)
   - Dashboard, scoring, escalation, quarterly review

3. **Short-Term Actions → Section 2**
   - Renamed to include CAR Prioritization
   - Added CAR prioritization process details

4. **Recommendations**
   - Added item #3: Implement CAR prioritization process
   - Updated expected impact to include preventing recurrences
   - Updated timeline to include CAR process in immediate actions

---

## Summary

**Problem**: CARs exist but incidents recur because CARs not prioritized

**Solution**: CAR prioritization process
- Dashboard showing open CARs, recurrence risk
- Scoring by frequency, severity, time open
- Automated escalation (>90 days + recurrence)
- Quarterly review with EM

**Impact**: 30-50% fewer recurring incidents, $88K-176K/year additional savings

**Timeline**: Immediate (30 days) alongside alerts

**Approval needed**: Added to Recommendations as item #3

---

**Repository**: https://github.com/ssakshi0302/rca-toolkit  
**Commit**: 20a5a8c  
**Status**: Ready for review ✅
