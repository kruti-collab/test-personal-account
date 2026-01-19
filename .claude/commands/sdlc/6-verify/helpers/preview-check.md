---
description: Verify PR preview deployment is ready before manual testing
argument-hint: "[pr-number]"
allowed-tools: Bash, Read
---

# Preview Deployment Readiness Check

**Purpose**: Verify PR preview is deployed and healthy before running manual tests.

## Usage

```bash
# Check current PR's preview
/sdlc:6-verify:helpers:preview-check

# Check specific PR's preview
/sdlc:6-verify:helpers:preview-check 523
```

## Why This Helper Exists

**Problem**: Phase 6 run.md has complex preview readiness logic:
- URL construction based on PR number
- Health check polling with retries
- Timeout handling
- Environment-specific URL patterns

**Solution**: Extract into reusable helper.

**Benefits**:
- Can verify preview independently
- Reusable in Phase 5 (review) and Phase 7 (deploy)
- Clear pass/fail output
- Reduces Phase 6 run.md by ~80 lines

## Implementation

```bash
# Parse arguments
PR_NUMBER="${1:-}"

# Auto-detect PR if not provided
if [ -z "$PR_NUMBER" ]; then
  PR_NUMBER=$(gh pr view --json number --jq '.number' 2>/dev/null || echo "")
fi

if [ -z "$PR_NUMBER" ]; then
  echo "❌ Cannot determine PR number"
  echo "   Usage: /sdlc:6-verify:helpers:preview-check [pr-number]"
  exit 1
fi

echo "┌─────────────────────────────────────────────┐"
echo "│  PREVIEW DEPLOYMENT CHECK                   │"
echo "├─────────────────────────────────────────────┤"
echo "│  PR: #$PR_NUMBER"
echo "└─────────────────────────────────────────────┘"
echo ""
```

### Determine Environment

```bash
# Get base branch to determine environment
BASE_BRANCH=$(gh pr view $PR_NUMBER --json baseRefName --jq '.baseRefName')

case "$BASE_BRANCH" in
  staging)
    API_URL="https://pr-${PR_NUMBER}---gal-api-wug5dzqj2a-uc.a.run.app"
    DASHBOARD_URL="https://pr-${PR_NUMBER}--gal-staging-dashboard.web.app"
    ENVIRONMENT="staging-preview"
    ;;
  main)
    # Main PRs use staging environment (no preview)
    API_URL="https://gal-api-staging-wug5dzqj2a-uc.a.run.app"
    DASHBOARD_URL="https://gal-staging-dashboard.web.app"
    ENVIRONMENT="staging"
    ;;
  *)
    # Default to PR preview pattern
    API_URL="https://pr-${PR_NUMBER}---gal-api-wug5dzqj2a-uc.a.run.app"
    DASHBOARD_URL="https://pr-${PR_NUMBER}--gal-staging-dashboard.web.app"
    ENVIRONMENT="staging-preview"
    ;;
esac

echo "🌐 Target Environment: $ENVIRONMENT"
echo "   API: $API_URL"
echo "   Dashboard: $DASHBOARD_URL"
echo ""
```

### Initial Propagation Delay

```bash
echo "⏳ Waiting for preview deployment to propagate..."
sleep 30  # CDN/DNS propagation
echo ""
```

### API Health Check

```bash
echo "🔍 Checking API health..."

API_READY=false
MAX_RETRIES=5
RETRY_INTERVAL=10

for i in $(seq 1 $MAX_RETRIES); do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$API_URL/health" 2>/dev/null || echo "000")

  if [ "$STATUS" = "200" ]; then
    echo "✅ API healthy (attempt $i/$MAX_RETRIES)"
    echo "   Status: $STATUS"
    echo "   URL: $API_URL"
    API_READY=true
    break
  else
    echo "⏳ Attempt $i/$MAX_RETRIES: API status $STATUS"
    if [ $i -lt $MAX_RETRIES ]; then
      echo "   Retrying in ${RETRY_INTERVAL}s..."
      sleep $RETRY_INTERVAL
    fi
  fi
done

if [ "$API_READY" = false ]; then
  echo ""
  echo "⚠️  API not ready after $MAX_RETRIES attempts"
  echo "   Last status: $STATUS"
  echo ""
  echo "Possible causes:"
  echo "  - Deployment still in progress"
  echo "  - API failed to start"
  echo "  - Cloud Run service not deployed"
  echo ""
  echo "Check deployment status:"
  echo "  gh run list --workflow orchestrate-preview.yml --limit 3"
fi

echo ""
```

### Dashboard Accessibility Check

```bash
echo "🔍 Checking Dashboard accessibility..."

DASHBOARD_READY=false
MAX_RETRIES=3

for i in $(seq 1 $MAX_RETRIES); do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$DASHBOARD_URL" 2>/dev/null || echo "000")

  if [ "$STATUS" = "200" ] || [ "$STATUS" = "304" ]; then
    echo "✅ Dashboard accessible (attempt $i/$MAX_RETRIES)"
    echo "   Status: $STATUS"
    echo "   URL: $DASHBOARD_URL"
    DASHBOARD_READY=true
    break
  else
    echo "⏳ Attempt $i/$MAX_RETRIES: Dashboard status $STATUS"
    if [ $i -lt $MAX_RETRIES ]; then
      sleep 10
    fi
  fi
done

if [ "$DASHBOARD_READY" = false ]; then
  echo ""
  echo "⚠️  Dashboard not accessible after $MAX_RETRIES attempts"
  echo "   Last status: $STATUS"
  echo ""
  echo "Possible causes:"
  echo "  - Firebase preview channel not created"
  echo "  - Deployment failed"
  echo "  - CDN propagation still in progress"
  echo ""
  echo "Check deployment status:"
  echo "  gh run list --workflow orchestrate-preview.yml --limit 3"
fi

echo ""
```

