---
description: Code quality review with complexity-based agent analysis
argument-hint: "[branch-name|pr-number] [--force] [--no-worktree]"
allowed-tools: Bash, Read, Grep, Task
handoffs:
  - label: "Next: VERIFY (SDLC Phase 6)"
    agent: sdlc:6-verify:run
    prompt: Verify deployment and merge after code review passes
  - label: "Fallback: IMPLEMENT (SDLC Phase 4)"
    agent: sdlc:4-implement:run
    prompt: Fix implementation to make tests pass
  - label: "Fallback: TEST (SDLC Phase 3)"
    agent: sdlc:3-test:run
    prompt: Fix tests that are broken or flaky
---

## ⚠️ CRITICAL SCOPE REMINDER

**Phase 5 (REVIEW) is CODE REVIEW ONLY.**

| What Phase 5 Does | What Phase 5 Does NOT Do |
|-------------------|--------------------------|
| ✅ Worktree setup & rebase | ❌ Manual CLI testing |
| ✅ CI status check | ❌ Manual UI testing |
| ✅ PR complexity analysis | ❌ Browser automation |
| ✅ Tier-based agent code review | ❌ Playwright MCP usage |
| ✅ Code review gate | ❌ Any testing beyond CI |

**If you find yourself doing manual tests → STOP → You're in Phase 6 territory.**

Manual testing is handled by `/sdlc:6-verify:run` which delegates to the `manual-tester` agent.

---

## MANDATORY: Initialize Progress Tracking

**Before ANY work, create TodoWrite:**

```javascript
TodoWrite([
  { content: "[P5] Verify code changes exist", status: "pending", activeForm: "Verifying code changes" },
  { content: "[P5] Set up review worktree", status: "pending", activeForm: "Setting up review worktree" },
  { content: "[P5] Rebase onto staging", status: "pending", activeForm: "Rebasing onto staging" },
  { content: "[P5] Wait for CI checks", status: "pending", activeForm: "Waiting for CI checks to pass" },
  { content: "[P5] Analyze PR complexity & tier", status: "pending", activeForm: "Analyzing PR complexity" },
  { content: "[P5] Run tier-based code review", status: "pending", activeForm: "Running tier-based agent review" },
  { content: "[P5] Check code review gate", status: "pending", activeForm: "Checking code review gate" },
  { content: "[P5] Clean up review worktree", status: "pending", activeForm: "Cleaning up worktree" },
  { content: "[P5] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

**Rules (from official docs):**
- Initialize ALL todos upfront before starting work
- Mark `in_progress` BEFORE beginning each step
- Only ONE todo `in_progress` at a time
- Mark `completed` immediately when done

---

## ⛔ STOP: Worktree Setup Required (MANDATORY FIRST STEP)

**CRITICAL: You MUST set up a worktree BEFORE doing ANY other work.**

This is not optional. Worktree prevents:
- Branch mixing issues (accidentally working in wrong branch)
- Conflict between main development and review work
- Lost work from improper rebases

**If you have not yet created a worktree, STOP and do it NOW:**

```bash
# 1. Determine the branch/PR to review
BRANCH_NAME="<branch-name-or-from-pr>"

# 2. Create isolated worktree
WORKTREE_PATH="/tmp/worktrees/review-${BRANCH_NAME//\//-}"
git fetch origin "$BRANCH_NAME"
git worktree add "$WORKTREE_PATH" "$BRANCH_NAME"

# 3. ALL subsequent work happens in worktree
cd "$WORKTREE_PATH"
```

**DO NOT proceed until worktree is active.**

---

## Step 0: Setup & Rebase (REQUIRED FIRST)

**CRITICAL: Before ANY review work, set up the environment properly.**

### 0a. Parse Arguments (Branch Name, PR Number, Flags)

```bash
BRANCH_NAME=""
PR_NUMBER=""
FORCE_FLAG=false
USE_WORKTREE=true  # MANDATORY: Worktree is DEFAULT (use --no-worktree to disable)

