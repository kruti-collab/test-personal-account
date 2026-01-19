---
description: Smart CI check waiting with timeout, progress updates, and failure detection
argument-hint: "[pr-number] [--timeout-minutes]"
allowed-tools: Bash
---

# CI Check Wait Helper

**Purpose**: Wait for ALL CI checks to pass with smart polling and failure detection.

## Usage

```bash
# Wait for current PR's CI
/sdlc:5-review:helpers:ci-wait

# Wait for specific PR
/sdlc:5-review:helpers:ci-wait 523

# Custom timeout (default: 60 minutes)
/sdlc:5-review:helpers:ci-wait 523 --timeout-minutes=30
```

## Why This Helper Exists

**Problem**: CI waiting logic is duplicated across:
- Phase 5 (review) - Step 0c
- Phase 6 (verify) - Step 1b
- Phase 7 (deploy) - Pre-deployment gate

**Solution**: Centralize smart CI waiting.

**Benefits**:
- DRY principle (Don't Repeat Yourself)
- Consistent timeout behavior
- Smart failure detection
- Progress updates every minute
- Reusable across phases

## Implementation

```bash
# Parse arguments
PR_NUMBER="${1:-}"
TIMEOUT_MINUTES=60

# Parse flags
for arg in "$@"; do
  if [[ "$arg" =~ --timeout-minutes=([0-9]+) ]]; then
    TIMEOUT_MINUTES="${BASH_REMATCH[1]}"
  fi
done

# Auto-detect PR if not provided
if [ -z "$PR_NUMBER" ]; then
  PR_NUMBER=$(gh pr view --json number --jq '.number' 2>/dev/null || echo "")
fi

if [ -z "$PR_NUMBER" ]; then
  echo "❌ Cannot determine PR number"
  echo "   Usage: /sdlc:5-review:helpers:ci-wait [pr-number] [--timeout-minutes=N]"
  exit 1
fi

echo "┌─────────────────────────────────────────────┐"
echo "│  CI CHECK WAIT                              │"
echo "├─────────────────────────────────────────────┤"
echo "│  PR: #$PR_NUMBER"
echo "│  Timeout: $TIMEOUT_MINUTES minutes"
echo "└─────────────────────────────────────────────┘"
echo ""
```

### Initial Status Check

```bash
echo "🔍 Checking initial CI status..."

# Unset GITHUB_TOKEN to avoid rate limit issues
unset GITHUB_TOKEN

# Get current check status
CHECKS_JSON=$(gh pr checks $PR_NUMBER --json name,state,conclusion 2>/dev/null)

if [ -z "$CHECKS_JSON" ] || [ "$CHECKS_JSON" = "[]" ]; then
  echo "⚠️  No CI checks found for PR #$PR_NUMBER"
  echo "   Either:"
  echo "   - CI hasn't started yet (push just happened)"
  echo "   - PR has no required checks configured"
  echo "   - PR is from a fork (checks may be delayed)"
  echo ""
  echo "Waiting 30s for checks to appear..."
  sleep 30
  CHECKS_JSON=$(gh pr checks $PR_NUMBER --json name,state,conclusion 2>/dev/null)
fi

# Calculate check counts
TOTAL=$(echo "$CHECKS_JSON" | jq 'length')
PASSING=$(echo "$CHECKS_JSON" | jq '[.[] | select(.conclusion == "success")] | length')
FAILING=$(echo "$CHECKS_JSON" | jq '[.[] | select(.conclusion == "failure")] | length')
PENDING=$(echo "$CHECKS_JSON" | jq '[.[] | select(.state == "PENDING" or .state == "IN_PROGRESS" or .state == "QUEUED")] | length')

echo ""
echo "📊 CI Status: PR #$PR_NUMBER"
echo "   ✅ Passing:  $PASSING"
echo "   ❌ Failing:  $FAILING"
echo "   ⏳ Pending:  $PENDING"
echo "   📋 Total:    $TOTAL"
echo ""
```

### Quick Exit if Already Done

```bash
if [ "$PENDING" -eq 0 ] && [ "$FAILING" -eq 0 ] && [ "$PASSING" -gt 0 ]; then
  echo "✅ All CI checks already passing!"
  exit 0
fi

if [ "$FAILING" -gt 0 ]; then
  echo "❌ CI checks already failing - cannot proceed"
  echo ""
  echo "Failed checks:"
  echo "$CHECKS_JSON" | jq -r '.[] | select(.conclusion == "failure") | "  - " + .name'
  echo ""
  echo "Fix failures first, then re-run this command."
  exit 1
fi
```

### Smart Polling Loop

```bash
echo "⏳ Waiting for CI checks to complete..."
echo "   Polling every 60s, timeout in $TIMEOUT_MINUTES minutes"
echo ""

TIMEOUT_SECONDS=$((TIMEOUT_MINUTES * 60))
INTERVAL=60
ELAPSED=0
LAST_PENDING=$PENDING

while [ $ELAPSED -lt $TIMEOUT_SECONDS ]; do
  # Fetch latest status
  CHECKS_JSON=$(gh pr checks $PR_NUMBER --json name,state,conclusion 2>/dev/null)

  # Recalculate counts
  TOTAL=$(echo "$CHECKS_JSON" | jq 'length')
  PASSING=$(echo "$CHECKS_JSON" | jq '[.[] | select(.conclusion == "success")] | length')
  FAILING=$(echo "$CHECKS_JSON" | jq '[.[] | select(.conclusion == "failure")] | length')
  PENDING=$(echo "$CHECKS_JSON" | jq '[.[] | select(.state == "PENDING" or .state == "IN_PROGRESS" or .state == "QUEUED")] | length')

  # Detect progress
  PROGRESS=""
  if [ "$PENDING" -lt "$LAST_PENDING" ]; then
    COMPLETED=$((LAST_PENDING - PENDING))
    PROGRESS=" (+$COMPLETED completed)"
  fi
  LAST_PENDING=$PENDING

  # Show progress update
  MINUTES_ELAPSED=$((ELAPSED / 60))
  MINUTES_REMAINING=$(((TIMEOUT_SECONDS - ELAPSED) / 60))
  echo "[$(date +%H:%M:%S)] [$MINUTES_ELAPSED/${TIMEOUT_MINUTES}m] Pass:$PASSING Fail:$FAILING Pending:$PENDING$PROGRESS"

  # Check exit conditions
  if [ "$FAILING" -gt 0 ]; then
    echo ""
    echo "❌ CI check failed!"
    echo ""
    echo "Failed checks:"
    echo "$CHECKS_JSON" | jq -r '.[] | select(.conclusion == "failure") | "  - " + .name'
    echo ""
    echo "View details:"
    echo "  gh run list --branch $(gh pr view $PR_NUMBER --json headRefName --jq '.headRefName') --limit 3"
    echo ""
    echo "After fixing, re-run:"
    echo "  /sdlc:5-review:helpers:ci-wait $PR_NUMBER"
    exit 1
  fi

  if [ "$PENDING" -eq 0 ] && [ "$PASSING" -gt 0 ]; then
    echo ""
    echo "✅ All CI checks passed!"
    echo ""
    echo "Final status:"
    echo "   ✅ Passing: $PASSING"
    echo "   📋 Total:   $TOTAL"
    echo ""
    exit 0
  fi

  # Wait before next poll
  sleep $INTERVAL
  ELAPSED=$((ELAPSED + INTERVAL))
done
```

### Timeout Handling

```bash
echo ""
echo "⏰ Timeout reached after $TIMEOUT_MINUTES minutes"
echo ""
echo "Current status:"
echo "   ✅ Passing:  $PASSING"
echo "   ❌ Failing:  $FAILING"
echo "   ⏳ Pending:  $PENDING"
echo ""

if [ "$PENDING" -gt 0 ]; then
  echo "Pending checks:"
  echo "$CHECKS_JSON" | jq -r '.[] | select(.state == "PENDING" or .state == "IN_PROGRESS" or .state == "QUEUED") | "  - " + .name + " (" + .state + ")"'
  echo ""
  echo "Possible issues:"
  echo "  - Workflow stuck or queued behind other runs"
  echo "  - Runner capacity exhausted"
  echo "  - Infinite loop in workflow"
  echo ""
  echo "Investigate:"
  echo "  gh run list --workflow <workflow-name> --limit 10"
  echo ""
  echo "Continue waiting:"
  echo "  /sdlc:5-review:helpers:ci-wait $PR_NUMBER --timeout-minutes=90"
fi

exit 1
```

## Output Example (Success)

```
┌─────────────────────────────────────────────┐
│  CI CHECK WAIT                              │
├─────────────────────────────────────────────┤
│  PR: #523                                   │
│  Timeout: 60 minutes                        │
└─────────────────────────────────────────────┘

🔍 Checking initial CI status...

📊 CI Status: PR #523
   ✅ Passing:  2
   ❌ Failing:  0
   ⏳ Pending:  3
   📋 Total:    5

⏳ Waiting for CI checks to complete...
   Polling every 60s, timeout in 60 minutes

[14:32:15] [0/60m] Pass:2 Fail:0 Pending:3
[14:33:15] [1/60m] Pass:3 Fail:0 Pending:2 (+1 completed)
[14:34:15] [2/60m] Pass:4 Fail:0 Pending:1 (+1 completed)
[14:35:15] [3/60m] Pass:5 Fail:0 Pending:0 (+1 completed)

✅ All CI checks passed!

Final status:
   ✅ Passing: 5
   📋 Total:   5
```

## Output Example (Failure)

```
[14:32:15] [0/60m] Pass:2 Fail:0 Pending:3
[14:33:15] [1/60m] Pass:2 Fail:1 Pending:2

❌ CI check failed!

Failed checks:
  - E2E Dashboard Tests

View details:
  gh run list --branch 523-oauth-preview-auth --limit 3

After fixing, re-run:
  /sdlc:5-review:helpers:ci-wait 523
```

## Integration with Phases

### Phase 5 (Review)

```markdown
## Step 0d: Wait for CI (BLOCKER)

Skill(skill: "sdlc:5-review:helpers:ci-wait")

If CI fails → STOP. Fix issues before code review.
If CI passes → Continue to Phase 0 (Prime Context).
```

### Phase 6 (Verify)

```markdown
## Step 1b: Wait for CI After Rebase

Skill(skill: "sdlc:5-review:helpers:ci-wait", args: "$PR_NUMBER")

Rebase may trigger new CI runs. Wait for them to complete.
```

### Phase 7 (Deploy)

```markdown
## Pre-Deployment Gate

Skill(skill: "sdlc:5-review:helpers:ci-wait", args: "--timeout-minutes=30")

Ensure CI is green before deployment.
```

## Advanced Usage

```bash
# Quick timeout for fast workflows
/sdlc:5-review:helpers:ci-wait --timeout-minutes=15

# Extended timeout for slow E2E tests
/sdlc:5-review:helpers:ci-wait --timeout-minutes=120
```

## Error Recovery

If command exits with failure:

1. **Check what failed**:
   ```bash
   gh run list --branch <branch> --limit 5
   gh run view <run-id>
   ```

2. **Fix the issue** (code, config, flaky test)

3. **Trigger re-run** (if CI supports it):
   ```bash
   gh run rerun <run-id>
   ```

4. **Re-run wait command**:
   ```bash
   /sdlc:5-review:helpers:ci-wait
   ```
