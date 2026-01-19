---
description: Scan and label PRs needing cleanup, splitting, or recreation
argument-hint: "[base-branch] [--apply-labels] [--dry-run]"
---

# Triage PRs

Automatically scan open PRs to detect issues (wrong base, mixed changes, stale branches) and identify healthy PRs ready for review, applying GitHub labels for systematic cleanup and prioritization.

## Purpose

Catch problematic PRs early through automated detection AND identify healthy PRs ready for review:

**Problem Detection:**
- **Wrong base branch** (PR created from stale commit)
- **Mixed changes** (unrelated changes in single PR)
- **Stale branches** (far behind base)
- **Large PRs** (too many files/commits)
- **Missing descriptions** (no PR description for reviewers)

**Healthy PR Identification:**
- **Ready for review** (clean, focused, well-described PRs)

## Variables

- **BASE_BRANCH**: First argument from $ARGUMENTS (optional, defaults to 'dev')
  - Base branch to check PRs against
  - Examples: 'dev', 'devops', 'main'

- **--apply-labels**: Flag to actually apply labels (default: dry-run)
- **--dry-run**: Flag to show what would be done without applying labels

## Detection Criteria

### 🔴 needs-clean-review (Wrong Base Branch)
**Indicators:**
- Total commits > 100 AND commits ahead > commits in GitHub issue
- Large file count (>30) when ticket suggests small change
- Branch includes files from already-merged tickets

**Example:** #322 had 1,201 commits instead of 1

**Action:** Use `/recreate-pr` to create clean branch

### 🟡 needs-split-review (Mixed Changes)
**Indicators:**
- Changes span 3+ unrelated categories:
  - Source code (lib/)
  - CI/CD (.github/workflows/)
  - Documentation (docs/, README.md)
  - Configuration (.claude/, .devcontainer/)
  - Tests (test/)
- Commits reference multiple ticket numbers
- File count > 20 with diverse purposes

**Action:** Use `/review-pr-scope` + `/split-pr`

### 🟠 stale (Branch Behind)
**Indicators:**
- > 100 commits behind base branch
- No activity for 14+ days
- Likely has merge conflicts

**Action:** Sync or recreate depending on severity

### 🔵 large-pr (Too Large)
**Indicators:**
- > 50 commits
- > 30 files changed
- Could be split for easier review

**Action:** Consider breaking into smaller PRs

### 📝 needs-description (Missing PR Description)
**Indicators:**
- PR body is empty or only whitespace
- No meaningful description of changes
- Missing test plan or related tickets

**Action:** Use `/describe-pr` to generate comprehensive description

### ✅ ready-for-review (Healthy PR)
**Indicators:**
- Commits ahead < 100 (not wrong base)
- Commits behind < 100 (not stale)
- File count < 30 (reasonable size)
- Commits < 50 (manageable review)
- Has PR description (not empty)
- Changes appear focused (< 3 categories OR single logical purpose)

**Benefits:**
- Quickly identify PRs ready for code review
- Prioritize review queue effectively
- Reward clean PR practices

**Action:** Ready for code review - no triage needed

## Label Management Strategy

**Smart Label Sync:**
This command intelligently manages labels by comparing current state with desired state:

**Triage-Managed Labels** (auto-added and auto-removed):
- `needs-clean-review` - Wrong base branch detected
- `needs-split-review` - Mixed unrelated changes
- `stale` - Branch far behind or inactive
- `large-pr` - Too many files/commits
- `needs-description` - Missing PR description
- `ready-for-review` - Healthy and ready for code review

**Label Lifecycle:**
1. **Initial triage**: Labels added based on detection
2. **Developer fixes issue**: (e.g., adds description, rebases branch)
3. **Re-run triage**: Outdated labels automatically removed
4. **New state reflected**: Only current issues remain labeled

**Protection Rules:**
- ✅ **Auto-remove**: Only triage-managed labels listed above
- ❌ **Never remove**: User-added labels (custom labels, priority, team labels)
- ✅ **Always sync**: Labels reflect current PR state, not historical issues

**Example Workflow:**
```bash
# Day 1: PR created without description
/triage-prs dev --apply-labels
# Result: Adds "needs-description" label

# Day 2: Developer adds comprehensive description
/triage-prs dev --apply-labels
# Result: Removes "needs-description", adds "ready-for-review"

# Day 3: PR becomes stale (100+ commits behind)
/triage-prs dev --apply-labels
# Result: Removes "ready-for-review", adds "stale"
```

## Steps to perform:

### Phase 1: Discovery

