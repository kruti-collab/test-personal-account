---
description: List local git branches sorted by age (oldest first)
argument-hint: "[limit]"
tags: [git, branch-management, cleanup, audit]
---

# Oldest Branches

List local git branches sorted by their last commit date (oldest first) to identify potentially stale branches that may need cleanup or review.

## Purpose

Identify oldest branches in your repository to:
- **Find stale branches** - Branches that haven't been touched in months
- **Prioritize cleanup** - Start cleanup with the oldest branches
- **Detect abandoned work** - Find work that was started but never completed
- **Audit inactive development** - See what work has been sitting idle
- **Plan rebase strategy** - Identify branches far behind that need rebasing

**Use Case:** You want to quickly find the oldest branches in your repository to decide which ones to investigate, rebase, or delete.

## Variables

- **LIMIT**: Optional first argument from $ARGUMENTS (default: 20)
  - Number of oldest branches to display
  - Examples: 10, 20, 50, "all"
  - If "all" is provided, shows all branches

## Steps to Perform

### Phase 1: Get Branch List with Dates

1. **List all local branches sorted by commit date (oldest first):**
   ```bash
   git for-each-ref --sort=committerdate --format='%(committerdate:short)|%(refname:short)|%(authorname)|%(subject)' refs/heads/
   ```

   **Output format:**
   ```
   2025-07-08|main|John Doe|Initial commit
   2025-07-21|copilot/fix-140b8fce|Copilot|Fix bug in auth
   2025-08-10|#156|Shay Panuilov|Add account deletion
   ```

2. **Apply limit if provided:**
   ```bash
   # If LIMIT is a number
   git for-each-ref ... | head -n $LIMIT

   # If LIMIT is "all"
   git for-each-ref ...

   # If LIMIT not provided, default to 20
   git for-each-ref ... | head -n 20
   ```

### Phase 2: Extract GitHub Tickets

3. **For each branch, extract GitHub issue if present:**
   ```bash
   # Look for #XXX pattern in branch name
   echo "$BRANCH_NAME" | grep -oE "^[0-9]+|#[0-9]+" || echo "No ticket"
   ```

### Phase 3: Calculate Age and Status

4. **Calculate days since last commit:**
   ```bash
   # Get last commit timestamp
   last_commit_epoch=$(git log -1 --format='%ct' $BRANCH_NAME)

   # Get current timestamp
   current_epoch=$(date +%s)

   # Calculate days
   days_old=$(( (current_epoch - last_commit_epoch) / 86400 ))
   ```

5. **Check if branch is merged to dev:**
   ```bash
   # Check if branch is fully merged into dev
   git branch --merged dev | grep "^  $BRANCH_NAME$"

   # If found, mark as "Merged ✓"
   # If not found, mark as "Active"
   ```

6. **Check commits ahead/behind dev:**
   ```bash
   # Get ahead/behind counts
   git rev-list --left-right --count dev...$BRANCH_NAME
   # Output: "<ahead>\t<behind>"
   ```

7. **Check remote tracking:**
   ```bash
   # Get upstream branch
   git rev-parse --abbrev-ref $BRANCH_NAME@{upstream} 2>/dev/null

   # Check if remote exists
   git ls-remote --heads origin $BRANCH_NAME
   ```

### Phase 4: Categorize by Age

8. **Apply age categorization:**

   **🟢 Recent (< 30 days):**
   - Still active or recently worked on
   - Likely relevant to current work

   **🟡 Aging (30-90 days):**
   - Getting old, may need attention
   - Review if still needed

   **🔴 Stale (90-180 days):**
   - Very old, likely needs cleanup
   - High priority for review

   **⚫ Ancient (> 180 days):**
   - Extremely old, almost certainly stale
   - Investigate or delete

### Phase 5: Generate Report

