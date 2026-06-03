# Adapting for Your Team

This agent is designed to be forked and customized per product. Here's how to set it up for your team.

---

## Step 1: Fork or Clone

```bash
git clone git@github.com:hotasarthak88/Product-Manager-Builder-Agent.git MyProductAgent
cd MyProductAgent
```

---

## Step 2: Edit Configuration

Open `docs/qa-config.yaml` and change:

```yaml
product:
  name: "Your Product Name"          # Replace
  base_url: "https://your-app.a2z.com"  # Replace
  description: "What your product does"  # Replace

tickets:
  tool: "ask"  # Leave as "ask" — agent will prompt you
```

---

## Step 3: Provide Your BRD/PRD

**Option A (automatic)**: Just tell the agent:
> "Here's my BRD: {paste or link}"

The agent extracts testable acceptance criteria and creates `docs/qa-scenarios/brd-acceptance-criteria.yaml` for you.

**Option B (manual)**: Create your own criteria YAML following the format in the existing file.

---

## Step 4: Add Product-Specific Scenarios (Optional)

Create `docs/qa-scenarios/your-product-scenarios.yaml` with test cases specific to your product's flows. See [Scenario Library](Scenario-Library) for the format.

---

## Step 5: Add Test Files (Optional)

If your product has evidence upload, form submission, or file processing — add dummy test files to `docs/qa-test-files/` for the agent to use in state-change testing.

---

## What Stays the Same (Don't Edit)

| File | Why |
|------|-----|
| `.kiro/steering/pm-builder-agent-core.md` | Agent personality and discovery |
| `.kiro/steering/pm-builder-agent-workflow.md` | PM lifecycle phases |
| `.kiro/steering/pm-builder-agent-qa-analysis.md` | Three-layer framework |
| `.kiro/steering/pm-builder-agent-qa-advanced.md` | QA engine capabilities |
| `.kiro/steering/pm-builder-agent-review.md` | Multi-persona reviews |

These are product-agnostic and work for any team.

---

## What You Customize

| File | What to Change |
|------|---------------|
| `docs/qa-config.yaml` | Product name, URLs, team, terms |
| `docs/qa-scenarios/*.yaml` | Your product's test scenarios |
| `docs/qa-test-files/` | Dummy files for your product |
| `README.md` | Optional — update description |

---

## Example: Setting Up for a Console Feature

```yaml
# docs/qa-config.yaml
product:
  name: "AWS Console Widgets"
  environment: "beta"
  base_url: "https://beta.console.aws.amazon.com"

heuristics:
  allowed_technical_terms:
    - "CloudFormation"
    - "CloudWatch"
    - "Cloudscape"
  blocked_internal_terms:
    - "widgetId"
    - "renderState"
    - "componentRef"
```

Then say:
> "QA this: https://beta.console.aws.amazon.com/my-feature"

The agent runs all heuristics, checks for jargon, validates consistency, and produces a three-layer report.

---

## Tips

- **Don't overthink the setup.** Start with just `product.name` and `product.base_url`. Add more config as you discover what you need.
- **Let the agent build your BRD registry.** Say "Here's my PRD" and it does the work.
- **Add scenarios incrementally.** Every time you find a bug manually, tell the agent: "Add a QA scenario: {description}" — it grows your test suite over time.
- **Share reports via GitHub.** Push QA reports to your repo for engineering visibility.
