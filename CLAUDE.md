# Incident Triaging Framework Evaluation - Project Context

## Project Identity

**Project**: Temporal Intelligent Incident Automation Framework Evaluation  
**Phase**: Research & Evaluation (no code yet)  
**Your Role**: Staff Engineer  
**Goal**: Evaluate platforms and design a strategy for automating Temporal incident **detection, diagnosis, and resolution**  
**Scope**: Full incident automation pipeline - detection → diagnosis → remediation  
**Target**: Reduce MTTD (detect), MTTD (diagnose), and MTTR (resolve)

## What This Workspace Is For

This is an **exploratory workspace** for researching, analyzing, and comparing incident automation platforms. Work here involves:

- Reading and synthesizing platform documentation
- Analyzing Temporal's past incidents and RCAs
- Comparing platform capabilities against requirements (detection, diagnosis, remediation)
- Designing evaluation criteria and decision frameworks
- Producing strategy recommendations
- Assessing remediation safety and phased rollout approaches

## Active Context Files

### Research & Evaluation Context
- `@.claude/context/platform-evaluation.md` - How to evaluate platforms, comparison frameworks
- `@.claude/context/temporal-incidents.md` - Temporal incident patterns, what needs to be triaged
- `@.claude/context/knowledge-sources.md` - Where incident data comes from (Argus, logs, GUS, Slack)

### Platforms Under Evaluation
- `@.claude/context/platforms/wardenaiops.md` - WardenAIOps capabilities and integration
- `@.claude/context/platforms/icd.md` - ICD platform details
- `@.claude/context/platforms/claude-skills.md` - Claude local skills approach
- `@.claude/context/platforms/mastra.md` - Mastra for knowledge agents
- `@.claude/context/platforms/hybrid.md` - Hybrid approach considerations

## Artifact Storage

As we work, save research outputs here:
- `research/` - Platform comparisons, evaluation matrices, meeting notes
- `.claude/artifacts/` - Generated analysis, decision docs, strategy drafts

## Key Questions We're Answering

1. What incident patterns does Temporal have that could be automated (detect → diagnose → resolve)?
2. Which platforms best handle Temporal's full incident lifecycle?
3. What's the integration effort for each platform?
4. What are the trade-offs (cost, accuracy, latency, maintainability, safety)?
5. Should we build, buy, or hybrid?
6. What does a phased rollout look like (detection first → remediation later)?
7. What safety mechanisms are needed for automated remediation?
8. What does a PoC look like?

## How to Work in This Project

### When You Have New Information
"I have a Google doc about how Team X uses WardenAIOps"
→ Claude will read it, extract key points, update relevant context

### When Analyzing Incidents
"Here's an RCA link for the March 9 replication failure"
→ Claude will analyze patterns, add to incident knowledge base

### When Comparing Platforms
"Compare WardenAIOps vs Mastra for our use case"
→ Claude will reference stored context and produce structured comparison

### When Making Decisions
"Help me decide between approaches"
→ Claude will use evaluation criteria, produce decision framework

## Domain Context Files

Load these as needed:

### Platform Evaluation
`@.claude/context/platform-evaluation.md`  
Use when: Comparing platforms, defining evaluation criteria, making decisions

### Temporal Incidents
`@.claude/context/temporal-incidents.md`  
Use when: Analyzing what needs to be automated, understanding incident patterns

### Knowledge Sources
`@.claude/context/knowledge-sources.md`  
Use when: Understanding data integration, building knowledge graphs

## Output Preferences

Since this is research/evaluation work:
- **Longer outputs are OK** - depth matters for strategy
- **Use tables and matrices** for comparisons
- **Cite sources** - reference docs, RCAs, team examples
- **Show reasoning** - explain trade-offs and assumptions
- **Save artifacts** - write comparison docs to `research/` or `.claude/artifacts/`

### Document Formatting Convention

**Heading structure** (use this convention for all documents):
- `##` for main sections (e.g., "Executive Summary", "Platform Comparison")
- `###` for subsections (e.g., "Detection Phase", "Causation Phase")
- `####` for sub-subsections (e.g., "Testing Results", "Data Sources")
- **Bold text** for inline emphasis within tables or paragraphs
- Avoid using `#` (top-level) - start with `##`

**Example**:
```markdown
## Platform Comparison Matrix

### Detection Phase

#### Temporal Testing Results
```

### Writing Style: Data-Driven, Not Marketing

**Always apply**:
- Data-driven: Use numbers, timelines, test results, concrete observations
- Factual statements: "Platform X supports Y" not "Game-changing capability Y"
- Remove hype words: Avoid "game-changer", "critical discovery", "exciting", "powerful", etc.
- Concrete comparisons: Tables, metrics, feature lists over narrative
- Direct language: State facts, not implications wrapped in enthusiasm
- Let data speak: Present evidence, let reader draw conclusions

**Examples**:
- ❌ "Game-Changer Discovery: Claude Agent SDK Portability"
- ✅ "Claude Agent SDK Integration Status"

- ❌ "This is a critical breakthrough that changes everything"
- ✅ "Both platforms support Claude Agent SDK (Matrix: native, Warden: cycle 264)"

- ❌ "Exciting new capability enables powerful workflows"
- ✅ "Workflow engine supports conditional branching (if/else logic)"

---

**Phase**: Evaluation  
**Timeline**: Exploratory  
**Last Updated**: 2026-05-07  
**Owner**: Sakshi Mehrotra
