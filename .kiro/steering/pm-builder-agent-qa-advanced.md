# PM Builder Agent — Advanced QA Automation Engine

## Overview

This steering file defines the advanced QA capabilities that extend the base QA phase (`pm-builder-agent-qa.md`) and the three-layer analysis framework (`pm-builder-agent-qa-analysis.md`). These capabilities enable deeper, faster, and more systematic QA with less manual PM effort.

---

## Capability 1: Full Analysis Modal Automation

### What It Does
Automatically opens the Full Analysis modal, extracts all per-question structured data, and cross-references against inline content.

### Execution Steps
1. Navigate to review page, wait for full render
2. Click "View Full Analysis" button (locate by text or `[ref]`)
3. Wait for modal to render (look for heading "Full Analysis" or similar)
4. For EACH question in the modal, extract:
   - Question text (with step/number if present)
   - Current Answer
   - Historical Pattern
   - Confidence level
   - Suggested Response
   - Insight text
   - Reasoning text
   - ⚠️ Flags (if any)
5. Close modal
6. Compare extracted Full Analysis data against inline CLARA card data
7. Flag any discrepancies per Heuristic #1 (Content Source Consistency)

### Playwright Sequence
```
browser_click → "View Full Analysis" button
browser_wait_for → modal heading or content render
browser_evaluate → extract all structured content from modal
browser_click → close button (X)
```

### Auto-Flags Generated
- `CONTENT_VARIANCE`: Inline insight text ≠ Full Analysis insight text
- `REASONING_AS_INSIGHT`: Inline insight text == Full Analysis reasoning text
- `FLAG_NOT_PROPAGATED`: Full Analysis has ⚠️ flag but inline has no warning
- `CONFIDENCE_MISMATCH`: Full Analysis confidence = Low but recommendation is affirmative

---

## Capability 2: State-Change Testing (Manipulate & Observe)

### What It Does
Performs actions on the review (attach files, select answers, type responses) and observes how CLARA reacts — checking if recommendations update, flags appear, and quality gates respond.

### Safety Rules (CRITICAL)
- **ONLY operate on reviews explicitly tagged for QA testing**
- Before ANY manipulation, ask the PM: "Is this review safe to modify for testing?"
- Never manipulate reviews that are in "Submitted" or "Approved" state
- After testing, offer to reset changes: "Want me to undo the test changes?"
- Document every action taken for audit trail

### Test Action Library

| Action | Playwright Steps | What to Observe |
|--------|-----------------|-----------------|
| Attach invalid file | `browser_click` → "Add attachment" → upload file | Does inline flag it? Does quality gate block? |
| Select "No" on effectiveness | `browser_click` → radio button "No" | Does auto-notification trigger? Does CLARA warn? |
| Type vague response | `browser_type` → "Reviewed" in text field | Does real-time quality feedback fire? |
| Select answer contradicting prior | Click "No" when history shows "Yes" | Does CLARA flag the deviation? |
| Remove evidence | Click remove on attachment | Does Q3 recommendation update? |
| Re-analyze after change | Click "Re-analyze" button | Does Executive Summary update? Do per-question insights change? |

### Execution Flow
```
1. Capture baseline state (all CLARA outputs)
2. Perform action
3. Wait for CLARA re-analysis (click "Re-analyze" or wait for auto-update)
4. Capture new state (all CLARA outputs)
5. Diff: baseline vs new state
6. Flag: what SHOULD have changed but didn't? What changed unexpectedly?
7. Document in report
8. Offer to undo changes
```

### PM Input Required
> **Question for PM**: Which reviews in gamma are safe to modify for destructive QA testing? Should I create a naming convention (e.g., "QA-TEST-" prefix) or is there a specific set of test reviews I should use?

---

## Capability 3: Cross-Surface Consistency Engine

### What It Does
For every question on a review, extracts data from THREE surfaces simultaneously and runs automated consistency checks.

### Data Extraction (Per Question)

