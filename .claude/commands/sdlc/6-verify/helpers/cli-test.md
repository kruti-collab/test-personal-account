---
description: Run CLI manual tests via manual-tester agent against any environment
argument-hint: "[pr-number|environment]"
allowed-tools: Task, Bash
---

# CLI Manual Testing Helper

**Purpose**: Standalone CLI testing via manual-tester agent for any environment.

## Usage

```bash
# Test current PR's preview
/sdlc:6-verify:helpers:cli-test

# Test specific PR's preview
/sdlc:6-verify:helpers:cli-test 523

# Test specific environment
/sdlc:6-verify:helpers:cli-test dev-local
/sdlc:6-verify:helpers:cli-test staging
/sdlc:6-verify:helpers:cli-test production
```

## Why This Helper Exists

**Problem**: CLI testing logic is duplicated:
- Phase 6 Step 2 (full verification)
- Phase 4 manual testing (pre-PR)
- Phase 8 maintenance (post-deploy)

**Solution**: Extract into reusable helper.

**Benefits**:
- Consistent CLI test suite
- Reusable across phases
- Standalone testing capability
- Delegates to manual-tester (preserves context)

## CLI Test Suite

All CLI tests use these 4 standard tests:

| Test | Command | Expected |
|------|---------|----------|
| Version | `gal --version` | Version number `/\d+\.\d+\.\d+/` |
| Auth Status | `gal auth status --json` | JSON with `authenticated` field |
| Sync Status | `gal sync status --json` | JSON with `synced`/`neverSynced` |
| Demo Sync | `gal sync --pull --demo` | Output contains `DEMO` or `demo mode` |

## Implementation

```bash
# Parse argument (PR number or environment)
ARG="${1:-}"

# Default environment
ENVIRONMENT="dev-local"
API_URL=""
PR_NUMBER=""

# Determine if argument is PR number or environment
if [[ "$ARG" =~ ^[0-9]+$ ]]; then
  # Argument is PR number
  PR_NUMBER="$ARG"
  ENVIRONMENT="staging-preview"
elif [ -n "$ARG" ]; then
  # Argument is environment name
  ENVIRONMENT="$ARG"
fi

# Auto-detect PR if environment is staging-preview
if [ "$ENVIRONMENT" = "staging-preview" ] && [ -z "$PR_NUMBER" ]; then
  PR_NUMBER=$(gh pr view --json number --jq '.number' 2>/dev/null || echo "")
  if [ -z "$PR_NUMBER" ]; then
    echo "❌ Cannot determine PR number for staging-preview environment"
    echo "   Usage: /sdlc:6-verify:helpers:cli-test [pr-number]"
    exit 1
  fi
fi

# Determine API URL based on environment
case "$ENVIRONMENT" in
  dev-local)
    API_URL="http://localhost:3000"
    ;;
  staging-preview)
    if [ -z "$PR_NUMBER" ]; then
      echo "❌ PR number required for staging-preview environment"
      exit 1
    fi
    API_URL="https://pr-${PR_NUMBER}---gal-api-wug5dzqj2a-uc.a.run.app"
    ;;
  staging)
    API_URL="https://gal-api-staging-wug5dzqj2a-uc.a.run.app"
    ;;
  production)
    API_URL="https://api.gal.run"
    ;;
  *)
    echo "❌ Invalid environment: $ENVIRONMENT"
    echo "   Valid: dev-local, staging-preview, staging, production"
    exit 1
    ;;
esac

echo "┌─────────────────────────────────────────────┐"
echo "│  CLI MANUAL TESTING                         │"
echo "├─────────────────────────────────────────────┤"
echo "│  Environment: $ENVIRONMENT"
echo "│  API URL: $API_URL"
if [ -n "$PR_NUMBER" ]; then
echo "│  PR: #$PR_NUMBER"
fi
echo "└─────────────────────────────────────────────┘"
echo ""
```

### Prerequisite Checks

```bash
# Check if API is accessible (quick ping)
echo "🔍 Checking API accessibility..."

STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$API_URL/health" 2>/dev/null || echo "000")

if [ "$STATUS" != "200" ]; then
  echo "⚠️  API not accessible (status: $STATUS)"
  echo ""
  echo "Possible causes:"
  case "$ENVIRONMENT" in
    dev-local)
      echo "  - Local server not running"
      echo "  - Run: ./scripts/run.sh"
      ;;
    staging-preview)
      echo "  - Preview deployment not ready"
      echo "  - Run: /sdlc:6-verify:helpers:preview-check $PR_NUMBER"
      ;;
    staging|production)
      echo "  - Service may be down"
      echo "  - Check deployment status"
      ;;
  esac
  echo ""
  echo "Proceeding anyway (tests may fail)..."
fi
echo ""
```

### Delegate to manual-tester Agent

