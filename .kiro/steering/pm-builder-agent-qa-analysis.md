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

Based on manual QA patterns identified for CLARA, apply these heuristics when scanning any review:

### Heuristic 1: Content Source Consistency
**Check**: Does the inline Questionnaire insight match the Full Analysis insight for the same question?
**Trigger**: If text differs between the two surfaces, flag it.
**Why**: Users who read the inline first form expectations. If Full Analysis contradicts or significantly enriches, trust erodes.

### Heuristic 2: Jargon/Internal Field Leak
**Check**: Does any user-facing text contain internal system identifiers (field names, API parameters, database columns)?
**Trigger**: CamelCase words that aren't proper nouns, terms not in the question text, technical identifiers.
**Pattern**: `ConfirmationField`, `reviewSetId`, `entityType`, `complianceFlag`
**Why**: Reveals the AI is referencing the data model, not the user experience.

### Heuristic 3: Insight vs Reasoning Conflation
**Check**: Does the inline "Insights" text read like justification (backward-looking: "prior reviews said X") rather than guidance (forward-looking: "verify X before answering")?
**Trigger**: If the inline text starts with "The control's..." or "All 5 prior reviews..." without telling the user what to DO.
**Why**: Reasoning explains the AI's choice; Insight guides the user's action. Showing Reasoning as Insight gives justification without direction.

### Heuristic 4: Information at Point of Decision
**Check**: Is critical information (flags, prior failures, discrepancies) visible at the exact point where the user makes their decision (the radio button/text field)?
**Trigger**: If important flags only appear in the Executive Summary or Full Analysis but NOT next to the relevant question.
**Why**: Reviewers answer questions top-to-bottom. If a flag is 5 screens above the question, they'll miss it.

### Heuristic 5: Template/Repetitive Content
**Check**: Do multiple reviews start with the same opening phrase or follow the same rigid template?
**Trigger**: If >50% of reviews share identical opening sentences.
**Why**: Template-feeling content trains users to skip it. The most valuable real estate (first sentence) is wasted.

### Heuristic 6: Obvious Statement Waste
**Check**: Does the AI tell the user something they can already see on the page?
**Trigger**: "All 13 questions are currently blank" (visible), "This is a draft review" (visible), "No attachments" (visible in attachment table).
**Why**: Wastes cognitive bandwidth. The first thing the AI says should be something the user DOESN'T already know.

### Heuristic 7: Word Limit Artifacts
**Check**: Does the inline text feel truncated, compressed, or like it chose the shorter of two options?
**Trigger**: Inline is significantly less specific than Full Analysis; inline drops names, dates, or specific data points that Full Analysis includes.
**Why**: Eval constraints (120-150 words) may force the model to pick shorter content even when the longer content is more valuable.

### Heuristic 8: Cross-Question Consistency
**Check**: Are CLARA's recommendations internally consistent across questions within the same review?
**Trigger**: If Q2 says "evidence is available" but the Evidence Matrix shows "no evidence found," flag the contradiction.
**Why**: Internal contradictions within a single review completely destroy trust.

### Heuristic 9: Control-Type Awareness
**Check**: Does the CLARA response correctly account for the control type (IT General vs Application vs Manual)?
**Trigger**: Suggesting IPA actions for automated controls, or N/A for manual controls with documented IPA.
**Why**: Wrong control-type handling produces incorrect recommendations that could lead to audit findings.

### Heuristic 10: Historical Context Accuracy
**Check**: Do claims about "prior reviews" match the actual historical data?
**Trigger**: "All 5 prior reviews answered Yes" but the control page shows a "No" in a recent quarter.
**Why**: Inaccurate historical claims are factual errors that could lead to incorrect attestations.

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
