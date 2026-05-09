# Output Format Specification

**Internal specification - DO NOT COMMIT**

---

## Purpose

Defines expected structure, tone, and consistency requirements for framework-generated operational analysis outputs.

---

## Standard Output Structure

### Operational Analysis Summary

```markdown
# Operational Analysis Summary

## Scope
- **Incidents analyzed**: [count] incidents
- **Services**: [service names]
- **Time range**: [date range]
- **Analysis date**: [YYYY-MM-DD]

## Key Observations

[Framework-generated observations from incident analysis]

### Detection Patterns
- [Observable delays, missing alerts, signal gaps]

### Diagnosis Patterns
- [Manual correlation steps, recurring investigation paths, runbook gaps]

### Remediation Patterns
- [Manual processes, approval delays, repeated actions]

### Recurring Root Causes
- [≥2 incidents with same service + symptom + root cause]

## Common Operational Gaps

[Evidence-based gaps across analyzed incidents]

### Observability Gaps
- [Missing metrics, dashboards, visibility]

### Alerting Gaps
- [Signals exist but no alerts configured]

### Runbook Gaps
- [Repeated diagnosis steps without documentation]

### Process Gaps
- [Approval bottlenecks, manual workflows, ownership unclear]

## Repeated Patterns

[Patterns occurring ≥2 times across incidents]

| Pattern | Service | Occurrences | Symptom | Root Cause |
|---------|---------|-------------|---------|------------|
| [Name] | [service] | [count] | [symptom] | [cause] |

## Potential Improvement Areas

[Framework-identified opportunities]

### Immediate (alerts, dashboards)
- [Specific, actionable items]

### Short-term (runbooks, correlation)
- [Specific, actionable items]

### Longer-term (automation candidates)
- [Specific, actionable items with safety notes]

---

## Engineering Recommendations

**Human-reviewed and operationally prioritized**

### Detection
- [Engineering-reviewed alert/monitoring recommendations]

### Diagnosis
- [Engineering-reviewed runbook/correlation recommendations]

### Remediation
- [Engineering-reviewed automation candidates with safety assessment]

---

## Operational Follow-Ups

### Immediate Actions
- [30-day timeframe]

### Short-Term Actions
- [60-90 day timeframe]

### Longer-Term Initiatives
- [6-12 month timeframe]

```

---

## Section Ordering

1. **Scope** - What was analyzed (incidents, services, time range)
2. **Key Observations** - Framework-generated patterns and gaps
3. **Common Operational Gaps** - Cross-incident themes
4. **Repeated Patterns** - Specific recurring incidents
5. **Potential Improvement Areas** - Framework-identified opportunities
6. **Engineering Recommendations** - Human-reviewed prioritization
7. **Operational Follow-Ups** - Actionable next steps

---

## Framework-Generated vs Human-Reviewed

### Framework-Generated Sections
- Key Observations
- Common Operational Gaps
- Repeated Patterns
- Potential Improvement Areas

**These sections**:
- Present evidence from incidents
- Identify patterns (≥2 occurrences)
- Document observable gaps
- List potential opportunities

**Do NOT**:
- Make prioritization decisions
- Assess safety/risk
- Recommend specific implementations
- Determine organizational changes

### Human-Reviewed Sections
- Engineering Recommendations
- Operational Follow-Ups

**These sections**:
- Require engineering review
- Include domain expertise
- Assess safety and risk
- Prioritize by operational impact
- Determine implementation approach

**Clearly marked**:
- "Engineering-reviewed"
- "Human-prioritized"
- "Domain-informed"

---

## Tone & Style Expectations

### Operational Language
**Use**:
- Detection delays
- Observable patterns
- Signal gaps
- Runbook opportunities
- Manual correlation
- Recurring failures

**Avoid**:
- Transformation initiatives
- Strategic recommendations
- Business value propositions
- Maturity assessments
- Organizational change

### Grounded Phrasing
**Use**:
- "Preliminary analysis indicates..."
- "Recurring operational bottlenecks identified..."
- "Observable delays contributed to..."
- "Potential efficiency opportunities..."

**Avoid**:
- "97% reduction guaranteed"
- "Massive savings"
- "Game-changing automation"
- "Revolutionary approach"

### Evidence-Based
**Always cite**:
- Specific incidents (RCA #1, RCA #3)
- Concrete timelines (20h TTD, 3-day detection delay)
- Observable metrics (DB CPU 100%, memory >70%)
- Repeated occurrences (2/6 incidents, 33% recurrence rate)

---

## Consistency Expectations

### Pattern Detection
**Consistent criteria**:
- ≥2 incidents required for "recurring pattern"
- Same service + symptom + root cause = pattern
- Frequency and severity documented
- Runbook generation triggered at ≥2 occurrences

### Gap Identification
**Consistent approach**:
- Missing alert = signal exists, alert missing
- Observability gap = metric/dashboard missing
- Runbook gap = repeated diagnosis steps undocumented
- Process gap = approval delay or manual workflow

### Time Breakdown
**Consistent methodology**:
- TTD = incident start → detection
- TTX = detection → root cause identified
- TTR = root cause → resolution
- Always cite incident-specific timelines

---

## Output Quality Checklist

Before finalizing any output, verify:

- [ ] Scope section present (incidents, services, time range)
- [ ] Observations tied to specific incidents
- [ ] Patterns supported by ≥2 occurrences
- [ ] Gaps documented with concrete examples
- [ ] Framework vs human sections clearly separated
- [ ] Language is operationally grounded (no hype)
- [ ] Recommendations marked as engineering-reviewed
- [ ] Time estimates based on actual incidents
- [ ] No overclaiming (97% reduction, massive savings)
- [ ] Tone is concise, technical, practical

---

## Example: Correct vs Incorrect

### ✅ Correct
```
## Key Observations

### Detection Patterns
- 4/6 incidents had detection delays >10 hours (average: 16.5h)
- Observable signals existed in all incidents; delays caused by missing alerts
- 10 specific missing alerts identified (DB CPU, memory pressure, OOMKilled events)
```

**Why**: Evidence-based, specific incidents cited, grounded language

### ❌ Incorrect
```
## Strategic Transformation Opportunity

Our AI-powered analysis reveals game-changing automation potential with 97% reduction 
in detection time, delivering $500K+ annual value. This revolutionary approach will 
transform incident management and position the team as industry leaders.
```

**Why**: Hype language, overclaiming, business case framing, no evidence

---

### ✅ Correct
```
## Engineering Recommendations

**Human-reviewed and operationally prioritized**

### Detection
- Implement DB CPU alerting (>80% sustained 10+ min)
- Add memory pressure monitoring (>70% sustained 10+ min)
- Configure OOMKilled event alerts (any occurrence)
```

**Why**: Clearly marked as human-reviewed, specific, actionable, operationally grounded

### ❌ Incorrect
```
## Framework Recommends

The AI system has determined optimal automation strategy:
1. Deploy autonomous incident detection (97% faster)
2. Implement self-healing remediation (zero human intervention)
3. Roll out across organization (expected $2M savings)
```

**Why**: Implies autonomous decisions, overclaims, business framing, no human review marker

---

## Summary

**Framework outputs should be**:
- Concise (1-3 pages typical)
- Operationally grounded
- Evidence-based
- Consistently structured
- Clear on framework vs human contributions

**Framework outputs should NOT be**:
- Consulting-style reports
- Business transformation proposals
- Hype-driven showcases
- Overclaiming guarantees
- Autonomous decision documents

---

**Last Updated**: 2026-05-09  
**Purpose**: Internal specification for output consistency and quality
