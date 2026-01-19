---
description: Automated CI workflow fix iteration cycle with resumable state management
argument-hint: "[--resume|--iteration N|--monitor-only]"
allowed-tools:
  - Task
  - Read
  - Write
  - Bash
  - Grep
  - Skill
---

# CI Auto-Fix - Resumable Iteration Lifecycle

Automated iteration cycle for fixing failing CI workflows with state persistence, resume capability, and delegation to devops-engineer.

## Purpose

Execute iterative fix cycles for failing GitHub Actions workflows with:
- **State persistence**: Save progress to `.ci-autofix-state.json`
- **Resume capability**: Continue from any step after interruption
- **Automated PR workflow**: Create, monitor, merge, repeat
- **Iteration tracking**: Full history of attempts and results

## Recent Findings (2025-11-22)

**Key Learnings from Iteration 1:**

1. **PR Approval Limitation**: Cannot use `gh pr review --approve` to approve own PRs
   - **Solution**: Use `/manual-approve-pr` slash command instead
   - **Implementation**: Step 6 now invokes SlashCommand tool

2. **PR Workflow Completion**: Must wait for ALL PR workflows to complete before approval
   - **Issue**: Previously only checked first status check
   - **Fix**: Now monitors all workflows with `gh pr checks` until none are pending/in_progress
   - **Timeout**: Increased to 10 minutes (from 5) to allow for longer-running workflows

3. **Merge Strategy Flexibility**: Repository doesn't allow `--squash --auto`
   - **Solution**: Try merge strategies in order: merge → squash → rebase
   - **Fallback**: If all fail, pause and require manual intervention
   - **State**: Track `manual_merge_required` status for resume capability

4. **Workflow Monitoring**: Dev branch workflows vs PR workflows are different
   - **Step 2**: Waits for dev branch workflows to complete (analyze baseline)
   - **Step 6**: Waits for PR-specific workflows to complete (validate fixes)
   - **Step 7**: Monitors dev branch workflows after merge (confirm fixes deployed)

5. **CRITICAL - Validate PR Workflows PASS Before Merge**: Must ensure workflows succeed, not just complete
   - **Risk**: Merging with failing workflows cascades failures through deployment hierarchy:
     - dev → staging → production (each push triggers next level)
   - **Fix**: Step 6 now validates ALL workflows pass (not just complete)
   - **Criteria**: Zero "fail" status workflows before merge
   - **Exception**: Allow "skipping" workflows (intentional skips)
   - **Blocking**: If any workflow fails, STOP and require manual investigation

## Variables

- **MODE**: First argument from $ARGUMENTS (optional)
  - Default (no argument): Start new iteration cycle
  - `--resume`: Continue from saved state
  - `--iteration N`: Skip to specific iteration number
  - `--monitor-only`: Skip to monitoring step only

## State File Format

```json
{
  "iteration": 8,
  "pr_number": 421,
  "branch": "fix/iteration8-final-three-workflows",
  "timestamp": "2025-11-22T17:38:00Z",
  "current_step": "monitoring",
  "workflow_status": "in_progress",
  "failures": ["Android", "Web", "iOS"],
  "history": [
    {"iteration": 6, "pr": 419, "result": "4 failures"},
    {"iteration": 7, "pr": 420, "result": "Docker fixed, 3 failures"}
  ]
}
```

## Step 1: Load or Initialize State

**Check for existing state:**

