# Scenario Library

PMs author test scenarios in YAML. The agent reads and executes them automatically.

---

## Location

```
docs/qa-scenarios/
├── clara-reviewer-scenarios.yaml    # CLARA-specific scenarios
├── brd-acceptance-criteria.yaml     # BRD testable assertions
└── your-product-scenarios.yaml      # Add your own
```

---

## Writing Scenarios

### Non-Destructive (Safe on Any Page)

```yaml
- name: "Content consistency check"
  description: "Verify inline content matches detail view"
  destructive: false
  steps:
    - action: navigate_to_page
    - action: extract_inline_content
    - action: open_detail_view
    - action: extract_detail_content
  assertions:
    - check: inline_matches_detail
      severity_if_fail: P1
```

### Destructive (Requires PM Approval)

```yaml
- name: "Upload invalid file"
  description: "Attach wrong file and verify system flags it"
  destructive: true
  preconditions:
    - pm_approved: true  # PM must say "test this one"
  steps:
    - action: attach_file
      params:
        name: "Wrong File"
        file_path: "docs/qa-test-files/random-evidence.pdf"
    - action: wait_for_update
      timeout_seconds: 30
  assertions:
    - surface: inline_warning
      check: shows_invalid_file_message
      expected: true
      severity_if_fail: P0
  cleanup:
    - action: remove_attachment
      params:
        name: "Wrong File"
```

---

## Available Actions

| Action | What It Does |
|--------|-------------|
| `navigate_to_page` | Opens the target URL |
| `extract_inline_content` | Reads visible AI/helper content |
| `open_detail_view` | Clicks into a modal or expanded view |
| `extract_detail_content` | Reads modal/expanded content |
| `attach_file` | Uploads a file |
| `type_in_field` | Types text into an input |
| `select_radio` | Selects a radio button |
| `click_button` | Clicks a specific button |
| `wait_for_update` | Waits for AI to re-process |
| `click_reanalyze` | Triggers re-analysis |
| `capture_baseline` | Saves current state for comparison |
| `capture_new_state` | Saves state after action for diff |
| `clear_field` | Removes text from a field |
| `remove_attachment` | Deletes an uploaded file |
| `blur_field` | Moves focus away (triggers validation) |

---

## Available Assertions

| Check | What It Verifies |
|-------|-----------------|
| `inline_matches_detail` | Content consistency across surfaces |
| `contains_evidence_warning` | Warning about invalid/missing evidence |
| `shows_validity_indicator` | Visual badge (🟢/🔴) on elements |
| `confidence_is_low` | AI confidence level is Low |
| `suggested_response_adapts` | Recommendation changes based on state |
| `shows_quality_warning` | Real-time quality feedback fires |
| `no_camel_case_identifiers` | No jargon in user text |
| `first_sentence_is_not_obvious` | Summary doesn't restate visible state |

---

## Running Scenarios

```
"Run QA scenarios"                    → All non-destructive
"Run scenarios on {url}"              → Non-destructive on specific page
"Test this one" + "Run QA scenarios"  → Includes destructive scenarios
```

---

## Adding Scenarios

Tell the agent:
```
"Add a QA scenario: When a user types only 'Yes' in a free-text field, 
CLARA should warn that the response needs more detail"
```

The agent appends it to your scenario YAML file.
