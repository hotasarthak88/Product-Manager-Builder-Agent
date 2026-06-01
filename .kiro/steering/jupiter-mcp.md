# Jupiter MCP — Always Use for UI Design

When the user asks to design, build, create, or prototype any UI component, page, form, dashboard, layout, or visual element:

1. **Always call `jupiter_design` first** with the user's requirements (and optional reference image) to generate a structured design JSON. Do NOT attempt to write UI code directly.
2. **Wait for design completion** — use `jupiter_design_status` to poll if the design is still in progress.
3. **Call `jupiter_generate_code`** with the executionId to get React implementation instructions.
4. **Implement the code** using components from `@amzn/uno-cloudscape-theme` — never import from `@cloudscape-design/components` directly.

## When to Use Jupiter MCP

- Any request involving UI mockups, prototypes, or wireframes
- Creating new pages, forms, tables, dashboards, or layouts
- Recreating a design from a screenshot or Figma reference
- Iterating on an existing UI design ("add a filter", "change the layout")
- Generating HTML mock files for review

## Component Imports

All Jupiter components MUST be imported from `@amzn/uno-cloudscape-theme`:

```tsx
import { Button, Form, FormField, Input, Container, Header } from "@amzn/uno-cloudscape-theme";
```

## Iterative Design

When refining an existing design, pass the previous design JSON as `existingDesign` to `jupiter_design` so changes are incremental rather than starting from scratch.