for arg in $ARGUMENTS; do
  if [ "$arg" = "--force" ]; then
    FORCE_FLAG=true
  elif [ "$arg" = "--no-worktree" ]; then
    USE_WORKTREE=false
    echo "⚠️  WARNING: Worktree disabled - reviewing in main repo"
  elif [[ "$arg" =~ ^[0-9]+$ ]]; then
    PR_NUMBER="$arg"
  elif [[ "$arg" =~ ^[a-zA-Z] ]]; then
    BRANCH_NAME="$arg"
  fi
done

echo "📋 Arguments parsed:"
echo "   Branch: ${BRANCH_NAME:-'(from current or PR)'}"
echo "   PR#: ${PR_NUMBER:-'(auto-detect)'}"
echo "   Worktree: $USE_WORKTREE (MANDATORY by default)"
echo "   Force: $FORCE_FLAG"
```

### 0b. Set Up Worktree (MANDATORY - First Step)

**⛔ BLOCKING: Worktree is MANDATORY for all PR reviews.**

```bash
if [ "$USE_WORKTREE" = true ]; then
  echo ""
  echo "🔧 Setting up git worktree for isolated review..."

  if [ -n "$BRANCH_NAME" ]; then
    TARGET_BRANCH="$BRANCH_NAME"
  elif [ -n "$PR_NUMBER" ]; then
    TARGET_BRANCH=$(gh pr view $PR_NUMBER --json headRefName --jq '.headRefName')
  else
    echo "❌ Cannot set up worktree: need branch name or PR number"
    exit 1
  fi

  WORKTREE_PATH="/tmp/worktrees/review-${TARGET_BRANCH//\//-}"
  git worktree remove "$WORKTREE_PATH" 2>/dev/null || true
  git fetch origin "$TARGET_BRANCH"
  git worktree add "$WORKTREE_PATH" "$TARGET_BRANCH"

  echo "✅ Worktree created: $WORKTREE_PATH"
  REVIEW_DIR="$WORKTREE_PATH"
else
  REVIEW_DIR="$(pwd)"
fi
```

### 0c. Rebase onto Staging (MANDATORY)

**⛔ BLOCKING: ALWAYS rebase before review to ensure clean merge.**

```bash
cd "$REVIEW_DIR"
git fetch origin staging

BEHIND=$(git rev-list --count HEAD..origin/staging 2>/dev/null || echo "0")

if [ "$BEHIND" -gt 0 ]; then
  echo "   Branch is $BEHIND commits behind staging, rebasing..."

  if git rebase origin/staging; then
    echo "✅ Rebase successful"
    BRANCH=$(git branch --show-current)
    git push origin "$BRANCH" --force-with-lease
    echo "⏳ Waiting 10s for CI to restart after rebase..."
    sleep 10
  else
    echo "❌ Rebase failed - conflicts detected"
    echo ""
    echo "⛔ STOP: Do NOT manually resolve conflicts."
    echo "   MUST invoke: Skill(skill: '_internal:git:resolve-conflicts')"
    echo ""
    echo "The resolve-conflicts skill provides:"
    echo "  - Systematic analysis of both conflict sides"
    echo "  - Context priming for informed decisions"
    echo "  - Documented resolution strategy"
    exit 1
  fi
else
  echo "✅ Branch is up to date with staging"
fi
```

---

## Step 0: Pre-flight CI Check (REQUIRED - BLOCKS REVIEW)

### 0a. Get PR Number

```bash
if [ -z "$PR_NUMBER" ]; then
  PR_NUMBER=$(gh pr view --json number --jq '.number' 2>/dev/null || echo "")
fi

if [ -z "$PR_NUMBER" ]; then
  echo "❌ No PR found for current branch"
  exit 1
fi

echo "📋 Checking PR #$PR_NUMBER"

