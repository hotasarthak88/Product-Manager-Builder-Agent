# CLARA Three-Layer QA Analysis — Review 407759

**Review**: #407759 — Review of investment holdings for compliance (Control 165)
**Control Type**: Application Control | **Frequency**: Monthly | **Performer**: Treasury's Investment team
**Evidence Required**: Clearwater Audit Report (sourced from State Street)
**Environment**: Gamma | **Date**: June 3, 2026

---

## Control Context (Reference for Analysis)

| Property | Value | Source |
|----------|-------|--------|
| Control Type | Application Control | Control page |
| Control Frequency | Monthly | Control page |
| "When is the control performed?" | "The control is performed quarterly. The portfolios are checked for compliance daily." | Control page |
| Reports Used | Clearwater Audit Report | Control page |
| Additional Info | Source = State Street (Amazon's global security custodian) | Control page |
| Review Attachments | 0 on review; 2 on control page | Attachment tables |
| Evidence Matrix | "No evidence found for this Test Procedure" | Evidence Matrix |
| Historical "No" | Q4 2022 Operating Effectiveness = No (resolved) | CLARA Q6 insight |

---

## Finding 1: "ConfirmationField" Jargon + Incorrect Framing

**Heuristics Triggered**: #2 (Jargon Leak), #1 (Content Consistency), #3 (Insight vs Reasoning)

**CLARA Output (Q1 inline Insight)**:
> "The ConfirmationField requires the reviewer to attest that control details are current. The control document shows the performer, frequency, reports, and objective — the reviewer must verify these reflect reality."

**Actual Question Text**:
> "Confirm the control details are **accurate** and up to date for the review period, including the geographies in which the control operates."

### Layer 1: Reasoning

**User Impact**: Reviewer sees an unfamiliar system term ("ConfirmationField") and incorrect framing ("current" vs "accurate"). This creates confusion — they may wonder if they're looking at the right question.

**Trust Signal**: CRITICAL. If the AI can't even correctly paraphrase the question it's helping with, why would the reviewer trust its answer recommendations? This is the FIRST CLARA interaction the reviewer sees — it sets the tone.

**Risk**: Low direct risk (the recommendation "Confirmed" is correct), but the incorrect framing ("current" vs "accurate") could lead a reviewer to only check if data is recent rather than if it's factually correct.

**Adoption Threat**: HIGH for first impressions. A jargon leak in the very first CLARA interaction signals "this AI doesn't understand what I'm doing."

### Layer 2: Issue Summary

## Issue: Internal field name "ConfirmationField" and incorrect framing ("current" vs "accurate") in Q1 Insight

| Field | Value |
|-------|-------|
| Severity | P1 |
| Category | Content Accuracy + Jargon Leak |
| Scope | Systemic (all reviews with per-question recs — confirmed in 407759, 407615) |
| Reproducibility | Always |
| Affected Persona | Reviewer |
| Affected Reviews | All reviews with CLARA per-question recommendations |
| Control Types Affected | All (that have per-question recs enabled) |

**What's happening**: The inline insight for Q1 (Confirm control details) uses the internal data model field name "ConfirmationField" and incorrectly states the field requires attestation that details are "current" — the actual question asks for "accurate."

**Expected behavior**: Insight should use user-facing language matching the question text, and correctly reflect the attestation requirement (accuracy, not just currency).

**Actual behavior**: Internal field name leaked; framing mismatches the question.

**Evidence**: Extracted inline text: `"The ConfirmationField requires the reviewer to attest that control details are current."`

### Layer 3: Remediation

**Root Cause**: The prompt template for generating per-question insights references the GRC data model field identifier (`ConfirmationField`) instead of the user-facing question text. The model also paraphrases the attestation requirement incorrectly.

**Proposed Fix**: 
1. Update the prompt template to inject the full user-facing question text (not the field ID) as context
2. Add a post-processing check: if any CamelCase token in the output doesn't appear in the question text, strip or replace it
3. Explicitly instruct the model: "Use the exact terminology from the question text when describing the requirement"

**Effort**: S (prompt template fix + post-processing regex)

**Fix Type**: Prompt + Backend logic

**Validation**: Re-generate Q1 insight for 407759; verify no CamelCase field names appear; verify "accurate" is used instead of "current"

**Trade-offs**: Stricter prompt templates may make insights slightly more verbose. The CamelCase regex could accidentally strip legitimate proper nouns (mitigate with an allowlist).

---

## Finding 2: Frequency Discrepancy Not Flagged at Q1 or Q2 (Only in Full Analysis)

**Heuristics Triggered**: #4 (Information at Point of Decision), #1 (Content Consistency), #7 (Word Limit Artifacts)

**The Discrepancy**:
- Control Frequency field = "**Monthly**"
- "When is the control performed?" field = "The control is performed **quarterly**. The portfolios are checked for compliance daily."

The Full Analysis correctly identifies this:
> "⚠️ Flag: The Control Frequency field says 'Monthly' but the 'When is the control performed?' field says 'The control is performed quarterly.'"

**But the inline Insights for Q1 and Q2 do NOT mention this discrepancy.**

### Layer 1: Reasoning

**User Impact**: The reviewer answering Q1 ("Confirm control details are accurate") doesn't see the frequency discrepancy inline. They may confirm "accurate" without noticing the Monthly vs Quarterly contradiction — because the flag is hidden in a modal they need to separately click.

**Trust Signal**: Moderate. If the reviewer later opens Full Analysis and sees the flag, they'll wonder "Why didn't CLARA tell me this when I was answering the question?"

**Risk**: HIGH. The reviewer could attest that control details are "accurate" while an internal inconsistency exists on the control page. This could surface during audit as a documentation error.

**Adoption Threat**: Moderate. The Full Analysis has the right answer — but most users won't click it if the inline seems "good enough."

### Layer 2: Issue Summary

## Issue: Critical frequency discrepancy (Monthly vs Quarterly) only visible in Full Analysis, not at point of decision (Q1/Q2)

| Field | Value |
|-------|-------|
| Severity | P1 |
| Category | Information Hierarchy + Progressive Disclosure Failure |
| Scope | This review (407759); likely systemic wherever control page has internal contradictions |
| Reproducibility | Always (discrepancy exists on control page) |
| Affected Persona | Reviewer |
| Affected Reviews | 407759; any review with control page inconsistencies |
| Control Types Affected | All |

**What's happening**: The Full Analysis correctly identifies that Control Frequency = "Monthly" contradicts the "When performed" field which says "quarterly." However, this flag is NOT surfaced in the inline Insight for Q1 (where the reviewer attests accuracy) or Q2 (where they confirm frequency).

**Expected behavior**: BRD Epic-1 specifies variance/inconsistency detection should be presented to the reviewer. Flags should appear at the point of decision.

**Actual behavior**: Flag only in Full Analysis modal. Inline Q1 Insight talks about "ConfirmationField" generically. Inline Q2 Insight says "All 5 prior reviews answered Yes" without mentioning the discrepancy.

**Evidence**: Control Frequency = "Monthly"; When performed = "quarterly"; Full Analysis screenshot shows flag; inline Q1/Q2 insights don't mention it.

### Layer 3: Remediation

**Root Cause**: The Full Analysis pipeline has access to field-by-field comparison logic and generates flags. The inline Insight pipeline either (a) doesn't run the same comparison, or (b) drops the flag due to word limit constraints.

**Proposed Fix**:
1. **Primary**: When the Full Analysis identifies a ⚠️ flag for a question, inject a condensed version into the inline Insight as a prefix: `"⚠️ Control page inconsistency detected — see Full Analysis. [Original insight continues...]"`
2. **Alternative**: Surface flags as a separate inline element (not inside the Insight text) — a red/yellow banner between the question and the CLARA recommendation card

**Effort**: M (requires piping flag data from Full Analysis to inline rendering)

**Fix Type**: Backend (flag propagation) + Frontend (flag display element)

**Validation**: After fix, open 407759 Q1 — verify frequency discrepancy flag is visible inline without opening Full Analysis.

**Trade-offs**: May make the inline card visually busier for reviews with multiple flags. Mitigate by limiting to 1 flag per question inline and linking to Full Analysis for details.

---

## Finding 3: Insight vs Reasoning Conflation (Q5 — Design Effectiveness)

**Heuristics Triggered**: #3 (Insight vs Reasoning Conflation), #7 (Word Limit Artifacts), #1 (Content Consistency)

**Q5 Inline (labeled "Insights")**:
> "The control's detective design — automated compliance flagging plus manual investigation — has been affirmed as effective in all 5 prior reviews. No design changes appear in the changelog."

**Q5 Full Analysis Insight** (from earlier session):
> "Verify that the control design has not changed. The control is designed to detect investment guideline violations via Clearwater's automated compliance flags, with the Investment team investigating and resolving each flag. This detective design has been consistently affirmed as effective across all prior reviews. The changelog shows no design changes in the review window."

**Q5 Full Analysis Reasoning**:
> "The control's detective design — automated compliance flagging plus manual investigation — has been affirmed as effective in all 5 prior reviews. No design changes appear in the changelog."

**The inline "Insights" is showing the Reasoning text word-for-word, not the Insight.**

### Layer 1: Reasoning

**User Impact**: The reviewer gets a backward-looking justification ("has been affirmed as effective") instead of forward-looking guidance ("Verify that the control design has not changed"). They're told WHY the AI recommends "Yes" but not WHAT to check before agreeing.

**Trust Signal**: Low immediate impact (the content sounds reasonable). But over time, reviewers realize the "Insights" label doesn't deliver insights — it delivers justifications. They learn to ignore it.

**Risk**: Moderate. A reviewer who just reads "affirmed effective in all 5 prior reviews" may rubber-stamp "Yes" without independently verifying. The Insight (from Full Analysis) explicitly says "Verify" — it instructs action. The Reasoning just confirms the pattern.

**Adoption Threat**: HIGH over time. If "Insights" consistently delivers Reasoning instead of actionable guidance, users will treat the label as meaningless and stop reading it.

### Layer 2: Issue Summary

## Issue: Inline "Insights" field displays Reasoning text instead of actual Insight — conflates two distinct content types

| Field | Value |
|-------|-------|
| Severity | P1 |
| Category | Semantic Confusion + Constraint Artifacts |
| Scope | Systemic (confirmed on Q1, Q5; likely all questions) |
| Reproducibility | Always |
| Affected Persona | Reviewer |
| Affected Reviews | All with per-question recs |
| Control Types Affected | All |

**What's happening**: The Full Analysis generates two distinct fields — "Insight" (forward-looking guidance: what to verify) and "Reasoning" (backward-looking justification: why this answer). The inline Questionnaire card shows the Reasoning text labeled as "Insights."

**Expected behavior**: Inline should show the Insight (actionable guidance) since that's what the label promises.

**Actual behavior**: Inline shows Reasoning (justification). The actual Insight is only available via Full Analysis modal.

**Evidence**: Q5 inline = "The control's detective design... has been affirmed as effective..." (Reasoning). Q5 Full Analysis Insight = "Verify that the control design has not changed..." (Guidance).

### Layer 3: Remediation

**Root Cause**: Likely a word-limit constraint (120-150 words) in the inline rendering pipeline. The Reasoning tends to be shorter/punchier than the Insight. When the system must choose which to display inline, it picks the shorter one — which happens to be Reasoning.

**Proposed Fix**:
1. **Primary**: Change the inline field source to explicitly pull from the "Insight" content, not "Reasoning." If the Insight exceeds the word limit, truncate it with "... [See Full Analysis]" rather than substituting Reasoning.
2. **Alternative**: Remove the word limit entirely and show the full Insight. The CLARA card can expand/collapse if too long.
3. **Minimum viable**: If both fields must coexist in limited space, show Insight first (1-2 sentences) and Reasoning second (collapsed by default).

**Effort**: S-M (field source mapping change; potentially prompt restructuring)

**Fix Type**: Backend (field routing) or Prompt (explicit instruction to generate Insight and Reasoning as separate tagged outputs)

**Validation**: After fix, compare Q5 inline text with Full Analysis Insight text — they should be the same or the inline should be a subset of the Insight (not Reasoning).

**Trade-offs**: If the Insight is longer than the current word limit, the inline card becomes taller. Mitigate with expand/collapse or "Read more" link.

---

## Finding 4: Q2 Recommends "Yes" for Evidence Availability But 0 Attachments Exist

**Heuristics Triggered**: #8 (Cross-Question Consistency), #4 (Information at Point of Decision)

**Q3 — "Is the evidence used to support this control available and retained?"**
- CLARA recommendation: **Yes**
- Insight: "Prior reviews consistently answered 'Yes.' However, the answer is only valid if the Clearwater Audit Report for the current quarter is available and retained. The reviewer must confirm access to this evidence."

**Actual state**: 
- Review attachments: **0**
- Evidence Matrix: "No evidence found for this Test Procedure"
- Executive Summary explicitly states: "No attachments are present... the Clearwater Audit Report is required evidence"

### Layer 1: Reasoning

**User Impact**: CLARA recommends "Yes" for evidence availability while the page visibly shows ZERO evidence attached. The reviewer who trusts CLARA's recommendation without checking would answer "Yes" to a factually false statement.

**Trust Signal**: CRITICAL. This is a direct contradiction between the recommendation and observable reality on the page. If a reviewer notices this, CLARA's credibility is severely damaged.

**Risk**: HIGH. Answering "Yes" to evidence availability when no evidence exists is the #1 rejection reason (per BRD: "10-15% of rejections caused by missing or broken evidence"). The approver will immediately reject.

**Adoption Threat**: HIGH. If CLARA recommends "Yes" for something visibly false, users will stop trusting ANY recommendation.

### Layer 2: Issue Summary

## Issue: CLARA recommends "Yes" for evidence availability (Q3) when zero attachments exist on the review

| Field | Value |
|-------|-------|
| Severity | P0 |
| Category | Cross-Question Consistency + Content Accuracy |
| Scope | This review; likely systemic for all blank reviews with historical "Yes" patterns |
| Reproducibility | Always (for reviews with 0 attachments and prior "Yes" history) |
| Affected Persona | Reviewer |
| Affected Reviews | 407759; likely all reviews with 0 attachments |
| Control Types Affected | All |

**What's happening**: CLARA recommends "Yes" for evidence availability based solely on historical pattern ("prior reviews answered Yes") without checking the current review's actual attachment state. The review has 0 attachments and the Evidence Matrix explicitly shows "No evidence found."

**Expected behavior**: BRD Epic-4 (Evidence Validation) specifies CLARA should "validate that all required evidence links and files are provided" and flag gaps. The recommendation should be conditional: "Yes — but only after attaching the Clearwater Audit Report."

**Actual behavior**: Unconditional "Yes" recommendation. The Insight does add a caveat ("answer is only valid if... available and retained"), but the top-line recommendation is still "Yes."

**Evidence**: CLARA rec = "Yes"; Attachment count = 0; Evidence Matrix = "No evidence found for this Test Procedure"

### Layer 3: Remediation

**Root Cause**: The recommendation is generated from historical pattern analysis (prior answers) WITHOUT cross-referencing the current review's attachment state. The model has access to "all 5 prior reviews answered Yes" but does not check "current review attachments = 0."

**Proposed Fix**:
1. **Primary**: Add a pre-recommendation check: If Q3 is about evidence availability AND current review attachments = 0 AND evidence matrix shows gaps → override recommendation to "Yes — contingent on attaching [specific evidence name]" or change to "Cannot confirm yet — evidence not attached"
2. **Add a blocking flag**: Display red warning: "⚠️ You cannot truthfully answer 'Yes' until evidence is attached. [Upload Evidence]"
3. **Minimum viable**: Change the recommendation format to "Yes (conditional)" with the condition displayed prominently

**Effort**: M (requires cross-referencing attachment state with recommendation generation)

**Fix Type**: Backend logic (current-state validation before recommendation)

**Validation**: Open 407759 after fix; verify Q3 recommendation reflects the 0-attachment reality.

**Trade-offs**: May increase false negatives (recommending caution when evidence exists but isn't attached to this specific review). Mitigate by also checking the control page attachments.

---

## Finding 5: Q6 (Operating Effectiveness) References Historical "No" But Doesn't Flag It Prominently

**Heuristics Triggered**: #4 (Information at Point of Decision), #10 (Historical Context Accuracy)

**Q6 Inline Insight**:
> "The most recent reviews (Q2 2023, Q1 2023) answered 'Yes.' The Q4 2022 'No' was a resolved issue. The reviewer must confirm current-quarter effectiveness independently."

**CLARA Recommendation**: Yes

### Layer 1: Reasoning

**User Impact**: The insight mentions the Q4 2022 "No" in passing — buried in the second sentence. But this is material: this control has a HISTORY of effectiveness failure. The reviewer should be explicitly alerted that they're answering "Yes" for a control that previously failed.

**Trust Signal**: Actually moderate-positive — CLARA does mention the historical "No." But the presentation doesn't give it the weight it deserves.

**Risk**: Moderate. The insight says "was a resolved issue" which implies it's fine now — but CLARA cannot independently verify resolution. The reviewer might take "resolved" at face value.

**Adoption Threat**: Low — this is a quality issue, not a trust-breaking one.

### Layer 2: Issue Summary

## Issue: Historical effectiveness failure (Q4 2022 "No") mentioned in passing rather than prominently flagged at Q6

| Field | Value |
|-------|-------|
| Severity | P2 |
| Category | Information Hierarchy |
| Scope | This review; any control with prior effectiveness failures |
| Reproducibility | Always (when historical "No" exists) |
| Affected Persona | Reviewer |
| Affected Reviews | 407759; any review with Q4 2022 or prior "No" for effectiveness |
| Control Types Affected | All |

**What's happening**: CLARA's inline insight mentions the prior "No" answer but treats it as resolved context rather than an active alert. The recommendation is "Yes" with the "No" noted in supporting text.

**Expected behavior**: A prior effectiveness failure should produce a visual flag (⚠️) or different recommendation format: "Yes (verify resolution of prior Q4 2022 failure first)"

**Actual behavior**: Plain text mention with "was a resolved issue" editorial judgment.

**Evidence**: Inline Q6 insight text references "Q4 2022 'No' was a resolved issue"

### Layer 3: Remediation

**Root Cause**: The model treats historical "No" answers as informational context rather than risk flags that need elevated presentation.

**Proposed Fix**: When any question has a historical "No" answer in the prior 4 quarters, add a visual flag prefix to the inline insight: "⚠️ PRIOR FAILURE: This question was answered 'No' in Q4 2022. Verify the issue is fully resolved before confirming 'Yes.'" Display as a distinct warning element, not inline text.

**Effort**: S (conditional flag injection based on historical pattern)

**Fix Type**: Backend (flag generation) + Frontend (flag display)

**Validation**: Open 407759 Q6; verify visual warning about Q4 2022 "No" appears prominently before/above the recommendation.

**Trade-offs**: May create visual noise for controls with old resolved issues. Mitigate by only flagging "No" answers within the last 4 quarters (1 year).

---

## Finding 6: Q8 (Carnaval Alarms) Recommends "Yes" — But Is This Correct for Application Controls?

**Heuristics Triggered**: #9 (Control-Type Awareness), #10 (Historical Context Accuracy)

**Q8 Inline Insight**:
> "All 5 prior reviews confirmed alarm registration. The reviewer should verify current state in GRC-Next before confirming."

**CLARA Recommendation**: Yes

**Context**: The control type is "Application Control." In Review 407615 (IT General Control), the same Carnaval question recommended **N/A** with the reasoning: "The control does not use Carnaval." 

**Question**: Does this Application Control actually use Carnaval alarms? Or should this be N/A like the IT General Control?

### Layer 1: Reasoning

**User Impact**: If prior reviewers incorrectly answered "Yes" for 5 quarters (perhaps before the N/A option was available, or through misunderstanding), CLARA perpetuates the error by pattern-matching on history.

**Trust Signal**: Moderate risk. CLARA is following historical pattern — which is its design. But if the historical pattern is WRONG, CLARA amplifies the error.

**Risk**: Moderate. If this control doesn't actually use Carnaval, answering "Yes" is incorrect. However, it's also possible this Application Control genuinely has Carnaval alarms (different from the IT General Control in 407615).

**Adoption Threat**: Low — this requires domain expertise to detect.

### Layer 2: Issue Summary

## Issue: CLARA recommends "Yes" for Carnaval alarm registration based purely on history — no current-state verification

| Field | Value |
|-------|-------|
| Severity | P2 |
| Category | Historical Context Accuracy + Control-Type Awareness |
| Scope | Isolated to this review (requires domain knowledge to confirm) |
| Reproducibility | Whenever historical pattern exists without current-state check |
| Affected Persona | Reviewer |
| Affected Reviews | 407759 |
| Control Types Affected | Application Control |

**What's happening**: CLARA recommends "Yes" for Carnaval alarm registration because all 5 prior reviews said "Yes." However, it does not independently verify whether Carnaval alarms actually exist for this control/application in GRC-Next.

**Expected behavior**: BRD Epic-2 specifies automated monitoring registration verification via GRC-Next APIs. The recommendation should be validated against current state, not just history.

**Actual behavior**: Pure pattern matching on historical answers without current-state API check.

**Evidence**: Recommendation = "Yes"; Insight = "All 5 prior reviews confirmed"; No evidence of GRC-Next API verification.

### Layer 3: Remediation

**Root Cause**: Per-question recommendations appear to be generated from historical analysis only, without the real-time monitoring verification that Epic-2 specifies. The monitoring check (GRC-Next API query) may only run as a standalone feature, not as input to recommendation generation.

**Proposed Fix**: Feed the monitoring registration check result (from Epic-2) into the recommendation generation for Q8. If GRC-Next shows alarms exist → "Yes" is appropriate. If GRC-Next shows no Carnaval alarms → recommend "N/A" regardless of history.

**Effort**: M (requires integration between monitoring check and recommendation pipeline)

**Fix Type**: Backend (pipeline integration)

**Validation**: Verify Q8 recommendation matches actual GRC-Next monitoring state, not just history.

**Trade-offs**: Requires GRC-Next API to be available at recommendation generation time. If API is down, fall back to historical pattern with disclaimer.

---

## Finding 7: Executive Summary Leads with Obvious Statement

**Heuristics Triggered**: #6 (Obvious Statement Waste), #5 (Template Content)

**Executive Summary Opening**:
> "This is a fresh draft review with all 13 questions blank."

### Layer 1: Reasoning

**User Impact**: The reviewer just opened a blank review — they can SEE it's blank. The first sentence tells them what they already know.

**Trust Signal**: Minor — doesn't break trust, but doesn't build it either. Feels robotic/templated.

**Risk**: None directly. But the real estate wasted here could be used for the MOST important fact (e.g., "Attach the Clearwater Audit Report before answering any questions" or "This control had a prior effectiveness failure in Q4 2022").

**Adoption Threat**: Moderate over time. If every summary starts the same way, users learn to skip the first sentence — which means they might also skip the valuable content that follows.

### Layer 2: Issue Summary

## Issue: Executive Summary opens with obvious "all questions blank" statement instead of most actionable information

| Field | Value |
|-------|-------|
| Severity | P2 |
| Category | Obvious Statement Waste + Information Hierarchy |
| Scope | Systemic (confirmed across 10/12 reviews) |
| Reproducibility | Always (for draft reviews) |
| Affected Persona | Reviewer |
| Affected Reviews | All draft reviews |
| Control Types Affected | All |

**What's happening**: Every draft review's Executive Summary begins with "This is a fresh draft review with all 13 questions blank" — information visible from the page itself.

**Expected behavior**: Lead with the most actionable or surprising insight — something the reviewer DOESN'T already know.

**Actual behavior**: Leads with obvious state, burying actionable content in subsequent sentences.

### Layer 3: Remediation

**Root Cause**: The summarization prompt likely lists "current state" as the first output section. The model dutifully reports it first.

**Proposed Fix**: Restructure the prompt to prioritize: (1) Flags/risks, (2) Required actions before submission, (3) Key context from history, (4) Current state (only if non-obvious). Add instruction: "Do not state information that is visually apparent on the page."

**Effort**: S (prompt restructuring)

**Fix Type**: Prompt

**Validation**: Re-generate summary for 407759; verify first sentence is actionable (e.g., "Attach the Clearwater Audit Report — it's required evidence and currently missing.").

**Trade-offs**: None significant. The obvious information can be moved to the end or dropped entirely.

---

## Finding 8: IPA Q1 (Q10) Recommendation Conflicts with N/A State Shown on Review 407615

**Heuristics Triggered**: #9 (Control-Type Awareness)

**Q10 (IPA Q1) Recommendation**:
> "Clearwater Audit Report sourced by Clearwater (compliance monitoring system). The data originates from State Street, Amazon's global security custodian."

**Q10 Insight**:
> "The 'Reports used in this control?' field explicitly names 'Clearwater Audit Report.' The data source is State Street per the Additional Information field."

**Context**: This is an **Application Control**. In Review 407615 (IT General Control), IPA questions all got N/A because "IT General Controls don't use IPA." But 407759's control type is "Application Control" — and it DOES use IPA (the Clearwater Audit Report). So this recommendation is **correct**.

### Layer 1: Reasoning

**User Impact**: This is actually a POSITIVE finding. CLARA correctly distinguishes that an Application Control with a named report SHOULD document IPA, unlike an IT General Control. The recommendation is appropriate.

**Trust Signal**: Positive. Shows control-type awareness working correctly for IPA.

**Risk**: None — recommendation is factually correct and control-type-appropriate.

**Adoption Threat**: None.

### Layer 2: Issue Summary

**No issue — POSITIVE VALIDATION.** CLARA correctly identifies that this Application Control uses IPA (Clearwater Audit Report from State Street) and provides the correct recommendation for IPA Q1. Control-type awareness is functioning for the IPA section on this review.

---

## Finding 9: IPA Q2 (Q11) Recommends G/L Agreement That May Not Be Reviewer's Actual Workflow

**Heuristics Triggered**: #3 (Insight vs Reasoning)

**Q11 Recommendation**:
> "I reviewed the Clearwater Audit Report output and confirmed the investment holdings data is consistent with the general ledger investment account balances."

**Q11 Insight**:
> "The control checks investment holdings for compliance — the underlying holdings data should tie to G/L investment accounts. However, the reviewer must verify this is actually performed."

### Layer 1: Reasoning

**User Impact**: The recommendation pre-fills a specific attestation: "I reviewed... and confirmed... consistent with the general ledger." This is a STRONG claim. If the reviewer accepts this without actually performing the reconciliation, they're making a false attestation.

**Trust Signal**: Moderate concern. The Insight correctly adds "the reviewer must verify this is actually performed" — but the recommendation itself reads as a completed attestation, not a suggestion.

**Risk**: MODERATE-HIGH. If a reviewer one-click accepts this recommendation without actually reviewing the Clearwater Audit Report against G/L balances, they've signed a false compliance attestation. This could surface during audit.

**Adoption Threat**: None (this is a correctness/compliance issue, not UX).

### Layer 2: Issue Summary

## Issue: IPA Q2 recommendation pre-fills a compliance attestation that the reviewer may not have actually performed

| Field | Value |
|-------|-------|
| Severity | P1 |
| Category | Content Accuracy + Compliance/Audit Risk |
| Scope | This review; any review where IPA has G/L reconciliation requirements |
| Reproducibility | Always (for controls with IPA reconciliation) |
| Affected Persona | Reviewer |
| Affected Reviews | 407759; Application Controls with IPA |
| Control Types Affected | Application Control, IT Dependent Manual |

**What's happening**: CLARA pre-fills text that reads as a completed attestation ("I reviewed... and confirmed...") rather than a template to be verified. The one-click accept flow means a reviewer could submit this attestation without actually performing the G/L reconciliation.

**Expected behavior**: Recommendations for attestation-type questions should be framed as templates: "After reviewing the Clearwater Audit Report, confirm: [investment holdings data is consistent with G/L balances]" — making it clear the action hasn't been performed yet.

**Actual behavior**: Pre-filled as if the action is already complete.

### Layer 3: Remediation

**Root Cause**: The model generates attestation language in past tense ("I reviewed... confirmed") because that's how prior approved responses were written. It's pattern-matching completed responses, not generating guidance for an uncompleted action.

**Proposed Fix**: For questions that require personal attestation (contain "I confirm," "I reviewed," "I verified"), frame recommendations as conditional templates:
- Instead of: "I reviewed the report and confirmed..."
- Use: "**[After reviewing the Clearwater Audit Report]**: I reviewed the output and confirmed the investment holdings data is consistent with G/L balances."
- Or add a mandatory confirmation: "⚠️ This response attests that you personally performed this verification. Accept only after completing the review."

**Effort**: M (attestation detection + template reformatting)

**Fix Type**: Prompt (instruction to frame attestations as conditional) + Frontend (confirmation gate before accepting attestation-type recommendations)

**Validation**: Verify Q11 recommendation clearly indicates it's a template requiring action, not a completed statement.

**Trade-offs**: Adds friction to the one-click accept flow. But this friction is appropriate for compliance attestations — it prevents false attestations.

---

## Findings Summary

| # | Finding | Severity | Heuristics | Category |
|---|---------|----------|-----------|----------|
| 1 | "ConfirmationField" jargon + "current" vs "accurate" | P1 | #2, #1, #3 | Content Accuracy |
| 2 | Frequency discrepancy (Monthly/Quarterly) not flagged inline | P1 | #4, #1, #7 | Information Hierarchy |
| 3 | Insight vs Reasoning conflation — Reasoning shown as "Insights" | P1 | #3, #7, #1 | Semantic Confusion |
| 4 | **Q3 recommends "Yes" for evidence when 0 attachments exist** | **P0** | #8, #4 | Cross-Question Consistency |
| 5 | Q6 historical "No" mentioned but not prominently flagged | P2 | #4, #10 | Information Hierarchy |
| 6 | Q8 (Carnaval) "Yes" based on history without current-state check | P2 | #9, #10 | Control-Type Awareness |
| 7 | Executive Summary leads with obvious "all blank" statement | P2 | #6, #5 | Template Content |
| 8 | IPA Q1 correctly handles Application Control ✅ | N/A | #9 | Positive Validation |
| 9 | IPA Q2 pre-fills completed attestation language | P1 | #3 | Compliance Risk |

**Total: 8 issues found + 1 positive validation**
- P0: 1 (evidence recommendation contradicts page state)
- P1: 4 (jargon, frequency flag missing, insight/reasoning conflation, attestation framing)
- P2: 3 (historical flag presentation, Carnaval history-only, template summary)

---

## Engineering Action Items (Prioritized)

| Priority | Ticket | Fix Type | Effort |
|----------|--------|----------|--------|
| P0 | CLARA recommends "Yes" for evidence availability despite 0 attachments — cross-reference current state | Backend | M |
| P1 | Inline displays Reasoning text labeled as "Insights" — route actual Insight content to inline | Backend/Prompt | S-M |
| P1 | Flags from Full Analysis (frequency discrepancy) not propagated to inline questions | Backend + Frontend | M |
| P1 | IPA attestation recommendation pre-fills completed statement — add conditional framing | Prompt + Frontend | M |
| P1 | "ConfirmationField" jargon in Q1 inline + "current" vs "accurate" mismatch | Prompt | S |
| P2 | Historical "No" answers need visual flag at question level, not just text mention | Backend + Frontend | S |
| P2 | Executive Summary leads with obvious state — restructure to lead with actionable insight | Prompt | S |
| P2 | Q8 Carnaval recommendation based on history only, no GRC-Next API verification | Backend (integration) | M |

---

*Analysis performed by pm-builder-agent using Three-Layer QA Framework*
*10 heuristics applied across 13 CLARA responses*
*Date: June 3, 2026*
