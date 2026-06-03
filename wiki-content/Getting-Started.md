# Getting Started

## Prerequisites

- [Kiro IDE](https://kiro.dev) installed
- MCP servers available (builder-mcp, pippin-mcp, playwright)
- Node.js 18+ (for Playwright MCP)

## Setup (5 minutes)

### Step 1: Clone the repo
```bash
git clone git@github.com:hotasarthak88/Product-Manager-Builder-Agent.git
```

### Step 2: Open in Kiro
Open the cloned folder as a workspace in Kiro. The steering files activate automatically.

### Step 3: Install Playwright MCP (for QA capabilities)
```bash
npm install -g @playwright/mcp
```

### Step 4: Start chatting
Just describe what you need. The agent figures out which phase applies.

---

## First Interaction

On your first message, the agent silently:
1. Discovers your identity (Phone Tool)
2. Maps your repos and pipelines (from workspace + code.amazon.com)
3. Identifies your engineering partners (from CRs and permissions)
4. Finds your goals (from Kingpin)
5. Sets up workspace structure

**You see**: An answer to your question that already knows your world. Zero onboarding questions.

---

## MCP Servers

| Server | Command | Purpose | Required? |
|--------|---------|---------|-----------|
| builder-mcp | `builder-mcp` | Code search, wikis, Phone Tool, tickets | ✅ Required |
| pippin-mcp | `pippin-mcp-server` | Document sharing | ✅ Required |
| playwright | `npx @playwright/mcp@latest --headed` | Browser QA | ✅ For QA phase |
| jupiter-mcp | varies | UI prototyping | Recommended |
| figma | varies | Design integration | Optional |

The agent auto-configures MCP connections on first use. If something's missing, it'll note the gap and proceed with reduced capabilities.

---

## What to Say

| Goal | Say This |
|------|----------|
| Define a problem | "I have an idea..." or "Help me think through this problem" |
| Write a document | "Write a PRD for..." or "Help me structure a BRD" |
| Create a prototype | "Prototype the settings page" or "Mock up the dashboard" |
| Scaffold code | "Hand this off to engineering" or "Scaffold the code" |
| Run QA | "QA this: https://your-app.com/page" |
| Get a review | "Review this PRD" or "Give me a leadership perspective" |
| Track progress | "Generate my weekly log" or "Are we ready to launch?" |
| See capabilities | "What can you do?" or "Help" |

---

## Folder Structure (Created Automatically)

```
your-workspace/
├── .kiro/steering/     # Agent behavior (don't edit unless customizing)
├── .kiro/hooks/        # Event automations
├── docs/
│   ├── qa-config.yaml  # Your product QA settings
│   ├── qa-scenarios/   # Test scenario library
│   ├── qa-test-files/  # Dummy files for testing
│   └── qa-reports/     # Generated QA reports
├── knowledge-base/     # Auto-generated context (gitignored)
├── mocks/              # Prototypes
└── README.md
```
