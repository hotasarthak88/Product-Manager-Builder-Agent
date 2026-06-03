# CLARA Three-Layer QA Analysis — Addendum: Evidence Validation Intelligence

**Review**: #407759 — Review of investment holdings for compliance (Control 165)
**Addendum to**: `clara-407759-three-layer-analysis.md`
**Date**: June 3, 2026
**Test Scenarios**: Manual QA — evidence upload with invalid/mismatched files

---

## Test Scenarios Performed

| Test | Action | CLARA Response (Full Analysis) | CLARA Response (Inline/Attachments) |
|------|--------|-------------------------------|-------------------------------------|
| A | Attached random file named "Random Evidence Test" with unrelated description | ✅ Flagged as invalid. Detected name/description don't relate to control. Priority Action: "Replace the invalid attachment." | ❌ NOTHING. No flag in Q3 inline, no indicator in attachments table. |
| B | Renamed file to "Clearwater Audit Report" (correct name, wrong content) | ✅ Detected content is PAC/Controllership data, NOT investment guideline compliance data. Confidence: Low. Flag: "Answer is 'Yes' but no valid evidence." | ❌ NOTHING. Inline Q3 unchanged. Attachment table shows file without validity indicator. |

---

## Finding 10: Evidence Validation Intelligence Trapped in Full Analysis

**Heuristics Triggered**: #4 (Information at Point of Decision), #1 (Content Consistency), Progressive Disclosure Failure

### What CLARA Can Do (Proven by Tests)

CLARA demonstrates sophisticated evidence analysis capabilities:
1. ✅ **Name/description matching** — detects "Random Evidence Test" doesn't relate to this control
2. ✅ **Content-level inspection** — reads INSIDE the file, identifies content as PAC/Controllership data (not Clearwater compliance data) even when the filename says "Clearwater Audit Report"
3. ✅ **Confidence calibration** — correctly sets confidence to "Low" when evidence is invalid
4. ✅ **Flag generation** — creates ⚠️ Flag: "Answer is 'Yes' but no valid evidence is attached"
5. ✅ **Actionable guidance** — "Remove the invalid attachment and replace it with the correct report"
6. ✅ **Priority Action Items** — generates numbered action list in Executive Summary

### What CLARA Fails To Do (The Architecture Problem)

All of this intelligence is confined to the Full Analysis modal. Three surfaces that SHOULD show it are completely silent:

| Surface | Should Show | Currently Shows |
|---------|------------|-----------------|
| **Q3 Inline CLARA Card** | "⚠️ Evidence invalid — replace before answering Yes" | Generic: "Prior reviews consistently answered Yes..." |
| **Files & Attachments Table** | 🔴 validity indicator on the file row | File name, description, size — no validity signal |
| **Pre-Submit Quality Gate** | Block or warn: "Invalid evidence detected" | Nothing — submit button logic doesn't check evidence validity |

### Layer 1: Reasoning

**User Impact**: The reviewer attaches a file, sees it in "Files & external link (1)", answers Q3 "Yes" trusting that evidence is present, and submits. At NO point does the inline experience warn them their evidence is invalid. The only way to discover this is clicking "View Full Analysis" — which most users won't do if the inline card seems fine.

**Trust Signal**: INVERSE — this is a trust-BUILDING opportunity being wasted. If CLARA surfaced "I read your file and it's not the right evidence" inline, reviewers would be impressed. Currently they'll never know this capability exists until an approver rejects them.

**Risk**: **CRITICAL for compliance.** A reviewer can submit with:
- Invalid evidence attached ✓ (looks fine in the UI)
- Q3 = "Yes, evidence is available" ✓ (CLARA recommended it)
- Pre-submit quality gate: doesn't catch it
- Result: Submitted → approver rejects → 60-90 min rework cycle

This is exactly the rejection scenario the BRD is designed to prevent (Epic-4: Evidence Validation, Epic-5: reducing 35-40% rejection rate).

**Adoption Threat**: MISSED OPPORTUNITY. This is CLARA's most impressive capability (reading file content and detecting mismatches) but users will never see it unless they click into Full Analysis. If surfaced inline, this would be the "wow moment" that drives adoption.

### Layer 2: Issue Summary

## Issue: CLARA evidence validation intelligence (content analysis, name matching, validity flags) only surfaces in Full Analysis — invisible at point of decision

| Field | Value |
|-------|-------|
| Severity | **P0** |
| Category | Progressive Disclosure Failure + Information at Point of Decision |
| Scope | Systemic (architectural — applies to ALL reviews with attachments) |
| Reproducibility | Always (verified with both wrong-name and right-name-wrong-content scenarios) |
| Affected Persona | Reviewer (directly), Approver (receives invalid submissions) |
| Affected Reviews | All reviews where evidence is attached |
| Control Types Affected | All |

