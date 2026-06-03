# PM Builder Agent — QA Analysis Framework (Three-Layer Model)

## Overview

When conducting QA on CLARA Reviews AI Assistant (or any AI-assisted feature), the agent evaluates every finding through three structured layers. This produces actionable output that engineering and product can immediately prioritize.

## The Three Layers

### Layer 1: Reasoning (Why Is This an Issue?)

For every anomaly detected, the agent must articulate WHY it's a problem — not just WHAT is wrong. The reasoning layer answers:

1. **User Impact** — What does the user experience because of this issue?
2. **Trust Erosion** — Does this make the user question the AI's competence or consistency?
3. **Task Failure Risk** — Could this cause the user to make a wrong decision or submit incorrect data?
4. **Adoption Risk** — Will users stop using the feature because of this?
5. **Compliance/Audit Risk** — Does this create regulatory exposure?

**Format:**
```
### Why This Is an Issue

**User Impact**: [What the user experiences]
**Trust Signal**: [How this affects confidence in the system]
**Risk**: [What could go wrong downstream]
**Adoption Threat**: [Will users abandon this feature?]
```

**Reasoning Categories** (apply one or more):

| Category | Trigger | Example |
|----------|---------|---------|
| Content Accuracy | AI output is factually wrong or misleading | "ConfirmationField" jargon, "current" vs "accurate" |
| Content Consistency | Same data shown differently in two places | Full Analysis shows richer insight than inline |
| Content Completeness | Important information is missing or truncated | Reasoning visible in Full Analysis but dropped from inline |
| UX Coherence | Information architecture breaks user's mental model | No numbering in Full Analysis to match questionnaire |
| Semantic Confusion | System conflates two distinct concepts | Insight vs Reasoning shown interchangeably |
| Information Hierarchy | Most important info not in most visible position | Critical flag buried in summary instead of at the question |
| Progressive Disclosure Failure | Deep content not accessible at point of decision | Full Analysis has high-value content but requires extra click |
| Constraint Artifacts | Technical constraints produce user-facing quality issues | Word limit causing Reasoning to surface instead of Insight |
| Silent Failure | Feature doesn't work but no error is shown | Per-question recs not generating for 83% of reviews |

### Layer 2: Issue Summarization

Every issue gets a structured summary card that can be directly used for ticket filing:

**Format:**
```
## Issue: [One-line title]

| Field | Value |
|-------|-------|
| Severity | P0 / P1 / P2 / P3 |
| Category | [From reasoning categories above] |
| Scope | Isolated / Partial / Systemic |
| Reproducibility | Always / Intermittent / Edge case |
| Affected Persona | Reviewer / Approver / Oversight |
| Affected Reviews | [List or "All"] |
| Control Types Affected | [IT General / Application / IT Dep Manual / Manual / All] |

**What's happening**: [2-3 sentence factual description]
**Expected behavior**: [What the BRD/PRD specifies]
**Actual behavior**: [What was observed]
**Evidence**: [Screenshot reference, extracted text, or data point]
```

### Layer 3: Remediation Proposal

For every issue, the agent proposes a fix with:
1. **Root Cause Hypothesis** — what's likely causing this technically
2. **Proposed Fix** — specific, actionable change
3. **Effort Estimate** — T-shirt size (S/M/L/XL)
4. **Fix Type** — prompt change / backend logic / frontend UX / data pipeline
5. **Validation** — how to confirm the fix worked
6. **Trade-offs** — what might break or change if this fix is applied

**Format:**
```
### Remediation

**Root Cause**: [Technical hypothesis]
**Proposed Fix**: [Specific change]
**Effort**: S / M / L / XL
**Fix Type**: Prompt / Backend / Frontend / Pipeline
**Validation**: [How to verify]
**Trade-offs**: [Side effects or risks of the fix]
**Alternative Approaches**: [If primary fix isn't feasible]
```

---

## QA Bug-Finding Heuristics (Learned Patterns)

