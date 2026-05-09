# AI Document Creation Template

**Purpose**: Get high-quality documents faster by defining requirements upfront instead of iterating after creation.

**Time savings**: 2-3 minutes upfront clarification vs 20+ minutes post-creation editing.

---

## Template

```
[Role] Create [document type] for [audience]

**Requirements**:
- Length: [X pages]
- Style: [data-driven/technical/executive/etc.]
- Format: [will paste to Google Docs/PDF/presentation/etc.]
- Constraints: [no team names/exclude sections X,Y/etc.]

**Content**:
- [Key points to cover]

**Before creating full doc**: Show me 1-paragraph example to confirm style/tone.
```

---

## Example Prompt

```
[Staff Engineer] Create platform comparison document for leadership decision-making

**Requirements**:
- Length: 4-6 pages max
- Style: Data-driven (numbers, timelines, concrete observations)
  - Use factual statements: "Platform X supports Y" not "Game-changing capability Y"
  - Avoid hype words: "game-changer", "critical discovery", "exciting", "powerful"
  - Heavy use of comparison tables
- Format: Will paste to Google Docs
  - Use ## and ### headers only (avoid # and ####)
  - Prefer tables over bullet lists
- Constraints: 
  - Do NOT reference specific internal team names (use "internal teams")
  - Do NOT include detailed cost analysis
  - Do NOT include execution phase details (focus on decision)

**Content**:
- Executive summary with problem/evaluation/recommendation
- Platform comparison table (Matrix vs WardenAIOps)
- Testing results (what was validated vs needs exploration)
- HyperForce context (why it matters)
- Strategic recommendation with 8-10 week timeline
- Risks & mitigation
- Next steps

**Before creating full doc**: Show me 1-paragraph example of the executive summary to confirm style/tone matches expectations.
```

---

## Key Principles

### 1. Define Output Requirements Upfront
- Length, audience, format
- Avoids creating wrong-sized documents

### 2. Establish Style Rules First
- Data-driven vs narrative
- Technical vs executive language
- Reference examples of what to avoid

### 3. Specify Exclusions/Constraints
- What NOT to include
- What NOT to reference
- Avoids cleanup passes

### 4. Request Example First
- 1-paragraph or 1-section sample
- Course-correct before full work
- "Show me an example of X section first"

### 5. Clarify Deliverable Format
- Final destination (Google Docs, PDF, presentation)
- Formatting constraints
- Heading structure

### 6. Plan Multi-Document Sets Together
```
"Create 3 versions:
1. Executive (1-2 pages) - decision brief
2. Strategic (4-5 pages) - detailed recommendation
3. Technical (as needed) - full comparison tables"
```

---

## Anti-Patterns (What NOT to Do)

❌ **Vague request**: "Create a platform comparison"
- Missing: length, audience, style, format

❌ **Iterate after creation**: Create 18 pages → "reduce to 6" → "reduce to 4"
- Should have specified upfront

❌ **Style corrections after**: "Remove hype words from all documents"
- Should have established style first

❌ **Format fixes late**: "This pastes badly into Google Docs"
- Should have specified format constraints

❌ **Sequential document creation**: Create doc 1, review, create doc 2, review...
- Should have planned all versions together

---

## Checklist Before Requesting Document

- [ ] Specified length/page count
- [ ] Defined audience (executives/technical/peer review)
- [ ] Described style (data-driven/technical/narrative)
- [ ] Clarified format (Google Docs/PDF/markdown/presentation)
- [ ] Listed constraints (what NOT to include)
- [ ] Outlined key content sections
- [ ] Requested example first (optional but recommended)

---

## Template for Research/Evaluation Documents

```
[Staff Engineer] Create [topic] evaluation document

**Requirements**:
- Length: [1-2 / 4-6 / 10-15] pages
- Audience: [Leadership/Technical team/Peer review]
- Style: Data-driven
  - Use numbers, timelines, test results, concrete observations
  - Factual statements over narrative
  - Avoid hype words: "game-changer", "exciting", "breakthrough"
- Format: Will paste to [Google Docs/Confluence/etc.]
  - [Heading structure preferences]
  - [Table vs text preferences]

**Content**:
- Problem statement
- What was tested/researched
- Key findings with evidence
- Comparison table (Option A vs Option B)
- Recommendation with rationale
- Timeline and next steps
- Risks & mitigation

**Constraints**:
- [Any exclusions]
- [Any confidentiality restrictions]

**Before creating**: Show 1-paragraph example of [key section] to confirm style.
```

---

## Real Example: This Project

**What worked well**:
```
"Remove hype words: avoid 'game-changer', 'critical discovery', 'exciting'"
→ Added to global CLAUDE.md
→ Now applies to all future documents automatically
```

**What could be faster next time**:
```
Specify upfront: "Create 3 versions (1-pager, 4-6 pages, comprehensive)"
Instead of: Creating versions one at a time
```

---

## Reusable Snippets

### Style: Data-Driven
```
Style: Data-driven
- Use numbers, timelines, test results, concrete observations
- Factual statements: "Platform X supports Y" not "Game-changing capability Y"
- Avoid hype words: "game-changer", "critical discovery", "exciting", "powerful"
- Concrete comparisons: tables, metrics, feature lists
- Direct language: state facts, let data speak
```

### Format: Google Docs
```
Format: Will paste to Google Docs
- Use ## and ### headers only (avoid # and ####)
- Prefer tables over bullet lists
- Bold text for emphasis within tables/paragraphs
```

### Audience: Leadership
```
Audience: Leadership 
- Length: 4-6 pages max
- Easy to skim
- Heavy on tables/data
- do not use emoticons/icons like tick ✅ , searching, cross, use text
- Clear recommendation upfront
```

---

**Saved**: 2026-05-07  
**Use for**: Any future research documents, platform evaluations, technical comparisons, strategy recommendations