**What's happening**: CLARA performs deep evidence analysis (reads file content, validates against test procedure requirements, detects name/description mismatches, sets confidence levels, generates flags) and surfaces findings ONLY in the Full Analysis modal. The inline Questionnaire cards, Files & Attachments table, and pre-submit quality gate do NOT display evidence validity findings.

**Expected behavior** (per BRD Epic-4 — Evidence Validation):
> "CLARA validates that all required evidence items are attached or referenced before submission"
> "Displays clear error for missing evidence identifying category and location"
> "Provides actionable guidance ('Attach repository link,' 'Upload supporting document')"

The BRD expects evidence validation to be **proactive and visible** — not hidden in a modal.

**Actual behavior**: Evidence analysis exists and is highly sophisticated (content-level inspection that detects PAC data inside a file named "Clearwater Audit Report") but is architecturally confined to the Full Analysis modal.

**Evidence**: 
- Screenshot 1: Full Analysis showing "Replace the invalid attachment" as Priority Action Item #1
- Screenshot 2: Full Analysis Q2 showing Confidence: Low, Flag: "no valid evidence attached"
- Screenshot 3: Files & Attachments table showing "Random Evidence Test" with NO validity indicator
- Inline Q3 card: generic insight with no mention of invalid evidence

### Layer 3: Remediation

**Root Cause**: The evidence validation logic runs as part of the Full Analysis generation pipeline. Its outputs (Confidence level, Flags, Priority Actions) are stored and rendered ONLY within the Full Analysis response object. Three other rendering surfaces (Q3 inline card, attachment table component, quality gate logic) do not consume this output because they operate on separate data paths.

**Proposed Fix — Three Surfaces (Priority Order)**:

#### Fix A: Inline Q3 Card Override (Highest Priority)

When evidence validation detects issues, override the generic Q3 Insight with the evidence-specific finding:

```
✦ CLARA recommendation: Yes (⚠️ CONDITIONAL)

⚠️ Evidence Issue Detected:
Current attachment "Random Evidence Test" is not valid evidence 
for this control. The test procedure requires a "Validated audit 
report" from Clearwater.

Action: Remove invalid attachment → Upload actual Clearwater 
Audit Report → Then confirm "Yes"

[View Full Analysis for details]
```

**Effort**: M
**Fix Type**: Backend (pipe evidence validation output to inline rendering when flags exist)

#### Fix B: Pre-Submit Quality Gate Check

Add evidence validity to quality gate checks:

```
⛔ Cannot submit — Evidence issues detected:
• "Random Evidence Test" does not match required evidence 
  (Clearwater Audit Report)
• Q3 answer "Yes" contradicts evidence state

[Fix Evidence] [View Details]
```

**Effort**: S (add evidence flag check to existing quality gate logic)
**Fix Type**: Backend (consume evidence validation output in quality gate)

#### Fix C: Attachment Table Validity Indicator

Add visual indicator to each file in the attachment table:

```
| ✓/⚠️ | Name                    | Description          | Size     |
|-------|-------------------------|----------------------|----------|
| 🔴    | Random Evidence Test    | Random Evidence Test | 335.84kB |
```

Indicator states:
- 🟢 Valid — matches expected evidence name AND content
- 🟡 Partial — name matches but content not verified OR content unclear
- 🔴 Invalid — name/content don't match test procedure requirements

**Effort**: M (new UI column + consume evidence validation output)
**Fix Type**: Frontend + Backend

#### Implementation Priority
1. **Fix B first** (S effort) — prevents invalid submission. Immediate value.
2. **Fix A second** (M effort) — guides reviewer at decision point.
3. **Fix C third** (M effort) — visual glanceability in attachment table.

**Validation**: 
1. Upload invalid evidence → verify Q3 inline shows warning (Fix A)
2. Upload invalid evidence → attempt submit → verify blocked/warned (Fix B)
3. Upload invalid evidence → verify 🔴 indicator in table (Fix C)
4. Upload VALID evidence → verify all three surfaces show positive/green state

**Trade-offs**: 
- Requires evidence validation to run on file upload (not just on Full Analysis click). May add latency to upload flow.
- False positives: if CLARA incorrectly flags valid evidence as invalid, it blocks the reviewer. Mitigate with "Override: I confirm this is correct evidence" option.
- Content-level inspection may be expensive (reading large files). Mitigate with file size limits for content analysis.

---

## Finding 11: CLARA Detects Invalid Evidence But Still Recommends "Yes"

**Heuristics Triggered**: #8 (Cross-Question Consistency), #3 (Insight vs Reasoning Conflation)

### The Contradiction (Screenshots Prove This)

From the Full Analysis for Q2 (evidence availability) after attaching "Random Evidence Test":