# SDLC Status Helper - updates PR labels and linked issues
update_sdlc_status() {
  local PR_NUM=$1 PHASE=$2 STATUS=$3
  gh pr edit $PR_NUM --remove-label "sdlc:5-review-in-progress,sdlc:5-review-passed,sdlc:5-review-blocked,sdlc:6-verify-in-progress,sdlc:6-verify-passed,sdlc:6-verify-blocked,sdlc:7-deploy-in-progress,sdlc:7-deploy-complete" 2>/dev/null || true
  gh pr edit $PR_NUM --add-label "sdlc:${PHASE}-${STATUS}" 2>/dev/null || true
  LINKED_ISSUE=$(gh pr view $PR_NUM --json body --jq '.body' 2>/dev/null | grep -oE 'Closes #[0-9]+|Fixes #[0-9]+|Resolves #[0-9]+' | grep -oE '[0-9]+' | head -1)
  [ -n "$LINKED_ISSUE" ] && gh issue comment $LINKED_ISSUE --body "**SDLC Update**: Phase ${PHASE} - ${STATUS}" 2>/dev/null || true
  echo "📊 SDLC Status: ${PHASE} → ${STATUS}"
}

# Set review in-progress status
update_sdlc_status $PR_NUMBER "5-review" "in-progress"
```

### 0b. Check for Merge Conflicts (MUST RESOLVE FIRST)

```bash
MERGEABLE=$(gh pr view $PR_NUMBER --json mergeable --jq '.mergeable')

if [ "$MERGEABLE" = "CONFLICTING" ]; then
  echo ""
  echo "🚨 MERGE CONFLICT DETECTED - BLOCKING"
  echo "   Use: Skill(skill: '_internal:git:resolve-conflicts')"
  exit 1
fi
```

### 0c. Check CI Status

```bash
unset GITHUB_TOKEN
CHECKS_JSON=$(gh pr checks $PR_NUMBER --json name,state,conclusion 2>/dev/null)

if [ -n "$CHECKS_JSON" ]; then
  PASSING=$(echo "$CHECKS_JSON" | jq '[.[] | select(.conclusion == "success")] | length')
  FAILING=$(echo "$CHECKS_JSON" | jq '[.[] | select(.conclusion == "failure")] | length')
  PENDING=$(echo "$CHECKS_JSON" | jq '[.[] | select(.state == "PENDING" or .state == "IN_PROGRESS")] | length')
  TOTAL=$(echo "$CHECKS_JSON" | jq 'length')

  echo ""
  echo "┌─────────────────────────────────────────────┐"
  echo "│  CI STATUS: PR #$PR_NUMBER"
  echo "├─────────────────────────────────────────────┤"
  echo "│  ✅ Passing:  $PASSING"
  echo "│  ❌ Failing:  $FAILING"
  echo "│  ⏳ Pending:  $PENDING"
  echo "│  Total:      $TOTAL"
  echo "└─────────────────────────────────────────────┘"
fi
```

### 0d. Handle Pending/Failing Checks

```bash
if [ "$PENDING" -gt 0 ] && [ "$FORCE_FLAG" != true ]; then
  echo "⏳ CI checks still running - waiting..."
  TIMEOUT=3600
  INTERVAL=60
  ELAPSED=0

  while [ $ELAPSED -lt $TIMEOUT ]; do
    CHECKS_JSON=$(gh pr checks $PR_NUMBER --json name,state,conclusion 2>/dev/null)
    FAILING=$(echo "$CHECKS_JSON" | jq '[.[] | select(.conclusion == "failure")] | length')
    PENDING=$(echo "$CHECKS_JSON" | jq '[.[] | select(.state == "PENDING" or .state == "IN_PROGRESS")] | length')

    if [ "$FAILING" -gt 0 ]; then break; fi
    if [ "$PENDING" -eq 0 ]; then echo "✅ All CI checks complete!"; break; fi

    sleep $INTERVAL
    ELAPSED=$((ELAPSED + INTERVAL))
  done
fi

if [ "$FAILING" -gt 0 ] && [ "$FORCE_FLAG" != true ]; then
  echo "❌ CI checks failing - cannot proceed with review"
  echo "   Fix failures first, then re-run: /sdlc:5-review:run"
  exit 1
