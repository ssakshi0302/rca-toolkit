# Temporal Incident Triaging Framework Evaluation

## Overview

**Status**: Evaluation Phase  
**Owner**: Sakshi Mehrotra  
**Role**: Staff Engineer  
**Goal**: Evaluate and recommend a framework for automating Temporal incident **detection, diagnosis, and resolution**  
**Scope**: Full incident automation pipeline with phased rollout - detection/diagnosis first (Phase 1), then automated remediation (Phases 2-3)  
**Target**: Reduce MTTD (Mean Time To Detect), MTTD (Mean Time To Diagnose), MTTR (Mean Time To Resolve)

## What This Repository Is

This is a **research workspace**, not a code repository (yet). It contains:
- Evaluation notes on various platforms
- Analysis of Temporal incident patterns
- Comparison matrices and decision frameworks
- Strategy documents and recommendations

## Platforms Being Evaluated

- **Matrix**: Salesforce's intelligent incident management platform
- **AI Exchange**: Salesforce's AI agent orchestration platform
- **Hybrid Approaches**: Combining multiple platforms or custom solutions

**Evaluation Focus**: End-to-end capability (detection → diagnosis → remediation), with emphasis on safety mechanisms for automated actions

## Structure

```
.
├── CLAUDE.md                    # Project context for Claude
├── README.md                    # This file
├── research/                    # Evaluation documents
│   ├── comparisons/            # Platform comparison matrices
│   ├── strategy/               # Strategy documents
│   └── notes/                  # Meeting notes, findings
├── .claude/
│   ├── context/                # Context files for Claude
│   │   ├── platform-evaluation.md
│   │   ├── temporal-incidents.md
│   │   ├── knowledge-sources.md
│   │   └── platforms/          # Per-platform deep dives
│   └── artifacts/              # Generated analysis documents
```

## How to Work Here

### Adding Platform Information
When you have docs, specs, or examples:
1. Share with Claude: "I have [doc/link] about [platform]"
2. Claude extracts key points and updates relevant context
3. Context persists across sessions

### Analyzing Incidents
When you have RCAs or incident data:
1. Share with Claude: "Analyze this RCA for triaging patterns"
2. Claude identifies automatable patterns
3. Adds to incident knowledge base in context files

### Comparing Platforms
When you need a comparison:
1. Ask Claude: "[Staff Engineer] Compare Platform A vs B for our use case"
2. Claude loads relevant context and produces structured comparison
3. Save output to `research/comparisons/`

### Strategy Development
When you're ready to recommend:
1. Ask Claude: "[Staff Engineer] Draft a strategy doc for [approach]"
2. Claude synthesizes learnings into decision document
3. Save to `research/strategy/`

## Decision Criteria (Evolving)

Key factors we're evaluating:
- Accuracy of incident detection, diagnosis, and remediation
- End-to-end capability (detection → diagnosis → remediation)
- Safety mechanisms for automated actions
- Integration complexity
- Temporal domain fit
- Operational burden
- Cost
- Team skillset match
- Salesforce paved path alignment
- Phased rollout support (observational → guided → automated)

## Current Phase

**Phase 1: Paper Evaluation** ✓ (In Progress)
- Gathering platform information
- Analyzing Temporal incident patterns
- Defining evaluation criteria

**Phase 2: PoC** (Future)
- Build minimal integration
- Test on historical incidents
- Measure accuracy/latency

**Phase 3: Decision** (Future)
- Recommend approach
- Get stakeholder buy-in
- Plan implementation

---

**Last Updated**: 2026-05-07
