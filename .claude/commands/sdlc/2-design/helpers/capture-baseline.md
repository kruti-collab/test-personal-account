---
description: "Capture current UI as design baseline screenshot"
argument-hint: "<page-name> [--mode=dark|light|both]"
allowed-tools: ["Bash", "mcp__playwright__browser_navigate", "mcp__playwright__browser_take_screenshot", "mcp__playwright__browser_resize", "mcp__playwright__browser_close"]
---

# Capture Design Baseline

Capture the current UI state as a baseline for visual regression testing.

## Arguments

- `page-name`: Name of the page to capture
- `--mode`: dark (default), light, or both

## Usage

```
/design/capture-baseline dashboard
/design/capture-baseline discovery --mode=both
```

## Workflow

1. **Ensure Services Running**
   ```bash
   curl -s http://localhost:5173 > /dev/null || {
     echo "Start dashboard first: ./scripts/run.sh"
     exit 1
   }
   ```

2. **Create Baseline Directory**
   ```bash
   mkdir -p docs/design/baselines/{page-name}
   ```

3. **Navigate and Screenshot**
   Using Playwright MCP:
   ```
   mcp__playwright__browser_resize(1280, 720)
   mcp__playwright__browser_navigate(http://localhost:5173/{route})
   mcp__playwright__browser_take_screenshot(
     filename: "docs/design/baselines/{page-name}/dark.png"
   )
   ```

4. **Light Mode** (if --mode=light or --mode=both)
   - Toggle theme if available
   - Or inject light mode class
   - Screenshot as `light.png`

5. **Report**
   ```
   Baseline captured:
   - docs/design/baselines/{page-name}/dark.png
   - docs/design/baselines/{page-name}/light.png (if applicable)

   Commit these files to track visual changes.
   ```

## Page Routes

| Page Name | Route |
|-----------|-------|
| dashboard | / |
| discovery | /discovery |
| approved-config | /approved-config |
| settings | /settings |

## Notes

- Baselines should be captured at consistent viewport (1280x720)
- Commit baselines to git for version tracking
- Re-capture after intentional design changes
- Compare using `/design/compare` command
