# CLARA Anomaly Report — Review 407759

**Review**: #407759 — Review of investment holdings for compliance (Control 165)
**Control Type**: Application Control
**Environment**: Gamma (GRC 2.0 Preview)
**Reported by**: Sarthak Hota (sarthah)
**Date**: June 3, 2026
**Severity**: Medium (UX quality + content consistency)

---

## Anomaly Summary

| # | Issue | Severity | Category |
|---|-------|----------|----------|
| 1 | "ConfirmationField" jargon leaked into user-facing Insight | Medium | Content Quality |
| 2 | Inconsistency between Questionnaire Insight vs Full Analysis Insight | High | Data Consistency |
| 3 | Full Analysis modal lacks question numbering/labeling | Medium | UX/Usability |

---

## Anomaly 1: "ConfirmationField" Internal Jargon in Insight

### What's Happening
In the **Questionnaire view** for Step 1 (Confirm Control Details), the CLARA Insight reads:

> "The **ConfirmationField** requires the reviewer to attest that control details are **current**."

### Why This Is Wrong (Two Issues)

**Issue A — Jargon leak**: "ConfirmationField" is an internal system/schema field name. It's meaningless to a reviewer. The user sees "Confirm the control details are accurate and up to date" — they don't know what a "ConfirmationField" is.

**Issue B — Incorrect framing**: The insight says the field requires attestation that details are "**current**." But the actual question text says:

> "Confirm the control details are **accurate** and up to date for the review period, including the geographies in which the control operates."

The emphasis should be on **accuracy**, not currency. Being current is one aspect, but accuracy is the primary attestation requirement.

### What It Should Say
Something like:
> "This question asks you to confirm that the control details on the control page are accurate and reflect the current review period. Verify the performer, frequency, reports used, and geography before confirming."

### Root Cause Hypothesis
The AI model is likely referencing the internal GRC data model field name (`ConfirmationField`) from the schema rather than the user-facing question label. The prompt template for generating per-question insights may be using the raw field identifier instead of the human-readable question text.

### Evidence
- **Screenshot**: Questionnaire view showing "ConfirmationField" in the insight
- **Extracted text**: `"Insights: The ConfirmationField requires the reviewer to attest that control details are current. The control document shows the performer, frequency, reports, and objective — the reviewer must verify these reflect reality."`

---

## Anomaly 2: Content Variance Between Full Analysis and Questionnaire Insight

### What's Happening
The **same question** (Step 1: Confirm control details) produces **different CLARA responses** depending on where you read it:

**Questionnaire Insight (inline, brief):**
> "The ConfirmationField requires the reviewer to attest that control details are current. The control document shows the performer, frequency, reports, and objective — the reviewer must verify these reflect reality."

**Full Analysis Modal (detailed, richer):**
> "Before confirming, verify that the control details on the Control 165 page are still accurate: Treasury's Investment team performs the control, it uses the Clearwater Audit Report, the frequency is Monthly (though the 'When' field says quarterly — see flag below), and it operates globally for Amazon's Fixed Income portfolio. If any detail is outdated, update the control page first rather than confirming."

### Why This Is a Problem

