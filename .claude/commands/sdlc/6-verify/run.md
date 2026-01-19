---
description: Deployment verification, manual testing, approval and merge
argument-hint: "[pr-number] [--force]"
allowed-tools: Bash, Read, Grep, Task, mcp__playwright__browser_navigate, mcp__playwright__browser_snapshot, mcp__playwright__browser_wait_for, mcp__playwright__browser_close
handoffs:
  - label: "Next: DEPLOY (SDLC Phase 7)"
    agent: sdlc:7-deploy:run
    prompt: Deploy to staging/production after PR is merged
  - label: "Fallback: REVIEW (SDLC Phase 5)"
    agent: sdlc:5-review:run
    prompt: Re-run code review if issues found during verification
---

## MANDATORY: Initialize Progress Tracking

**Before ANY work, create TodoWrite:**

```javascript
TodoWrite([
  { content: "[P6] Verify Phase 5 (REVIEW) passed", status: "pending", activeForm: "Verifying code review gate" },
  { content: "[P6] Prime context", status: "pending", activeForm: "Priming verification context" },
  { content: "[P6] Check preview deployment", status: "pending", activeForm: "Verifying preview deployment" },
  { content: "[P6] Delegate CLI testing", status: "pending", activeForm: "Running CLI tests via manual-tester" },
  { content: "[P6] Delegate UI testing", status: "pending", activeForm: "Running UI tests via manual-tester" },
  { content: "[P6] Comprehensive verification (if Tier 3+)", status: "pending", activeForm: "Verifying PR comprehensively" },
  { content: "[P6] MCP verification (if applicable)", status: "pending", activeForm: "Verifying MCP integration" },
  { content: "[P6] Fix-on-spot loop", status: "pending", activeForm: "Fixing issues on the spot" },
  { content: "[P6] Check merge conflicts", status: "pending", activeForm: "Checking merge conflicts" },
  { content: "[P6] Approve & enable auto-merge", status: "pending", activeForm: "Approving and enabling auto-merge" },
  { content: "[P6] Monitor merge", status: "pending", activeForm: "Monitoring PR merge" },
  { content: "[P6] Post-merge actions", status: "pending", activeForm: "Running post-merge actions" },
  { content: "[P6] Clean up worktree", status: "pending", activeForm: "Cleaning up resources" },
  { content: "[P6] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

**Rules (from official docs):**
- Initialize ALL todos upfront before starting work
- Mark `in_progress` BEFORE beginning each step
- Only ONE todo `in_progress` at a time
- Mark `completed` immediately when done

---

## ⛔ CRITICAL: Worktree Setup (MANDATORY BEFORE STARTING)

**NEVER run verification in the main worktree - ALWAYS create a dedicated worktree.**

**Why?** Verification involves merging conflicts, rebasing, and potentially disruptive git operations. Using a dedicated worktree prevents interference with other sessions.

```bash
# REQUIRED: Create verification worktree before proceeding
PR_NUMBER=${1:-$(gh pr view --json number --jq '.number' 2>/dev/null || echo "")}

if [ -z "$PR_NUMBER" ]; then
  echo "❌ No PR number provided or found"
  exit 1
fi

BRANCH_NAME=$(gh pr view $PR_NUMBER --json headRefName --jq '.headRefName')
WORKTREE_DIR="/tmp/worktrees/verify-$PR_NUMBER-$(date +%s)"

echo "🔧 Setting up verification worktree..."
echo "   PR: #$PR_NUMBER"
echo "   Branch: $BRANCH_NAME"
echo "   Worktree: $WORKTREE_DIR"

# Create worktree and switch to it
git worktree add "$WORKTREE_DIR" "$BRANCH_NAME"
cd "$WORKTREE_DIR"