```bash
STATE_FILE=".ci-autofix-state.json"
MODE="${1:-new}"

if [ -f "$STATE_FILE" ]; then
  if [ "$MODE" != "--resume" ] && [ "$MODE" != "new" ]; then
    echo "⚠️  Existing state found. Use --resume to continue or delete $STATE_FILE"
    echo ""
    jq '.' "$STATE_FILE"
    exit 1
  fi
fi

# Load state if resuming
if [ "$MODE" = "--resume" ] && [ -f "$STATE_FILE" ]; then
  echo "📂 Loading saved state..."
  ITERATION=$(jq -r '.iteration' "$STATE_FILE")
  PR_NUMBER=$(jq -r '.pr_number' "$STATE_FILE")
  BRANCH=$(jq -r '.branch' "$STATE_FILE")
  CURRENT_STEP=$(jq -r '.current_step' "$STATE_FILE")
  WORKFLOW_STATUS=$(jq -r '.workflow_status' "$STATE_FILE")

  echo "   Iteration: $ITERATION"
  echo "   PR: #$PR_NUMBER"
  echo "   Branch: $BRANCH"
  echo "   Step: $CURRENT_STEP"
  echo "   Status: $WORKFLOW_STATUS"
  echo ""

  # Jump to saved step
  case "$CURRENT_STEP" in
    "wait-workflows") RESUME_STEP=2 ;;
    "analyze") RESUME_STEP=3 ;;
    "delegate") RESUME_STEP=4 ;;
    "create-pr") RESUME_STEP=5 ;;
    "approve") RESUME_STEP=6 ;;
    "monitoring") RESUME_STEP=7 ;;
    "evaluate") RESUME_STEP=8 ;;
    *) RESUME_STEP=1 ;;
  esac
else
  # Initialize new state
  ITERATION=1
  RESUME_STEP=1

  # Check for existing state to continue iteration numbering
  if [ -f "$STATE_FILE" ]; then
    PREV_ITERATION=$(jq -r '.iteration' "$STATE_FILE")
    ITERATION=$((PREV_ITERATION + 1))
  fi
fi
```

## Step 2: Wait for Running Workflows

**Skip if resuming past this step:**

```bash
if [ $RESUME_STEP -le 2 ]; then
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo "⏳ Step 2: Waiting for Running Workflows"
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo ""

  # Check for running or queued workflows
  RUNNING_COUNT=$(gh run list --branch dev --limit 50 --json status \
    | jq '[.[] | select(.status == "in_progress" or .status == "queued")] | length')

  if [ "$RUNNING_COUNT" -eq 0 ]; then
    echo "✅ No workflows currently running"
    echo ""
  else
    echo "Found $RUNNING_COUNT running/queued workflows"
    echo "Waiting for all workflows to complete before analyzing failures..."
    echo ""

    # Monitor workflows (max 60 minutes)
    TIMEOUT=3600
    ELAPSED=0
    INTERVAL=60

    while [ $ELAPSED -lt $TIMEOUT ]; do
      RUNNING_COUNT=$(gh run list --branch dev --limit 50 --json status \
        | jq '[.[] | select(.status == "in_progress" or .status == "queued")] | length')

      if [ "$RUNNING_COUNT" -eq 0 ]; then
        echo "✅ All workflows completed"
        echo ""
        break
      fi

      echo "⏳ Workflows still running: $RUNNING_COUNT - waiting $INTERVAL seconds..."
      sleep $INTERVAL
      ELAPSED=$((ELAPSED + INTERVAL))
    done

    if [ "$RUNNING_COUNT" -gt 0 ]; then
      echo "⚠️  Timeout reached - $RUNNING_COUNT workflows still running"
      echo "   Proceeding with analysis anyway..."
      echo ""
    fi
  fi
fi
```

## Step 3: Analyze Workflow Failures

**Skip if resuming past this step.**

**Uses `github-actions-skill` for enhanced analysis:**