9. **Create ordered list with details:**

   ```
   📅 Oldest Branches Report
   ═════════════════════════════════════════
   Generated: [date/time]
   Branches analyzed: X
   Showing: [oldest X] or [all branches]

   🎯 Age Distribution
   ───────────────────
   🟢 Recent (<30d):    X branches
   🟡 Aging (30-90d):   X branches
   🔴 Stale (90-180d):  X branches
   ⚫ Ancient (>180d):  X branches

   ═════════════════════════════════════════

   📋 OLDEST BRANCHES (sorted oldest → newest)
   ═════════════════════════════════════════

   1. main
      📅 Age: 95 days old (2025-07-08)
      🏷️  Ticket: N/A (main branch)
      👤 Author: Initial commit by John Doe
      📊 Status: Base branch
      🔄 Commits: N/A (base branch)
      🌐 Remote: origin/main (tracking)
      💡 Action: Keep (main branch)

   2. copilot/fix-140b8fce-54aa-4290-8ae6-883594b5b414
      📅 Age: 81 days old (2025-07-21)
      🏷️  Ticket: No SMR pattern found
      👤 Author: Copilot
      📊 Status: Active
      🔄 Commits: 3 ahead, 245 behind dev
      🌐 Remote: None (never pushed)
      💡 Action: ⚠️ Review - likely abandoned Copilot experiment

   3. #156
      📅 Age: 61 days old (2025-08-10)
      🏷️  Ticket: #156 - Account deletion button
      👤 Author: Shay Panuilov
      📊 Status: Active (not merged)
      🔄 Commits: 4 ahead, 320 behind dev
      🌐 Remote: origin/#156 (pruned)
      💡 Action: 🔴 Rebase or delete - far behind dev

   4. #184
      📅 Age: 61 days old (2025-08-10)
      🏷️  Ticket: #184 - Feature description
      👤 Author: Developer Name
      📊 Status: Merged ✓
      🔄 Commits: 0 ahead, 0 behind dev
      🌐 Remote: origin/#184 (exists)
      💡 Action: ✅ Safe to delete - fully merged

   5. #213
      📅 Age: 61 days old (2025-08-10)
      🏷️  Ticket: #213 - Another feature
      👤 Author: Developer Name
      📊 Status: Active
      🔄 Commits: 8 ahead, 450 behind dev
      🌐 Remote: None
      💡 Action: ⚠️ Investigate - 61 days old, far behind

   ... [continue for all branches]

   ═════════════════════════════════════════

   💡 CLEANUP RECOMMENDATIONS
   ═════════════════════════════════════════

   Immediate Cleanup (Merged):
   ──────────────────────────
   git branch -d #184 #265 #266

   Review for Deletion (Ancient + Far Behind):
   ───────────────────────────────────────────
   - #156 (61 days, 320 behind)
   - #213 (61 days, 450 behind)
   - copilot/fix-* (81 days, 245 behind)

   Needs Investigation:
   ────────────────────
   - Branches > 90 days old without PRs
   - Branches with pruned remotes
   - Branches far behind dev (>100 commits)

   ═════════════════════════════════════════

   🎯 NEXT STEPS
   ═════════════════════════════════════════

   1. Delete merged branches (safe cleanup)
   2. Investigate ancient/stale branches:
      /analyze-branch #156
      /analyze-branch #213
   3. Review branches without GitHub issues
   4. Rebase or delete far-behind branches
   5. Set up regular branch cleanup schedule

   💡 TIP: Use /analyze-branch <name> for detailed analysis
   💡 TIP: Use /review-branches for comprehensive audit
   ```

## Output Format

For each branch, display:
- **Rank**: Position in oldest-to-newest list
- **Branch name**: Full branch name
- **Age**: Days old and last commit date
- **Ticket**: GitHub issue if found (#XXX pattern)
- **Author**: Last commit author
- **Status**: Merged or Active
- **Commits**: Ahead/behind dev
- **Remote**: Tracking status and existence
- **Action**: Recommended next step

## Age-based Recommendations

### 🟢 Recent (< 30 days)
```
💡 Action: Continue monitoring
```

### 🟡 Aging (30-90 days)
```
💡 Action: Review if still needed
         Consider rebasing if far behind
         Check GitHub issue status
```

### 🔴 Stale (90-180 days)
```
💡 Action: High priority review
         Rebase or delete
         Check if work was merged elsewhere
```

### ⚫ Ancient (> 180 days)
```
💡 Action: Investigate immediately
         Likely safe to delete
         Verify no valuable work would be lost
```

## Integration with Other Commands

**Works with:**
- `/analyze-branch` - Deep dive into specific old branches
- `/review-branches` - Comprehensive audit of all branches
- `/triage-prs` - Check PR status for old branches

**Workflow:**
```bash
# 1. Find oldest branches
/oldest-branches 30

# 2. Analyze suspicious old branches
/analyze-branch #156
/analyze-branch #213

# 3. Comprehensive review
/review-branches

# 4. Execute cleanup based on recommendations
git branch -d <merged-branches>
git branch -D <safe-to-delete-branches>
```

## Common Use Cases

### Quick Cleanup Audit
```bash
# Find 10 oldest branches
/oldest-branches 10

# Quick visual scan for obviously stale branches
```

### Monthly Branch Maintenance
```bash
# List top 30 oldest branches
/oldest-branches 30

# Identify branches older than 90 days
# Use /analyze-branch for detailed investigation
```

### Pre-Release Cleanup
```bash
# Find all old branches
/oldest-branches all

# Focus on branches > 180 days
# Clean up before major release
```

### Historical Analysis
```bash
# See all branches by age
/oldest-branches all

# Understand development timeline
# Identify long-running feature branches
```

## Safety Considerations

**Safe to Delete Indicators:**
- ✅ Merged to dev
- ✅ PR merged
- ✅ GitHub issue Done/Closed
- ✅ Remote pruned
- ✅ No commits ahead of dev

**Review Before Delete:**
- ⚠️ Has commits ahead of dev
- ⚠️ No PR but has work
- ⚠️ GitHub issue still open
- ⚠️ Recent activity despite age
- ⚠️ Not merged

**NEVER Delete:**
- ❌ Main/dev/master branches
- ❌ Branches with unmerged work needed
- ❌ Active sprint branches
- ❌ Branches with open PRs

## Benefits

✅ **Quick identification** - Find oldest branches instantly
✅ **Cleanup prioritization** - Start with most stale
✅ **Age visualization** - See branch timeline clearly
✅ **Actionable recommendations** - Know what to do next
✅ **Historical insight** - Understand development patterns
✅ **Maintenance planning** - Regular cleanup workflow
✅ **Team coordination** - Everyone sees stale work

## Example Output Summary

```
📅 Oldest Branches Report
═════════════════════════
Showing oldest 20 branches

🎯 Age Distribution
🟢 Recent:  5 branches
🟡 Aging:   8 branches
🔴 Stale:   4 branches
⚫ Ancient: 3 branches

💡 Cleanup Potential
• 6 merged (safe delete)
• 7 stale (review needed)
• 4 ancient (high priority)
```

---

**IMPORTANT:** This command provides quick age-based analysis. For comprehensive branch investigation including GitHub and PR status, use `/analyze-branch <name>` for specific branches or `/review-branches` for complete audit.
