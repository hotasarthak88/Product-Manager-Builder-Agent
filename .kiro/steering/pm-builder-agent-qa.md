# PM Builder Agent — Browser QA Phase

## Overview
The QA phase sits between **Bridge** (code scaffolded) and **Ship** (tracking progress). It uses browser automation to validate that scaffolded features work correctly before engineering takes ownership.

```
Problem → Plan → Build → Bridge → **QA** → Validate → Ship
```

## When QA Triggers

The agent offers browser QA when:
- A prototype has been finalized and the PM wants to verify it works
- Code has been scaffolded (Bridge phase complete) and deployed to a beta/gamma URL
- The PM explicitly asks: "test this", "QA the feature", "check if it works"
- A PRD has acceptance criteria that can be verified in a browser

## Prerequisites — What the Agent Checks

Before running browser QA, the agent verifies:

1. **Playwright MCP is available** — check if `playwright` server is in MCP config and responding
2. **Target URL exists** — ask the PM for the URL if not already known:
   > "What URL should I test against? (e.g., `http://localhost:3000` or a beta environment URL)"
3. **Acceptance criteria exist** — pull from the PRD or ask the PM what to verify

If Playwright MCP is NOT available:
> "Browser QA requires Playwright MCP. To set it up, run: `npx @playwright/mcp@latest` — then I can navigate your app and run tests. Want me to generate a test plan instead that you can run manually?"

## QA Execution Flow

### Step 1: Generate Test Scenarios from PRD

Extract acceptance criteria from the PRD and convert to browser test scenarios:

```markdown
## Test Scenarios for: {feature-name}

### Scenario 1: {user story title}
- **Given**: {precondition}
- **When**: {action — what to click/type/navigate}
- **Then**: {expected outcome — what should be visible/changed}
- **Screenshot**: capture after "Then" step

### Scenario 2: ...
```

Always include these default scenarios:
- **Happy path** — primary user flow works end to end
- **Empty state** — page loads correctly with no data
- **Error state** — graceful handling when something fails (disconnect network, invalid input)
- **Loading state** — something visible while data fetches
- **Responsive** — viewport at 1440px, 1024px, and 768px

### Step 2: Execute in Browser

Using Playwright MCP tools:
1. **Navigate** to the target URL
2. **Wait** for page load (network idle or specific element visible)
3. **Execute** each test scenario step by step
4. **Screenshot** after each key state change
5. **Record** pass/fail for each scenario
6. **Check accessibility** — run axe-core scan on each page state

### Step 3: Report Results

Generate a structured QA report:

```markdown
# QA Report: {feature-name}
**Date**: {timestamp}
**Target URL**: {url}
**Agent**: pm-builder-agent
**PRD**: {link to PRD}

## Summary
- ✅ Passed: {N}/{total}
- ❌ Failed: {N}/{total}
- ⚠️ Accessibility issues: {N}

## Results

### ✅ Scenario 1: {title}
- Steps executed successfully
- Screenshot: [link to screenshot]

### ❌ Scenario 2: {title}
- **Failed at step**: {step description}
- **Expected**: {what should have happened}
- **Actual**: {what actually happened}
- **Screenshot**: [link to screenshot]
- **Suggested fix**: {what might be wrong}

### ⚠️ Accessibility
- {violation 1}: {element} — {impact level}
- {violation 2}: {element} — {impact level}

## Recommendations
- {what needs fixing before ship}
- {what's acceptable to ship with and fix later}
```

Save report to: `docs/qa-reports/{feature-name}-{date}.md`
Save screenshots to: `docs/qa-reports/screenshots/`

### Step 4: Act on Results

**If all pass:**
> "All test scenarios passed. Screenshots saved. Ready to move to Ship phase?"

