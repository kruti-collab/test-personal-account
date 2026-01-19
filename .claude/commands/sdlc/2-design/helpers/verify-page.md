---
description: "Verify a page implementation matches its design spec"
argument-hint: "<page-name> [--mode=dark|light|both] [--headed]"
allowed-tools: ["Task", "Read", "Bash"]
---

# Design Page Verification

Verify that a page's UI implementation matches its design specification.

## Arguments

- `page-name`: Name of the page to verify (dashboard, discovery, approved-config, settings)
- `--mode`: dark (default), light, or both
- `--headed`: Show browser during verification

## Usage

```
/design/verify-page dashboard
/design/verify-page discovery --mode=both
/design/verify-page settings --headed
```

## Workflow

1. **Locate Design Spec**
   ```
   docs/design/pages/{page-name}.design.md
   ```
   If spec doesn't exist, report and exit.

2. **Start Services** (if not running)
   ```bash
   # Check if dashboard is running
   curl -s http://localhost:5173 > /dev/null || echo "Dashboard not running"
   ```

3. **Launch Design Verifier Agent**
   ```
   Task(subagent_type: "design-verifier", prompt: "
     Verify page against design spec:
     - page_url: http://localhost:5173/{route}
     - spec_path: docs/design/pages/{page-name}.design.md
     - mode: {mode}
   ")
   ```

4. **Collect Report**
   Display structured verification report to user.

5. **Decision Point**
   Based on verdict:
   - PASS: "Design verification passed!"
   - FAIL: "Found {N} issues. Fix UI or update spec?"
   - PARTIAL: "Minor issues found. Review recommended."

## Page Routes

| Page Name | Route |
|-----------|-------|
| dashboard | / |
| discovery | /discovery |
| approved-config | /approved-config |
| settings | /settings |
| billing | /billing |
| cli | /cli |
| docs | /docs |

## Example Output

```
Design Verification: Dashboard
==============================

Spec: docs/design/pages/dashboard.design.md
Mode: dark
URL: http://localhost:5173/

Results:
✓ Page Title: "Dashboard"
✓ Stats Cards: 3 found
✓ Feature Cards: 3 found
✓ Layout: Correct

VERDICT: PASS

Screenshot saved: docs/design/baselines/dashboard/current-dark.png
```

## Creating Missing Specs

If no design spec exists for a page:

```
No design spec found for: {page-name}

Would you like to:
1. Create a new design spec from current UI
2. Skip verification

Use /design/capture-spec {page-name} to generate spec from current implementation.
```
