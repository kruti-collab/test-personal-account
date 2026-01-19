---
description: Delegate manual testing to manual-tester agent for UI/CLI verification
argument-hint: "[test-type] [environment]"
allowed-tools: Task, Bash
---

# Manual Testing Helper

**Purpose**: Run manual testing via manual-tester agent without full implementation workflow.

## Usage

```bash
# Test CLI commands
/sdlc:4-implement:helpers:manual-test cli dev-local

# Test UI flows
/sdlc:4-implement:helpers:manual-test ui dev-local

# Test both
/sdlc:4-implement:helpers:manual-test all staging
```

## Arguments

- `$1`: Test type (required)
  - `cli` - CLI commands only
  - `ui` - UI flows only
  - `all` - Both CLI and UI
- `$2`: Environment (optional, default: dev-local)
  - `dev-local` - Local development
  - `staging-preview` - PR preview
  - `staging` - Staging environment

## Why This Helper Exists

**Problem**: Phase 4 run.md includes manual testing inline, bloating context with:
- Playwright MCP calls
- tmux session management
- Screenshot handling
- Test result aggregation

**Solution**: Delegate ALL manual testing to the `manual-tester` agent.

**Benefits**:
- Reduces Phase 4 run.md by ~100 lines
- Enables standalone manual testing
- Can test implementation without full workflow
- Preserves main conversation context

## Implementation

```bash
# Parse arguments
TEST_TYPE="${1:-all}"
ENVIRONMENT="${2:-dev-local}"

# Validate test type
if [[ ! "$TEST_TYPE" =~ ^(cli|ui|all)$ ]]; then
  echo "❌ Invalid test type: $TEST_TYPE"
  echo "   Valid: cli, ui, all"
  exit 1
fi

# Determine URLs based on environment
case "$ENVIRONMENT" in
  dev-local)
    API_URL="http://localhost:3000"
    DASHBOARD_URL="http://localhost:5173"
    ;;
  staging-preview)
    PR_NUMBER=$(gh pr view --json number --jq '.number' 2>/dev/null || echo "")
    if [ -z "$PR_NUMBER" ]; then
      echo "❌ Cannot determine PR number for staging-preview environment"
      exit 1
    fi
    API_URL="https://pr-${PR_NUMBER}---gal-api-wug5dzqj2a-uc.a.run.app"
    DASHBOARD_URL="https://pr-${PR_NUMBER}--gal-staging-dashboard.web.app"
    ;;
  staging)
    API_URL="https://gal-api-staging-wug5dzqj2a-uc.a.run.app"
    DASHBOARD_URL="https://gal-staging-dashboard.web.app"
    ;;
  production)
    API_URL="https://api.gal.run"
    DASHBOARD_URL="https://app.gal.run"
    ;;
  *)
    echo "❌ Invalid environment: $ENVIRONMENT"
    exit 1
    ;;
esac

echo "┌─────────────────────────────────────────────┐"
echo "│  MANUAL TESTING                             │"
echo "├─────────────────────────────────────────────┤"
echo "│  Test Type: $TEST_TYPE"
echo "│  Environment: $ENVIRONMENT"
echo "│  API: $API_URL"
echo "│  Dashboard: $DASHBOARD_URL"
echo "└─────────────────────────────────────────────┘"
echo ""
```

### CLI Testing

```bash
if [ "$TEST_TYPE" = "cli" ] || [ "$TEST_TYPE" = "all" ]; then
  echo "🔧 Running CLI tests via manual-tester agent..."

  Task(
    subagent_type: "manual-tester",
    description: "CLI manual testing for $ENVIRONMENT",
    prompt: "Test GAL CLI against $ENVIRONMENT:

    Environment: $ENVIRONMENT
    API URL: $API_URL

    TESTS:
    1. Version check:
       - Command: gal --version
       - Expected: Version number matching /\\d+\\.\\d+\\.\\d+/

    2. Auth status:
       - Command: GAL_API_URL=$API_URL gal auth status --json
       - Expected: JSON with 'authenticated' field

    3. Sync status:
       - Command: GAL_API_URL=$API_URL gal sync status --json
       - Expected: JSON with 'synced' or 'neverSynced' field

    4. Demo sync:
       - Command: gal sync --pull --demo
       - Expected: Output contains 'DEMO' or 'demo mode'

    REPORT FORMAT:
    - Test 1: PASS/FAIL + output
    - Test 2: PASS/FAIL + output
    - Test 3: PASS/FAIL + output
    - Test 4: PASS/FAIL + output
    - Overall: PASS/FAIL
    - Issues Found: [list any errors]"
  )
fi
```

