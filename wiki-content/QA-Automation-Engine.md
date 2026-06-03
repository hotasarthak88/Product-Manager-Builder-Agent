# QA Automation Engine

The agent includes a comprehensive browser-based QA system that tests web applications against their BRD/PRD requirements.

---

## Capabilities

| # | Capability | What It Does |
|---|-----------|-------------|
| 1 | Full Analysis Modal Automation | Opens modals, extracts structured content, cross-references |
| 2 | State-Change Testing | Performs actions on the app and observes AI/system reactions |
| 3 | Cross-Surface Consistency Engine | Compares content across multiple UI surfaces |
| 4 | BRD Acceptance Criteria Validation | Validates live product against spec |
| 5 | Batch Regression Runner | Same checks across multiple pages with systemic pattern detection |
| 6 | Screenshot Diff | Visual regression detection against baselines |
| 7 | PM-Authored Scenario Library | Test scenarios in YAML, executed automatically |
| 8 | Confidence Scoring | Each finding rated 0-100% with breakdown |
| 9 | Session Persistence | Headed browser with cookie persistence |
| 10 | Automated Ticket Filing | Generates tickets from findings |

---

## How to Use

### Standard QA Run
```
"QA this: https://your-product.a2z.com/page"
```
Runs: Session check → Data extraction → Full Analysis modal → Cross-surface consistency → BRD criteria → Three-layer analysis → Report

### Batch Run
```
"QA these reviews: 
- https://your-app.com/page1
- https://your-app.com/page2
- https://your-app.com/page3"
```
Runs all pages, produces comparison matrix, identifies systemic patterns.

### Scenario-Based Run
```
"Run QA scenarios"
```
Executes all non-destructive scenarios from `docs/qa-scenarios/`.

### Destructive Testing
```
"Test this one: https://your-app.com/page"
```
Approves the URL for state-change testing (attach files, click buttons, etc.).

---

## 10 Bug-Finding Heuristics

Every QA run applies these patterns automatically:

| # | Heuristic | What It Catches |
|---|-----------|----------------|
| 1 | Content Source Consistency | Same data shown differently across surfaces |
| 2 | Jargon/Internal Field Leak | Internal identifiers in user-facing text |
| 3 | Insight vs Reasoning Conflation | Justification shown instead of guidance |
| 4 | Information at Point of Decision | Critical info hidden away from decision point |
| 5 | Template/Repetitive Content | AI output feels robotic across instances |
| 6 | Obvious Statement Waste | AI states what's already visible on the page |
| 7 | Word Limit Artifacts | Compressed content loses important specifics |
| 8 | Cross-Element Consistency | Internal contradictions within same page |
| 9 | Context-Type Awareness | Wrong logic applied for the entity type |
| 10 | Historical Context Accuracy | Claims about past data don't match reality |

---

## Output

Every QA run produces:
1. **Findings report** with three-layer analysis (see [Three-Layer Analysis](Three-Layer-Analysis))
2. **Screenshots** at desktop, tablet, and mobile breakpoints
3. **Consistency matrix** (per-element pass/fail)
4. **Confidence scores** on each finding
5. **Recommended ticket list** (file with one click)

Reports are saved to `docs/qa-reports/` and can be pushed to GitHub.

---

## Safety Rules

- **Never tests production** — only localhost, beta, gamma
- **Never submits real data** — uses obvious test data
- **Destructive testing** requires explicit PM approval per URL
- **Always screenshots** — visual evidence for every finding
- **Cleans up** — offers to undo state-change test modifications
