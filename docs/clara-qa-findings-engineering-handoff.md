# CLARA Reviews AI Assistant — QA Findings & Engineering Recommendations

**Author**: Sarthak Hota (sarthah)
**Date**: June 3, 2026
**Environment**: Gamma (`gamma.grc.a2z.com`)
**Reviews Tested**: 12 (primary deep-dive on #407759)
**Method**: Automated browser QA + Manual scenario testing

---

## TL;DR

Tested CLARA's Reviewer AI Assistant across 12 gamma reviews. Found **11 issues** (3× P0, 5× P1, 3× P2) and confirmed 1 positive (AI Governance suppression works). The three P0s share the same architectural root cause: **the evidence validation pipeline operates independently from the recommendation pipeline and inline rendering — CLARA has excellent intelligence that's architecturally trapped in the Full Analysis modal.**

---

## Executive Summary of Findings

| # | Finding | Severity | Root Cause Category |
|---|---------|----------|---------------------|
| 1 | Q3 recommends "Yes" for evidence availability when 0 attachments exist | **P0** | Recommendation ignores current state |
| 2 | Evidence validation intelligence (content analysis, name matching) only visible in Full Analysis — not inline, not in attachments table, not in quality gate | **P0** | Pipeline siloing |
| 3 | CLARA detects invalid evidence but Suggested Response remains "Yes" — doesn't adapt to own findings | **P0** | Recommendation ignores validation output |
| 4 | Inline "Insights" shows Reasoning text, not actual Insight — conflates two content types | P1 | Word limit constraint + field routing |
| 5 | Full Analysis flags (frequency discrepancy) not propagated to inline questions | P1 | Pipeline siloing |
| 6 | "ConfirmationField" internal field name leaked into user-facing insight text | P1 | Prompt template uses data model field IDs |
| 7 | IPA Q2 pre-fills completed attestation ("I reviewed and confirmed...") — should be conditional | P1 | Historical pattern used as completed statement |
| 8 | IPA Q3 recommendation contradicts Q2 recommendation (both can't apply) | P1 | No cross-question coherence check |
| 9 | Historical "No" (Q4 2022 Operating Effectiveness) mentioned in text but not visually flagged | P2 | No flag injection for historical failures |
| 10 | Executive Summary leads with obvious page state ("13 questions blank") | P2 | Prompt prioritizes current state over actions |
| 11 | Q8 (Carnaval) "Yes" based on historical pattern without GRC-Next current-state verification | P2 | No API integration for live checks |
| ✅ | AI Governance suppression works correctly (submitted review 407615 shows zero CLARA content) | Positive | — |

---

## The Core Architecture Problem

All three P0 findings stem from the same issue:

```
CURRENT (Siloed):

Evidence Upload → Validation Pipeline → Full Analysis ONLY
                                        ↕ NO CONNECTION ↕
Historical Data → Recommendation Pipeline → Inline Cards
                                        ↕ NO CONNECTION ↕
                  Quality Gate Logic → Submit Button


NEEDED (Integrated):

Evidence Upload → Validation Pipeline ──┬──→ Full Analysis
                                        ├──→ Inline Cards (override when flags exist)
                                        ├──→ Recommendation (adapt when Confidence=Low)
                                        └──→ Quality Gate (block when invalid)
```

CLARA can read file content, detect mismatches, set confidence levels, and generate flags. This intelligence exists. It's just not wired to where decisions happen.

---

## Detailed Findings

### P0-1: Evidence Recommendation Contradicts Page State

**Scenario**: Review has 0 attachments. Evidence Matrix says "No evidence found for this Test Procedure."

**CLARA says**: Q3 "Is evidence available?" → Recommended answer: **Yes**

**Why**: Recommendation generated from historical pattern ("all 5 prior reviews answered Yes") without checking current attachment count.

**Fix**: Add pre-recommendation check: IF question is about evidence availability AND attachments = 0 AND evidence matrix shows gaps → change recommendation to "Yes (conditional — attach evidence first)" or "Cannot confirm yet."

**Effort**: M | **Type**: Backend logic

---

### P0-2: Evidence Validation Intelligence Trapped in Full Analysis

**Scenario A**: Attached file named "Random Evidence Test" with random content.
- **Full Analysis**: ✅ Correctly flags it — "not valid evidence, name/description don't relate to this control"
- **Inline Q3 card**: ❌ Generic insight ("Prior reviews answered Yes")
- **Attachments table**: ❌ No validity indicator
- **Quality gate**: ❌ Doesn't block submission

**Scenario B**: Renamed file to "Clearwater Audit Report" (correct name, wrong content — PAC data).
- **Full Analysis**: ✅ Correctly identifies content mismatch — "contains PAC/Controllership data, not investment compliance data"
- **Inline Q3 card**: ❌ Still generic
- **Attachments table**: ❌ Still no indicator
- **Quality gate**: ❌ Still doesn't block

**CLARA can**: Read file content, validate against test procedure, detect name mismatches, detect content mismatches, set confidence levels, generate flags. **All of this is invisible at the point of decision.**

**Fix (3 surfaces)**:
1. **Inline Q3 override** (M effort): When evidence flags exist, replace generic insight with evidence finding
2. **Quality gate check** (S effort): Block/warn on submission when evidence is invalid
3. **Attachment table indicator** (M effort): 🟢/🔴 badge per file

---

### P0-3: Recommendation Doesn't Adapt to Own Validation

**What the Full Analysis simultaneously shows**:
- Confidence: **Low** — "the only attachment is invalid"
- Suggested Response: **Yes**
- ⚠️ Flag: "Answer is 'Yes' but no valid evidence is attached. This contradicts the claim."

**The contradiction**: The system says "Yes" AND "this answer contradicts reality" in the same view.

**Fix**: Post-processing rule — IF Flag severity = ⚠️ AND Confidence = Low → override Suggested Response to "Yes (⚠️ CONDITIONAL)" with warning styling. Never present an unconditional "Yes" alongside a flag that says it's wrong.

**Effort**: S | **Type**: Backend post-processing

---

### P1-1: Insight vs Reasoning Conflation

**Full Analysis shows two distinct fields**:
- **Insight** (forward-looking): "Verify that the control design has not changed. The control is designed to detect investment guideline violations via Clearwater's automated compliance flags..."
- **Reasoning** (backward-looking): "The control's detective design has been affirmed as effective in all 5 prior reviews. No design changes appear in the changelog."

**Inline card shows**: The **Reasoning** text, labeled as "Insights."

**The problem**: Reasoning justifies the AI's choice. Insight guides the user's action. Users get justification without direction.

**Hypothesis**: Word limit (120-150 words) causes the system to pick the shorter text (Reasoning) over the longer, more valuable text (Insight).

**Fix**: Explicitly route the "Insight" field content to the inline card. If it exceeds word limit, truncate with "... [See Full Analysis]" — don't substitute Reasoning.

**Effort**: S-M | **Type**: Backend field routing

---

### P1-2: Frequency Discrepancy Not Flagged Inline

Control page has contradictory data:
- Control Frequency = **"Monthly"**
- "When is the control performed?" = **"The control is performed quarterly"**

Full Analysis correctly flags this: "⚠️ Flag: Control Frequency says 'Monthly' but 'When performed?' says 'quarterly.'"

Inline Q1 (where reviewer confirms details are "accurate"): **No mention of the discrepancy.**

**Fix**: When Full Analysis generates a ⚠️ flag for a question, inject condensed flag into the inline card: "⚠️ Inconsistency detected — see Full Analysis."

**Effort**: M | **Type**: Backend (flag propagation) + Frontend (display)

---

### P1-3: "ConfirmationField" Internal Field Name in User-Facing Text

Q1 inline insight reads: "The **ConfirmationField** requires the reviewer to attest that control details are **current**."

Two issues:
1. "ConfirmationField" is an internal schema name — meaningless to reviewers
2. Says "current" but the question asks for "accurate"

**Fix**: Update prompt template to inject user-facing question text (not field ID). Add post-processing: strip any CamelCase token not present in the question text.

**Effort**: S | **Type**: Prompt

---

### P1-4: Attestation Pre-Fill Risk (IPA Q2)

Recommendation pre-fills: "I reviewed the Clearwater Audit Report output and **confirmed** the investment holdings data is consistent with the general ledger..."

This reads as a **completed attestation**. One-click accept = signing a compliance statement the reviewer may not have actually performed.

**Fix**: Frame as conditional template: "**[After reviewing the Clearwater Audit Report]**: I reviewed the output and confirmed..." — making it clear the action hasn't been performed yet. Or add confirmation gate: "⚠️ This attests you personally performed this verification."

**Effort**: M | **Type**: Prompt + Frontend

---

### P2 Issues (Lower Priority)

| # | Issue | Fix | Effort |
|---|-------|-----|--------|
| P2-1 | Historical "No" mentioned in text but not visually flagged at Q6 | Inject ⚠️ prefix when prior "No" exists within 4 quarters | S |
| P2-2 | Executive Summary leads with "13 questions blank" (obvious) | Restructure prompt to lead with actions/flags | S |
| P2-3 | Q8 Carnaval "Yes" from history only, no GRC-Next verification | Integrate monitoring API result into recommendation | M |

---

## Proposed Executive Summary Rewrite (P2-2 Demonstration)

### Current (Today):
> "This draft review has 13 questions, of which 10 are blank, 1 is confirmed, 1 answered "Yes," and 1 is an empty array. The single attachment ("Clearwater Audit Report") is invalid — its extracted content shows a PAC/Controllership Contact spreadsheet..."

### Proposed (After Fix):

> ### ⚠️ 2 Blocking Issues — Resolve Before Submission
>
> **1. Invalid evidence attached.** The file named "Clearwater Audit Report" contains PAC/Controllership data — not the required investment compliance audit showing guideline violations and resolutions. Remove it and attach the actual Clearwater Audit Report.
>
> **2. Frequency inconsistency on control page.** Control Frequency = "Monthly" but description says "performed quarterly." Clarify with control owner and update before confirming Q1.
>
> ---
> ### Context
> - **Key control** — L8+ approval required
> - **New reviewer** — prior reviews by tiawu and sxholi
> - **Historical note** — Q4 2022 Operating Effectiveness was "No" (resolved)
> - **Required evidence** — Clearwater Audit Report for current quarter

### Why It's Better

| Dimension | Current | Proposed |
|-----------|---------|----------|
| First thing read | "13 questions, 10 blank" (obvious) | "⚠️ 2 Blocking Issues" (actionable) |
| Tells reviewer what to DO | Buried at end | Numbered steps at top |
| Scannable | Dense paragraph | Header + list — 10 seconds |
| Jargon | "empty array" | None |
| Redundancy | Critical Context repeats summary | No repetition |

---

## Systemic Finding: Per-Question Recommendations Missing for 83% of Reviews

| Control Type | Reviews | Has Per-Question CLARA Recs? |
|-------------|---------|------------------------------|
| IT General Control | 1 | ✅ Yes (13 recs) |
| Application Control | 2 | ✅ 1 of 2 has recs |
| IT Dependent Manual | 7 | ❌ None |
| Manual Control | 2 | ❌ None |

**83% of reviews** (10/12) only get the Insights panel summary with zero per-question guidance. The core value proposition (pre-fill saving 15-20 min per review) is not being delivered for IT Dependent Manual and Manual Controls.

**Question for engineering**: Is this intentional pilot scoping or a bug?

---

## Positive Confirmation: AI Governance Works

Review 407615 was submitted on 6/2/2026. After submission:
- Zero CLARA content visible
- No "Insights" or "Generated by Clara" anywhere
- Only "Approve" and "Reject with comment" buttons (approver view)
- Human-authored responses only

**Epic-11 confirmed working.** State-based suppression correctly removes all AI content on state transition.

---

## Recommended Priority for Sprint Planning

| Sprint | Tickets | Impact |
|--------|---------|--------|
| **This sprint** | P0-1 + P0-3: Recommendation adapts to evidence state (S+S effort) | Prevents false attestation submissions |
| **This sprint** | P0-2 (Fix B): Quality gate blocks on invalid evidence (S effort) | Prevents rejections from invalid evidence |
| **Next sprint** | P0-2 (Fix A): Evidence findings surfaced inline (M effort) | Guidance at point of decision |
| **Next sprint** | P1-1: Route Insight (not Reasoning) to inline cards (S-M effort) | Correct content in correct place |
| **Next sprint** | P1-2: Propagate Full Analysis flags to inline (M effort) | Critical flags visible without clicking modal |
| **Backlog** | P1-3, P1-4, P2s | Quality improvements |

---

## Test Methodology

- **Automated browser QA** using Playwright MCP — navigated pages, extracted CLARA content, validated cross-question consistency
- **Manual evidence scenarios** — uploaded invalid files to test CLARA's detection capabilities
- **Cross-review scanning** — checked 12 reviews for systemic patterns
- **Three-layer analysis framework** — each finding evaluated through Reasoning (why it matters) → Summarization (ticket-ready card) → Remediation (root cause + fix proposal)

---

## Full Reports (on GitHub)

All detailed reports with three-layer analysis, screenshots, and extracted data:
https://github.com/hotasarthak88/Product-Manager-Builder-Agent/tree/main/docs/qa-reports

- `clara-407759-three-layer-analysis.md` — primary deep-dive (9 findings)
- `clara-407759-addendum-evidence-validation.md` — evidence scenarios (2 P0 findings)
- `clara-systemic-findings-2026-06-03.md` — cross-review systemic analysis
- `clara-review-owner-test-cases.md` — 49 test cases for Review Owner persona
- `clara-reviewer-test-cases.md` — 35 test cases for Approver persona

---

*Questions? Reach out to sarthah. All findings reproducible on gamma.grc.a2z.com.*
