---
allowed-tools: Skill, Bash, Write, Read
argument-hint: <bug-url-or-issue-number> [description]
description: Bug fix workflow using SDLC steps 1-7 with TDD enforcement
handoffs:
  - label: "Manual Review Needed"
    agent: sdlc:5-review:run
    prompt: Review and finalize PR
---

# Bug Fix - SDLC Steps 1-7 Workflow

Bug fixes follow the **same SDLC workflow as features**, enforcing TDD throughout.

## Philosophy

**Bugs are NOT special cases.** They follow the same quality process:
- SPECIFY: Document what's broken and expected behavior
- DESIGN: Plan the minimal fix approach
- TEST: Write failing regression test (TDD Red)
- IMPLEMENT: Fix to pass test (TDD Green)
- REVIEW: Code review and PR
- VERIFY: Manual testing
- DEPLOY: Merge to staging

**MAINTAIN (step 8)** is for bug **triage**, not bug **fixing**.

## Variables

- **BUG_INPUT**: First argument from $ARGUMENTS
  - GitHub issue number (e.g., `#431`, `431`)
  - Full GitHub issue URL (e.g., `https://github.com/Scheduler-Systems/gal/issues/431`)
  - GitHub issue (e.g., `#278`)
- **DESCRIPTION**: Additional context from remaining arguments (optional)

## Usage

```bash
# GitHub issue by number
/bug-fix 431

# GitHub issue by URL
/bug-fix https://github.com/Scheduler-Systems/gal/issues/431

# GitHub issue
/bug-fix #278

# With additional context
/bug-fix 431 "Org sync deletes orgs when user not admin"
```

## Workflow (SDLC Steps 1-7)

### Phase 0: Parse Input and Setup

```bash
BUG_INPUT="$1"
DESCRIPTION="${@:2}"

# Determine bug source (GitHub vs GitHub)
if echo "$BUG_INPUT" | grep -qE '^(https?://github\.com/.*/issues/[0-9]+|#?[0-9]+)$'; then
  BUG_TYPE="github"
  ISSUE_NUMBER=$(echo "$BUG_INPUT" | grep -oE '[0-9]+')
elif echo "$BUG_INPUT" | grep -qE '^#[0-9]+$'; then
  BUG_TYPE="gh"
  ISSUE_ID="$BUG_INPUT"
else
  echo "Invalid bug identifier. Use GitHub issue (#431) or GitHub issue (#278)"
  exit 1
fi

echo "═══════════════════════════════════════════════════════════"
echo "  SDLC BUG FIX WORKFLOW (Steps 1-7)"
echo "═══════════════════════════════════════════════════════════"
echo ""
echo "Bug: $BUG_INPUT"
echo "Description: ${DESCRIPTION:-[From ticket]}"
echo ""
echo "Step 1: SPECIFY  - Create bug specification"
echo "Step 2: DESIGN   - Plan fix approach"
echo "Step 3: TEST     - Write failing test (TDD Red)"
echo "Step 4: IMPLEMENT - Fix bug (TDD Green)"
echo "Step 5: REVIEW   - Code review and PR"
echo "Step 6: VERIFY   - Manual testing"
echo "Step 7: DEPLOY   - Merge to staging"
echo ""
echo "═══════════════════════════════════════════════════════════"
```

### Step 1: SPECIFY - Create Bug Specification

Create a bug spec documenting:
- Current behavior (broken)
- Expected behavior (correct)
- Root cause analysis
- Reproduction steps

```markdown
Skill(skill: "sdlc:1-specify:run", arguments: "$BUG_INPUT $DESCRIPTION")
```

**Gate Check**: Bug spec must exist before proceeding.

### Step 2: DESIGN - Plan Fix Approach

Design the minimal fix:
- Identify exact code location
- Plan changes needed
- Consider side effects

```markdown
Skill(skill: "sdlc:2-design:run")
```

**Gate Check**: plan.md must exist before proceeding.

### Step 3: TEST - Write Failing Test (TDD Red)