```bash
if [ $RESUME_STEP -le 3 ]; then
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo "🔍 Step 3: Analyzing Workflow Failures"
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo ""

  # Use github-actions-skill for detection
  SKILL_DIR=".claude/skills/github-actions-skill"
  echo "Using github-actions-skill for analysis..."
  echo ""

  # Detect failures using skill
  SKILL_OUTPUT=$(cd $SKILL_DIR && node run.js detect-failures --branch main --json 2>/dev/null)

  if [ $? -eq 0 ]; then
    FAILURE_COUNT=$(echo "$SKILL_OUTPUT" | jq -r '.failing | length')

    if [ "$FAILURE_COUNT" -eq 0 ]; then
      echo "✅ No failed workflows found!"
      echo ""
      echo "🎉 All workflows passing - iteration cycle complete!"
      rm -f "$STATE_FILE"
      exit 0
    fi

    echo "Found $FAILURE_COUNT failing workflow(s)"
    echo ""

    # Get detailed analysis for each failure
    for RUN_ID in $(echo "$SKILL_OUTPUT" | jq -r '.failing[].id'); do
      echo "Analyzing run #$RUN_ID..."
      cd $SKILL_DIR && node run.js analyze $RUN_ID 2>/dev/null || true
      echo ""

      # Get fix suggestions
      echo "Fix suggestions for run #$RUN_ID:"
      cd $SKILL_DIR && node run.js suggest-fix $RUN_ID 2>/dev/null || true
      echo ""
    done

    # Extract failure names for state
    FAILURES=($(echo "$SKILL_OUTPUT" | jq -r '.failing[].name'))
  else
    # Fallback to basic gh command if skill fails
    echo "Skill unavailable, using fallback detection..."
    FAILED_WORKFLOWS=$(gh run list --branch main --limit 20 --json conclusion,name,databaseId \
      | jq -r '.[] | select(.conclusion == "failure") | "\(.name) (Run #\(.databaseId))"')

    if [ -z "$FAILED_WORKFLOWS" ]; then
      echo "✅ No failed workflows found!"
      rm -f "$STATE_FILE"
      exit 0
    fi

    echo "Failed workflows:"
    echo "$FAILED_WORKFLOWS"
    FAILURES=($(echo "$FAILED_WORKFLOWS" | awk '{print $1}'))
  fi

  # Save state with analysis results
  cat > "$STATE_FILE" <<EOF
{
  "iteration": $ITERATION,
  "pr_number": null,
  "branch": null,
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "current_step": "analyze",
  "workflow_status": "analyzing",
  "failures": $(printf '%s\n' "${FAILURES[@]}" | jq -R . | jq -s .),
  "history": $([ -f "$STATE_FILE" ] && jq -c '.history // []' "$STATE_FILE" || echo "[]")
}
EOF

  echo "💾 State saved to $STATE_FILE"
  echo ""
fi
```

## Step 4: Delegate to DevOps Engineer

**Skip if resuming past this step.**

**DevOps engineer uses `github-actions-skill` for analysis:**