echo "✅ Verification worktree ready: $(pwd)"
echo ""
```

**⚠️ STOP GATE: Do NOT proceed until you're in the verification worktree.**

---

## Step Skip Criteria (IMPORTANT)

**Some steps may be N/A based on PR type. DOCUMENT the reason when skipping.**

| Step | Can Skip When | Required Documentation |
|------|---------------|----------------------|
| Step 2: CLI Testing | PR has NO CLI changes AND NO API changes | Comment: "CLI N/A: No CLI/API changes" |
| Step 3: UI Testing | PR has NO dashboard changes AND NO API changes | Comment: "UI N/A: No dashboard/API changes" |
| Step 4: Comprehensive | PR is Tier 1-2 AND NOT a promotion PR | (implicit skip) |
| Step 5: MCP | PR has NO .mcp.json or settings.json changes | (implicit skip) |
| Step 9: Post-merge | PR not yet merged (waiting for auto-merge) | Note: "Post-merge deferred to auto-merge completion" |
| Step 10: Worktree | Not in a worktree | (implicit skip) |

**⛔ NEVER SKIP without documenting reason. If uncertain, RUN THE STEP.**

---

## ⚠️ CRITICAL: Infrastructure Blockers Can Affect ANY PR (Learning 2026-01-18)

**DO NOT skip manual testing based on file type alone.**

Even "CSS-only" or "docs-only" PRs can be blocked by infrastructure issues:
- **SSL certificate mismatches** on PR preview deployments
- **OAuth/GitHub App configuration** blocking sign-in
- **Preview URL routing** issues
- **API deployment failures** that curl 200 doesn't catch

### Minimum Verification Required (ALL PRs)

Before marking UI/CLI testing as skipped:

1. **Actually open the preview URL in a browser** - Don't trust curl status codes
2. **Verify sign-in works** - OAuth/GitHub App issues block ALL functionality
3. **Check browser console for errors** - SSL and routing issues show here

```bash
# A 200 status does NOT mean the site works!
curl -s -o /dev/null -w "%{http_code}" "$DASHBOARD_URL"  # ← NOT ENOUGH

# MUST also verify in actual browser via manual-tester agent
Task(
  subagent_type: "manual-tester",
  prompt: "Open $DASHBOARD_URL in browser. Check for:
    1. SSL certificate errors (NET::ERR_CERT_*)
    2. Sign-in functionality
    3. Console errors
    Report any infrastructure issues found."
)
```

**If infrastructure blockers found → Update SDLC status to `6-verify-blocked` and halt.**

---

## Phase Gate Check (REQUIRED)

**CRITICAL: Verify Phase 5 (REVIEW) completed - code review must pass before verification.**

```bash
PR_NUMBER=${1:-$(gh pr view --json number --jq '.number' 2>/dev/null || echo "")}

if [ -z "$PR_NUMBER" ]; then
  echo "❌ No PR number provided or found"
  exit 1
fi

echo "📋 Verifying PR #$PR_NUMBER"

# Check if code review comment exists (indicates Phase 5 completed)
REVIEW_COMMENTS=$(gh pr view $PR_NUMBER --json comments --jq '.comments[].body' | grep -c "Code Review" || echo "0")

if [ "$REVIEW_COMMENTS" -eq 0 ]; then
  echo ""
  echo "┌─────────────────────────────────────────────────────────────┐"
  echo "│ SDLC Phase Order Violation                                  │"
  echo "├─────────────────────────────────────────────────────────────┤"
  echo "│ Current Phase:  6-VERIFY                                    │"
  echo "│ Required Phase: 5-REVIEW (not completed)                    │"
  echo "│ Missing:        Code review must pass first                 │"
  echo "└─────────────────────────────────────────────────────────────┘"
  echo ""
  echo "Run this command first:"
  echo "  /sdlc:5-review:run $PR_NUMBER"
  exit 1
fi

echo "✅ Phase Gate passed. Code review completed."

# SDLC Status Helper - updates PR labels and linked issues
update_sdlc_status() {
  local PR_NUM=$1 PHASE=$2 STATUS=$3
  gh pr edit $PR_NUM --remove-label "sdlc:5-review-in-progress,sdlc:5-review-passed,sdlc:5-review-blocked,sdlc:6-verify-in-progress,sdlc:6-verify-passed,sdlc:6-verify-blocked,sdlc:7-deploy-in-progress,sdlc:7-deploy-complete" 2>/dev/null || true
  gh pr edit $PR_NUM --add-label "sdlc:${PHASE}-${STATUS}" 2>/dev/null || true
  LINKED_ISSUE=$(gh pr view $PR_NUM --json body --jq '.body' 2>/dev/null | grep -oE 'Closes #[0-9]+|Fixes #[0-9]+|Resolves #[0-9]+' | grep -oE '[0-9]+' | head -1)
  [ -n "$LINKED_ISSUE" ] && gh issue comment $LINKED_ISSUE --body "**SDLC Update**: Phase ${PHASE} - ${STATUS}" 2>/dev/null || true
  echo "📊 SDLC Status: ${PHASE} → ${STATUS}"
}