Write regression test that:
- Reproduces the bug
- MUST FAIL initially (proves bug exists)
- Will PASS after fix

```markdown
Skill(skill: "sdlc:3-test:run")
```

**TDD Red Validation:**
```bash
# Test MUST fail before fix
echo "TDD RED: Running test to confirm bug exists..."
# If test passes, bug may already be fixed - warn user
```

**Gate Check**: Test file must exist AND fail.

### Step 4: IMPLEMENT - Fix Bug (TDD Green)

Implement minimal fix to make test pass:
- Focus on root cause
- Avoid over-engineering
- Test must pass after fix

```markdown
Skill(skill: "sdlc:4-implement:run")
```

**TDD Green Validation:**
```bash
# Test MUST pass after fix
echo "TDD GREEN: Running test to confirm fix works..."
# If test fails, fix is incomplete
```

**Gate Check**: Test must pass.

#### Step 4.1: Visual Verification (MANDATORY)

**STOP and verify manually before committing.**

Provide user with manual testing guide:

```
═══════════════════════════════════════════════════════════
  MANUAL VERIFICATION REQUIRED
═══════════════════════════════════════════════════════════

Before committing, please verify the fix manually:

1. Start local environment:
   ./scripts/run.sh

2. Open browser:
   http://localhost:5173

3. Test the bug scenario:
   [Provide specific steps to reproduce the original bug]

4. Verify fix works:
   [Describe expected behavior after fix]

5. Check for regressions:
   - Test related functionality
   - Verify no new errors in console

═══════════════════════════════════════════════════════════
```

**ASK USER**: "Have you verified the fix works manually? (yes/no)"

**Only proceed to commit after user confirms.**

#### Step 4.2: Commit and Create PR (User Consent Required)

**After user confirms fix works visually, ask for commit consent:**

```
═══════════════════════════════════════════════════════════
  READY TO COMMIT AND CREATE PR
═══════════════════════════════════════════════════════════

You have verified the fix works. Ready to:

1. Commit changes with message:
   [branch-name] Fix for #$ISSUE_NUMBER - $DESCRIPTION

2. Push to remote branch

3. Create PR targeting staging with:
   - Title: Fix #$ISSUE_NUMBER - $DESCRIPTION
   - Body: Closes #$ISSUE_NUMBER
   - Link to issue for auto-close

═══════════════════════════════════════════════════════════
```

**ASK USER**: "Should I commit, push, and create the PR now? (yes/no)"

**If user says YES:**
```bash
# Stage all changes
git add -A

# Commit with branch name prefix
BRANCH=$(git branch --show-current)
git commit -m "[$BRANCH] Fix #$ISSUE_NUMBER - $DESCRIPTION

Closes #$ISSUE_NUMBER

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"

# Push to remote
git push origin $BRANCH

# Create PR
gh pr create --base staging --title "Fix #$ISSUE_NUMBER - $DESCRIPTION" --body "## Summary
- Fixed bug where [describe bug]
- Root cause: [describe root cause]
- Solution: [describe fix]

## Test Plan
- [ ] Unit/integration tests pass
- [ ] Manual verification completed
- [ ] No regressions observed

Closes #$ISSUE_NUMBER

🤖 Generated with [Claude Code](https://claude.com/claude-code)"
```

**If user says NO:**
- Ask what they want to change before committing
- Make requested changes
- Ask again

**Gate Check**: PR created successfully.

#### Step 4.3: Capture Learnings

**After PR is created, ask user to capture any SDLC improvements:**

```
═══════════════════════════════════════════════════════════
  CAPTURE LEARNINGS
═══════════════════════════════════════════════════════════

Did you discover any improvements to SDLC commands during this fix?

Examples:
- Missing validation steps
- Better error messages needed
- New patterns to document
- Process improvements

If yes, run: /capture-learnings

This creates a separate PR for agentic layer improvements.
═══════════════════════════════════════════════════════════
```

**This step is optional but encouraged.**

### Step 5: REVIEW - Code Review