```yaml
Surface 1 - Inline Card:
  recommendation: string  # "Yes", "No", "N/A", "Confirmed", free text
  insight_text: string    # The "Insights:" content
  has_flag: boolean       # Any warning/flag visible
  
Surface 2 - Full Analysis Modal:
  insight: string         # "Insight:" field
  reasoning: string       # "Reasoning:" field  
  confidence: string      # "High", "Medium", "Low" + explanation
  flags: array            # All ⚠️ flags
  historical_pattern: string
  suggested_response: string
  
Surface 3 - Page State:
  attachment_count: number
  evidence_matrix_gaps: array
  radio_button_state: string  # "Yes", "No", "N/A", "unchecked"
  text_field_content: string
  control_type: string
  control_frequency: string
```

### Automated Consistency Checks

```python
# Check 1: Insight/Reasoning conflation
if inline.insight_text == full_analysis.reasoning:
    flag("REASONING_AS_INSIGHT", severity="P1",
         msg="Inline shows Reasoning text labeled as Insights")

# Check 2: Content variance
if inline.insight_text != full_analysis.insight:
    flag("CONTENT_VARIANCE", severity="P1", 
         msg=f"Inline: '{inline.insight_text[:50]}...' vs Full: '{full_analysis.insight[:50]}...'")

# Check 3: Recommendation contradicts state
if inline.recommendation in ["Yes", "Confirmed"] and page.attachment_count == 0:
    if question_is_about_evidence(question_text):
        flag("REC_CONTRADICTS_STATE", severity="P0",
             msg="Recommends Yes but 0 attachments exist")

# Check 4: Confidence mismatch
if full_analysis.confidence.startswith("Low") and full_analysis.suggested_response in ["Yes", "Confirmed"]:
    flag("CONFIDENCE_IGNORED", severity="P0",
         msg="Confidence=Low but still recommends affirmative")

# Check 5: Flag not propagated
if len(full_analysis.flags) > 0 and not inline.has_flag:
    flag("FLAG_HIDDEN", severity="P1",
         msg=f"Full Analysis has {len(full_analysis.flags)} flags but inline shows none")

# Check 6: Jargon detection
for word in inline.insight_text.split():
    if is_camel_case(word) and word not in question_text:
        flag("JARGON_LEAK", severity="P1",
             msg=f"Internal identifier '{word}' in user-facing text")

# Check 7: Cross-question contradiction
if q3.recommendation == "Yes" and evidence_matrix_has_gaps:
    flag("CROSS_Q_CONTRADICTION", severity="P0",
         msg="Q3=Yes (evidence available) but Evidence Matrix shows gaps")

# Check 8: Control-type mismatch
if control_type == "IT General Control" and question_is_manual_only(question_text):
    if recommendation != "N/A":
        flag("CONTROL_TYPE_MISMATCH", severity="P1",
             msg=f"IT General Control but manual-only question not recommended as N/A")

# Check 9: Historical accuracy
if "all 5 prior reviews" in insight and any_prior_no_exists:
    flag("HISTORICAL_INACCURACY", severity="P1",
         msg="Claims all prior reviews were Yes but historical No exists")

# Check 10: Attestation language
if contains_attestation_language(recommendation):  # "I reviewed", "I confirmed", "I verified"
    flag("ATTESTATION_PREFILL", severity="P1",
         msg="Pre-fills completed attestation — reviewer may accept without performing action")
```

### Output
Produces a per-question consistency matrix:
```
| Q# | Rec | Inline=Insight? | Flags Propagated? | State Consistent? | Issues |
|----|-----|-----------------|-------------------|-------------------|--------|
| 1  | Confirmed | ❌ Shows Reasoning | N/A | ✅ | JARGON_LEAK |
| 2  | Yes | ❌ Generic | ❌ Flag hidden | ✅ | CONTENT_VARIANCE |
| 3  | Yes | ❌ Generic | ❌ Flag hidden | ❌ 0 attachments | P0: REC_CONTRADICTS_STATE |
```

---

## Capability 4: BRD-Driven Acceptance Criteria Validation