fi
```

---

## ⛔ STOP: CI Must Pass Before Agent Launch (CRITICAL - Learning 2026-01-18)

**DO NOT launch any review agents until ALL CI checks complete and pass.**

| ❌ WRONG | ✅ RIGHT |
|----------|----------|
| Launch agents while CI is "pending" | Wait for ALL checks to complete |
| Parallelize agents before CI passes | Sequential: CI passes → THEN agents |
| "CI is mostly passing" | ALL checks must be ✅ success |

**Why this matters:**
- Review agents consume significant context
- If CI fails, you'll need to fix code and re-review anyway
- Wasted agent cycles = wasted context = potential compaction

**Verification before proceeding:**
```bash
# MUST show 0 pending and 0 failing before launching agents
PENDING=$(gh pr checks $PR_NUMBER --json state --jq '[.[] | select(.state == "PENDING" or .state == "IN_PROGRESS")] | length')
FAILING=$(gh pr checks $PR_NUMBER --json conclusion --jq '[.[] | select(.conclusion == "failure")] | length')

if [ "$PENDING" -gt 0 ] || [ "$FAILING" -gt 0 ]; then
  echo "⛔ BLOCKED: Cannot launch review agents"
  echo "   Pending: $PENDING | Failing: $FAILING"
  echo "   Wait for CI to complete before proceeding"
  exit 1
fi

echo "✅ All CI checks passed - proceeding to agent review"
```

---

## Phase Gate Check (REQUIRED)

```bash
HAS_CHANGES=false

if [ -n "$(git status --porcelain)" ]; then HAS_CHANGES=true; fi
COMMITS_AHEAD=$(git log @{u}..HEAD --oneline 2>/dev/null | wc -l | tr -d ' ')
if [ "$COMMITS_AHEAD" -gt 0 ]; then HAS_CHANGES=true; fi
PR_EXISTS=$(gh pr view --json number --jq '.number' 2>/dev/null || echo "")
if [ -n "$PR_EXISTS" ]; then HAS_CHANGES=true; fi

if [ "$HAS_CHANGES" = false ]; then
  echo "❌ PHASE GATE FAILED: No code changes to review"
  echo "   Run: /sdlc:4-implement:run first"
  exit 1
fi

echo "✅ Phase Gate passed. Ready for review."
```

---

## ⛔ Phase 0: Prime Context (OPTIONAL - Context Already Loaded)

**Note (2026-01-14):** The `sdlc:prime:5-review` skill was never implemented.
Context is already loaded via path-specific rules in `.claude/rules/`:
- `sdlc-review-workflow.md` - Review workflow rules
- `sdlc-phase-gates.md` - Phase gate requirements
- `e2e-testing.md` - E2E testing patterns

If you need additional context, use:
```
Skill(skill: "_internal:utilities:prime")
```

| What Happens If Skipped | Why This Matters |
|-------------------------|------------------|
| ~~Missing project context~~ | Rules auto-load via path matching |
| ~~Incomplete understanding~~ | SDLC rules already attached |
| ~~Wasted review cycles~~ | Proceed directly to Step 1 |

**Mark your TodoWrite item as `completed` and proceed to Step 1.**

---

## Step 1: Analyze PR Complexity

```bash
PR_DATA=$(gh pr view $PR_NUMBER --json number,title,files,additions,deletions,changedFiles)
FILES_CHANGED=$(echo "$PR_DATA" | jq '.changedFiles')
TOTAL_LINES=$(echo "$PR_DATA" | jq '.additions + .deletions')
FILE_PATHS=$(echo "$PR_DATA" | jq -r '.files[].path')

# Detect sensitive files
SENSITIVE_PATTERNS=("\.env" "\.github/workflows/" "firebase\.json" "firestore\.(rules|indexes)" "auth" "api.*key")
IS_SENSITIVE=false
for pattern in "${SENSITIVE_PATTERNS[@]}"; do
  if echo "$FILE_PATHS" | grep -qE "$pattern"; then
    IS_SENSITIVE=true
    break
  fi
done