**If some fail:**
> "Found {N} issues. Here's what failed: [summary]. Want me to:
> 1. Fix the code and re-test (if it's scaffolded code I own)
> 2. File tickets for engineering to fix
> 3. Run QA again after you fix it"

**If accessibility issues found:**
> "Found {N} accessibility violations. {critical count} are WCAG AA blockers. Want me to fix those before handoff?"

## Accessibility QA (Built-In)

Every browser QA run includes accessibility checks:
- **axe-core scan** — WCAG 2.1 AA violations
- **Keyboard navigation** — Tab through all interactive elements, verify focus indicators
- **Color contrast** — check text against backgrounds (4.5:1 ratio minimum)
- **Screen reader landmarks** — verify proper heading hierarchy and ARIA labels
- **Focus management** — modals/drawers trap focus, route changes move focus

Report accessibility separately from functional QA — PMs need to know which a11y issues are launch-blockers vs nice-to-haves.

## Responsive QA

Test at three breakpoints by default:
- **Desktop**: 1440 × 900
- **Tablet**: 1024 × 768
- **Mobile**: 375 × 812

For each breakpoint, verify:
- Layout doesn't break (no horizontal scroll, no overlapping elements)
- Navigation is accessible (hamburger menu works on mobile)
- Text is readable (no truncation that hides meaning)
- Touch targets are adequate on mobile (44×44px minimum)

## QA Without Playwright MCP (Fallback)

If Playwright MCP isn't available, the agent still provides value:

1. **Generate test plan** — structured scenarios from the PRD that the PM or QA engineer can execute manually
2. **Generate Cypress tests** — write `.cy.ts` files that engineering can run in their pipeline
3. **Review screenshots** — if the PM pastes screenshots, analyze them against acceptance criteria
4. **Checklist mode** — provide an interactive checklist the PM walks through manually:

```markdown
## Manual QA Checklist: {feature-name}

### Functional
- [ ] Page loads without errors
- [ ] Primary action (button/form) works
- [ ] Data displays correctly
- [ ] Error state shows friendly message
- [ ] Empty state has guidance for the user
- [ ] Loading indicator appears during fetch

### Accessibility
- [ ] Can Tab to all interactive elements
- [ ] Focus indicator is visible
- [ ] Screen reader announces page content logically
- [ ] Color is not the only indicator of state

### Responsive
- [ ] Desktop (1440px) — layout intact
- [ ] Tablet (1024px) — layout adapts
- [ ] Mobile (375px) — usable without horizontal scroll
```

## Integration with the PM Lifecycle

### Phase Tracker Update

After QA completes, the status tracker shows:

```
│ Problem │  Plan  │ Build  │ Bridge │   QA   │ Validate │  Ship  │
│   ✅    │  ✅    │  ✅    │  ✅    │ 🔵 NOW │    ○     │   ○    │
```

### QA → Validate Transition

After QA passes:
> "QA complete — {N}/{N} scenarios passed. Want me to run a formal review (Developer/Product/UX/Leadership) on the PRD before shipping, or move straight to tracking?"

### QA → Bridge (Loop Back)

If QA reveals code issues the agent can fix:
> "Found issues in the scaffold. Let me fix and re-run QA."
The agent fixes code, re-runs the self-review loop (SE → PE), then re-runs QA. Maximum 2 fix-and-retest cycles before escalating to engineering.

## Rules for Browser QA

- **Never test production** — only localhost, beta, or gamma environments
- **Never submit real data** — use obvious test data ("Test User", "test@example.com")
- **Never authenticate with real credentials** — if auth is needed, ask the PM how to bypass (feature flag, test account, mock auth)
- **Always screenshot** — visual evidence for every pass and every fail
- **Always check accessibility** — it's not optional, it's built into every run
- **Report honestly** — don't mark something as passed if it's "mostly working"
- **Respect rate limits** — don't hammer the dev server with rapid requests
- **Clean up** — if the test created any data (form submissions, etc.), note it so it can be cleaned

## Playwright MCP Tools Reference

When Playwright MCP is available, the agent uses these capabilities:
- `browser_navigate` — go to a URL
- `browser_click` — click an element (by text, role, or selector)
- `browser_type` — type into an input field
- `browser_screenshot` — capture the current viewport
- `browser_wait` — wait for an element or network idle
- `browser_evaluate` — run JavaScript in the page (for axe-core, reading state)
- `browser_resize` — change viewport for responsive testing
- `browser_get_text` — read text content of elements
- `browser_select` — select dropdown options

The agent chains these tools to execute test scenarios step by step, taking screenshots at each validation point.

---
*Built with pm-builder-agent*