# Set verify in-progress status
update_sdlc_status $PR_NUMBER "6-verify" "in-progress"
```

---

## Phase 0: Prime Context (REQUIRED - AUTO-EXECUTE)

```
Skill(skill: "sdlc:prime:6-verify")
```

This loads verification context including:
- PR state and CI status
- Preview environment URLs
- CLI and UI test cases
- Worktree status

**DO NOT PROCEED until prime command completes.**

---

## Step 1: Preview Readiness Check (REQUIRED)

**After CI passes, verify the PR preview deployment is ready before testing.**

### 1a. Determine Target Environment

```bash
BASE_BRANCH=$(gh pr view $PR_NUMBER --json baseRefName --jq '.baseRefName')

case "$BASE_BRANCH" in
  staging)
    API_URL="https://pr-${PR_NUMBER}---gal-api-wug5dzqj2a-uc.a.run.app"
    DASHBOARD_URL="https://pr-${PR_NUMBER}--gal-staging-dashboard.web.app"
    ENVIRONMENT="staging-preview"
    ;;
  main)
    API_URL="https://gal-api-staging-wug5dzqj2a-uc.a.run.app"
    DASHBOARD_URL="https://gal-staging-dashboard.web.app"
    ENVIRONMENT="staging"
    ;;
  *)
    API_URL="https://pr-${PR_NUMBER}---gal-api-wug5dzqj2a-uc.a.run.app"
    DASHBOARD_URL="https://pr-${PR_NUMBER}--gal-staging-dashboard.web.app"
    ENVIRONMENT="staging-preview"
    ;;
esac

echo "🌐 Target Environment: $ENVIRONMENT"
echo "   API: $API_URL"
echo "   Dashboard: $DASHBOARD_URL"
```

### 1b. Wait for Preview Deployment

```bash
echo ""
echo "⏳ Waiting for preview deployment to be ready..."
sleep 30  # Initial propagation delay

# API health check (5 retries, 10s apart)
API_READY=false
for i in {1..5}; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$API_URL/health" 2>/dev/null || echo "000")
  if [ "$STATUS" = "200" ]; then
    echo "✅ API healthy ($API_URL)"
    API_READY=true
    break
  fi
  echo "   Attempt $i: API status $STATUS, retrying in 10s..."
  sleep 10
done

if [ "$API_READY" = false ]; then
  echo "⚠️  API not ready after 5 attempts - proceeding with caution"
fi

# Dashboard accessibility check (3 retries)
DASHBOARD_READY=false
for i in {1..3}; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$DASHBOARD_URL" 2>/dev/null || echo "000")
  if [ "$STATUS" = "200" ] || [ "$STATUS" = "304" ]; then
    echo "✅ Dashboard accessible ($DASHBOARD_URL)"
    DASHBOARD_READY=true
    break
  fi
  sleep 10
done
```

---

## ⛔ STOP GATE: MANUAL TESTING DECISION

**BEFORE proceeding to Steps 2-3, make an explicit decision:**

```bash
# Analyze PR changes to determine testing requirements
FILES_CHANGED=$(gh pr view $PR_NUMBER --json files --jq '.files[].path')

HAS_CLI_CHANGES=$(echo "$FILES_CHANGED" | grep -E "^apps/cli/" | wc -l | tr -d ' ')
HAS_API_CHANGES=$(echo "$FILES_CHANGED" | grep -E "^apps/api/" | wc -l | tr -d ' ')
HAS_DASHBOARD_CHANGES=$(echo "$FILES_CHANGED" | grep -E "^apps/dashboard/src/" | wc -l | tr -d ' ')
HAS_TEST_ONLY=$(echo "$FILES_CHANGED" | grep -vE "\.test\.(ts|tsx)$" | wc -l | tr -d ' ')