```bash
if [ $RESUME_STEP -le 4 ]; then
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo "🛠️  Step 4: Delegating to DevOps Engineer"
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo ""

  # Load failures from state
  FAILURES=($(jq -r '.failures[]' "$STATE_FILE"))

  echo "Delegating fix for iteration $ITERATION:"
  echo "  Failing workflows: ${FAILURES[*]}"
  echo ""

  # Delegate to devops-engineer
  Task(
    subagent_type="devops-engineer",
    description="Fix failing CI workflows - Iteration $ITERATION",
    prompt="
# CI Workflow Fix - Iteration $ITERATION

## Failed Workflows
${FAILURES[*]}

## IMPORTANT: Use github-actions-skill

You have access to the **github-actions-skill** for CI analysis. Use it first:

\`\`\`bash
SKILL_DIR=\".claude/skills/github-actions-skill\"

# Get detailed analysis of failures
cd \$SKILL_DIR && node run.js detect-failures --branch main

# For each failing run, get deep analysis
cd \$SKILL_DIR && node run.js analyze <run-id>

# Get fix suggestions
cd \$SKILL_DIR && node run.js suggest-fix <run-id>

# Parse logs for specific errors
cd \$SKILL_DIR && node run.js parse-logs <run-id>

# Validate your workflow changes before committing
cd \$SKILL_DIR && node run.js validate .github/workflows/<workflow>.yml
\`\`\`

## Your Task

1. **Use Skill for Analysis**: Run the skill commands above to get structured error analysis
2. **Review Fix Suggestions**: The skill provides actionable fix recommendations
3. **Create Fix Branch**: \`fix/iteration${ITERATION}-workflow-fixes\`
4. **Implement Fixes**: Apply the suggested fixes to workflows/scripts
5. **Validate Changes**: Use \`node run.js validate\` on modified workflows
6. **Commit Changes**: Use descriptive commit message
7. **Do NOT push or create PR**: Leave branch ready for review

## Branch Naming
\`fix/iteration${ITERATION}-workflow-fixes\`

## Commit Message Format
\`\`\`
fix(iteration-${ITERATION}): [Brief description of fixes]

- Fix 1: [Description]
- Fix 2: [Description]
- Fix 3: [Description]

Addresses: ${FAILURES[*]}
\`\`\`

## Deliverables

- Branch created with fixes
- Changes committed locally
- Summary of changes made (including skill analysis results)
- Validation results from skill
- Expected impact on failing workflows
    "
  )

  # Get branch name from devops-engineer output
  BRANCH="fix/iteration${ITERATION}-workflow-fixes"

  # Update state
  jq --arg branch "$BRANCH" \
     '.branch = $branch | .current_step = "delegate"' \
     "$STATE_FILE" > "${STATE_FILE}.tmp" && mv "${STATE_FILE}.tmp" "$STATE_FILE"

  echo ""
  echo "💾 State updated: branch = $BRANCH"
  echo ""
fi
```

## Step 5: Create/Update Pull Request

**Skip if resuming past this step:**

```bash
if [ $RESUME_STEP -le 5 ]; then
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo "📝 Step 5: Creating Pull Request"
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo ""

  # Load state
  BRANCH=$(jq -r '.branch' "$STATE_FILE")
  FAILURES=($(jq -r '.failures[]' "$STATE_FILE"))

  # Push branch
  echo "🔄 Pushing branch: $BRANCH"
  git push -u origin "$BRANCH"
  echo ""

  # Create PR
  echo "📤 Creating PR..."

  PR_TITLE="fix(iteration-${ITERATION}): CI workflow fixes for ${FAILURES[*]}"
  PR_BODY="## CI Auto-Fix Iteration $ITERATION

### Failed Workflows
$(printf '- %s\n' "${FAILURES[@]}")

### Changes Made
[DevOps engineer will have documented changes]

### Testing
- [ ] Workflows triggered on this PR
- [ ] Monitor for success/failure
- [ ] Analyze results

### Iteration History
$(jq -r '.history[] | "- Iteration \(.iteration): PR #\(.pr) - \(.result)"' "$STATE_FILE")

---
🤖 Auto-generated by \`/ci-auto-fix\` iteration cycle"

  PR_NUMBER=$(gh pr create \
    --title "$PR_TITLE" \
    --body "$PR_BODY" \
    --base dev \
    --head "$BRANCH" \
    --json number --jq '.number')

  echo "✅ PR Created: #$PR_NUMBER"
  echo "   URL: $(gh pr view $PR_NUMBER --json url --jq '.url')"
  echo ""

  # Update state
  jq --arg pr "$PR_NUMBER" \
     '.pr_number = ($pr | tonumber) | .current_step = "create-pr"' \
     "$STATE_FILE" > "${STATE_FILE}.tmp" && mv "${STATE_FILE}.tmp" "$STATE_FILE"

  echo "💾 State updated: pr_number = $PR_NUMBER"
  echo ""
fi
```

## Step 6: Approve and Merge PR

**Skip if resuming past this step:**

