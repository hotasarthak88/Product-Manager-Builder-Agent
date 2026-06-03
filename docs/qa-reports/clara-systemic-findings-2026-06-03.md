# CLARA Systemic QA Findings — Cross-Review Analysis

**Date**: June 3, 2026
**Environment**: Gamma (GRC 2.0 Preview)
**Reviews Scanned**: 12 total
**Agent**: pm-builder-agent

---

## Reviews Inventory

| Review ID | Control Name | Control Type | CLARA Recs | Insights Panel | Status |
|-----------|-------------|-------------|-----------|---------------|--------|
| 407615 | Windows Domain Administrator Rights | IT General Control | ~~13~~ → 0 | ~~Yes~~ → Suppressed | **Submitted** (AI suppressed ✅) |
| 407759 | Review of investment holdings | Application Control | 13 | Yes | Draft |
| 407763 | Corporate AP purchases (2-way/3-way) | IT Dependent Manual | 0 | Yes (summary only) | Draft |
| 407771 | Security features of bank portals | Application Control | 0 | Yes (summary only) | Draft |
| 407780 | Treasury Wire Approval Limit | IT Dependent Manual | 0 | Yes (summary only) | Draft |
| 407784 | AP Segregation of Duties | IT Dependent Manual | 0 | Yes (summary only) | Draft |
| 407790 | Unfavorable Purchase Price Variance | IT Dependent Manual | 0 | Yes (summary only) | Draft |
| 407791 | Retail revenue vs cash analysis | IT Dependent Manual | 0 | Yes (summary only) | Draft |
| 407808 | Payroll and Payment Funding Approval | IT Dependent Manual | 0 | Yes (summary only) | Draft |
| 407821 | Unrealized gain/loss recalculation | Manual Control | 0 | Yes (summary only) | Draft |
| 407827 | Logical access to AP check printers | Manual Control | 0 | Yes (summary only) | Draft |
| 407828 | Segregation of Duties Review | IT Dependent Manual | 0 | Yes (summary only) | Draft |

---

## SYSTEMIC ISSUE #1: Per-Question CLARA Recommendations Only Work for 2 of 12 Reviews (Critical)

### The Pattern
Only **2 out of 12 reviews** (407615 and 407759) have per-question CLARA recommendations with inline insights. The other **10 reviews show the Executive Summary/Insights panel but ZERO per-question recommendations**.

### Breakdown by Control Type

| Control Type | Reviews | Has Per-Question Recs? |
|-------------|---------|----------------------|
| IT General Control | 1 (407615) | ✅ Yes (13 recs) — now suppressed post-submission |
| Application Control | 2 (407759, 407771) | ✅ Only 407759 has recs; 407771 has 0 |
| IT Dependent Manual Control | 7 | ❌ None have per-question recs |
| Manual Control | 2 (407821, 407827) | ❌ None have per-question recs |

### Why This Is Critical

The BRD explicitly states (Epic-1.2, Pre-fill Response Generation):
> "CLARA pre-fills suggested response text based on detected changes or lack thereof"
> Persona: **Review Owner** (all control types)

The feature is supposed to work for **all review types**. Currently:
- 83% of reviews (10/12) only get the Insights panel summary — no actionable per-question help
- The primary value proposition (30-40% time reduction via pre-fill) is not being delivered to IT Dependent Manual and Manual Control reviewers

### Root Cause Hypotheses

1. **Feature flag scoping** — per-question recommendations may only be enabled for specific control types (IT General Control + Application Control) in the pilot
2. **Model/prompt pipeline not triggered** — the per-question generation pipeline may not fire for IT Dependent Manual / Manual control types
3. **Missing historical data** — these controls may lack the structured historical response data needed to generate per-question suggestions (the system says "several questions are new to this template version" for multiple reviews)
4. **Bug** — the generation was supposed to run but silently failed

### Impact
- **10 out of 12 reviewers** get the Insights panel but no actionable per-question help
- These reviewers face 13 blank questions with only a high-level executive summary
- The primary BRD value proposition (pre-fill saving 15-20 minutes per review) is NOT being delivered to the majority of reviews

### Recommendation
**P0** — Investigate why per-question recommendations are not generating for IT Dependent Manual and Manual Controls. If intentional (pilot scope), document it clearly. If a bug, fix immediately — this is the core feature.

---

## SYSTEMIC ISSUE #2: "ConfirmationField" Jargon — Isolated to Reviews With Per-Question Recs

### Finding
The "ConfirmationField" jargon only appears in reviews that HAVE per-question CLARA recommendations (407759 confirmed, 407615 had it when it was in Draft). Reviews without per-question recs don't show this because they don't have inline insights at the question level.

### Scope
- Confirmed in: 407759
- Likely present in: 407615 (observed in earlier session before submission)
- Not observable in: 10 other reviews (no per-question insights to contain the jargon)

### Verdict
**Not systemic across all reviews**, but systemic within the subset that has per-question recommendations. Since only 2/12 reviews currently have this feature, the blast radius is small — but it will grow as the feature rolls out.

---