echo ""
echo "┌─────────────────────────────────────────────┐"
echo "│  MANUAL TESTING DECISION                    │"
echo "├─────────────────────────────────────────────┤"
echo "│  CLI changes:       $HAS_CLI_CHANGES files"
echo "│  API changes:       $HAS_API_CHANGES files"
echo "│  Dashboard changes: $HAS_DASHBOARD_CHANGES files"
echo "│  Test-only PR:      $([[ $HAS_TEST_ONLY -eq 0 ]] && echo 'YES' || echo 'NO')"
echo "└─────────────────────────────────────────────┘"

if [[ $HAS_TEST_ONLY -eq 0 ]]; then
  echo ""
  echo "📝 TEST-ONLY PR DETECTED"
  echo "   Skipping CLI and UI manual testing."
  echo "   Reason: No production code changes, only test files."
  SKIP_CLI_TESTING=true
  SKIP_UI_TESTING=true
elif [[ $HAS_CLI_CHANGES -eq 0 && $HAS_API_CHANGES -eq 0 ]]; then
  echo "   CLI Testing: SKIP (no CLI/API changes)"
  SKIP_CLI_TESTING=true
else
  echo "   CLI Testing: REQUIRED"
  SKIP_CLI_TESTING=false
fi

if [[ $HAS_DASHBOARD_CHANGES -eq 0 && $HAS_API_CHANGES -eq 0 ]]; then
  echo "   UI Testing: SKIP (no dashboard/API changes)"
  SKIP_UI_TESTING=true
else
  echo "   UI Testing: REQUIRED"
  SKIP_UI_TESTING=false
fi
```

**If skipping, update todo as completed with skip reason. If required, MUST delegate to manual-tester agent.**

---

## Step 2: Manual CLI Testing (MUST USE manual-tester AGENT)

**⛔ CRITICAL: Do NOT use tmux directly. ALWAYS delegate to the `manual-tester` agent.**

### 2a. CLI Test Cases

| # | Test | Command | Expected |
|---|------|---------|----------|
| 1 | Version | `gal --version` | Matches `/\d+\.\d+\.\d+/` |
| 2 | Auth Status | `gal auth status --json` | Contains `"authenticated":` |
| 3 | Sync Status | `gal sync status --json` | Contains `"synced":` or `"neverSynced":` |
| 4 | Demo Sync | `gal sync --pull --demo` | Contains `DEMO MODE` or `synced` |

### 2b. Execute CLI Tests via manual-tester Agent

```
Task(
  subagent_type: "manual-tester",
  description: "CLI testing for PR #${PR_NUMBER}",
  prompt: "Test GAL CLI against preview API at $API_URL:

  1. Run: gal --version
     Expected: Version number matching /\\d+\\.\\d+\\.\\d+/

  2. Run: GAL_API_URL=$API_URL gal auth status --json
     Expected: JSON with 'authenticated' field

  3. Run: GAL_API_URL=$API_URL gal sync status --json
     Expected: JSON with 'synced' or 'neverSynced' field

  4. Run: gal sync --pull --demo
     Expected: Contains 'DEMO' or 'demo mode'

  Report:
  - Test results (PASS/FAIL for each)
  - Any error messages
  - Overall CLI verdict: [PASS/FAIL]"
)
```

---

## Step 3: Manual UI Testing (MUST USE manual-tester AGENT)

**⛔ CRITICAL: Do NOT use Playwright MCP directly. ALWAYS delegate to the `manual-tester` agent.**

### 3a. UI Test Cases

| # | Test | Action | Expected |
|---|------|--------|----------|
| 1 | No Login Redirect | Navigate to `/` | NOT redirected to `/login` |
| 2 | Dashboard Loads | Wait for content | Shows org name OR onboarding |
| 3 | Get Started | Navigate to `/get-started` | Page loads without error |
| 4 | Settings | Navigate to `/settings` | Shows user data |

### 3b. Execute UI Tests via manual-tester Agent

```
Task(
  subagent_type: "manual-tester",
  description: "UI testing for PR #${PR_NUMBER}",
  prompt: "Test Dashboard UI at $DASHBOARD_URL:

  1. Navigate to $DASHBOARD_URL
     Check: NOT redirected to /login

  2. Wait for dashboard content to load
     Check: Shows organization name OR onboarding flow

  3. Navigate to $DASHBOARD_URL/get-started
     Check: Page loads without error, no 404/500

  4. Navigate to $DASHBOARD_URL/settings
     Check: Shows user data or settings form

  Take screenshots as evidence.
  Report:
  - Test results (PASS/FAIL for each)
  - Screenshots captured
  - Any anomalies detected
  - Overall UI verdict: [PASS/FAIL]"
)
```

---

## Step 4: Comprehensive PR Verification (Tier 3+ or Promotion PRs)

**For large PRs or promotion PRs, run systematic commit-by-commit verification.**

```bash
# Get PR tier from Phase 5 or determine based on changes
FILES_CHANGED=$(gh pr view $PR_NUMBER --json changedFiles --jq '.changedFiles')
TIER=$([[ $FILES_CHANGED -gt 10 ]] && echo "3" || echo "2")