### Final Assessment

```bash
echo "┌─────────────────────────────────────────────┐"
echo "│  PREVIEW READINESS ASSESSMENT               │"
echo "├─────────────────────────────────────────────┤"

if [ "$API_READY" = true ] && [ "$DASHBOARD_READY" = true ]; then
  echo "│  Status: ✅ READY                           │"
  echo "├─────────────────────────────────────────────┤"
  echo "│  API: ✅ Healthy                            │"
  echo "│  Dashboard: ✅ Accessible                   │"
  echo "└─────────────────────────────────────────────┘"
  echo ""
  echo "Next Steps:"
  echo "  - Run CLI tests: /sdlc:6-verify:helpers:cli-test $PR_NUMBER"
  echo "  - Run UI tests: /sdlc:6-verify:helpers:ui-test $PR_NUMBER"
  echo "  - Or full verify: /sdlc:6-verify:run $PR_NUMBER"
  exit 0

elif [ "$API_READY" = true ]; then
  echo "│  Status: ⚠️  PARTIAL (API only)             │"
  echo "├─────────────────────────────────────────────┤"
  echo "│  API: ✅ Healthy                            │"
  echo "│  Dashboard: ❌ Not accessible               │"
  echo "└─────────────────────────────────────────────┘"
  echo ""
  echo "Can proceed with CLI testing only."
  echo "  - Run CLI tests: /sdlc:6-verify:helpers:cli-test $PR_NUMBER"
  echo ""
  echo "Dashboard verification blocked."
  exit 1

elif [ "$DASHBOARD_READY" = true ]; then
  echo "│  Status: ⚠️  PARTIAL (Dashboard only)       │"
  echo "├─────────────────────────────────────────────┤"
  echo "│  API: ❌ Not healthy                        │"
  echo "│  Dashboard: ✅ Accessible                   │"
  echo "└─────────────────────────────────────────────┘"
  echo ""
  echo "Can proceed with UI testing (API may not work)."
  echo "  - Run UI tests: /sdlc:6-verify:helpers:ui-test $PR_NUMBER"
  echo ""
  echo "CLI verification blocked."
  exit 1

else
  echo "│  Status: ❌ NOT READY                       │"
  echo "├─────────────────────────────────────────────┤"
  echo "│  API: ❌ Not healthy                        │"
  echo "│  Dashboard: ❌ Not accessible               │"
  echo "└─────────────────────────────────────────────┘"
  echo ""
  echo "Preview deployment not ready for testing."
  echo ""
  echo "Troubleshooting:"
  echo "  1. Check workflow status:"
  echo "     gh run list --workflow orchestrate-preview.yml --limit 3"
  echo ""
  echo "  2. Check if workflow completed:"
  echo "     gh run view <run-id>"
  echo ""
  echo "  3. Retry this check in 2-3 minutes:"
  echo "     /sdlc:6-verify:helpers:preview-check $PR_NUMBER"
  exit 1
fi
```

## Output Example (Success)

```
┌─────────────────────────────────────────────┐
│  PREVIEW DEPLOYMENT CHECK                   │
├─────────────────────────────────────────────┤
│  PR: #523                                   │
└─────────────────────────────────────────────┘

🌐 Target Environment: staging-preview
   API: https://pr-523---gal-api-wug5dzqj2a-uc.a.run.app
   Dashboard: https://pr-523--gal-staging-dashboard.web.app

⏳ Waiting for preview deployment to propagate...

🔍 Checking API health...
✅ API healthy (attempt 2/5)
   Status: 200
   URL: https://pr-523---gal-api-wug5dzqj2a-uc.a.run.app

🔍 Checking Dashboard accessibility...
✅ Dashboard accessible (attempt 1/3)
   Status: 200
   URL: https://pr-523--gal-staging-dashboard.web.app

┌─────────────────────────────────────────────┐
│  PREVIEW READINESS ASSESSMENT               │
├─────────────────────────────────────────────┤
│  Status: ✅ READY                           │
├─────────────────────────────────────────────┤
│  API: ✅ Healthy                            │
│  Dashboard: ✅ Accessible                   │
└─────────────────────────────────────────────┘

Next Steps:
  - Run CLI tests: /sdlc:6-verify:helpers:cli-test 523
  - Run UI tests: /sdlc:6-verify:helpers:ui-test 523
  - Or full verify: /sdlc:6-verify:run 523
```

## Integration with Phase 6

Phase 6 run.md Step 1 can now be simplified to:

```markdown
## Step 1: Preview Readiness Check

Skill(skill: "sdlc:6-verify:helpers:preview-check")

If preview not ready → STOP and troubleshoot.
If preview ready → Continue to Step 2 (CLI testing).
```

## Standalone Usage

Useful for CI debugging:

```bash
# Check if preview is ready before triggering E2E tests
/sdlc:6-verify:helpers:preview-check $PR_NUMBER
```
