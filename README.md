# PM Builder Agent

A Kiro-powered AI agent that acts as a product management colleague — not a tool, not an assistant. It pairs with PMs to drive the full product lifecycle from problem definition through shipping.

## What It Does

| Phase | Capability |
|-------|-----------|
| 🎯 **Problem** | Define problems, challenge assumptions, validate customers, create Impact Canvases |
| 📋 **Plan** | Write PRDs, BRDs, feature specs, executive summaries |
| 🎨 **Build** | Generate interactive prototypes matching your design system |
| 🔧 **Bridge** | Scaffold production code on feature branches, hand off to engineering |
| ✅ **Validate** | Multi-persona expert reviews (Developer, Product, UX, Leadership, Cross-Org) |
| 🚀 **Ship** | Track weekly progress, flag risks, assess launch readiness |
| 📚 **Learn** | Capture patterns, run retrospectives |

## How It Works

The agent uses Kiro steering files (`.kiro/steering/`) to define its behavior, personality, and workflows. On first interaction it autonomously discovers the PM's context — team, repos, goals, engineering partners — and adapts to their expertise level.

## Setup

1. Open this folder as a workspace in [Kiro](https://kiro.dev)
2. The steering files activate automatically
3. Start chatting — the agent will handle the rest

### Required MCP Servers (auto-configured on first use)

| Server | Purpose |
|--------|---------|
| `builder-mcp` | Code search, internal wikis, Phone Tool, tickets |
| `pippin-mcp` | Document sharing with stakeholders |
| `jupiter-mcp` | UI design and prototyping |
| `figma` | Figma design integration (recommended) |

## Project Structure

```
PMBuilderAgent/
├── .kiro/
│   ├── steering/          # Agent behavior definitions
│   │   ├── pm-builder-agent-core.md        # Identity, discovery, personality
│   │   ├── pm-builder-agent-workflow.md     # PM lifecycle & phases
│   │   ├── pm-builder-agent-build.md       # Prototyping & engineering handoff
│   │   ├── pm-builder-agent-review.md      # Multi-persona document reviews
│   │   ├── pm-builder-agent-engineering.md # Code quality standards
│   │   └── jupiter-mcp.md                 # Jupiter UI design integration
│   └── hooks/             # Event-driven automations
│       ├── block-spec-workflow.kiro.hook
│       └── ignore-spec-session-prompt.kiro.hook
├── .gitignore
└── README.md
```

## Key Design Decisions

- **Autonomous discovery** — the agent builds context silently, never blocks the PM with onboarding questions
- **Data-driven opinions** — pushes back only when evidence supports it, cites sources
- **Adaptive expertise** — calibrates to the PM's level and operates one step ahead
- **Two-hat self-review** — all scaffolded code goes through a Senior Engineer → Principal Engineer review loop before handoff
- **Phase-aware** — tracks where the PM is in the lifecycle and suggests next steps proactively

## License

Internal use only.

---
*Built with pm-builder-agent*