```bash
if [ $RESUME_STEP -le 6 ]; then
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo "✅ Step 6: Approving and Merging PR"
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo ""

  # Load state
  PR_NUMBER=$(jq -r '.pr_number' "$STATE_FILE")

  echo "Waiting for PR workflows to complete..."
  echo ""

  # Wait for ALL PR workflows to complete (max 10 minutes)
  TIMEOUT=600
  ELAPSED=0
  INTERVAL=30

  while [ $ELAPSED -lt $TIMEOUT ]; do
    # Check for pending or in-progress workflows on the PR
    PENDING_COUNT=$(gh pr checks $PR_NUMBER 2>&1 | grep -c "pending\|in_progress" || echo "0")

    if [ "$PENDING_COUNT" -eq 0 ]; then
      echo "✅ All PR workflows completed"
      echo ""
      break
    fi

    echo "⏳ Workflows still running: $PENDING_COUNT - waiting $INTERVAL seconds..."
    sleep $INTERVAL
    ELAPSED=$((ELAPSED + INTERVAL))
  done

  if [ "$PENDING_COUNT" -gt 0 ]; then
    echo "⚠️  Timeout reached - $PENDING_COUNT workflows still running"
    echo "   Proceeding with approval anyway..."
    echo ""
  fi

  # Display final workflow status
  echo "📊 Final PR Workflow Status:"
  gh pr checks $PR_NUMBER || true
  echo ""

  # CRITICAL: Validate workflows PASS before merge
  # Prevent cascading failures through deployment hierarchy (dev → staging → production)
  echo "🔍 Validating workflow results..."
  FAILED_COUNT=$(gh pr checks $PR_NUMBER 2>&1 | grep -c "fail" || echo "0")

  if [ "$FAILED_COUNT" -gt 0 ]; then
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "🚨 CRITICAL: PR Workflows Failed"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo ""
    echo "❌ Found $FAILED_COUNT failing workflow(s) on PR #$PR_NUMBER"
    echo ""
    echo "Failed workflows:"
    gh pr checks $PR_NUMBER 2>&1 | grep "fail" || true
    echo ""
    echo "⚠️  DEPLOYMENT HIERARCHY RISK:"
    echo "   Merging with failing workflows will cascade failures:"
    echo "   dev → staging → production (each push triggers next level)"
    echo ""
    echo "🛑 BLOCKING MERGE - Manual investigation required:"
    echo "   1. Review workflow logs: gh run view <run-id> --log-failed"
    echo "   2. Determine if failures are:"
    echo "      - New issues introduced by this PR"
    echo "      - Pre-existing issues that need fixing"
    echo "      - False positives that can be ignored"
    echo "   3. Options:"
    echo "      a) Fix issues and push new commits to PR"
    echo "      b) Close PR and create new iteration"
    echo "      c) Override and merge anyway (DANGEROUS)"
    echo ""
    echo "   View PR: gh pr view $PR_NUMBER --web"
    echo ""
    echo "⏸️  Pausing workflow - use /ci-auto-fix --resume after fixing failures"

    # Update state to indicate workflow failures
    jq --argjson failed "$FAILED_COUNT" \
       '.current_step = "approve" | .workflow_status = "pr_workflows_failed" | .failed_workflow_count = $failed' \
       "$STATE_FILE" > "${STATE_FILE}.tmp" && mv "${STATE_FILE}.tmp" "$STATE_FILE"

    echo "💾 State saved: workflow_status = pr_workflows_failed"
    echo ""
    exit 1
  fi

  echo "✅ All workflows passed or were skipped - safe to merge"
  echo ""

  # Approve PR using /manual-approve-pr slash command
  # Note: Can't use `gh pr review --approve` because GitHub doesn't allow approving own PRs
  echo "✅ Approving PR #$PR_NUMBER using /manual-approve-pr..."

  # Use SlashCommand tool to invoke /manual-approve-pr
  SlashCommand("/manual-approve-pr $PR_NUMBER")

  echo "⏳ Waiting 10 seconds for approval to process..."
  sleep 10
  echo ""

  # Merge PR
  # Note: Repository doesn't allow --squash --auto, use regular merge
  echo "🔀 Merging PR #$PR_NUMBER..."

  # Try different merge strategies in order of preference
  if gh pr merge $PR_NUMBER --merge --delete-branch 2>/dev/null; then
    echo "✅ PR merged successfully (merge commit)"
  elif gh pr merge $PR_NUMBER --squash --delete-branch 2>/dev/null; then
    echo "✅ PR merged successfully (squash merge)"
  elif gh pr merge $PR_NUMBER --rebase --delete-branch 2>/dev/null; then
    echo "✅ PR merged successfully (rebase merge)"
  else
    echo "❌ Failed to merge PR #$PR_NUMBER"
    echo "   Manual intervention required"
    echo "   Use: gh pr view $PR_NUMBER --web"
    echo ""
    echo "⏸️  Pausing workflow - use /ci-auto-fix --resume after manual merge"

    # Update state to indicate manual merge needed
    jq '.current_step = "approve" | .workflow_status = "manual_merge_required"' \
       "$STATE_FILE" > "${STATE_FILE}.tmp" && mv "${STATE_FILE}.tmp" "$STATE_FILE"

    exit 1
  fi
  echo ""

  # Update state
  jq '.current_step = "approve" | .workflow_status = "merged"' \
     "$STATE_FILE" > "${STATE_FILE}.tmp" && mv "${STATE_FILE}.tmp" "$STATE_FILE"

  echo "💾 State updated: PR merged successfully"
  echo ""
fi
```

