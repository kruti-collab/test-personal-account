---
description: Audit local git branches and their connection to GitHub issues and GitHub PRs
name: review-branches
tags: [git, gh, pr, cleanup, audit]
---

# Branch Review & Audit

Comprehensive audit of local git branches to identify:
- Which branches have open PRs
- Which branches need PRs created
- Which branches are connected to GitHub issues
- Which branches are stale or can be deleted
- Branch-ticket-PR relationship status

## Purpose

Maintain a clean git workflow by:
- **Tracking branch status** - Know which branches are active vs abandoned
- **GitHub integration** - Ensure branches are connected to valid tickets
- **PR management** - Identify branches ready for PR or with merged PRs
- **Cleanup recommendations** - Suggest branches safe to delete
- **Release readiness** - Identify work-in-progress that needs completion

## Variables

- **--stale-days**: Optional number of days to consider a branch stale (default: 30)
- **--include-merged**: Include branches that have been merged (default: false)

## Steps to Perform

### Phase 1: Collect Branch Information

1. **Get all local branches:**
   ```bash
   git branch --format='%(refname:short)'
   ```

2. **For each branch, gather:**
   ```bash
   # Last commit date
   git log -1 --format='%ci' <branch>

   # Last commit author
   git log -1 --format='%an' <branch>

   # Commits ahead/behind main branch
   git rev-list --left-right --count dev...<branch>

   # Check if merged to dev
   git branch --merged dev | grep <branch>
   ```

3. **Extract GitHub issue from branch name:**
   ```bash
   # Look for #XXX pattern
   echo "#243" | grep -oE "^[0-9]+|#[0-9]+"
   ```

### Phase 2: Check GitHub Status

4. **For branches with GitHub issues:**
   ```bash
   # Get ticket status
   gh issue view #XXX --plain | grep STATUS

   # Check if ticket is in current milestone
   gh issue list --search "key = #XXX AND sprint in openSprints()"
   ```

5. **Categorize based on ticket status:**
   - To Do / In Progress → Active work
   - Done / Closed → May be safe to delete
   - Not found → Orphaned branch

### Phase 3: Check PR Status

6. **For each branch, check if PR exists:**
   ```bash
   # List PRs for the branch
   gh pr list --head <branch> --json number,state,title,url
   ```

7. **Categorize PR status:**
   - Open PR → Active review
   - Merged PR → Can delete branch
   - Closed PR → Investigate why closed
   - No PR → Needs PR creation

### Phase 4: Analyze and Categorize

8. **Calculate staleness:**
   ```bash
   # Get days since last commit
   last_commit_date=$(git log -1 --format='%ct' <branch>)
   current_date=$(date +%s)
   days_old=$(( (current_date - last_commit_date) / 86400 ))
   ```

9. **Categorize branches:**

   **🟢 Active Development:**
   - Last commit < 7 days
   - GitHub issue in "In Progress"
   - Has open PR or ready for PR

   **🟡 Needs Attention:**
   - Last commit 7-30 days
   - GitHub issue exists but not in sprint
   - No PR created yet

   **🔴 Stale/Cleanup:**
   - Last commit > 30 days
   - GitHub issue closed/done
   - PR merged or closed
   - Branch merged to dev

   **⚪ Unknown Status:**
   - No GitHub issue pattern
   - Ticket not found
   - Ambiguous state

### Phase 5: Generate Report