### What It Does
Parses BRD acceptance criteria into testable assertions and validates them against the live review.

### Acceptance Criteria Registry

The agent maintains a registry of testable BRD criteria. Each entry maps to:
- An Epic/requirement number
- A testable condition
- A measurement method
- A pass/fail threshold

```yaml
acceptance_criteria:
  - epic: "Epic-1.1"
    requirement: "Change detection within 3 seconds"
    test: "Measure time from page load to CLARA content render"
    threshold_ms: 3000
    
  - epic: "Epic-1.2"
    requirement: "Pre-fill includes concrete data points"
    test: "Check if recommendation contains specific names, dates, or counts"
    check: "recommendation contains at least one of: date pattern, number, proper noun"
    
  - epic: "Epic-4.1"
    requirement: "Evidence completeness validation before submission"
    test: "If evidence matrix has gaps AND CLARA doesn't flag it inline"
    expected: "Flag displayed at point of decision"
    
  - epic: "Epic-6.1"
    requirement: "Approver summary in <3 seconds with quality status"
    test: "Summary present, color-coded readiness, no numeric score visible"
    
  - epic: "Epic-11.1"
    requirement: "AI suppressed in Submitted state"
    test: "Navigate to submitted review, search for 'CLARA' or 'Generated by Clara'"
    expected: "Zero matches"
```

### PM Input Required
> **Question for PM**: Should I extract the full BRD acceptance criteria into this registry now? I have the BRD content from our earlier session. This would create ~50 testable assertions from the 11 Epics.

---

## Capability 5: Batch Regression Runner

### What It Does
Takes a list of review URLs and runs the same consistency checks against all of them, producing a comparison matrix that surfaces systemic patterns.

### Input Format
```yaml
batch_qa:
  reviews:
    - url: "https://gamma.grc.a2z.com/docs/Review/407759/draft"
      label: "Investment Holdings (App Control)"
    - url: "https://gamma.grc.a2z.com/docs/Review/407763/draft"
      label: "Corporate AP (IT Dep Manual)"
    - url: "https://gamma.grc.a2z.com/docs/Review/407615/draft"
      label: "Windows Domain Admin (IT General)"
  checks:
    - per_question_recs_present
    - insights_panel_present
    - evidence_flags_inline
    - jargon_free
    - no_content_variance
```

### Output: Systemic Pattern Matrix
```
| Review | Type | Q Recs | Insights | Evidence Flag | Jargon | Confidence |
|--------|------|--------|----------|---------------|--------|-----------|
| 407759 | App Control | 13 ✅ | ✅ | Hidden ❌ | ❌ | Shows ✅ |
| 407763 | IT Dep Man | 0 ❌ | ✅ | N/A | ✅ | N/A |
| 407615 | IT General | Suppressed | Suppressed | Suppressed | Suppressed | Suppressed |
```

### Auto-Generated Insights
- "Per-question recommendations: only available for 2/12 reviews (Application Control + IT General)"
- "Evidence flags: NEVER surfaced inline across ALL reviews tested"
- "Jargon leak: only present in reviews WITH per-question recommendations"

### Execution
```
for review in batch:
    navigate(review.url)
    wait_for_render()
    extract_all_surfaces()
    run_consistency_checks()
    record_results()
    
generate_comparison_matrix()
identify_systemic_patterns()
produce_findings_report()
```

---

## Capability 6: Screenshot Diff for Visual Regression

### What It Does
Captures screenshots of key UI elements and compares against stored baselines to detect visual regressions.

### Baseline Capture
On first QA run (or when PM says "set baseline"):
1. Screenshot full page at 1440×900
2. Screenshot CLARA Insights panel (cropped)
3. Screenshot each inline CLARA card
4. Screenshot Evidence Matrix
5. Screenshot Attachment table
6. Save to `docs/qa-baselines/{review-id}/`

