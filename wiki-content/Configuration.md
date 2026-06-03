# Configuration

All product-specific settings live in `docs/qa-config.yaml`. Edit this file to customize the agent for your product.

---

## Full Configuration Reference

```yaml
# ─────────────────────────────────────────────────────────────
# PRODUCT
# ─────────────────────────────────────────────────────────────
product:
  name: "Your Product Name"
  environment: "gamma"          # gamma | beta | prod | localhost
  base_url: "https://your-app.a2z.com"
  description: "One-line description"

# ─────────────────────────────────────────────────────────────
# TICKETS
# ─────────────────────────────────────────────────────────────
tickets:
  tool: "ask"                   # ask | taskei | sim-t | jira | github
  sim_room_id: ""              # Taskei room UUID (if using Taskei)
  default_assignees:
    all: "engineer-alias"      # Single assignee for all issues
    # Or per-category:
    # ai_content: "alias1"
    # frontend: "alias2"
    # backend: "alias3"
  ticket_format: "summary_impact_fix"
  auto_link_to_report: true

# ─────────────────────────────────────────────────────────────
# QA SETTINGS
# ─────────────────────────────────────────────────────────────
qa_settings:
  approved_destructive_urls: []    # PM adds via "test this one"
  queue_url: ""                    # Dashboard/queue page URL
  brd_source: "docs/qa-scenarios/brd-acceptance-criteria.yaml"
  scenarios_path: "docs/qa-scenarios/"
  test_files_path: "docs/qa-test-files/"
  baselines_path: "docs/qa-baselines/"
  reports_path: "docs/qa-reports/"

# ─────────────────────────────────────────────────────────────
# HEURISTIC TUNING
# ─────────────────────────────────────────────────────────────
heuristics:
  # Words that WON'T trigger jargon detection
  allowed_technical_terms:
    - "YourProductTerm"
    - "AnotherAllowedTerm"
  
  # Words that ALWAYS flag as jargon
  blocked_internal_terms:
    - "internalFieldName"
    - "dbColumnName"
  
  # Which elements relate to evidence (for cross-check)
  evidence_questions:
    - question_number: 3
      topic: "Is evidence available?"

# ─────────────────────────────────────────────────────────────
# PERSONAS
# ─────────────────────────────────────────────────────────────
personas:
  primary:
    name: "End User"
    description: "Primary user of your product"
    test_cases_file: "docs/qa-reports/test-cases-primary.md"
  secondary:
    name: "Admin/Approver"
    description: "Reviews or approves work"
    test_cases_file: "docs/qa-reports/test-cases-secondary.md"
```

---

## Minimal Config (Quick Start)

For a new product, you only need:

```yaml
product:
  name: "My Product"
  base_url: "https://my-app.a2z.com"

tickets:
  tool: "ask"  # Agent will ask for details on first ticket filing
```

Everything else has sensible defaults.

---

## Ticket Tool Options

| Value | Tool | Notes |
|-------|------|-------|
| `"ask"` | Agent asks on first use | Default — zero config needed |
| `"taskei"` | Amazon Taskei (SIM) | Needs `sim_room_id` |
| `"sim-t"` | SIM-T (t.corp.amazon.com) | Needs CTI category |
| `"jira"` | JIRA | For teams using JIRA |
| `"github"` | GitHub Issues | For open-source or external |

The PM/SDM always provides the assignee and destination. The agent never assumes.