1. **The Full Analysis version is significantly better** — it names the specific control (Control 165), identifies the performer (Treasury's Investment team), names the evidence (Clearwater Audit Report), flags a discrepancy (Monthly vs quarterly), and gives clear action guidance.

2. **The Questionnaire version is generic and unhelpful** — it doesn't reference ANY control-specific data. It could apply to literally any control in the system.

3. **Users won't trust CLARA** if the system gives them a shallow answer inline but a deep answer in a modal they have to separately open. Most users will never click "View Full Analysis" — they'll judge CLARA entirely on the inline insight.

4. **The Full Analysis correctly identifies a frequency discrepancy** (Control Frequency = Monthly, but "When is the control performed?" says quarterly) which the inline insight completely misses. This is a material finding the reviewer needs.

### What Should Happen
The **inline Questionnaire Insight should match the depth and specificity of the Full Analysis** — or at minimum, include the key data points:
- Reference the specific control by name/number
- Name the performer and evidence
- Surface flags (like the frequency discrepancy)
- Give clear action guidance

If the Full Analysis is intentionally more detailed, the inline insight should at least **reference** the flag: "⚠️ See Full Analysis for a frequency discrepancy flag."

### Root Cause Hypothesis
Two possible causes:

**Hypothesis A — Separate generation pipelines**: The Questionnaire Insight and Full Analysis are generated by different prompts or different model calls. The Full Analysis gets richer context (full control page data, historical patterns, field-by-field comparison) while the Questionnaire Insight gets only the question text + minimal context.

**Hypothesis B — Caching/timing issue**: The Full Analysis may be generated fresh on demand (with full context), while the Questionnaire Insight was generated earlier (possibly at review creation time) with less context available.

### Recommendation
- **Short-term**: Surface the Full Analysis "Insight" text in the Questionnaire view instead of the generic version. The Full Analysis already has the right content — just pipe it to the inline card.
- **Long-term**: Ensure both surfaces use the same generation pipeline with the same context window. One model call, one response, shown in two places with different formatting.

### Evidence
- **Screenshot 1**: Questionnaire view — generic insight
- **Screenshot 2**: Full Analysis modal — rich, specific insight with flag about frequency discrepancy
- **Extracted Questionnaire text**: `"The ConfirmationField requires the reviewer to attest that control details are current..."`
- **Extracted Full Analysis text**: `"Before confirming, verify that the control details on the Control 165 page are still accurate: Treasury's Investment team performs the control..."`

---

## Anomaly 3: Full Analysis Modal — Poor Information Architecture

### What's Happening
The Full Analysis modal presents question-by-question analysis, but **without step numbers or labels** that match the questionnaire UI.

In the questionnaire, questions are labeled:
- "**Step 1**: Confirm Control Details and Related GRC Items"
- "**Step 2**: Control Review Questions" → "1. Is the control being performed..." "2. Is the evidence..."
- "**Step 3**: IPA Questions" → "1. What pieces of IPA..." "2. For each piece of IPA..."

In the Full Analysis modal, the same questions appear as:
- "Confirm the control details are accurate and up to date..."
- (no number, no step label)

### Why This Is a Problem

1. **Cross-referencing is impossible** — the reviewer can't easily match a Full Analysis section to the corresponding question in the questionnaire. They have to read the full question text and mentally map it.

2. **Scanning is difficult** — without numbered headers, a 13-question analysis becomes a wall of text. Users scan by number ("what did CLARA say about Q5?") — they can't do that here.

3. **Adoption risk** — if the Full Analysis is hard to navigate, reviewers won't use it. They'll stick with the inline insights (which, per Anomaly 2, are less helpful). The richer content in the Full Analysis is wasted.

### What Should Happen
Each section in the Full Analysis modal should be labeled to match the questionnaire:

```
Step 1: Confirm Control Details
─────────────────────────────────
[Current Answer] [Historical Pattern] [Confidence]
Suggested Response: Confirmed
Insight: ...
Flags: ...

Step 2, Question 1: Is the control being performed at the documented frequency?
─────────────────────────────────
[Current Answer] [Historical Pattern] [Confidence]
...
```

This creates a 1:1 map between the questionnaire and the Full Analysis that users can cross-reference instantly.

### Root Cause Hypothesis
The Full Analysis modal likely renders questions using the raw question text from the GRC schema without adding the Step/Question numbering that the questionnaire UI adds as a presentation layer. The numbering is a frontend concern that the Full Analysis backend doesn't have access to.

### Recommendation
- Map question IDs to their Step/Question numbers in the Full Analysis rendering logic
- Add collapsible sections with numbered headers
- Optionally: add "Jump to question" links that scroll to the corresponding questionnaire field

---

## Impact Assessment

| Issue | User Impact | Frequency | Confidence |
|-------|-----------|-----------|-----------|
| ConfirmationField jargon | Confusing — undermines trust in AI | Every review with this question type | High (verified in screenshots) |
| Content variance | Misleading — reviewer misses material flags | Every question across all reviews | High (verified in 407759) |
| Missing numbering in Full Analysis | Usability barrier — reduces adoption | Every Full Analysis view | High (structural issue) |

### Combined Effect
These three issues together tell a story: **the best content CLARA produces (Full Analysis) is hard to find and hard to navigate, while the most visible content (inline Insights) is generic and sometimes incorrect.** This inverts the value proposition — the feature that could save 15-20 minutes per review is hidden behind a modal that most users won't click.

---

## Recommended Priority

1. **Fix content variance FIRST** (Anomaly 2) — highest impact. Pipe the Full Analysis insight content into the inline card. This is likely a backend change to use the same generation context for both surfaces.

2. **Fix jargon leak SECOND** (Anomaly 1) — quick win. Strip internal field names from the prompt template used for insight generation. Replace `ConfirmationField` with the user-facing question label.

3. **Fix numbering THIRD** (Anomaly 3) — UX improvement. Add Step/Question numbering to the Full Analysis modal to match the questionnaire. This is a frontend change.

---

## Tickets to File

| # | Title | Type | Assignee Suggestion |
|---|-------|------|---------------------|
| 1 | [CLARA] Internal field name "ConfirmationField" leaked into user-facing Insight text | Bug | CLARA AI/Prompt team |
| 2 | [CLARA] Inline Questionnaire Insight is generic while Full Analysis has specific, actionable content — align them | Bug/Enhancement | CLARA AI/Backend team |
| 3 | [CLARA] Full Analysis modal lacks Step/Question numbering — hard to cross-reference with questionnaire | Enhancement | GRC Frontend team |

---

*Report generated by pm-builder-agent based on manual QA observations from sarthah*
*Date: June 3, 2026*