Review the PR created in Step 4.3:
- Wait for CI checks to pass
- Review code quality
- Address any review comments

```markdown
Skill(skill: "sdlc:5-review:run")
```

**Note**: PR was already created in Step 4.3 after user consent.

**Gate Check**: CI checks pass, code review approved.

### Step 6: VERIFY - Manual Testing

Manual verification of the fix:
- Test in preview environment
- Verify bug is fixed
- Check for regressions

```markdown
Skill(skill: "sdlc:6-verify:run")
```

**Gate Check**: Manual verification passed.

### Step 7: DEPLOY - Merge to Staging

Merge PR and deploy:
- Wait for CI to pass
- Merge to staging
- Close GitHub issue

```markdown
Skill(skill: "sdlc:7-deploy:run")
```

**Final Action**: Auto-close the GitHub issue.

## Workflow Summary

```
═══════════════════════════════════════════════════════════
  BUG FIX COMPLETE (SDLC Steps 1-7)
═══════════════════════════════════════════════════════════

Bug: $BUG_INPUT

SDLC Steps Completed:
  1. SPECIFY   - Bug spec created
  2. DESIGN    - Fix approach planned
  3. TEST      - Failing test written (TDD Red)
  4. IMPLEMENT - Bug fixed (TDD Green)
     4.1 Visual Verification - User confirmed fix works
     4.2 Commit & PR         - Changes committed, PR created
     4.3 Capture Learnings   - SDLC improvements captured
  5. REVIEW    - Code review passed
  6. VERIFY    - Manual testing passed
  7. DEPLOY    - Merged to staging

TDD Workflow:
  RED   - Test failed (bug confirmed)
  GREEN - Test passed (bug fixed)
  USER  - Visually verified fix works

PR Link: [PR URL]
Issue Status: CLOSED

═══════════════════════════════════════════════════════════
```

## TDD Principles

1. **Red First**: Test MUST fail before fix
   - Proves test actually catches the bug
   - Prevents false positives

2. **Green Second**: Minimal fix to pass test
   - Focus on the specific issue
   - Avoid scope creep

3. **Refactor**: Clean up in code review
   - PR review handles cleanup suggestions
   - Keep fix focused and reviewable

## Why Steps 1-7 for Bugs?

| Step | For Features | For Bugs |
|------|--------------|----------|
| 1-SPECIFY | Requirements doc | Bug description + root cause |
| 2-DESIGN | Architecture plan | Fix approach |
| 3-TEST | E2E/unit tests | Regression test |
| 4-IMPLEMENT | Build feature | Fix bug |
| 5-REVIEW | Code review | Code review |
| 6-VERIFY | Manual testing | Verify fix |
| 7-DEPLOY | Deploy feature | Deploy fix |

**Same process, same quality.**

## MAINTAIN (Step 8) is Different

**Step 8 (MAINTAIN)** is for:
- Bug **triage** (not fixing)
- CI/CD maintenance
- Infrastructure issues
- Post-deployment monitoring

**Don't confuse triage with fixing.** Triage identifies bugs. Steps 1-7 fix them.

## Error Recovery

If any step fails:

```bash
# Step 1 failed - no spec
echo "Recovery: Run /sdlc:1-specify:run $BUG_INPUT"

# Step 2 failed - no plan
echo "Recovery: Run /sdlc:2-design:run"

# Step 3 failed - no test
echo "Recovery: Run /sdlc:3-test:run"

# Step 4 failed - test still fails
echo "Recovery: Run /sdlc:4-implement:run"

# Step 5 failed - no PR
echo "Recovery: Run /sdlc:5-review:run"

# Step 6 failed - verification failed
echo "Recovery: Run /sdlc:6-verify:run"

# Step 7 failed - not merged
echo "Recovery: Run /sdlc:7-deploy:run"
```

## Notes

- **Default branch**: PRs target `staging` (not main)
- **TDD enforced**: Tests must fail before fix, pass after
- **Same workflow**: Features and bugs use identical steps
- **MAINTAIN is step 8**: Triage, not fixing