| Full Analysis Field | Value |
|---------------------|-------|
| Current Answer | Yes |
| Confidence | **Low — the only attachment is invalid and the required Clearwater Audit Report is not attached** |
| Suggested Response | **Yes** |
| Insight | "Attach the current quarter's Clearwater Audit Report before submitting this answer. The current attachment ('Random Evidence Test') is not valid evidence..." |
| ⚠️ Flag | "Answer is 'Yes' but no valid evidence is attached. This contradicts the claim that evidence is available and retained." |

**The system simultaneously says:**
- ✅ "Suggested Response: Yes" (headline — most visible)
- ❌ "Confidence: Low" (buried in table)
- ❌ "Flag: Answer contradicts reality" (below insight)
- ❌ "Insight: Attach before submitting" (between response and flag)

### Layer 1: Reasoning

**User Impact**: The reviewer sees **"Suggested Response: Yes"** as the dominant signal. Even in the Full Analysis where all the context is available, the headline says "Yes." A time-pressured reviewer who scans rather than reads will take "Yes" at face value.

**Trust Signal**: CONFUSING. The system is arguing with itself. It says "Yes" AND "this answer contradicts reality" in the same view. The user doesn't know which voice of CLARA to trust.

**Risk**: **HIGH.** The headline "Yes" gives implicit permission to accept. The flag below contradicts it. In compliance attestation contexts, this ambiguity is dangerous — it creates plausible deniability ("CLARA said Yes") while simultaneously documenting the risk ("CLARA also said it's wrong").

**Adoption Threat**: Moderate. Users who notice the contradiction will question CLARA's logic: "It knows the evidence is wrong but still recommends Yes? Why?"

### Layer 2: Issue Summary

## Issue: CLARA recommends "Yes" for evidence availability after detecting evidence is invalid — recommendation doesn't adapt to its own analysis findings