```bash
echo "🧪 Running CLI tests via manual-tester agent..."
echo ""

Task(
  subagent_type: "manual-tester",
  description: "CLI manual testing for $ENVIRONMENT",
  prompt: "Test GAL CLI against $ENVIRONMENT environment.

ENVIRONMENT DETAILS:
- Environment: $ENVIRONMENT
- API URL: $API_URL
$([ -n "$PR_NUMBER" ] && echo "- PR: #$PR_NUMBER")

CLI TEST SUITE (4 tests):

1. Version Check
   Command: gal --version
   Expected: Version number matching /\\d+\\.\\d+\\.\\d+/
   Validation: Output shows semver format (e.g., 1.2.3)

2. Auth Status
   Command: GAL_API_URL=$API_URL gal auth status --json
   Expected: JSON output with 'authenticated' field
   Validation: Parses as valid JSON, has boolean authenticated field

3. Sync Status
   Command: GAL_API_URL=$API_URL gal sync status --json
   Expected: JSON output with 'synced' or 'neverSynced' field
   Validation: Parses as valid JSON, has sync status field

4. Demo Sync
   Command: gal sync --pull --demo
   Expected: Output contains 'DEMO' or 'demo mode' indicator
   Validation: String matching confirms demo mode active

EXECUTION INSTRUCTIONS:
- Use tmux for terminal command execution
- Capture full output for each command
- Note any errors, warnings, or unexpected output
- Take screenshot of terminal if visual verification needed
- Test in clean shell environment (no cached auth)

REPORT FORMAT:
For each test:
  - Test Name: [name]
  - Command: [exact command run]
  - Exit Code: [0 or error code]
  - Output: [stdout]
  - Stderr: [stderr if any]
  - Result: PASS or FAIL
  - Notes: [any observations]

Summary:
  - Total: 4 tests
  - Passed: [count]
  - Failed: [count]
  - Overall: PASS (if all pass) or FAIL
  - Issues Found: [list if any]

If any test fails, provide troubleshooting suggestions."
)
```

### Post-Test Summary

```bash
echo ""
echo "┌─────────────────────────────────────────────┐"
echo "│  CLI TESTING COMPLETE                       │"
echo "└─────────────────────────────────────────────┘"
echo ""
echo "Test results delegated to manual-tester agent."
echo "Check agent output for detailed results."
echo ""
echo "Expected Results:"
echo "  - 4 tests run"
echo "  - All tests PASS"
echo "  - No errors or warnings"
echo ""
echo "If tests fail:"
case "$ENVIRONMENT" in
  dev-local)
    echo "  1. Verify local server is running: ./scripts/run.sh"
    echo "  2. Check API health: curl http://localhost:3000/health"
    echo "  3. Rebuild CLI: cd apps/cli && pnpm build"
    ;;
  staging-preview)
    echo "  1. Verify preview deployed: /sdlc:6-verify:helpers:preview-check $PR_NUMBER"
    echo "  2. Check CI status: gh pr checks $PR_NUMBER"
    echo "  3. Review deployment logs: gh run list --workflow orchestrate-preview.yml"
    ;;
  staging)
    echo "  1. Check staging deployment status"
    echo "  2. Verify API health: curl $API_URL/health"
    echo "  3. Check recent deployments: gh run list --workflow orchestrate-staging.yml"
    ;;
  production)
    echo "  1. Check production status page"
    echo "  2. Verify API health: curl $API_URL/health"
    echo "  3. Review incident reports"
    ;;
esac
```

## Output Example (Success Flow)

```
┌─────────────────────────────────────────────┐
│  CLI MANUAL TESTING                         │
├─────────────────────────────────────────────┤
│  Environment: staging-preview                        │
│  API URL: https://pr-523---gal-api...       │
│  PR: #523                                   │
└─────────────────────────────────────────────┘

🔍 Checking API accessibility...
API accessible (status: 200)

🧪 Running CLI tests via manual-tester agent...

[manual-tester agent output appears here]

┌─────────────────────────────────────────────┐
│  CLI TESTING COMPLETE                       │
└─────────────────────────────────────────────┘

Test results delegated to manual-tester agent.
Expected Results:
  - 4 tests run
  - All tests PASS
  - No errors or warnings
```

## Integration with Phases

### Phase 6 (Verify)

```markdown
## Step 2: Manual CLI Testing

Skill(skill: "sdlc:6-verify:helpers:cli-test", args: "$PR_NUMBER")

If CLI tests fail → Fix issues and re-run.
If CLI tests pass → Continue to Step 3 (UI testing).
```

### Phase 4 (Implement)

```markdown
## Step 10.5: Manual Testing

# Test CLI only
Skill(skill: "sdlc:6-verify:helpers:cli-test", args: "dev-local")
```

### Phase 8 (Maintain)

```markdown
## Post-Deploy Verification

# Verify CLI works in production
Skill(skill: "sdlc:6-verify:helpers:cli-test", args: "production")
```

## Standalone Usage Examples

```bash
# Quick local CLI check after bug fix
/sdlc:6-verify:helpers:cli-test dev-local

# Verify staging after deployment
/sdlc:6-verify:helpers:cli-test staging

# Test PR preview before full verification
/sdlc:6-verify:helpers:cli-test 523

# Production smoke test
/sdlc:6-verify:helpers:cli-test production
```

## Skip Criteria

Can skip CLI testing when:

| Condition | Reason |
|-----------|--------|
| Test-only PR | No CLI/API code changes |
| Documentation-only PR | No functional changes |
| UI-only PR (no API) | CLI not affected |

**Document skip reason explicitly:**
```
CLI Testing: SKIPPED
Reason: Test-only PR with no CLI/API changes
```

## Common Failures and Fixes

| Failure | Likely Cause | Fix |
|---------|--------------|-----|
| Version check fails | CLI not built | `cd apps/cli && pnpm build` |
| Auth status timeout | API not running | Start API or wait for deployment |
| Sync status 404 | Wrong API URL | Verify environment URL |
| Demo sync fails | Backend issue | Check API logs |

## Related Helpers

- `/sdlc:6-verify:helpers:preview-check` - Verify API is ready first
- `/sdlc:6-verify:helpers:ui-test` - Pair with CLI testing
- `/sdlc:4-implement:helpers:manual-test` - Full manual testing (CLI + UI)