# Determine tier
TIER=1
if [ "$IS_SENSITIVE" = true ] || [ $FILES_CHANGED -gt 25 ] || [ $TOTAL_LINES -gt 1000 ]; then
  TIER=4
elif [ $FILES_CHANGED -gt 10 ] || [ $TOTAL_LINES -gt 500 ]; then
  TIER=3
elif [ $FILES_CHANGED -gt 3 ] || [ $TOTAL_LINES -gt 150 ]; then
  TIER=2
fi

echo "📊 PR #$PR_NUMBER Complexity Analysis"
echo "   Files: $FILES_CHANGED | Lines: $TOTAL_LINES | Sensitive: $IS_SENSITIVE"
echo "   → Tier $TIER Review Selected"
```

### Detect Promotion PRs

```bash
PR_INFO=$(gh pr view $PR_NUMBER --json headRefName,baseRefName)
HEAD_BRANCH=$(echo "$PR_INFO" | jq -r '.headRefName')
BASE_BRANCH=$(echo "$PR_INFO" | jq -r '.baseRefName')

IS_PROMOTION=false
if [ "$HEAD_BRANCH" = "staging" ] && [ "$BASE_BRANCH" = "main" ]; then
  IS_PROMOTION=true
  TIER=2  # Promotion PRs use Tier 2
  echo "📦 PROMOTION PR DETECTED (staging → main) - Using Tier 2"
fi
```

---

## Step 2: Test Coverage & Documentation Check

### Test Coverage Analysis

```bash
SOURCE_FILES=$(echo "$FILE_PATHS" | grep -E '\.(ts|tsx|js|jsx)$' | grep -vE '(test|spec|\.d\.ts|config)')

MISSING_TESTS=()
for file in $SOURCE_FILES; do
  HAS_TEST=false
  for pattern in "${file%.ts}.test.ts" "${file%.ts}.spec.ts"; do
    if [ -f "$pattern" ]; then HAS_TEST=true; break; fi
  done
  if [ "$HAS_TEST" = false ]; then MISSING_TESTS+=("$file"); fi
done

