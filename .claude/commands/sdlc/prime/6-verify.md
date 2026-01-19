# Prime VERIFY - Load Verification Context

Prime context window with everything needed for deployment verification and merge.

## Step 1: Check PR State

```bash
# Get current PR info
PR_NUMBER=$(gh pr view --json number --jq '.number' 2>/dev/null || echo "")

if [ -n "$PR_NUMBER" ]; then
  echo "📋 PR #$PR_NUMBER"
  gh pr view $PR_NUMBER --json title,state,mergeable,baseRefName,headRefName,changedFiles
else
  echo "⚠️  No PR found for current branch"
fi
```

## Step 2: Check CI Status

```bash
if [ -n "$PR_NUMBER" ]; then
  echo ""
  echo "📊 CI Status:"
  gh pr checks $PR_NUMBER --json name,state,conclusion | jq -r '.[] | "\(.state) \(.name)"' | head -10
fi
```

## Step 3: Check Worktree Status

```bash
echo ""
echo "📂 Worktree Status:"
git worktree list

CURRENT_PATH=$(pwd)
if [[ "$CURRENT_PATH" == /tmp/worktrees/* ]]; then
  echo "✅ Working in worktree: $CURRENT_PATH"
else
  echo "⚠️  Not in worktree - consider setting one up"
fi
```

## Step 4: Determine Preview Environment

```bash
BASE_BRANCH=$(gh pr view $PR_NUMBER --json baseRefName --jq '.baseRefName' 2>/dev/null || echo "staging")

case "$BASE_BRANCH" in
  staging)
    echo ""
    echo "🌐 Preview Environment (staging-preview):"
    echo "   API: https://pr-${PR_NUMBER}---gal-api-wug5dzqj2a-uc.a.run.app"
    echo "   Dashboard: https://pr-${PR_NUMBER}--gal-staging-dashboard.web.app"
    ;;
  main)
    echo ""
    echo "🌐 Preview Environment (staging):"
    echo "   API: https://gal-api-staging-wug5dzqj2a-uc.a.run.app"
    echo "   Dashboard: https://gal-staging-dashboard.web.app"
    ;;
esac
```

## Verification Test Cases (Memorize These)

### CLI Test Cases

| # | Test | Command | Expected |
|---|------|---------|----------|
| 1 | Version | `gal --version` | `/\d+\.\d+\.\d+/` |
| 2 | Auth Status | `gal auth status --json` | `"authenticated":` |
| 3 | Sync Status | `gal sync status --json` | `"synced":` or `"neverSynced":` |
| 4 | Demo Sync | `gal sync --pull --demo` | `DEMO MODE` |

### UI Test Cases

| # | Test | Action | Expected |
|---|------|--------|----------|
| 1 | No Login Redirect | Navigate to `/` | NOT redirected to `/login` |
| 2 | Dashboard Loads | Wait for content | Shows org name OR onboarding |
| 3 | Get Started | Navigate to `/get-started` | Page loads without error |
| 4 | Settings | Navigate to `/settings` | Shows user data |

## CRITICAL RULES (MUST FOLLOW)

### Manual Testing is MANDATORY

**NEVER approve a PR without:**
- Running all 4 CLI tests
- Running all 4 UI tests
- MCP verification (if `.mcp.json` or `settings.json` changed)

### Use manual-tester Agent

**DO NOT use Playwright MCP or tmux directly.** Always delegate to:
```
Task(
  subagent_type: "manual-tester",
  prompt: "Test CLI/UI at preview URL..."
)
```

### Fix-On-Spot

When issues are found during verification:
1. Fix immediately while having context
2. Commit with `[skip ci]` for batch fixes
3. Push and re-verify
4. Never use `--admin` to bypass

### CI Check Protocol

**NEVER approve or merge a PR until ALL CI checks pass.** Don't use `--admin` to bypass failures - fix them first.

### GitHub Issue Linkage

**ALWAYS use "Closes #X" (not "Relates to") in PR descriptions for auto-close.**

## After Priming

You are now ready to run:
- `/sdlc:6-verify:run` - Full verification workflow
- Manual CLI tests in worktree
- Manual UI tests via manual-tester agent

## Report After Priming

Summarize:
1. PR number and state
2. CI check status
3. Preview environment URLs
4. Ready to proceed with verification