## SYSTEMIC ISSUE #3: Executive Summary Is Repetitive/Templated Across Reviews

### The Pattern
Across all 10 reviews without per-question recommendations, the Executive Summary follows an almost identical template:

> "All 13 questions are currently blank. [This is a Key control / No attachments / new reviewer sarthah]. [Historical pattern]. [Specific gap]."

Every single summary starts with "All 13 questions are currently blank" — which is obvious to the reviewer (they just opened a blank review). This wastes the most valuable real estate (top of the summary) on information the user already knows.

### Examples

| Review | Executive Summary Opening |
|--------|--------------------------|
| 407763 | "All 13 questions are currently blank. This is a Key control with a significant historical context..." |
| 407771 | "All 13 questions are currently blank. This is a new reviewer (sarthah)..." |
| 407784 | "All 13 questions are currently blank. This is a Key control with a strong historical pattern..." |
| 407790 | "All 13 questions are currently blank. No attachments are present..." |
| 407827 | "All 13 questions are currently blank. This is a Key control with a strong historical pattern..." |
| 407828 | "All 13 questions are currently blank. This is a new reviewer (sarthah)..." |

### Why This Is a Problem

1. **Leading with obvious information** — the reviewer knows the questions are blank (they just opened the review). This wastes cognitive bandwidth.
2. **Template feel reduces trust** — if every summary starts the same way, users learn to skip it. The valuable parts (prior ineffectiveness, evidence gaps, reviewer transition) are buried after the obvious statement.
3. **Not actionable** — "All 13 questions are blank" doesn't tell the reviewer what to DO. The summary should lead with the most important insight: "⚠️ This control was marked ineffective last quarter" or "You'll need the Clearwater Audit Report before you can submit."

### Recommendation
Restructure the Executive Summary to lead with the **most actionable insight**, not the current state that's visible on the page. Prioritize: risk flags > evidence requirements > reviewer transition > historical pattern > current blank state.

---

## SYSTEMIC ISSUE #4: "Generated by Clara" Count = 1 for All Non-Recommendation Reviews

### The Pattern
All reviews without per-question recommendations show exactly **1** "Generated by Clara" instance — this is the Insights panel summary. But the panel itself contains rich content (Executive Summary + Critical Context + Evidence Analysis).

The issue: **there's no visual connection between the Insights panel and the individual questions**. The reviewer sees a high-level summary and then faces 13 blank questions with no guidance.

### The Gap

```
Current experience (10/12 reviews):
┌─────────────────────────────────────────────┐
│ Insights Panel (executive summary, flags)    │ ← Rich context here
└─────────────────────────────────────────────┘
              ⬇️ DISCONNECT ⬇️
┌─────────────────────────────────────────────┐
│ Q1: Confirm control details [BLANK]          │ ← No help here
│ Q2: Is control at frequency? [BLANK]         │ ← No help here
│ Q3: Is evidence available? [BLANK]           │ ← No help here
│ ... 10 more blank questions ...              │
└─────────────────────────────────────────────┘

Desired experience (what 407759 has):
┌─────────────────────────────────────────────┐
│ Insights Panel (executive summary, flags)    │ ← Rich context
└─────────────────────────────────────────────┘
              ⬇️ CONNECTED ⬇️
┌─────────────────────────────────────────────┐
│ Q1: [CLARA rec: Confirmed] + Insight         │ ← Actionable
│ Q2: [CLARA rec: Yes] + Insight               │ ← Actionable
│ Q3: [CLARA rec: Yes] + Evidence flag         │ ← Actionable
│ ... all questions have guidance ...           │
└─────────────────────────────────────────────┘
```

### Recommendation
Even if per-question full recommendations can't be generated (due to missing historical data), the system should at minimum provide:
- The historical answer pattern ("Prior reviews answered Yes")
- A confidence level ("High/Medium/Low")
- A basic guidance note ("Attach evidence before answering Yes")

This is less expensive than full pre-fill and bridges the gap between "helpful summary" and "blank questions."

---

## SYSTEMIC ISSUE #5: Review 407763 — Prior Quarter Ineffectiveness Not Prominently Flagged

### Finding
Review 407763 (Corporate AP purchases) has a critical piece of information:

> "In Q2 2023, all four parallel reviewers marked this control as NOT operating effectively due to invoices paid without approval or receipt on PO (PO #53-09985318)."

This is in the Executive Summary — but there is **no per-question flag on Q5 (Operating Effectiveness)** because per-question recommendations are missing for this control type.

### Why This Is Dangerous

A reviewer opening this blank review needs to know IMMEDIATELY that the previous quarter had an effectiveness failure. Currently:
- The info is in the summary (requires reading carefully)
- It is NOT surfaced at the point of decision (Q5 radio buttons)
- A reviewer could answer "Yes" to Q5 without realizing the prior quarter was "No"

### Recommendation
Even without full per-question recommendations, critical flags (prior ineffectiveness, prior rejection) should be injected as **inline warnings** at the relevant question — regardless of whether the full recommendation pipeline ran.

---