10. **Create comprehensive report:**

    ```
    🌳 Branch Review Report
    ======================
    Generated: [date/time]
    Branches analyzed: X
    Stale threshold: X days

    📊 Summary
    ──────────
    🟢 Active: X branches
    🟡 Needs attention: X branches
    🔴 Safe to delete: X branches
    ⚪ Unknown: X branches

    ═════════════════════════════════════════

    🟢 ACTIVE DEVELOPMENT (X branches)
    ═════════════════════════════════════════

    Branch: #295
    ├─ Ticket: #295 - Onboarding Blocked
    ├─ Status: In Progress ✓
    ├─ Sprint: Sprint 239 (v3.82) ✓
    ├─ Last commit: 2 days ago
    ├─ Author: KRUTI
    ├─ Commits: 5 ahead, 2 behind dev
    ├─ PR: #242 (Open) ✓
    └─ Action: Continue development

    Branch: #314
    ├─ Ticket: #314 - Forgot Password
    ├─ Status: To Do
    ├─ Sprint: Sprint 239 (v3.82) ✓
    ├─ Last commit: 1 day ago
    ├─ Author: KRUTI
    ├─ Commits: 3 ahead, 0 behind dev
    ├─ PR: None ⚠️
    └─ Action: Create PR when ready

    ═════════════════════════════════════════

    🟡 NEEDS ATTENTION (X branches)
    ═════════════════════════════════════════

    Branch: #243
    ├─ Ticket: #243 - Claude Code hooks
    ├─ Status: To Do
    ├─ Sprint: Backlog (removed from Sprint 239)
    ├─ Last commit: 15 days ago
    ├─ Author: Shay Panuilov
    ├─ Commits: 12 ahead, 45 behind dev
    ├─ PR: None
    └─ Action: ⚠️ Rebase or abandon - ticket moved to backlog

    Branch: #278
    ├─ Ticket: #278 - Agent management docs
    ├─ Status: To Do
    ├─ Sprint: Backlog
    ├─ Last commit: 22 days ago
    ├─ Author: Shay Panuilov
    ├─ Commits: 8 ahead, 67 behind dev
    ├─ PR: None
    └─ Action: ⚠️ Needs rebase - 67 commits behind

    ═════════════════════════════════════════

    🔴 SAFE TO DELETE (X branches)
    ═════════════════════════════════════════

    Branch: #266
    ├─ Ticket: #266 - Bug going back after creating schedule
    ├─ Status: Done ✓
    ├─ Sprint: Sprint 239 (completed)
    ├─ Last commit: 5 days ago
    ├─ Author: KRUTI
    ├─ PR: #238 (Merged) ✓
    ├─ Merged to dev: Yes ✓
    └─ Action: ✅ Safe to delete

    Branch: #265
    ├─ Ticket: #265 - Employer/Employee building schedule
    ├─ Status: Done ✓
    ├─ Sprint: Sprint 239 (completed)
    ├─ Last commit: 6 days ago
    ├─ Author: KRUTI
    ├─ PR: #237 (Merged) ✓
    ├─ Merged to dev: Yes ✓
    └─ Action: ✅ Safe to delete

    Branch: #211
    ├─ Ticket: #211 - Intercom messenger alignment
    ├─ Status: Done ✓
    ├─ Last commit: 8 days ago
    ├─ PR: #235 (Merged) ✓
    ├─ Merged to dev: Yes ✓
    └─ Action: ✅ Safe to delete

    Branch: #156
    ├─ Ticket: #156 - Account deletion button
    ├─ Status: To Do
    ├─ Sprint: Backlog
    ├─ Last commit: 45 days ago (STALE)
    ├─ Author: Shay Panuilov
    ├─ Commits: 4 ahead, 120 behind dev
    ├─ PR: None
    └─ Action: 🗑️ Delete - Stale and moved to backlog

    ═════════════════════════════════════════

    ⚪ UNKNOWN STATUS (X branches)
    ═════════════════════════════════════════

    Branch: 243
    ├─ Ticket: Unable to parse (no issue number)
    ├─ Last commit: 60 days ago
    ├─ Author: Shay Panuilov
    ├─ Commits: 2 ahead, 150 behind dev
    └─ Action: ⚠️ Investigate or delete

    Branch: experiment-new-feature
    ├─ Ticket: No GitHub issue pattern found
    ├─ Last commit: 3 days ago
    ├─ Author: Shay Panuilov
    ├─ Commits: 7 ahead, 1 behind dev
    └─ Action: ⚠️ Review - no ticket association

    ═════════════════════════════════════════

    💡 RECOMMENDATIONS
    ═════════════════════════════════════════

    Immediate Actions:
    ──────────────────
    1. ✅ DELETE 5 branches (merged PRs):
       git branch -d #266 #265 #211 #249 #250

    2. 🗑️ DELETE 3 stale branches (>30 days, in backlog):
       git branch -D #156 #184 243

    3. 📝 CREATE PRs for 2 ready branches:
       - #314: Forgot password (3 commits, ready)
       - #281: RevenueCat webhook (5 commits, ready)

    4. 🔄 REBASE 4 branches (far behind dev):
       - #243: 45 commits behind
       - #278: 67 commits behind
       - #213: 89 commits behind
       - #204: 102 commits behind

    Sprint 239 Focus:
    ─────────────────
    Keep only these 5 active branches:
    - #271: iOS Supply Chain Attack
    - #281: RevenueCat Webhook Bug
    - #314: Forgot Password
    - #295: Onboarding Blocked
    - #229: Web Upgrades

    Cleanup Impact:
    ───────────────
    Before: 35 local branches
    After cleanup: 12 branches
    Space saved: ~X MB
    Clarity gained: 65% reduction in branch clutter

    ═════════════════════════════════════════

    🎯 NEXT STEPS
    ═════════════════════════════════════════

    1. Review the report above
    2. Execute safe deletions (merged branches)
    3. Investigate unknown/stale branches
    4. Create PRs for ready branches
    5. Rebase or abandon behind branches
    6. Focus on Sprint 239 active work

    Run with --execute to perform recommended cleanup:
    $ /review-branches --execute
    ```

