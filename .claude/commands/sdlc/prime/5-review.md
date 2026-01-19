# Prime REVIEW - Load PR Review Context

Prime context window with everything needed for code review and PR approval.

## Step 1: Read Review Documentation

Read these files to understand review process:

1. Read `docs/sdlc/5-review/README.md`
2. Read `docs/sdlc/5-review/operations/PRODUCTION_READINESS.md` (if exists)
3. Read `.claude/rules/project-overview.md`

## Step 2: Read Security Guidelines

Read security requirements for review:

1. Read `apps/api/SECURITY.md`
2. Read `docs/security/SECURITY-POLICY.md` (if exists)

## Step 3: Check Current PR State

```bash
# Current branch
BRANCH=$(git branch --show-current)
echo "Branch: $BRANCH"

# Check if PR exists
gh pr view --json number,title,state,additions,deletions,changedFiles 2>/dev/null || echo "No PR for this branch"

# List changed files
git diff --name-only origin/staging...HEAD 2>/dev/null | head -20

# Check CI status
gh pr checks 2>/dev/null || echo "No PR checks available"
```

## Step 4: Load PR Context

```bash
# Get PR diff summary
gh pr diff --stat 2>/dev/null | head -30

# Get PR description
gh pr view --json body --jq '.body' 2>/dev/null | head -50

# List PR comments
gh pr view --json comments --jq '.comments[].body' 2>/dev/null | head -20
```

## Step 5: Read Changed Files

For each significantly changed file:
1. Read the current version
2. Understand the changes in context
3. Check for patterns violations

## PR Review Checklist (Memorize This)

### Code Quality
- [ ] No hardcoded secrets or credentials
- [ ] Error handling is comprehensive
- [ ] No console.log in production code
- [ ] Types are properly defined
- [ ] No `any` types without justification

### Architecture
- [ ] Follows Clean Architecture layers
- [ ] Dependencies point inward
- [ ] Business logic in @gal/core
- [ ] No direct Firestore calls from routes

### Security
- [ ] Auth checks on all protected routes
- [ ] Input validation present
- [ ] No sensitive data in logs
- [ ] Rate limiting on auth endpoints

### Testing
- [ ] Tests added for new functionality
- [ ] Existing tests still pass
- [ ] Edge cases covered
- [ ] No flaky test patterns

### Documentation
- [ ] Complex code is documented
- [ ] API changes documented
- [ ] Breaking changes noted

### Git Hygiene
- [ ] Commit messages follow convention
- [ ] No merge commits (rebase preferred)
- [ ] Branch named correctly

## Review Tiers

| Tier | Files | Lines | Review Depth |
|------|-------|-------|--------------|
| 1: Lightweight | ≤3 | ≤150 | Quick scan |
| 2: Standard | 4-10 | 151-500 | Logic + Tests |
| 3: Thorough | 11-25 | 501-1000 | + Architecture |
| 4: Ultra-Strict | >25 | >1000 | Full audit |

### Sensitive Files (Auto Tier 4)
- `.env*`
- `.github/workflows/`
- `firebase.json`
- `firestore.rules`
- `**/auth/**`
- `**/secrets/**`

## CRITICAL RULES (MUST FOLLOW)

### CI Check Protocol

**NEVER approve or merge a PR until ALL CI checks pass.**

| Bad Pattern | Better Approach |
|-------------|-----------------|
| Run tests locally while CI runs | Wait for CI checks to complete |
| Approve while "pending" checks exist | Wait for ALL checks (including informational) |
| Use `--admin` to bypass failing checks | Fix the failure first |
| Continue code review with failed checks | Stop, investigate, and fix on the spot |

```bash
# Wait for all checks
gh pr checks $PR_NUMBER --watch

# Don't proceed until all passing:
gh pr checks $PR_NUMBER | grep -E "fail|pending"
```

### Fix Failures On The Spot

When discovering a bug or failure during SDLC workflow:

1. **Don't skip past it** - Fix immediately while having context
2. **Include fix in same PR** if related to the feature
3. **Create separate PR** if unrelated infrastructure fix
4. **Never use `--admin` bypass** - That's hiding the problem

### GitHub Issue Linkage

**ALWAYS link GitHub issues in PR descriptions for auto-close:**

```markdown
# CORRECT - Auto-closes issue when PR merges:
Closes #123
Fixes #456
Resolves #789

# WRONG - Does NOT auto-close:
Relates to #123
Related: #123
```

**SDLC Review Phase must verify issue linkage:**
```bash
# Check if PR body has closing keywords
gh pr view $PR_NUMBER --json body --jq '.body' | grep -iE "closes|fixes|resolves"

# If not found, edit PR to add:
gh pr edit $PR_NUMBER --body "$(gh pr view $PR_NUMBER --json body --jq '.body')

Closes #$ISSUE_NUMBER"
```

## After Priming

You are now ready to run:
- `/sdlc/review/run [pr-number]` - Full PR review
- `/sdlc/review/helpers/commit-changes` - Commit with proper message
- `/sdlc/review/helpers/describe-pr` - Generate PR description

## Report After Priming

Summarize:
1. Current branch and PR status
2. Files changed and line count
3. CI check status
4. Review tier recommendation
5. Ready to proceed with review