# Check if promotion PR
HEAD_BRANCH=$(gh pr view $PR_NUMBER --json headRefName --jq '.headRefName')
BASE_BRANCH=$(gh pr view $PR_NUMBER --json baseRefName --jq '.baseRefName')
IS_PROMOTION=$([[ "$HEAD_BRANCH" = "staging" && "$BASE_BRANCH" = "main" ]] && echo "true" || echo "false")

if [ "$TIER" -ge 3 ] || [ "$IS_PROMOTION" = "true" ]; then
  echo ""
  echo "┌─────────────────────────────────────────────┐"
  echo "│  🔍 COMPREHENSIVE VERIFICATION REQUIRED     │"
  echo "└─────────────────────────────────────────────┘"

  Task(
    subagent_type: "manual-tester",
    prompt: "Systematically verify PR #$PR_NUMBER:

    1. Get all commits: gh pr view $PR_NUMBER --json commits
    2. Categorize commits by type (product, infra, agentic, docs)
    3. For each PRODUCT commit:
       - Identify what changed (UI, CLI, API)
       - Test the change in browser (Playwright) or terminal (tmux)
       - Take screenshots as evidence
       - Report anomalies
    4. Create GitHub issues for new bugs found
    5. Comment on existing issues if fixes are verified

    Preview URL: $DASHBOARD_URL
    API URL: $API_URL
    Environment: $ENVIRONMENT"
  )
fi
```

---

## Step 5: MCP Configuration Verification (If MCP Changes)

**If PR modifies `.mcp.json`, `.claude/settings.json`, or MCP-related files, verify in fresh Claude Code session.**

### 5a. Detect MCP Changes

```bash
FILE_PATHS=$(gh pr view $PR_NUMBER --json files --jq '.files[].path')
MCP_FILES=$(echo "$FILE_PATHS" | grep -E '\.mcp\.json|settings\.json|mcp' || true)

if [ -n "$MCP_FILES" ]; then
  echo ""
  echo "┌─────────────────────────────────────────────┐"
  echo "│  🔧 MCP CONFIGURATION CHANGE DETECTED       │"
  echo "├─────────────────────────────────────────────┤"
  for f in $MCP_FILES; do
    echo "│    - $f"
  done
  echo "│                                             │"
  echo "│  ⚠️  MANUAL VERIFICATION REQUIRED           │"
  echo "└─────────────────────────────────────────────┘"

  MCP_VERIFICATION_REQUIRED=true
else
  MCP_VERIFICATION_REQUIRED=false
