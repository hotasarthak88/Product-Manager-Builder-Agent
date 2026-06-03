# Three-Layer Analysis Framework

Every QA finding is evaluated through three structured layers. This produces output that engineering and product can immediately act on.

---

## The Three Layers

### Layer 1: Reasoning (Why Is This an Issue?)

Articulates WHY a finding matters — not just WHAT is wrong.

**Questions answered**:
1. What does the user experience?
2. Does this erode trust in the system?
3. Could this cause a wrong decision?
4. Will users stop using the feature?
5. Does this create compliance/audit risk?

**Categories**:

| Category | Example |
|----------|---------|
| Content Accuracy | AI says "current" when question asks for "accurate" |
| Content Consistency | Different text in two surfaces for same data |
| Content Completeness | Important info truncated or dropped |
| UX Coherence | Info architecture breaks user's mental model |
| Semantic Confusion | Two distinct concepts conflated |
| Information Hierarchy | Important info not in most visible position |
| Progressive Disclosure Failure | Best content hidden behind extra click |
| Constraint Artifacts | Word limits causing quality loss |
| Silent Failure | Feature doesn't work with no error shown |

---

### Layer 2: Issue Summarization

Structured card ready for ticket filing:

```markdown
## Issue: [Title]

| Field | Value |
|-------|-------|
| Severity | P0 / P1 / P2 |
| Category | [From above] |
| Scope | Isolated / Partial / Systemic |
| Reproducibility | Always / Intermittent / Edge case |
| Affected Persona | [Role] |

What's happening: [2-3 sentences]
Expected behavior: [Per BRD/PRD]
Actual behavior: [Observed]
Evidence: [Screenshot or extracted text]
```

---

### Layer 3: Remediation Proposal

Actionable fix proposal:

```markdown
### Remediation

Root Cause: [Technical hypothesis]
Proposed Fix: [Specific change]
Effort: S / M / L / XL
Fix Type: Prompt / Backend / Frontend / Pipeline
Validation: [How to confirm fix worked]
Trade-offs: [Side effects]
Alternative Approaches: [If primary isn't feasible]
```

---

## Confidence Scoring

Each finding gets a confidence score (0-100%):

| Factor | Effect |
|--------|--------|
| Direct data contradiction | +30% |
| Same pattern across multiple instances | +20% |
| PM manually confirmed | +25% |
| BRD explicitly specifies different behavior | +25% |
| Heuristic-only (no hard proof) | -20% |
| Could be intentional design | -15% |
| Edge case | -10% |

**Tiers**:
- 90-100%: Confirmed → file ticket immediately
- 75-89%: High → include in report, recommend ticket
- 60-74%: Medium → flag for PM verification
- Below 60%: Low → "Needs Investigation" section

---

## Example (Real Finding)

```
## Finding: Q3 recommends "Yes" for evidence when 0 attachments exist

Confidence: 95% (Confirmed)
├── +30% — Direct contradiction (0 attachments vs "Yes")
├── +25% — BRD Epic-4 says validate evidence before submission
├── +20% — Observed across multiple reviews
└── +20% — PM manually confirmed

### Reasoning
User Impact: Reviewer trusts "Yes" recommendation, submits without evidence
Trust Signal: CRITICAL — AI contradicts what's visible on the page
Risk: Approver rejects → 60-90 min rework cycle
Adoption Threat: If users notice, they stop trusting ANY recommendation

### Summary
| Severity | P0 |
| Scope | Systemic |
| Reproducibility | Always |

### Remediation
Root Cause: Recommendation from historical pattern, doesn't check current attachments
Proposed Fix: Cross-reference attachment count before recommending "Yes" for evidence questions
Effort: M
Fix Type: Backend logic
Validation: Attach nothing → verify recommendation changes to conditional
```