if [ ${#MISSING_TESTS[@]} -gt 0 ]; then
  echo "⚠️  Missing Tests: ${#MISSING_TESTS[@]} files"
fi
```

### Documentation Check

```bash
NEEDS_DOCS=false

if echo "$FILE_PATHS" | grep -qE "pages/.*\.tsx$"; then
  if ! echo "$FILE_PATHS" | grep -qE "docs/|\.md$"; then
    NEEDS_DOCS=true
    echo "⚠️  New page without docs update"
  fi
fi

if echo "$FILE_PATHS" | grep -qE "cli/src/commands"; then
  if ! echo "$FILE_PATHS" | grep -qE "docs/|README"; then
    NEEDS_DOCS=true
    echo "⚠️  CLI command changes without docs"
  fi
fi
```

---

## Step 3: Execute Tier-Appropriate Agent Review

### Tier 1: Lightweight Review (~15 seconds)

**Criteria**: ≤3 files, ≤150 lines, non-sensitive

```bash
if [ "$TIER" -eq 1 ]; then
  Task(
    subagent_type: "general-purpose",
    description: "Quick review for PR #${PR_NUMBER}",
    prompt: "Quick Code Review - PR #${PR_NUMBER}. Check for obvious issues, code style, breaking changes. Report: Issues Found: [COUNT], Recommendation: [APPROVE/NEEDS_FIXES]"
  )
fi
```

### Tier 2: Standard Review (~30 seconds)

**Criteria**: 4-10 files, 151-500 lines

```bash
if [ "$TIER" -eq 2 ]; then
  # Launch 2 agents in parallel
  Task(
    subagent_type: "general-purpose",
    description: "Logic review for PR #${PR_NUMBER}",
    prompt: "Code Logic Review - PR #${PR_NUMBER}. Analyze logic correctness, error handling, state management. Score: __/100, Threshold: ≥75"
  )

  Task(
    subagent_type: "qa-engineer",
    description: "Test review for PR #${PR_NUMBER}",
    prompt: "Test Quality Review - PR #${PR_NUMBER}. Analyze test coverage, quality, edge cases. Score: __/100, Threshold: ≥70"
  )
fi
```

### Tier 3: Thorough Review (~45 seconds)

**Criteria**: 11-25 files, 501-1000 lines

```bash
if [ "$TIER" -eq 3 ]; then
  # Launch 3 agents in parallel
  Task(
    subagent_type: "general-purpose",
    description: "Deep logic review for PR #${PR_NUMBER}",
    prompt: "Deep Logic Review - PR #${PR_NUMBER}. Analyze logic, error handling, null safety, state management. Score: __/100, Threshold: ≥80"
  )

  Task(
    subagent_type: "solutions-architect",
    description: "Architecture review for PR #${PR_NUMBER}",
    prompt: "Architecture Review - PR #${PR_NUMBER}. Analyze MVVM compliance, SOLID principles, code organization. MVVM Score: __/100, Threshold: ≥85"
  )

  Task(
    subagent_type: "qa-engineer",
    description: "Test quality review for PR #${PR_NUMBER}",
    prompt: "Test Quality Review - PR #${PR_NUMBER}. Analyze test completeness, quality, coverage. Score: __/100, Threshold: ≥80"
  )
fi
```

### Tier 4: Ultra-Strict Review (~60 seconds)

**Criteria**: >25 files, >1000 lines, OR sensitive files

```bash
if [ "$TIER" -eq 4 ]; then
  # Launch 5 agents in parallel
  Task(
    subagent_type: "general-purpose",
    description: "Deep logic review for PR #${PR_NUMBER}",
    prompt: "Code Logic & Correctness Review - PR #${PR_NUMBER}. Full analysis of logic, error handling, null safety, edge cases, state management. Score: __/100, Threshold: ≥80"
  )

  Task(
    subagent_type: "solutions-architect",
    description: "Architecture review for PR #${PR_NUMBER}",
    prompt: "Architecture & MVVM Compliance Review - PR #${PR_NUMBER}. Full MVVM pattern analysis, Provider pattern, SOLID principles. MVVM Score: __/100, Threshold: ≥90"
  )

  Task(
    subagent_type: "security-researcher",
    description: "Security audit for PR #${PR_NUMBER}",
    prompt: "Security Audit Review - PR #${PR_NUMBER}. Check secrets management, auth, data validation, API security, dependencies. Critical/High issues: BLOCK PR"
  )

  Task(
    subagent_type: "general-purpose",
    description: "Performance analysis for PR #${PR_NUMBER}",
    prompt: "Performance Analysis Review - PR #${PR_NUMBER}. Analyze algorithm complexity, memory usage, widget performance, network efficiency. Score: __/100, Threshold: ≥70"
  )

  Task(
    subagent_type: "qa-engineer",
    description: "Test quality review for PR #${PR_NUMBER}",
    prompt: "Test Quality Assessment - PR #${PR_NUMBER}. Full test completeness, quality, organization, coverage analysis. Score: __/100, Threshold: ≥85"
  )
fi
```

---

## Step 4: Code Review Gate (BLOCKS PHASE 6)

**CRITICAL: Phase 6 (VERIFY) ONLY runs if code review passes with NO WARNINGS.**

```bash
CODE_REVIEW_PASSED=true
CODE_REVIEW_WARNINGS=()

# Check thresholds based on tier
case "$TIER" in
  1)
    if [ ${ISSUES_FOUND:-0} -gt 0 ]; then
      CODE_REVIEW_PASSED=false
      CODE_REVIEW_WARNINGS+=("Tier 1 review found issues")
    fi
    ;;
  2)
    if [ ${LOGIC_SCORE:-0} -lt 75 ]; then
      CODE_REVIEW_PASSED=false
      CODE_REVIEW_WARNINGS+=("Logic score below 75 threshold")
    fi
    if [ ${TEST_SCORE:-0} -lt 70 ]; then
      CODE_REVIEW_PASSED=false
      CODE_REVIEW_WARNINGS+=("Test score below 70 threshold")
    fi
    ;;
  3)
    if [ ${LOGIC_SCORE:-0} -lt 80 ]; then CODE_REVIEW_PASSED=false; fi
    if [ ${MVVM_SCORE:-0} -lt 85 ]; then CODE_REVIEW_PASSED=false; fi
    if [ ${TEST_SCORE:-0} -lt 80 ]; then CODE_REVIEW_PASSED=false; fi
    ;;
  4)
    if [ ${LOGIC_SCORE:-0} -lt 80 ]; then CODE_REVIEW_PASSED=false; fi
    if [ ${MVVM_SCORE:-0} -lt 90 ]; then CODE_REVIEW_PASSED=false; fi
    if [ ${TEST_SCORE:-0} -lt 85 ]; then CODE_REVIEW_PASSED=false; fi
    if [ ${SECURITY_CRITICAL:-0} -gt 0 ]; then CODE_REVIEW_PASSED=false; fi
    ;;
