# PM Builder Agent

An AI-powered product management agent built on Kiro that drives the full PM lifecycle — from problem definition through shipping. Designed for Amazon PMTs to use across any product, any team, any domain.

## What It Does

| Phase | Capability |
|-------|-----------|
| 🎯 **Problem** | Define problems, challenge assumptions, validate customers, create Impact Canvases |
| 📋 **Plan** | Write PRDs, BRDs, feature specs, executive summaries |
| 🎨 **Build** | Generate interactive prototypes matching your design system |
| 🔧 **Bridge** | Scaffold production code on feature branches, hand off to engineering |
| 🧪 **QA** | Automated browser testing, three-layer analysis, BRD validation, regression detection |
| ✅ **Validate** | Multi-persona expert reviews (Developer, Product, UX, Leadership, Cross-Org) |
| 🚀 **Ship** | Track weekly progress, flag risks, assess launch readiness |
| 📚 **Learn** | Capture patterns, run retrospectives |

## Who Is This For?

**Any PM, TPM, or PMT member** who wants to:
- Accelerate the problem → ship lifecycle
- Generate and validate PRDs/BRDs faster
- Run automated QA against their product's BRD acceptance criteria
- Surface bugs with structured reasoning, summarization, and remediation proposals
- Track feature progress across sprints
- Hand off engineering-ready code scaffolds

## Quick Start

1. Clone this repo into your Kiro workspace
2. Open it in [Kiro](https://kiro.dev) — steering files activate automatically
3. Start chatting — the agent discovers your context and adapts

### First Time Setup

The agent auto-discovers:
- Your identity and team (from Phone Tool)
- Your repos and pipelines (from workspace + code.amazon.com)
- Your goals (from Kingpin)
- Your engineering partners (from CRs and repo permissions)

**No onboarding questions.** You ask a question → you get an answer that already knows your world.

## QA Capabilities (New)

The agent includes a full QA automation engine for testing web applications against their PRD/BRD:

| Capability | What It Does |
|-----------|-------------|
| Cross-Surface Consistency | Compares content across multiple UI surfaces for contradictions |
| BRD Acceptance Criteria | Validates live product against spec requirements |
| Three-Layer Analysis | Every finding → Reasoning (why) + Summary (what) + Remediation (how to fix) |
| Scenario Library | PM-authored test scenarios in YAML, executed automatically |
| Batch Regression | Run same checks across multiple pages/reviews, surface systemic patterns |
| Confidence Scoring | Each finding rated 0-100% confidence with scoring breakdown |
| Automated Ticket Filing | Generates SIM tickets from findings with one-click confirmation |
| Session Persistence | Headed browser with cookie persistence for internal Amazon apps |

### How to Use QA

```
"QA this: https://your-product.a2z.com/page"        → Full automated analysis
"QA these: [url1, url2, url3]"                      → Batch analysis with systemic patterns
"Run QA scenarios"                                   → Execute scenario library
"File tickets for the last QA run"                   → Generate SIM tickets from findings
"Test this one" + url                                → Approve for destructive testing
```

### Customizing for Your Product

Edit `docs/qa-config.yaml` to configure:
- Your product name and base URL
- Team assignees for ticket filing
- Allowed/blocked technical terms
- Evidence question mappings
- Persona definitions

## Project Structure

```
PMBuilderAgent/
├── .kiro/
│   ├── steering/                           # Agent behavior definitions
│   │   ├── pm-builder-agent-core.md        # Identity, discovery, personality
│   │   ├── pm-builder-agent-workflow.md     # PM lifecycle & phases
│   │   ├── pm-builder-agent-build.md       # Prototyping & engineering handoff
│   │   ├── pm-builder-agent-review.md      # Multi-persona document reviews
│   │   ├── pm-builder-agent-engineering.md # Code quality standards
│   │   ├── pm-builder-agent-qa.md          # Browser QA phase
│   │   ├── pm-builder-agent-qa-analysis.md # Three-layer analysis framework
│   │   ├── pm-builder-agent-qa-advanced.md # Advanced QA automation engine
│   │   └── jupiter-mcp.md                  # Jupiter UI design integration
│   └── hooks/                              # Event-driven automations
├── docs/
│   ├── qa-config.yaml                      # Product-specific QA configuration
│   ├── qa-scenarios/                       # Test scenario library (YAML)
│   │   ├── clara-reviewer-scenarios.yaml   # Example: CLARA-specific scenarios
│   │   └── brd-acceptance-criteria.yaml    # Example: BRD criteria registry
│   ├── qa-test-files/                      # Dummy files for evidence testing
│   └── qa-reports/                         # Generated QA reports
├── .gitignore
└── README.md
```

## Adapting for Your Team

### For a new product:
1. Fork/clone this repo
2. Replace `docs/qa-config.yaml` with your product settings
3. Replace `docs/qa-scenarios/brd-acceptance-criteria.yaml` with YOUR BRD's criteria (or let the agent extract them — just say "Here's my BRD: {link}" and it builds the registry)
4. Add product-specific scenarios to `docs/qa-scenarios/`
5. The steering files work generically — no changes needed

### What stays the same across products:
- Three-layer analysis framework (Reasoning → Summary → Remediation)
- 10 QA heuristics (consistency, jargon, cross-surface, etc.)
- PM lifecycle phases (Problem → Plan → Build → Bridge → QA → Validate → Ship)
- Multi-persona review system
- Engineering handoff standards

### What you customize per product:
- `qa-config.yaml` — URLs, team, terms, personas
- `qa-scenarios/*.yaml` — product-specific test flows
- `qa-test-files/` — dummy files for your product's evidence types

## Required MCP Servers

| Server | Purpose | Required? |
|--------|---------|-----------|
| `builder-mcp` | Code search, internal wikis, Phone Tool, tickets | REQUIRED |
| `pippin-mcp` | Document sharing with stakeholders | REQUIRED |
| `playwright` | Browser automation for QA | REQUIRED for QA phase |
| `jupiter-mcp` | UI design and prototyping | RECOMMENDED |
| `figma` | Figma design integration | OPTIONAL |

## License

Internal use only.

---
*Built with pm-builder-agent*