### Regression Detection (Subsequent Runs)
1. Capture same screenshots
2. Pixel-diff against baseline (using Playwright's built-in comparison or image analysis)
3. Flag regions with >5% pixel change
4. Report: "CLARA card for Q3 has changed: [before] vs [after]"

### When to Run
- After each CLARA release/deployment to gamma
- After engineering claims a fix is deployed
- Weekly as part of regression suite

### PM Input Required
> **Question for PM**: Want me to capture baselines for all 12 reviews now, so we can detect regressions after engineering makes fixes? Or just baseline the primary test review (407759)?

---

## Capability 7: PM-Authored Scenario Library

### What It Does
Allows PMs to author test scenarios in plain language (YAML). The agent reads and executes them automatically.

### Scenario File Location
`docs/qa-scenarios/clara-reviewer-scenarios.yaml`

### Scenario Format
```yaml
scenarios:
  - name: "Invalid evidence — wrong name"
    description: "Attach a file with obviously wrong name/description"
    preconditions:
      - review_state: "draft"
      - safe_to_modify: true
    steps:
      - action: attach_file
        params:
          name: "Random Evidence Test"
          description: "Test file — not related to this control"
          file_path: "docs/qa-test-files/random-evidence.pdf"
      - action: wait_for_reanalysis
        timeout_seconds: 30
      - action: click_reanalyze  # Force re-analysis if needed
    assertions:
      - surface: inline_q3
        check: contains_evidence_warning
        expected: true
        severity_if_fail: P0
      - surface: attachment_table
        check: shows_validity_indicator
        expected: true
        severity_if_fail: P0
      - surface: quality_gate
        check: blocks_or_warns_on_submit
        expected: true
        severity_if_fail: P0
      - surface: full_analysis
        check: detects_invalid_evidence
        expected: true
        severity_if_fail: P1
    cleanup:
      - action: remove_attachment
        params:
          name: "Random Evidence Test"

  - name: "Correct name, wrong content"
    description: "Attach file with matching name but unrelated content"
    preconditions:
      - review_state: "draft"
      - safe_to_modify: true
    steps:
      - action: attach_file
        params:
          name: "Clearwater Audit Report"
          description: "Q2 2026 Clearwater compliance report"
          file_path: "docs/qa-test-files/wrong-content-clearwater.pdf"
      - action: wait_for_reanalysis
        timeout_seconds: 30
    assertions:
      - surface: full_analysis_q3
        check: detects_content_mismatch
        expected: true
      - surface: full_analysis_q3
        check: confidence_is_low
        expected: true
      - surface: full_analysis_q3
        check: suggested_response_is_conditional
        expected: true
        severity_if_fail: P0
      - surface: inline_q3
        check: shows_content_mismatch_warning
        expected: true
        severity_if_fail: P0
    cleanup:
      - action: remove_attachment
        params:
          name: "Clearwater Audit Report"

  - name: "Vague response detection"
    description: "Type a vague response and check for real-time feedback"
    steps:
      - action: type_in_field
        params:
          question: 1  # Step 1: Confirm control details
          text: "Reviewed"
      - action: blur_field  # Move focus away to trigger validation
      - action: wait
        timeout_seconds: 3
    assertions:
      - surface: inline_q1
        check: shows_quality_warning
        expected: true
        message: "Vague response should trigger 'add specific details' guidance"
    cleanup:
      - action: clear_field
        params:
          question: 1

  - name: "Operating effectiveness = No triggers notification"
    description: "Select No for operating effectiveness"
    preconditions:
      - review_state: "draft"
      - safe_to_modify: true
      - confirm_destructive: true  # Will trigger GRC Task creation
    steps:
      - action: select_radio
        params:
          question: 5  # Operating Effectiveness
          value: "No"
      - action: submit_review
    assertions:
      - surface: grc_task
        check: task_created_for_controllership
        expected: true
      - surface: email
        check: notification_sent
        expected: true
    cleanup:
      - action: reopen_review  # If possible
      - action: clear_radio
        params:
          question: 5
```

### PM Input Required
> **Question for PM**: 
> 1. Should I create the `docs/qa-scenarios/` directory and seed it with the scenarios we tested today (invalid evidence, renamed file)?
> 2. Do you have test PDF files I should use, or should I generate dummy PDFs for testing?
> 3. Any additional scenarios you want me to add based on your domain knowledge of common failure modes?

---

## Capability 8: Confidence Scoring on Findings

### What It Does
Assigns a confidence score (0-100%) to each finding, indicating how certain the agent is that it's a real issue vs a potential false positive.

### Scoring Criteria

| Factor | +Confidence | -Confidence |
|--------|------------|-------------|
| Direct data contradiction (page shows X, CLARA says Y) | +30% | — |
| Same pattern across multiple reviews | +20% | — |
| Confirmed by PM's manual observation | +25% | — |
| Based on heuristic only (no hard proof) | — | -20% |
| Could be intentional design decision | — | -15% |
| Edge case (unusual control type or state) | — | -10% |
| BRD explicitly specifies different behavior | +25% | — |

### Confidence Tiers

| Tier | Range | Action |
|------|-------|--------|
| **Confirmed** | 90-100% | File ticket immediately |
| **High** | 75-89% | Include in report, recommend ticket |
| **Medium** | 60-74% | Include in report, flag for PM verification |
| **Low** | Below 60% | "Needs Investigation" section — ask PM |

### Output Format
```
## Finding: Q3 recommends "Yes" with 0 attachments

Confidence: 95% (Confirmed)
├── +30% — Direct contradiction (0 attachments vs "Yes" recommendation)
├── +25% — BRD Epic-4 explicitly states evidence validation before submission
├── +20% — Observed across multiple reviews (407759, 407615)
└── +20% — PM manually confirmed this is a real issue

No deductions. Filing as P0.
```

---

## Capability 9: Session Persistence (Cookie Management)

### What It Does
Maintains browser authentication state between QA runs, eliminating repeated Midway authentication.

### Implementation

**First run (setup)**:
1. Launch Playwright in headed mode (`--headed`)
2. Navigate to gamma.grc.a2z.com
3. Wait for AEA/Midway to authenticate (user may need to approve)
4. Save browser context (cookies, localStorage) to persistent profile
5. Confirm: "✅ Authenticated. Saving session for future runs."

**Subsequent runs**:
1. Launch Playwright with saved profile/context
2. Test authentication: navigate to a protected page
3. If authenticated → proceed with QA
4. If auth expired → alert PM: "Session expired. Please re-authenticate in the browser window."

### Playwright Configuration
```json
{
  "command": "npx",
  "args": ["@playwright/mcp@latest", "--headed", "--user-data-dir", "~/.clara-qa-browser-profile"]
}
```

### Session Health Check (Run at Start of Every QA Session)
```
1. Navigate to gamma.grc.a2z.com
2. Wait 5 seconds
3. Check URL: 
   - Still on gamma.grc.a2z.com → ✅ authenticated
   - Redirected to midway-auth.amazon.com → ❌ session expired
4. If expired: prompt PM to authenticate in headed browser
```

### PM Input Required
> **Question for PM**: Should I update your MCP config to use `--headed` mode with a persistent user-data-dir? This means a Chrome window stays open during QA runs. Alternatively, I can keep headless and just prompt you to re-auth when sessions expire (current behavior).

---

## Capability 10: Automated Ticket Filing

### What It Does
After a QA run, converts findings into structured SIM tickets with one confirmation step.

### Ticket Generation Flow
```
1. QA run completes → findings report generated
2. Agent groups findings by severity and team assignment
3. Agent drafts ticket content from Layer 2 (Issue Summary) of each finding
4. Presents to PM: "I have 3 tickets ready to file. Review and confirm?"
5. On confirmation → creates tickets via Taskei/SIM
6. Records ticket IDs in the report for tracking
```

### Ticket Template (Auto-Generated from Three-Layer Analysis)
```
Title: [CLARA][{severity}] {one-line finding title}

Description:
## What's Happening
{Layer 2: What's happening}

## Expected Behavior
{Layer 2: Expected behavior}

## Actual Behavior  
{Layer 2: Actual behavior}

## Evidence
{Layer 2: Evidence reference}
{Link to full QA report on GitHub}

## Proposed Fix
{Layer 3: Proposed Fix}
Effort: {Layer 3: Effort}
Fix Type: {Layer 3: Fix Type}

## Reproduction Steps
1. Open {review URL}
2. {specific steps to see the issue}
3. Compare {surface A} with {surface B}

## Acceptance Criteria for Fix
{Layer 3: Validation steps}

---
*Filed by pm-builder-agent from QA report dated {date}*
```

### Team Assignment Logic
- Prompt/AI content issues → CLARA AI team
- Frontend/UX rendering → GRC Frontend team
- Backend pipeline/data flow → CLARA Backend team
- Quality gate logic → GRC Platform team

### PM Input Required
> **Question for PM**: 
> 1. What SIM room should CLARA tickets go to? (I need the room ID for Taskei)
> 2. Who should be the default assignee for CLARA AI issues vs GRC Frontend issues?
> 3. Should tickets include the full three-layer analysis, or just the summary + proposed fix?

---

## Orchestration: How Capabilities Work Together

### Standard QA Run (Triggered by "QA this: {url}")
```
Step 1: Session Check (Cap 9)
  → Verify authentication
  
Step 2: Page Load + Data Extraction
  → Navigate to review
  → Extract inline content (all questions)
  → Open Full Analysis modal (Cap 1)
  → Extract Full Analysis content
  → Extract page state (attachments, evidence matrix, control properties)

Step 3: Cross-Surface Consistency (Cap 3)
  → Run all automated checks
  → Generate per-question consistency matrix

Step 4: BRD Criteria Validation (Cap 4)
  → Run applicable acceptance criteria
  → Flag any spec violations

Step 5: Three-Layer Analysis (from qa-analysis.md)
  → For each finding: Reasoning → Summarization → Remediation
  → Assign confidence scores (Cap 8)

Step 6: Report Generation
  → Produce findings ranked by confidence × severity
  → Include comparison matrix
  → Include proposed fixes with effort estimates

Step 7: Action Offer
  → "File tickets?" (Cap 10)
  → "Run state-change tests?" (Cap 2) — if PM confirms review is safe to modify
  → "Compare against baselines?" (Cap 6) — if baselines exist
```

### Batch QA Run (Triggered by "QA these reviews: {list}")
```
Step 1: Session Check
Step 2: For each review → Steps 2-5 above
Step 3: Cross-Review Comparison (Cap 5)
  → Generate systemic pattern matrix
  → Flag patterns appearing in >50% of reviews
Step 4: Consolidated Report
  → Single report with per-review sections + systemic findings
Step 5: Action Offer
```

### Scenario-Based QA Run (Triggered by "Run QA scenarios")
```
Step 1: Session Check
Step 2: Load scenario file (Cap 7)
Step 3: For each scenario:
  → Verify preconditions
  → Execute steps
  → Check assertions
  → Run cleanup
Step 4: Report pass/fail per scenario
Step 5: Three-layer analysis on failures
```

---

## Trigger Phrases

The PM can invoke these capabilities with natural language:

| Say | Capability Triggered |
|-----|---------------------|
| "QA this: {url}" | Standard QA Run (all capabilities) |
| "QA these reviews: {list}" | Batch QA Run |
| "Run QA scenarios" | Scenario-Based Run |
| "Run scenarios on {url}" | Scenario-Based on specific review |
| "Set QA baseline for {url}" | Screenshot Baseline Capture (Cap 6) |
| "Check for regressions on {url}" | Screenshot Diff (Cap 6) |
| "File tickets for the last QA run" | Ticket Filing (Cap 10) |
| "Is {url} safe to test destructively?" | Precondition check for Cap 2 |
| "Add a QA scenario: {description}" | Append to scenario library (Cap 7) |
| "How confident are you about finding X?" | Explain confidence scoring (Cap 8) |

---

*Built with pm-builder-agent*