esac

echo ""
echo "┌─────────────────────────────────────────────┐"
echo "│  🚦 CODE REVIEW GATE                        │"
echo "├─────────────────────────────────────────────┤"

if [ "$CODE_REVIEW_PASSED" = true ]; then
  echo "│  ✅ PASS - Proceeding to Phase 6 (VERIFY)  │"
  echo "└─────────────────────────────────────────────┘"

  # Update SDLC status to passed
  update_sdlc_status $PR_NUMBER "5-review" "passed"

  echo ""
  echo "Next step: /sdlc:6-verify:run $PR_NUMBER"
else
  echo "│  ❌ BLOCKED - Issues must be fixed first   │"
  for warning in "${CODE_REVIEW_WARNINGS[@]}"; do
    echo "│  ⚠️  $warning"
  done
  echo "├─────────────────────────────────────────────┤"
  echo "│  Fix issues → Re-run: /sdlc:5-review:run   │"
  echo "└─────────────────────────────────────────────┘"

  # Update SDLC status to blocked
  update_sdlc_status $PR_NUMBER "5-review" "blocked"

  exit 1
fi
```

---

## Tier Summary Table

| Tier | Criteria | Agents | Thresholds | Time |
|------|----------|--------|------------|------|
| **1: Lightweight** | ≤3 files, ≤150 lines | 1 | No issues | ~15s |
| **2: Standard** | 4-10 files, 151-500 lines | 2 | Logic≥75, Test≥70 | ~30s |
| **3: Thorough** | 11-25 files, 501-1000 lines | 3 | Logic≥80, MVVM≥85, Test≥80 | ~45s |
| **4: Ultra-Strict** | >25 files, >1000 lines, or sensitive | 5 | Logic≥80, MVVM≥90, Test≥85, Security=0 | ~60s |

---

## Error Handling

**If any agent fails:**
```
❌ Code Review Failed

Failed Agent: [Name]
Score: [Score]/[Threshold]
Issues: [List]

Required Actions:
- Fix code issues
- Re-run: /sdlc:5-review:run

Phase 6 (VERIFY) is BLOCKED until fixed.
```

---

## 🧹 Worktree Cleanup (REQUIRED)

After code review completes (pass or fail):

```bash
cd /Users/karabil/Documents/scheduler-systems/products/gal
git worktree remove "$WORKTREE_PATH" --force 2>/dev/null || true
echo "✅ Worktree cleaned up"
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

**Update TodoWrite when complete:**
```javascript
TodoWrite([
  // ... previous steps as completed ...
  { content: "[P5] Capture learnings", status: "completed", activeForm: "Learnings captured" }
])
```

**Even if review was successful, invoke capture-learnings.**

---

## Next Phase

After code review gate passes:
```
/sdlc:6-verify:run $PR_NUMBER
```

This hands off to Phase 6 for:
- Preview deployment verification
- Manual CLI testing (via `manual-tester` agent)
- Manual UI testing (via `manual-tester` agent)
- MCP verification (if applicable)
- Approval and merge

**⚠️ REMINDER: Phase 6 handles ALL manual testing. Do NOT test manually in Phase 5.**