| Field | Value |
|-------|-------|
| Severity | **P0** |
| Category | Cross-Question Consistency + Content Accuracy |
| Scope | Systemic (architectural — recommendation pipeline independent of validation pipeline) |
| Reproducibility | Always (tested: invalid evidence doesn't change recommendation) |
| Affected Persona | Reviewer |
| Affected Reviews | Any review where evidence validation detects issues |
| Control Types Affected | All |

**What's happening**: CLARA's evidence analysis correctly identifies invalid evidence (sets Confidence: Low, generates ⚠️ Flag). However, the Suggested Response remains "Yes" unchanged. The recommendation pipeline and the validation pipeline produce output independently — the recommendation doesn't consume validation results.

**Expected behavior**: When evidence validation determines:
- Confidence = Low, AND
- ⚠️ Flag exists for the question

Then the Suggested Response should adapt:
- Change to "No — required evidence not attached" OR
- Change to "Yes — CONDITIONAL on attaching valid evidence first" with distinct visual styling

**Actual behavior**: "Suggested Response: Yes" + "Flag: contradicts the claim" — contradictory outputs from the same system displayed in the same view.

**Evidence**: Full Analysis screenshot showing simultaneous "Suggested Response: Yes" and "Confidence: Low — the only attachment is invalid"

### Layer 3: Remediation

**Root Cause**: Two independent pipelines:
1. **Recommendation pipeline** — generates Suggested Response from historical pattern analysis ("prior reviews answered Yes")
2. **Validation pipeline** — generates Confidence/Flags from current-state evidence analysis

These pipelines produce output in parallel. The recommendation doesn't have a post-processing step that checks validation results before finalizing.

**Proposed Fix**:

#### Option 1: Post-Processing Rule (Recommended — simplest)
After both pipelines complete, apply a reconciliation rule:

```
IF Flag.severity = ⚠️ AND Confidence IN (Low, Medium):
  IF Suggested_Response = affirmative ("Yes", "Confirmed"):
    Suggested_Response = "Yes (⚠️ CONDITIONAL — see flag below)"
    Card_Style = warning (yellow/orange border)
```

#### Option 2: Recommendation Depends on Validation
Change pipeline ordering: run validation FIRST, feed results into recommendation generation as context. The model then generates a response-aware recommendation:
- "Based on current evidence state (invalid attachment), I cannot recommend 'Yes' until valid evidence is attached."

#### Option 3: Visual Hierarchy Fix (Minimum Viable)
Keep "Suggested Response: Yes" but reorder the Full Analysis card to show the Flag FIRST:

```
⚠️ FLAG: Answer is "Yes" but no valid evidence is attached.
─────────────────────────────────────────────────────────
Suggested Response: Yes (conditional)
Confidence: Low
─────────────────────────────────────────────────────────
Insight: Attach the Clearwater Audit Report before submitting...
Reasoning: Prior reviews consistently answered "Yes"...
```

**Effort**: 
- Option 1: S (post-processing logic)
- Option 2: M-L (pipeline reordering)
- Option 3: S (frontend card reordering)

**Fix Type**: Backend (Option 1/2) or Frontend (Option 3)

**Validation**: 
1. Attach invalid evidence → re-analyze → verify Suggested Response reflects conditional/flagged state
2. Attach valid evidence → verify Suggested Response shows clean "Yes" without flags

**Trade-offs**: 
- Option 1 may produce too many "conditional" recommendations if flags are overly sensitive. Mitigate: only trigger on Confidence = Low (not Medium).
- Option 2 adds latency (sequential pipeline). Mitigate: run in parallel with async reconciliation.
- Option 3 doesn't fix the fundamental issue (still says "Yes") but at least ensures the flag is seen first.

---

## Updated Master Findings Table

| # | Finding | Severity | Category | Status |
|---|---------|----------|----------|--------|
| 1 | "ConfirmationField" jargon + "current" vs "accurate" | P1 | Content Accuracy | Original |
| 2 | Frequency discrepancy not flagged inline | P1 | Information Hierarchy | Original |
| 3 | Insight vs Reasoning conflation | P1 | Semantic Confusion | Original |
| 4 | Q3 recommends "Yes" when 0 attachments exist | P0 | Cross-Question Consistency | Original |
| 5 | Q6 historical "No" not prominently flagged | P2 | Information Hierarchy | Original |
| 6 | Q8 Carnaval "Yes" without current-state check | P2 | Control-Type Awareness | Original |
| 7 | Executive Summary leads with obvious statement | P2 | Template Content | Original |
| 8 | IPA Q1 correctly handles Application Control | ✅ Positive | N/A | Original |
| 9 | IPA Q2 pre-fills completed attestation language | P1 | Compliance Risk | Original |
| **10** | **Evidence validation intelligence trapped in Full Analysis** | **P0** | **Progressive Disclosure Failure** | **Addendum** |
| **11** | **Recommendation doesn't adapt to its own validation findings** | **P0** | **Cross-Question Consistency** | **Addendum** |

### Updated Severity Counts
- **P0: 3** (Finding 4, 10, 11 — all evidence-related)
- **P1: 5** (Finding 1, 2, 3, 9 + original 4 refactored)
- **P2: 3** (Finding 5, 6, 7)
- **Positive: 1** (Finding 8)

### The Common Thread

All three P0 issues share the same root cause: **the evidence validation pipeline operates independently from the recommendation pipeline and the inline rendering pipeline**. CLARA has excellent evidence intelligence but it's architecturally siloed.

```
Current Architecture (Siloed):

Evidence Upload → Validation Pipeline → Full Analysis ONLY
                                         ↕ NO CONNECTION ↕
Historical Data → Recommendation Pipeline → Inline Cards
                                         ↕ NO CONNECTION ↕
                  Quality Gate Logic → Submit Button

Needed Architecture (Integrated):

Evidence Upload → Validation Pipeline ──┬──→ Full Analysis
                                        ├──→ Inline Cards (override when flags exist)
                                        ├──→ Recommendation (adapt when Confidence=Low)
                                        └──→ Quality Gate (block when invalid)
```

---

## Engineering Action Items (Complete — All Findings)

| # | Priority | Ticket Title | Fix Type | Effort |
|---|----------|-------------|----------|--------|
| 1 | **P0** | Evidence validation output not consumed by recommendation pipeline — "Yes" recommended despite "Low" confidence and flags | Backend | S |
| 2 | **P0** | Evidence validation findings not surfaced in inline Q3 card — only in Full Analysis | Backend + Frontend | M |
| 3 | **P0** | Pre-submit quality gate doesn't check evidence validity — allows submission with invalid evidence | Backend | S |
| 4 | P1 | Inline displays Reasoning text labeled as "Insights" — conflates two distinct content types | Backend/Prompt | S-M |
| 5 | P1 | Full Analysis flags (frequency discrepancy) not propagated to inline questions | Backend + Frontend | M |
| 6 | P1 | IPA attestation pre-fills completed statement — needs conditional framing | Prompt + Frontend | M |
| 7 | P1 | "ConfirmationField" internal field name leaked into user-facing insight text | Prompt | S |
| 8 | P2 | Historical "No" answers need visual flag at question level | Backend + Frontend | S |
| 9 | P2 | Executive Summary leads with obvious page state — restructure to lead with actionable insight | Prompt | S |
| 10 | P2 | Carnaval recommendation based on history only — no GRC-Next API current-state verification | Backend (integration) | M |
| 11 | P2 | Attachment table lacks validity indicator column (🟢/🔴) | Frontend | M |

---

*Addendum to CLARA 407759 Three-Layer Analysis*
*Manual QA scenarios: Evidence validation with invalid/mismatched files*
*Date: June 3, 2026*
*Analyst: pm-builder-agent + sarthah (manual testing)*