## Step 7: Monitor Workflow Results

**Skip if resuming past this step:**

```bash
if [ $RESUME_STEP -le 7 ]; then
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo "📡 Step 7: Monitoring Workflow Results"
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo ""

  echo "Waiting for workflows to complete..."
  echo ""

  # Wait for merge to trigger workflows
  sleep 60

  # Monitor workflows (max 60 minutes)
  TIMEOUT=3600
  ELAPSED=0
  INTERVAL=120

  while [ $ELAPSED -lt $TIMEOUT ]; do
    # Get latest workflow runs on dev
    IN_PROGRESS=$(gh run list --branch dev --limit 20 --json status,conclusion \
      | jq '[.[] | select(.status == "in_progress")] | length')

    if [ "$IN_PROGRESS" -eq 0 ]; then
      echo "✅ All workflows completed"
      break
    fi

    echo "⏳ Workflows in progress: $IN_PROGRESS - waiting $INTERVAL seconds..."
    sleep $INTERVAL
    ELAPSED=$((ELAPSED + INTERVAL))
  done

  echo ""

  # Update state
  jq '.current_step = "monitoring" | .workflow_status = "completed"' \
     "$STATE_FILE" > "${STATE_FILE}.tmp" && mv "${STATE_FILE}.tmp" "$STATE_FILE"

  echo "💾 State updated: monitoring complete"
  echo ""
fi
```

## Step 8: Evaluate Results and Continue

**Always execute this step:**