fi
```

### 5b. Verify MCP in Fresh Session (tmux)

```bash
if [ "$MCP_VERIFICATION_REQUIRED" = true ]; then
  echo ""
  echo "🧪 Starting MCP verification in fresh Claude Code session..."

  # Get worktree path
  REVIEW_DIR=$(git rev-parse --show-toplevel 2>/dev/null || pwd)

  # Create tmux session
  tmux kill-session -t mcp-verify 2>/dev/null || true
  tmux new-session -d -s mcp-verify -c "$REVIEW_DIR"

  # Start Claude Code
  tmux send-keys -t mcp-verify 'claude' Enter
  sleep 8  # Wait for startup

  # Check MCP status
  tmux send-keys -t mcp-verify '/mcp' Enter
  sleep 3

  # Capture output
  MCP_OUTPUT=$(tmux capture-pane -t mcp-verify -p)

  echo ""
  echo "MCP Status Output:"
  echo "$MCP_OUTPUT" | grep -E '(connected|disabled|enabled)' || echo "(no MCP status found)"

  # Clean up
  tmux send-keys -t mcp-verify Escape
  sleep 1
  tmux send-keys -t mcp-verify '/exit' Enter
  sleep 2
  tmux kill-session -t mcp-verify 2>/dev/null || true

  echo ""
  echo "✅ MCP verification complete"
fi
```

### 5c. MCP Verification Checklist

| Check | Expected Result |
|-------|-----------------|
| Enabled servers show "✔ connected" | Yes |
| Disabled servers show "◯ disabled" | Yes |
| No unexpected servers connected | Yes |
| No error messages during startup | Yes |

---

## Step 6: Fix-On-Spot Loop

**If any step found issues, fix them immediately before proceeding to approval.**

### 6a. Aggregate Issues

```bash
ALL_ISSUES=()
ALL_ISSUES+=("${CLI_ISSUES[@]}")
ALL_ISSUES+=("${UI_ISSUES[@]}")