### UI Testing

```bash
if [ "$TEST_TYPE" = "ui" ] || [ "$TEST_TYPE" = "all" ]; then
  echo "🌐 Running UI tests via manual-tester agent..."

  Task(
    subagent_type: "manual-tester",
    description: "UI manual testing for $ENVIRONMENT",
    prompt: "Test GAL Dashboard at $DASHBOARD_URL:

    Environment: $ENVIRONMENT
    Dashboard URL: $DASHBOARD_URL

    TESTS:
    1. Homepage load:
       - Navigate to $DASHBOARD_URL
       - Expected: NOT redirected to /login

    2. Dashboard content:
       - Wait for content to load
       - Expected: Shows org name OR onboarding flow

    3. Get Started page:
       - Navigate to $DASHBOARD_URL/get-started
       - Expected: Page loads without error, no 404/500

    4. Settings page:
       - Navigate to $DASHBOARD_URL/settings
       - Expected: Shows user data or settings form

    INSTRUCTIONS:
    - Use Playwright MCP for browser automation
    - Take screenshots as evidence
    - Check browser console for errors

    REPORT FORMAT:
    - Test 1: PASS/FAIL + screenshot
    - Test 2: PASS/FAIL + screenshot
    - Test 3: PASS/FAIL + screenshot
    - Test 4: PASS/FAIL + screenshot
    - Overall: PASS/FAIL
    - Console Errors: [list any errors]
    - Anomalies: [list any unexpected behavior]"
  )
fi
```

### Final Report

```bash
echo ""
echo "┌─────────────────────────────────────────────┐"
echo "│  MANUAL TESTING COMPLETE                    │"
echo "└─────────────────────────────────────────────┘"
echo ""
echo "Results have been delegated to manual-tester agent."
echo "Check agent output for detailed test results."
echo ""
echo "Next Steps:"
if [ "$TEST_TYPE" = "cli" ]; then
  echo "  - Review CLI test results"
  echo "  - If tests pass: Continue to UI testing"
  echo "  - If tests fail: Fix CLI issues"
elif [ "$TEST_TYPE" = "ui" ]; then
  echo "  - Review UI test results"
  echo "  - If tests pass: Continue to commit/PR"
  echo "  - If tests fail: Fix UI issues"
else
  echo "  - Review both CLI and UI test results"
  echo "  - If all pass: Continue to commit/PR"
  echo "  - If any fail: Fix issues and re-test"
fi
```

## Integration with Phase 4

Phase 4 run.md can now delegate manual testing:

```markdown
## Step 10.5: Manual Testing (REQUIRED)

Instead of inline testing, run:

Skill(skill: "sdlc:4-implement:helpers:manual-test", args: "all dev-local")

This delegates to manual-tester agent and preserves context.
```

## Standalone Usage

Can also be used independently:

```bash
# Quick CLI check after bug fix
/sdlc:4-implement:helpers:manual-test cli dev-local

# Verify UI after merge
/sdlc:4-implement:helpers:manual-test ui staging
```

## Error Handling

```bash
# If environment not accessible
if ! curl -s "$API_URL/health" > /dev/null 2>&1; then
  echo "⚠️  API not accessible at $API_URL"
  echo "   - If dev-local: Run ./scripts/run.sh first"
  echo "   - If staging-preview: Wait for preview deployment"
  echo "   - If staging: Check deployment status"
  exit 1
fi
```
