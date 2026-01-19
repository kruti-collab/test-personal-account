---
description: "Generate design spec from current UI implementation"
argument-hint: "<page-name>"
allowed-tools: ["Task", "Read", "Write", "Bash", "mcp__playwright__browser_navigate", "mcp__playwright__browser_snapshot"]
---

# Capture Design Spec

Generate a design specification file from the current UI implementation. Use this to document existing pages or bootstrap new design specs.

## Arguments

- `page-name`: Name of the page to analyze

## Usage

```
/design/capture-spec dashboard
/design/capture-spec discovery
```

## Workflow

1. **Navigate to Page**
   ```
   mcp__playwright__browser_navigate(http://localhost:5173/{route})
   ```

2. **Capture Accessibility Snapshot**
   ```
   mcp__playwright__browser_snapshot()
   ```

3. **Analyze Snapshot**
   Extract from accessibility tree:
   - Headings (h1, h2, h3)
   - Interactive elements (buttons, links)
   - Structural elements (main, nav, aside)
   - Text content

4. **Generate Spec Template**
   ```markdown
   # Page: {PageName}

   > Auto-generated from UI on {date}

   ## Mode: dark (default)

   ### Required Elements

   | Element | Selector | Expected Text | Required |
   |---------|----------|---------------|----------|
   | Page Title | h1 | "{title}" | yes |
   | ... | ... | ... | ... |

   ### Layout

   - Header: {description}
   - Main content: {description}

   ### Colors

   | Element | Dark Mode | Light Mode |
   |---------|-----------|------------|
   | Background | (inspect) | (inspect) |
   | Text | (inspect) | (inspect) |

   ## Notes

   - Review and refine this auto-generated spec
   - Add specific color values after inspection
   - Mark optional elements appropriately
   ```

5. **Write Spec File**
   ```
   docs/design/pages/{page-name}.design.md
   ```

6. **Report**
   ```
   Design spec generated: docs/design/pages/{page-name}.design.md

   Next steps:
   1. Review generated spec
   2. Add specific styling requirements
   3. Run /design/verify-page {page-name} to validate
   ```

## Example Output Spec

```markdown
# Page: Dashboard

> Auto-generated from UI on 2025-01-15

## Mode: dark (default)

### Required Elements

| Element | Selector | Expected Text | Required |
|---------|----------|---------------|----------|
| Page Title | h1 | "Dashboard" | yes |
| Subtitle | p.text-gray-400 | "Overview of your AI agent governance" | yes |
| Org Stats | .dashboard-card:nth(1) | contains "Organizations" | yes |
| Config Stats | .dashboard-card:nth(2) | contains "Configs Found" | yes |
| Type Stats | .dashboard-card:nth(3) | contains "Config Types" | yes |
| Feature Card 1 | [data-testid="feature-card"]:nth(1) | "Auto-Discovery" | yes |
| Feature Card 2 | [data-testid="feature-card"]:nth(2) | "Approved Config" | yes |
| Feature Card 3 | [data-testid="feature-card"]:nth(3) | "CLI Sync" | yes |

### Layout

- Header: Sticky top navigation
- Main: max-w-6xl mx-auto, responsive padding
- Stats: 3-column grid on desktop, stack on mobile
- Features: 3-column grid on desktop, stack on mobile

### Colors

| Element | Dark Mode | Light Mode |
|---------|-----------|------------|
| Background | bg-gray-900 | TBD |
| Text Primary | text-white | TBD |
| Text Secondary | text-gray-400 | TBD |
| Accent | #00FF41 | TBD |
| Card Background | bg-white/10 | TBD |
```