## SYSTEMIC ISSUE #6: AI Governance Suppression Working Correctly (Positive Finding)

### Finding
Review 407615 was submitted on 6/2/2026 at 11:42 PM UTC. When accessed now:
- ❌ Zero CLARA content visible
- ❌ No "Insights" word anywhere on page
- ❌ No "Generated by Clara" markers
- ✅ "Approve" and "Reject with comment" buttons present (approver view)
- ✅ Human-authored responses only visible

### Verdict
**Epic-11 (AI Governance) is working as designed.** State-based suppression correctly removes all AI content when the review transitions from Draft → Submitted. This is a positive confirmation.

---

## SYSTEMIC ISSUE #7: Consistent 16-20 Console Errors Across All Reviews

### Finding
Every single review page loads with **16-20 console errors** and **2 warnings**. This is consistent across all 12 reviews tested.

### Breakdown
- Fresh page load: 6 errors, 2 warnings
- After full render: 16-20 errors, 2 warnings
- This pattern is identical regardless of control type, review state, or CLARA status

### Why This Matters
- Console errors may indicate failed API calls that affect functionality
- They may explain why per-question recommendations aren't loading for some reviews (silent API failures?)
- For QA sign-off, having 16+ errors on every page load is concerning

### Recommendation
Engineering should investigate the console errors to determine:
1. Are any related to CLARA recommendation pipeline failures?
2. Are any affecting user-visible functionality?
3. Are they suppressible (known harmless) or indicative of real issues?

---

## Summary Table — All Issues

| # | Issue | Severity | Systemic? | Reviews Affected | Category |
|---|-------|----------|-----------|-----------------|----------|
| 1 | Per-question CLARA recs only work for 2/12 reviews | **P0 Critical** | ✅ Yes | 10/12 missing | Feature Gap |
| 2 | "ConfirmationField" jargon in insights | P2 Medium | Partial (2 reviews) | 407759, 407615 | Content Quality |
| 3 | Executive Summary is templated/repetitive | P2 Medium | ✅ Yes (all 10) | All non-rec reviews | Content Quality |
| 4 | No guidance bridge between Insights panel and questions | P1 High | ✅ Yes (all 10) | All non-rec reviews | UX Gap |
| 5 | Prior-quarter ineffectiveness not flagged at question level | P1 High | At least 407763 | Controls with history | Risk/Safety |
| 6 | AI Governance suppression working ✅ | N/A (positive) | ✅ Yes | 407615 confirmed | Compliance |
| 7 | 16-20 console errors on every page load | P2 Medium | ✅ Yes (all 12) | All reviews | Technical Debt |
| 8 | Content variance: Questionnaire insight vs Full Analysis (from earlier report) | P1 High | Confirmed in 407759 | Reviews with recs | Content Consistency |
| 9 | Full Analysis modal lacks numbering (from earlier report) | P2 Medium | Confirmed in 407759 | Reviews with recs | UX/Usability |

---

## Priority Ranking for Engineering

### P0 — Must Fix Before Pilot Expansion
1. **Issue #1**: Per-question recommendations not generating for 83% of reviews. This is the core feature.

### P1 — Must Fix Before GA
2. **Issue #4**: No guidance bridge between Insights panel and blank questions
3. **Issue #5**: Critical historical flags (prior ineffectiveness) not surfaced at question level
4. **Issue #8**: Content variance between inline insight and Full Analysis

### P2 — Should Fix
5. **Issue #3**: Executive Summary leading with obvious "all blank" statement
6. **Issue #2**: "ConfirmationField" jargon leak
7. **Issue #7**: Console errors (investigate root cause)
8. **Issue #9**: Full Analysis modal numbering

### Positive Confirmation
9. **Issue #6**: AI Governance working correctly ✅

---

## Tickets to File

| # | Title | Priority | Team |
|---|-------|----------|------|
| 1 | [CLARA][P0] Per-question recommendations not generating for IT Dependent Manual and Manual Control types | P0 | CLARA Backend/AI |
| 2 | [CLARA][P1] No guidance bridge between Insights summary and individual questions for reviews without per-question recs | P1 | CLARA UX/Frontend |
| 3 | [CLARA][P1] Prior-quarter ineffectiveness not flagged inline at Operating Effectiveness question | P1 | CLARA AI/Rules |
| 4 | [CLARA][P1] Content variance between inline Questionnaire Insight and Full Analysis modal | P1 | CLARA Backend |
| 5 | [CLARA][P2] Executive Summary templated opening — leads with obvious "all blank" instead of actionable insight | P2 | CLARA AI/Prompt |
| 6 | [CLARA][P2] Internal field name "ConfirmationField" leaked into user-facing Insight text | P2 | CLARA AI/Prompt |
| 7 | [GRC][P2] 16-20 console errors on every review page load — investigate root cause | P2 | GRC Frontend |
| 8 | [CLARA][P2] Full Analysis modal lacks Step/Question numbering matching questionnaire | P2 | GRC Frontend |

---

*Report generated by pm-builder-agent*
*Cross-review analysis: 12 reviews scanned*
*Date: June 3, 2026*