if [ ${#ALL_ISSUES[@]} -gt 0 ]; then
  echo ""
  echo "┌─────────────────────────────────────────────┐"
  echo "│  🔧 FIX-ON-SPOT REQUIRED                    │"
  echo "├─────────────────────────────────────────────┤"
  echo "│  Total Issues: ${#ALL_ISSUES[@]}"
  for issue in "${ALL_ISSUES[@]}"; do
    echo "│  - $issue"
  done
  echo "└─────────────────────────────────────────────┘"
fi
```

### 6b. Fix Workflow

**When issues are found:**

1. **Analyze each issue** - Determine root cause
2. **Fix immediately** - Edit the affected files
3. **Test locally** - Verify fix works
4. **Commit with `[skip ci]`** - Batch fixes without triggering full CI
5. **Push to branch** - Update PR
6. **Re-run failed tests** - Verify issues resolved
7. **When ALL pass** - Remove `[skip ci]` for final commit

```bash
if [ ${#ALL_ISSUES[@]} -gt 0 ]; then
  # Fix and commit
  git add -A
  git commit -m "[fix] Address verification issues [skip ci]

- Fixed: Issue descriptions here

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
  git push origin $(git branch --show-current)

  echo ""
  echo "⏳ Re-waiting for CI after fixes..."
  # Loop back to Step 1
fi
```

---

## Step 7: Merge Conflict Check (BEFORE APPROVAL)

```bash
MERGEABLE=$(gh pr view $PR_NUMBER --json mergeable --jq '.mergeable')

if [ "$MERGEABLE" = "CONFLICTING" ]; then
  echo ""
  echo "⚠️  MERGE CONFLICT DETECTED"
  echo "   PR #$PR_NUMBER has conflicts with base branch"
  echo ""
  echo "   Use: Skill(skill: '_internal:git:resolve-conflicts')"
  echo "   Then: git push origin $BRANCH_NAME --force-with-lease"
  exit 1
fi
```

---

## ⛔ BLOCKING GATE: ALL TESTS MUST PASS

**STOP. Review TodoWrite status. DO NOT APPROVE until ALL of these pass:**

| Gate | Status | Verification | Skip Allowed? |
|------|--------|--------------|---------------|
| Step 1: Preview Ready | ⬜ | API + Dashboard accessible | NO |
| Step 2: CLI Tests | ⬜ | All 4 CLI tests pass | YES (if test-only or no CLI/API) |
| Step 3: UI Tests | ⬜ | All 4 UI tests pass | YES (if test-only or no dashboard/API) |
| Step 5: MCP (if applicable) | ⬜ | MCP servers configured correctly | YES (if no MCP changes) |
| Step 6: Fix-on-spot | ⬜ | All issues resolved | YES (if no issues found) |
| Step 7: No conflicts | ⬜ | PR is mergeable | NO |

**Before proceeding, verbally confirm each gate:**
```
"Gate 1: Preview ✅ (API and Dashboard accessible)"
"Gate 2: CLI ✅/SKIPPED (reason: ...)"
"Gate 3: UI ✅/SKIPPED (reason: ...)"
...
```

---

## Step 8: Final Approval & Auto-Merge

**Approval is ONLY granted when ALL gates pass.**

### 8a. Post Approval Comment

```bash
gh pr comment $PR_NUMBER --body "## ✅ VERIFY Phase Complete - APPROVED

**Phase 6 Verification Complete**

### Preview Environment
- API: $API_URL ✅
- Dashboard: $DASHBOARD_URL ✅

### Manual Testing
- CLI Tests: ✅ All 4 tests passed
- UI Tests: ✅ All 4 tests passed
$([ "$MCP_VERIFICATION_REQUIRED" = true ] && echo "- MCP Config: ✅ Verified in fresh session")

### Quality Gates
- Code Review: ✅ (Phase 5)
- Manual Testing: ✅ (Phase 6)
- No Merge Conflicts: ✅

**VERDICT: APPROVE AND MERGE** ✅

🤖 Verified with [Claude Code](https://claude.com/claude-code)"

# Update SDLC status to passed
update_sdlc_status $PR_NUMBER "6-verify" "passed"
```

### 8b. Enable Auto-Merge

```bash
echo "🔀 Enabling auto-merge for PR #$PR_NUMBER..."
echo "🧹 Will auto-delete branch after merge to keep git clean..."

# CRITICAL: Delete branch after merge to prevent git graph clutter
gh pr merge $PR_NUMBER --auto --merge --delete-branch

MERGE_STATUS=$(gh pr view $PR_NUMBER --json autoMergeRequest --jq '.autoMergeRequest.enabledAt // "not enabled"')
if [ "$MERGE_STATUS" != "not enabled" ]; then
  echo "✅ Auto-merge enabled - PR will merge when all checks pass"
  echo "🧹 Branch will be automatically deleted after merge"
else
  echo "⚠️  Auto-merge not enabled - may need manual merge"
fi
```

### 8c. Monitor Until Merged

```bash
echo ""
echo "⏳ Monitoring PR until merged (max 10 minutes)..."

MERGE_TIMEOUT=600
MERGE_ELAPSED=0
MERGE_INTERVAL=30

while [ $MERGE_ELAPSED -lt $MERGE_TIMEOUT ]; do
  PR_STATE=$(gh pr view $PR_NUMBER --json state --jq '.state')

  if [ "$PR_STATE" = "MERGED" ]; then
    echo ""
    echo "✅ PR #$PR_NUMBER merged successfully!"
    MERGE_COMMIT=$(gh pr view $PR_NUMBER --json mergeCommit --jq '.mergeCommit.oid')
    echo "   Merge commit: $MERGE_COMMIT"
    break
  fi

  echo "$(date +%H:%M:%S) - PR state: $PR_STATE (waiting for merge...)"
  sleep $MERGE_INTERVAL
  MERGE_ELAPSED=$((MERGE_ELAPSED + MERGE_INTERVAL))
done

if [ "$PR_STATE" != "MERGED" ]; then
  echo ""
  echo "⚠️  PR not merged after 10 minutes"
  echo "   Auto-merge is enabled - will merge when blocking checks pass"
fi
```

---

## Step 9: Post-Merge Actions (MANDATORY AFTER MERGE)

**⛔ CRITICAL: This step MUST run after PR is merged. Do NOT skip.**

**Update TodoWrite:**
```
TodoWrite: Mark "Step 9: Post-merge actions" as in_progress
```

```bash
if [ "$PR_STATE" = "MERGED" ]; then
  # Get linked issue
  ISSUE_NUMBER=$(gh pr view $PR_NUMBER --json body --jq '.body' | grep -oE '#[0-9]+' | head -1 | tr -d '#')

  if [ -n "$ISSUE_NUMBER" ]; then
    echo ""
    echo "📋 Closing linked issue #$ISSUE_NUMBER..."
    gh issue close $ISSUE_NUMBER --comment "Resolved by PR #$PR_NUMBER (merged to staging)" 2>/dev/null || true
  fi

  # Sync git state
  echo ""
  echo "🔄 Syncing git state after merge..."
  git fetch --all --prune

  # Clean up merged local branches
  echo ""
  echo "🧹 Cleaning up merged local branches..."
  MERGED_BRANCHES=$(git branch --merged staging | grep -v "staging\|main\|^\*" || true)
  if [ -n "$MERGED_BRANCHES" ]; then
    echo "$MERGED_BRANCHES" | xargs git branch -d 2>/dev/null || true
    echo "   Deleted merged branches"
  else
    echo "   No merged branches to clean"
  fi

  # Clear stale stashes
  STASH_COUNT=$(git stash list | wc -l | tr -d ' ')
  if [ "$STASH_COUNT" -gt 0 ]; then
    echo ""
    echo "🧹 Found $STASH_COUNT stash(es) - clearing stale ones..."
    git stash clear
    echo "   Stashes cleared"
  fi

  echo "✅ Git state synced and cleaned"

  # Suggest comprehensive branch pruning
  echo ""
  echo "💡 MAINTENANCE SUGGESTION:"
  echo "   For comprehensive git cleanup, run: /prune-branches --dry-run"
  echo "   This will show all branches that can be safely cleaned up"
fi
```

---

## Step 10: Worktree Cleanup (MANDATORY IF IN WORKTREE)

**⛔ CRITICAL: If working in /tmp/worktrees/*, MUST clean up before ending session.**

**Update TodoWrite:**
```
TodoWrite: Mark "Step 10: Worktree cleanup" as in_progress
```

```bash
WORKTREE_PATH=$(git rev-parse --show-toplevel 2>/dev/null || pwd)

# Check if we're in a worktree
if [[ "$WORKTREE_PATH" == /tmp/worktrees/* ]]; then
  echo ""
  echo "🧹 Cleaning up worktree..."

  # Get main repo path
  MAIN_REPO=$(git worktree list | grep -v '/tmp/worktrees' | head -1 | awk '{print $1}')

  # Return to main repo
  cd "$MAIN_REPO"

  # Remove the worktree
  if git worktree remove "$WORKTREE_PATH" --force 2>/dev/null; then
    echo "✅ Worktree removed: $WORKTREE_PATH"
  else
    echo "⚠️  Worktree cleanup failed - manual removal may be needed:"
    echo "   git worktree remove $WORKTREE_PATH --force"
  fi

  # Prune stale worktrees
  git worktree prune
  echo "✅ Stale worktrees pruned"

  # List remaining worktrees
  echo ""
  echo "📋 Active worktrees:"
  git worktree list
fi
```

---

## Summary

```bash
echo ""
echo "┌─────────────────────────────────────────────┐"
echo "│  ✅ VERIFY PHASE COMPLETE                   │"
echo "├─────────────────────────────────────────────┤"
echo "│  PR: #$PR_NUMBER                            │"
echo "│  Status: $PR_STATE                          │"
echo "│  Next: /sdlc:7-deploy:run (if not auto)    │"
echo "└─────────────────────────────────────────────┘"
```

---

## MANDATORY: Capture Learnings (AUTO-EXECUTE)

**DO NOT end without invoking:**

```
Skill(skill: "capture-learnings")
```

This captures:
- Process deviations that occurred
- Manual interventions from user
- Improvements to agentic layer
- Any `.claude/` or `CLAUDE.md` improvements made during verification

**Update TodoWrite when complete:**
```javascript
TodoWrite([
  // ... previous steps as completed ...
  { content: "[P6] Capture learnings", status: "completed", activeForm: "Learnings captured" }
])
```

---

## Session Completion Checklist

**Before ending the SDLC Phase 6 session, verify ALL todos are completed:**

```
TodoWrite: Verify all items show "completed" status
```

**If any item is still "pending" or "in_progress", either:**
1. Complete the step
2. Document why it was skipped (with explicit reason)

**⛔ DO NOT END SESSION with incomplete todos without documenting reasons.**