These heuristics apply to ANY AI-assisted product feature — not just compliance review tools. They detect common patterns where AI systems fail to deliver consistent, actionable, and trustworthy experiences.

### Heuristic 1: Content Source Consistency
**Check**: Does the same information appear differently across multiple UI surfaces?
**Trigger**: If text/data differs between two views of the same content (e.g., summary vs detail, inline vs modal, card vs full page).
**Why**: Users who see content in one place form expectations. If another surface contradicts or significantly enriches, trust erodes.

### Heuristic 2: Jargon/Internal Field Leak
**Check**: Does any user-facing text contain internal system identifiers?
**Trigger**: CamelCase words that aren't proper nouns, terms not in the UI label, database column names, API parameter names.
**Pattern examples**: `ConfirmationField`, `reviewSetId`, `entityType`, `statusCode`, `recordId`
**Why**: Reveals the AI is referencing the data model, not the user experience.

### Heuristic 3: Insight vs Reasoning Conflation
**Check**: Does the "explanation" or "insight" text read like justification (backward-looking) rather than guidance (forward-looking)?
**Trigger**: If helper text starts with "The system shows..." or "All previous records..." without telling the user what to DO.
**Why**: Reasoning explains why the AI chose something; Insight guides the user's next action. Showing Reasoning as Insight gives justification without direction.

### Heuristic 4: Information at Point of Decision
**Check**: Is critical information visible at the exact point where the user makes their decision?
**Trigger**: If important warnings only appear in a modal, a separate page, or above/below the decision point — not AT it.
**Why**: Users act top-to-bottom, left-to-right. If a warning is 5 screens above the decision field, they'll miss it.

### Heuristic 5: Template/Repetitive Content
**Check**: Does AI-generated content across multiple instances start with the same phrase or follow the same rigid template?
**Trigger**: If >50% of instances share identical opening sentences or structure.
**Why**: Template-feeling content trains users to skip it. The most valuable real estate (first sentence) is wasted.

### Heuristic 6: Obvious Statement Waste
**Check**: Does the AI tell the user something they can already see on the page?
**Trigger**: Restating visible UI state ("This form is empty," "No items selected," "You haven't entered anything").
**Why**: Wastes cognitive bandwidth. AI should add value the user can't derive from the page alone.

### Heuristic 7: Word Limit Artifacts
**Check**: Does compressed/inline content feel truncated or like it chose a shorter option over a richer one?
**Trigger**: Inline is significantly less specific than the expanded/detailed view; names, dates, or data points present in detail view but absent from inline.
**Why**: Character/word limits may force the AI to pick shorter content even when longer content is more valuable.

### Heuristic 8: Cross-Element Consistency
**Check**: Are AI recommendations internally consistent across different parts of the same page/flow?
**Trigger**: If one element says "X is true" but another element's data shows X is false.
**Why**: Internal contradictions within a single session completely destroy trust.

### Heuristic 9: Context-Type Awareness
**Check**: Does the AI correctly adapt its behavior based on the type/category of the entity it's operating on?
**Trigger**: Applying the wrong logic for the entity type (e.g., suggesting actions for Type A that only apply to Type B).
**Why**: Wrong context-type handling produces incorrect recommendations that could lead to downstream errors.

### Heuristic 10: Historical Context Accuracy
**Check**: Do claims about past behavior/data match actual records?
**Trigger**: "All previous instances show X" but one previous instance clearly shows Y.
**Why**: Inaccurate historical claims are factual errors that compound when users rely on them for decisions.

---

## Applying the Framework

When running QA, the agent:

1. **Scans** every CLARA response (inline Insight + Full Analysis if accessible) for each question
2. **Applies** all 10 heuristics to each response
3. **For each finding**: generates all three layers (Reasoning → Summarization → Remediation)
4. **Ranks** findings by severity (P0 > P1 > P2)
5. **Produces** a single consolidated report with all findings structured per the format above

---

*Built with pm-builder-agent*