```bash
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🎯 Step 8: Evaluating Results"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""

# Load state
ITERATION=$(jq -r '.iteration' "$STATE_FILE")
PR_NUMBER=$(jq -r '.pr_number' "$STATE_FILE")

# Get current failures
CURRENT_FAILURES=$(gh run list --branch dev --limit 20 --json conclusion,name \
  | jq -r '.[] | select(.conclusion == "failure") | .name' | sort -u)

FAILURE_COUNT=$(echo "$CURRENT_FAILURES" | grep -c . || echo "0")

echo "Results for Iteration $ITERATION:"
echo "  PR: #$PR_NUMBER"
echo "  Remaining failures: $FAILURE_COUNT"
echo ""

if [ "$FAILURE_COUNT" -gt 0 ]; then
  echo "Failed workflows:"
  echo "$CURRENT_FAILURES"
  echo ""
fi

# Update history
RESULT_MSG="$FAILURE_COUNT failures remaining"
if [ "$FAILURE_COUNT" -eq 0 ]; then
  RESULT_MSG="All workflows passing ✅"
fi

jq --arg iter "$ITERATION" \
   --arg pr "$PR_NUMBER" \
   --arg result "$RESULT_MSG" \
   '.history += [{"iteration": ($iter | tonumber), "pr": ($pr | tonumber), "result": $result}] | .current_step = "evaluate"' \
   "$STATE_FILE" > "${STATE_FILE}.tmp" && mv "${STATE_FILE}.tmp" "$STATE_FILE"

echo "💾 History updated"
echo ""

# Decide next action
if [ "$FAILURE_COUNT" -eq 0 ]; then
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo "🎉 SUCCESS - All Workflows Passing!"
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo ""
  echo "Iteration History:"
  jq -r '.history[] | "  Iteration \(.iteration): PR #\(.pr) - \(.result)"' "$STATE_FILE"
  echo ""

  # Cleanup state
  echo "🧹 Cleaning up state file..."
  rm -f "$STATE_FILE"

  echo ""
  echo "✅ CI auto-fix cycle complete!"
else
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo "🔄 Next Iteration Required"
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo ""
  echo "To continue to next iteration, run:"
  echo "  /ci-auto-fix"
  echo ""
  echo "Or to resume this iteration:"
  echo "  /ci-auto-fix --resume"
  echo ""
fi
```

## Resume Capability

**State persistence enables:**

- **Context limit interruption**: Resume exactly where you left off
- **Claude Code restart**: Reload state and continue
- **Manual intervention**: Pause, make changes, resume
- **Step-by-step execution**: Run one step at a time

**State file tracks:**
- Current iteration number
- PR number and branch
- Current step in workflow
- List of failing workflows
- Full iteration history

## Monitor-Only Mode

**Quick monitoring without full cycle:**

```bash
if [ "$MODE" = "--monitor-only" ]; then
  if [ ! -f "$STATE_FILE" ]; then
    echo "❌ No state file found. Start a new iteration first."
    exit 1
  fi

  # Jump directly to monitoring step
  RESUME_STEP=7

  echo "🔍 Monitor-Only Mode"
  echo "Jumping to workflow monitoring..."
  echo ""
fi
```

## Iteration-Specific Mode

**Skip to specific iteration:**

```bash
if [[ "$MODE" =~ ^--iteration[[:space:]]+([0-9]+)$ ]]; then
  TARGET_ITERATION="${BASH_REMATCH[1]}"

  echo "🎯 Skipping to Iteration $TARGET_ITERATION"

  # Update state to start at target iteration
  if [ -f "$STATE_FILE" ]; then
    jq --arg iter "$TARGET_ITERATION" \
       '.iteration = ($iter | tonumber) | .current_step = "analyze"' \
       "$STATE_FILE" > "${STATE_FILE}.tmp" && mv "${STATE_FILE}.tmp" "$STATE_FILE"
  fi

  RESUME_STEP=3
fi
```

## Example Usage

### Start new iteration cycle:
```bash
/ci-auto-fix
```

### Resume from saved state (after interruption):
```bash
/ci-auto-fix --resume
```

### Skip to specific iteration:
```bash
/ci-auto-fix --iteration 8
```

### Monitor workflows only:
```bash
/ci-auto-fix --monitor-only
```

## Benefits

✅ **Resumable**: Never lose progress due to interruptions
✅ **Trackable**: Full history of all iterations
✅ **Automated**: End-to-end PR workflow
✅ **Delegated**: Leverages devops-engineer expertise
✅ **Persistent**: State survives restarts
✅ **Flexible**: Multiple execution modes

## State Cleanup

**State file is automatically removed when:**
- All workflows pass (success condition)
- Manual cleanup with: `rm .ci-autofix-state.json`

**State file is preserved when:**
- Failures remain (enables resume)
- Interrupted mid-workflow
- User wants to continue later
