# PM Lifecycle

The agent drives PMs through 8 phases. You can enter at any phase — the agent detects what you have and suggests what's missing.

```
┌─────────┬────────┬────────┬─────────┬──────┬──────────┬────────┬────────┐
│ Problem │  Plan  │ Build  │ Bridge  │  QA  │ Validate │  Ship  │ Learn  │
└─────────┴────────┴────────┴─────────┴──────┴──────────┴────────┴────────┘
```

---

## Phase 0: Problem

**Purpose**: Define the problem before jumping to solutions.

**What the agent does**:
- Listens to your idea
- Asks probing questions (2-3 at a time)
- Challenges assumptions with data
- Synthesizes into an Idea Canvas
- Generates an Impact Canvas (mandatory before docs)

**Trigger phrases**: "I have an idea...", "Help me think through this problem"

---

## Phase 1: Plan

**Purpose**: Define what to build.

**What the agent does**:
- Writes PRDs, BRDs, feature specs, executive summaries
- Checks each requirement traces back to the stated problem
- Sizes solutions (Small / Medium / Large)
- Breaks greenfield projects into shippable phases

**Trigger phrases**: "Write a PRD for...", "Help me structure a BRD"

---

## Phase 2: Build

**Purpose**: Make it tangible.

**What the agent does**:
- Generates interactive HTML prototypes
- Matches your product's design system (auto-discovered)
- Iterates based on feedback
- Pushes to Figma if available

**Trigger phrases**: "Prototype the...", "Mock up the..."

---

## Phase 3: Bridge

**Purpose**: Scaffold code for engineering.

**What the agent does**:
- Determines delivery path (existing repo vs greenfield)
- Scaffolds components, hooks, types, test stubs
- Runs two-hat self-review (Senior Engineer → Principal Engineer)
- Pushes to feature branch behind feature flag
- Generates engineering handoff README

**Trigger phrases**: "Hand this off to engineering", "Scaffold the code"

---

## Phase 4: QA

**Purpose**: Validate the feature works before ship.

**What the agent does**:
- Navigates your app via browser automation (Playwright MCP)
- Runs cross-surface consistency checks
- Validates against BRD acceptance criteria
- Applies 10 bug-finding heuristics
- Produces three-layer analysis for every finding
- Takes screenshots at desktop/tablet/mobile

**Trigger phrases**: "QA this: {url}", "Run QA scenarios"

See: [QA Automation Engine](QA-Automation-Engine)

---

## Phase 5: Validate

**Purpose**: Confirm with stakeholders.

**What the agent does**:
- Runs expert reviews from 5 personas:
  - 🔧 Developer — technical feasibility
  - 📦 Product — customer value, business impact
  - 🎨 UX — user experience, accessibility
  - 👔 Leadership — strategic value, investment readiness
  - 🌐 Cross-Org — dependencies, coordination
- Produces structured review with scores and action items

**Trigger phrases**: "Review this PRD", "Give me a leadership perspective"

---

## Phase 6: Ship

**Purpose**: Track execution.

**What the agent does**:
- Generates weekly progress logs
- Flags risks proactively
- Assesses launch readiness
- Surfaces blockers from ticket load and pipeline health

**Trigger phrases**: "Generate my weekly log", "Are we ready to launch?"

---

## Phase 7: Learn

**Purpose**: Capture what worked.

**What the agent does**:
- Runs retrospectives
- Captures patterns and recipes
- Documents decisions for future reference

**Trigger phrases**: "What worked this sprint?", "Run a retro"

---

## Dynamic Entry

You don't have to start at Phase 0. The agent detects what you provide:

| You Provide | Agent Starts At |
|------------|----------------|
| Raw idea | Phase 0 (Problem) |
| PRD/BRD document | Phase 2 (Build) — offers to backfill Problem |
| Prototype | Phase 3 (Bridge) — offers to backfill PRD |
| Existing code | Phase 4 (QA) — offers to backfill docs |
| URL to test | Phase 4 (QA) — runs immediately |

The agent shows a status tracker at phase transitions and suggests next steps.