### Phase 6: Interactive Cleanup (Optional)

11. **If --execute flag provided:**
    ```bash
    # Prompt for confirmation before each action
    echo "Delete merged branch #266? (y/n)"
    read response

    if [ "$response" = "y" ]; then
        git branch -d #266
        echo "✓ Deleted #266"
    fi
    ```

## Output Format

The report uses color-coded categories:

- 🟢 **Active** - Current work, keep
- 🟡 **Attention** - Needs action (rebase, PR, etc.)
- 🔴 **Delete** - Safe to remove
- ⚪ **Unknown** - Needs investigation

For each branch, show:
- Branch name
- GitHub issue number and title
- Ticket status (To Do/In Progress/Done)
- Sprint assignment
- Last commit date and age
- Commits ahead/behind dev
- PR status (number, state, URL)
- Recommended action

## Safety Features

✅ **Read-only by default** - Only analyzes, doesn't modify
✅ **Confirmation prompts** - Requires approval for deletions
✅ **Merged verification** - Only suggests deleting merged branches
✅ **Ticket status check** - Verifies GitHub issue before suggesting deletion
✅ **PR status check** - Ensures PR is merged before cleanup
✅ **Stale threshold** - Configurable days before marking stale

## Integration

**Works with:**
- `/release-readiness` - Focus on sprint branches
- `/sprint-cleanup` - Align branches with milestone scope
- `/triage-prs` - Check PR health
- `/describe-pr` - Add context to existing PRs

**Workflow:**
```bash
# 1. Review all branches
/review-branches

# 2. Clean up safe branches
/review-branches --execute

# 3. Focus on sprint work
/release-readiness 239

# 4. Create PRs for ready branches
gh pr create ...
```

## Common Use Cases

### Weekly Branch Cleanup
```bash
# Check what can be cleaned
/review-branches

# Execute cleanup
/review-branches --execute
```

### Sprint Planning
```bash
# See all branches and their tickets
/review-branches

# Focus on current milestone
/review-branches --sprint 239
```

### Pre-Release Audit
```bash
# Identify incomplete work
/review-branches --stale-days 14

# Ensure all release branches have PRs
/review-branches --no-pr-only
```

### Post-Release Cleanup
```bash
# Find merged branches to delete
/review-branches --include-merged

# Clean them up
/review-branches --execute
```

## Best Practices

**Before Cleanup:**
1. Run without --execute first
2. Review recommendations carefully
3. Verify PR merge status
4. Check GitHub issue status
5. Communicate with team

**During Cleanup:**
1. Delete merged branches first
2. Archive instead of delete if unsure
3. Document reasoning for deletions
4. Update team on removed branches

**After Cleanup:**
1. Verify remaining branches
2. Update sprint board
3. Create PRs for ready branches
4. Set up regular cleanup cadence

## Benefits

✅ **Clean repository** - Remove clutter
✅ **Clear focus** - See active work only
✅ **Release confidence** - Know what's in progress
✅ **Team alignment** - Everyone knows branch status
✅ **Faster workflows** - Less confusion
✅ **Audit trail** - Track branch lifecycle

---

**IMPORTANT:** Always verify branch is fully merged and ticket is completed before deleting.