1. **Get all open PRs:**
   ```bash
   gh pr list --state open --json number,title,headRefName,baseRefName,body,createdAt --limit 100
   ```

2. **For each PR, gather data:**
   ```bash
   # Get PR details including description
   gh pr view $PR_NUMBER --json number,title,body,headRefName,baseRefName,files,commits

   # Get branch stats
   git fetch origin $HEAD_BRANCH
   git rev-list --count origin/$BASE_BRANCH..origin/$HEAD_BRANCH  # Commits ahead
   git rev-list --count origin/$HEAD_BRANCH..origin/$BASE_BRANCH  # Commits behind

   # Get file categories
   gh pr view $PR_NUMBER --json files | jq -r '.files[].path'
   ```

3. **Extract GitHub issue from PR:**
   - Parse ticket ID from title or branch name (e.g., #XXX)
   - Fetch ticket details: `gh issue view #XXX`
   - Estimate expected change size from ticket type/description

### Phase 2: Analysis

4. **Categorize files by type:**
   ```javascript
   Categories:
   - source_code: lib/**, src/**
   - tests: test/**, __tests__/**
   - ci_cd: .github/workflows/**, cloudbuild.yaml
   - documentation: docs/**, README.md, *.md
   - configuration: .claude/**, .devcontainer/**, .vscode/**
   - build: scripts/**, Dockerfile, pubspec.yaml
   ```

5. **Detect issues for each PR:**

   **Wrong Base Detection:**
   ```
   IF commits_ahead > 100 AND (
      commits_ahead > (expected_from_gh * 10) OR
      file_count > 30 AND expected_files < 10 OR
      categories.length > 5
   ) THEN
      issue = "needs-clean-review"
      reason = "Branch appears created from wrong base"
      recommendation = "/recreate-pr $PR_NUMBER [commits-to-keep]"
   ```

   **Mixed Changes Detection:**
   ```
   IF categories.length >= 3 AND (
      has_unrelated_categories(categories) OR
      multiple_ticket_references(commits)
   ) THEN
      issue = "needs-split-review"
      reason = "Multiple unrelated changes detected"
      recommendation = "/review-pr-scope $PR_NUMBER"
   ```

   **Stale Detection:**
   ```
   IF commits_behind > 100 OR age_days > 14 THEN
      issue = "stale"
      reason = "Branch significantly behind base"
      recommendation = "Sync or recreate"
   ```

   **Large PR Detection:**
   ```
   IF commits_ahead > 50 OR file_count > 30 THEN
      issue = "large-pr"
      reason = "PR too large for effective review"
      recommendation = "Consider splitting"
   ```

   **Missing Description Detection:**
   ```
   IF pr_body is empty OR pr_body.trim().length == 0 THEN
      issue = "needs-description"
      reason = "PR lacks description for reviewer context"
      recommendation = "/describe-pr $PR_NUMBER"
   ```

   **Healthy PR Detection:**
   ```
   IF commits_ahead < 100 AND
      commits_behind < 100 AND
      file_count < 30 AND
      commits_ahead < 50 AND
      pr_body is not empty AND
      (categories.length < 3 OR has_single_logical_purpose(files))
   THEN
      status = "ready-for-review"
      reason = "PR is healthy and ready for code review"
      recommendation = "Ready for review"
   ```

### Phase 3: Labeling & Reporting

6. **Fetch current labels and calculate changes (if --apply-labels):**
   ```bash
   # Get current labels for each PR
   gh pr list --state open --json number,labels --limit 100

   # For each PR, compare current labels vs. should-have labels
   # Determine which labels to ADD and which to REMOVE

   # Triage-managed labels (only these can be auto-removed):
   TRIAGE_LABELS=(
     "needs-clean-review"
     "needs-split-review"
     "stale"
     "large-pr"
     "needs-description"
     "ready-for-review"
   )

   # For each PR:
   # - Current labels: What PR currently has
   # - Should-have labels: What analysis determined PR needs
   # - To remove: Current triage labels NOT in should-have list
   # - To add: Should-have labels NOT in current list
   ```

7. **Remove outdated labels (if --apply-labels):**
   ```bash
   # Remove labels that no longer apply
   # ONLY remove triage-managed labels, never user-added labels

   # Example: PR had "needs-description" but now has description
   gh pr edit $PR_NUMBER --remove-label "needs-description"

   # Example: PR had "stale" but was rebased
   gh pr edit $PR_NUMBER --remove-label "stale"

   # Example: PR had "large-pr" but was split
   gh pr edit $PR_NUMBER --remove-label "large-pr"
   ```

8. **Add new labels (if --apply-labels):**
   ```bash
   # Add labels for detected issues
   gh pr edit $PR_NUMBER --add-label "needs-clean-review"
   gh pr edit $PR_NUMBER --add-label "needs-split-review"
   gh pr edit $PR_NUMBER --add-label "stale"
   gh pr edit $PR_NUMBER --add-label "large-pr"
   gh pr edit $PR_NUMBER --add-label "needs-description"

   # For healthy PRs
   gh pr edit $PR_NUMBER --add-label "ready-for-review"
   ```

9. **Add comment to PR (if --apply-labels):**
   ```bash
   gh pr comment $PR_NUMBER --body "## 🤖 Automated PR Triage

This PR has been flagged for review:

**Issue**: needs-clean-review
**Reason**: Branch appears created from wrong base (1,201 commits vs expected ~1)

**Detected:**
- Total commits: 1,201
- Files changed: 98
- Categories: 8 (CI/CD, agents, commands, hooks, docs, security, ...)

**Recommendation:**
Use \`/recreate-pr $PR_NUMBER [commit-hashes]\` to create clean branch with only relevant commits.

**Related Commands:**
- \`/review-pr-scope $PR_NUMBER\` - Analyze which changes to keep
- \`/recreate-pr $PR_NUMBER a98082c3\` - Recreate with specific commits

---
🤖 Auto-generated by /triage-prs"
   ```

10. **Generate summary report:**

   ```
   📊 PR Triage Report
   ===================

   Scanned: 15 open PRs
   Base: $BASE_BRANCH
   Mode: [DRY RUN|APPLIED LABELS]

   🔴 needs-clean-review (2 PRs):
   ─────────────────────────────

   #249: #322
   • Commits: 1,201 (expected ~1)
   • Files: 98 (expected ~3)
   • Categories: 8
   • Reason: Wrong base branch
   • Action: /recreate-pr 249 a98082c3

   #250: #249
   • Commits: 590 (expected ~2)
   • Files: 95
   • Categories: 7
   • Reason: Wrong base branch
   • Action: /recreate-pr 250 b1d4c6b7 8328d83a

   🟡 needs-split-review (3 PRs):
   ──────────────────────────────

   #248: SCH-170
   • Commits: 45
   • Files: 28
   • Categories: CI/CD (10 files), Source (15 files), Docs (3 files)
   • Reason: Mixed changes
   • Action: /review-pr-scope 248

   #244: #312
   • Commits: 30
   • Files: 25
   • Categories: Multiple unrelated
   • Reason: Mixed changes
   • Action: /review-pr-scope 244

   🟠 stale (2 PRs):
   ─────────────────

   #165: #274
   • Behind: 1,115 commits
   • Age: 28 days
   • Action: Sync or recreate

   🔵 large-pr (3 PRs):
   ────────────────────

   #240: #294
   • Commits: 67
   • Files: 35
   • Action: Consider splitting

   📝 needs-description (6 PRs):
   ─────────────────────────────

   #251: Devops
   • Empty description
   • Action: /describe-pr 251

   #250: #249
   • Empty description
   • GitHub: #249
   • Action: /describe-pr 250

   #244, #243, #240, #177
   • All missing descriptions
   • Action: Run /describe-pr on each

   ✅ ready-for-review (5 PRs):
   ────────────────────────────

   #252: #XXX
   • Commits: 15 ahead, 5 behind
   • Files: 8
   • Has description: Yes
   • Status: Ready for code review ✅

   #246, #238, #232, #211
   • All healthy and ready for review

   ═══════════════════════════════
   Summary:
   • Total scanned: 15
   • Need attention: 10 (67%)
   • Ready for review: 5 (33%)
   • Missing descriptions: 6 (40%)

   Labels added: [13 if --apply-labels, 0 if dry-run]
   Labels removed: [5 if --apply-labels, 0 if dry-run]
   Comments added: [10 if --apply-labels, 0 if dry-run]

   ═══════════════════════════════

   🚨 ISSUES DETECTED & FIXES NEEDED
   ═══════════════════════════════

   **Priority 1: Critical Issues** (2 PRs)
   ────────────────────────────────────────

   🔴 Wrong Base Branch:
   • #249 (#322) - 1,201 commits instead of ~1
   • #250 (#249) - 590 commits instead of ~2

   Execute these commands:
   ```bash
   /recreate-pr 249 a98082c3
   /recreate-pr 250 b1d4c6b7 8328d83a
   ```

   **Priority 2: Requires Review** (3 PRs)
   ────────────────────────────────────────

   🟡 Mixed Changes:
   • #248 (SCH-170) - CI/CD + Source + Docs
   • #244 (#312) - Multiple unrelated changes

   Execute these commands:
   ```bash
   /review-pr-scope 248
   /review-pr-scope 244
   ```

   🟠 Stale Branches:
   • #165 (#274) - 1,115 commits behind, 28 days old

   Action needed:
   ```bash
   # Option 1: Rebase
   git checkout #274 && git rebase dev

   # Option 2: Recreate if conflicts too severe
   /recreate-pr 165 [commit-hashes]
   ```

   **Priority 3: Quick Fixes** (9 PRs)
   ────────────────────────────────────────

   📝 Missing Descriptions:
   • #251, #250, #244, #243, #240, #177

   Execute these commands:
   ```bash
   /describe-pr 251
   /describe-pr 250
   /describe-pr 244
   /describe-pr 243
   /describe-pr 240
   /describe-pr 177
   ```

   🔵 Large PRs (review recommended):
   • #240 (#294) - 67 commits, 35 files
   • #235 (#XXX) - Consider splitting

   Optional actions:
   ```bash
   # Evaluate if splitting makes sense
   /review-pr-scope 240
   /split-pr 240  # If needed
   ```

   ═══════════════════════════════

   💡 EXECUTE ALL FIXES:

   Copy and run this command block to fix all issues:
   ```bash
   # Critical: Recreate PRs with wrong base
   /recreate-pr 249 a98082c3
   /recreate-pr 250 b1d4c6b7 8328d83a

   # Review: Check for mixed changes
   /review-pr-scope 248
   /review-pr-scope 244

   # Quick: Add descriptions
   /describe-pr 251
   /describe-pr 250
   /describe-pr 244
   /describe-pr 243
   /describe-pr 240
   /describe-pr 177
   ```
   ```

### Phase 4: Metrics Tracking

11. **Log triage results:**
   ```bash
   # Save to .git/pr-triage-log.jsonl
   {
     "timestamp": "2025-10-10T16:30:00Z",
     "pr_number": 249,
     "issues": ["needs-clean-review"],
     "commits_ahead": 1201,
     "commits_behind": 0,
     "files": 98,
     "categories": 8,
     "action_taken": "labeled"
   }
   ```

## Advanced Options

### Filter by base branch:
```bash
/triage-prs devops  # Only scan PRs targeting devops
```

### Filter by age:
```bash
/triage-prs dev --older-than 7  # Only scan PRs older than 7 days
```

### Filter by size:
```bash
/triage-prs dev --min-commits 20  # Only scan large PRs
```

## Safety Features

- ✅ **Dry-run by default** - Must explicitly apply labels
- ✅ **Smart label management** - Adds AND removes labels based on current state
- ✅ **Protected user labels** - Only removes triage-managed labels, preserves user-added labels
- ✅ **Detailed explanations** - Comments explain why flagged
- ✅ **Actionable recommendations** - Exact commands to fix
- ✅ **Audit trail** - Logs all triage actions

## Integration with Other Commands

Works with:
- **After scan**: Use `/review-pr-scope` on flagged PRs
- **After review**: Use `/recreate-pr` for clean recreation
- **After split**: Use `/split-pr` to separate changes
- **Periodic**: Run weekly to catch issues early

## Example Workflow

```bash
# 1. Scan PRs (dry-run)
/triage-prs dev

# Review output, then apply labels
/triage-prs dev --apply-labels

# 2. Fix detected issues
/recreate-pr 249 a98082c3
/review-pr-scope 248
/split-pr 244

# 3. Re-scan to confirm
/triage-prs dev
```

## Continuous Monitoring

Can be run:
- **Manually**: On-demand PR health check
- **GitHub Action**: Daily automated scan (future enhancement)
- **Pre-merge hook**: Catch issues before merge
- **Milestone review**: Clean up board systematically

## Benefits

✅ **Early detection** - Catch issues before review
✅ **Automated labeling** - Save manual triage time
✅ **Clear actions** - Developers know exactly what to do
✅ **Actionable issue summary** - Copy-paste commands to fix all issues
✅ **Prioritized fixes** - Critical, review-needed, and quick fixes clearly separated
✅ **Systematic cleanup** - Process all PRs consistently
✅ **Prevents technical debt** - Stop bad PRs early
✅ **Metrics tracking** - Measure PR health over time

## When to Use

- **Weekly maintenance**: Scan all open PRs
- **After merges**: Check for stale branches
- **Milestone planning**: Identify PRs needing cleanup
- **Board grooming**: Systematic PR health check
- **Before releases**: Ensure all PRs are clean
