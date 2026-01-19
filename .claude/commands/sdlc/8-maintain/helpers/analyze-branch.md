---
description: Deep forensic investigation of a single git branch with GitHub and PR analysis
argument-hint: "<branch-name>"
tags: [git, gh, pr, investigation, branch-management, forensics]
---

# Analyze Branch

Comprehensive branch analysis to recommend KEEP, REBASE, or DELETE based on branch metadata, GitHub status, PR state, merge detection, and activity.

## Analysis Steps

### 1. Branch Metadata
```bash
# Verify exists
git rev-parse --verify $BRANCH_NAME

# Get details
git rev-parse $BRANCH_NAME
git log -1 --format='%ci|%an <%ae>|%s' $BRANCH_NAME
git rev-list --count $BRANCH_NAME
git rev-list --left-right --count dev...$BRANCH_NAME  # ahead|behind

# Remote status
git rev-parse --abbrev-ref $BRANCH_NAME@{upstream} 2>/dev/null
git ls-remote --heads origin $BRANCH_NAME

# Age
current_date=$(date +%s)
last_commit_epoch=$(git log -1 --format='%ct' $BRANCH_NAME)
days_old=$(( (current_date - last_commit_epoch) / 86400 ))
```

### 2. Merge Detection
```bash
# Commits unique to branch
git rev-list --count dev..$BRANCH_NAME

# Check if work merged (sample commits)
git log dev..$BRANCH_NAME --format='%h|%s' | head -5
```

### 3. GitHub & PR Status
```bash
# Extract ticket
ticket=$(echo "$BRANCH_NAME" | grep -oE "^[0-9]+|#[0-9]+")

# Find PRs
gh pr list --head $BRANCH_NAME --json number,state,title,url
gh pr list --search "$ticket" --state all --json number,state,title,url,headRefName
```

## Recommendation Logic

### ✅ KEEP
**If ANY:**
- Ticket in current milestone
- Recent activity (< 7 days)
- Open PR with recent updates
- Not far behind dev (< 50 commits)
- High/Highest priority ticket

### 🔄 REBASE
**If ALL:**
- (Issue in milestone OR recent < 30 days)
- Work NOT merged (< 50% merged)
- Behind 50-500 commits (manageable)
- PR has conflicts OR needs update

### 🔄 RECREATE WITH CHERRY-PICK
**If ALL:**
- Has real work (commits ahead with actual changes, not just artifacts)
- PR closed/abandoned due to conflicts or review issues
- Far behind dev (> 1000 commits)
- Ticket still active/in progress
- Multiple merge commits creating messy history
- Work needs to be preserved but branch is unrecoverable

**Key insight:** Branch has valuable work but is too diverged to rebase cleanly. Cherry-pick the actual work commits to a fresh branch from updated dev.

### 🗑️ DELETE - Scenario 1: Clean Merged
**If ALL:**
- PR merged
- All commits in dev (>= 90%)
- Remote pruned
- Ticket Done

### 🗑️ DELETE - Scenario 2: Stale Backlog
**If ALL:**
- Stale (> 30 days)
- Ticket in backlog (not sprint)
- Far behind (> 1000 commits)
- No PR or PR closed
- Low/Medium priority

### 🗑️ DELETE - Scenario 3: Superseded
**If:**
- Another branch/PR with same ticket
- Other branch is cleaner/more recent

### 🗑️ DELETE - Scenario 4: Hopeless Divergence
**If ALL:**
- Far behind (> 1000 commits)
- Has commits ahead BUT no PR ever created
- Stale (> 30 days)
- Last commit is merge (artifact indicator)
- Remote exists but not tracking

**Key insight:** "Commits ahead" are divergence artifacts, not real work. DELETE and recreate from fresh dev when work resumes.

## Output Format

```
🔍 Branch Analysis: $BRANCH_NAME
═══════════════════════════════════════════════════════

📊 BRANCH METADATA
──────────────────
Name: $BRANCH_NAME
Last Commit: $COMMIT_HASH ($DAYS_AGO days ago)
Author: $AUTHOR_NAME <$AUTHOR_EMAIL>
Message: $COMMIT_MESSAGE

Commits ahead: $AHEAD_COUNT | behind: $BEHIND_COUNT
Remote: [EXISTS|PRUNED|NEVER_PUSHED]
Activity: [🟢 ACTIVE <7d | 🟡 AGING 7-30d | 🔴 STALE >30d]

═══════════════════════════════════════════════════════

🎫 GITHUB: $TICKET_ID
──────────────────────────
[Fetch details if available, otherwise note extraction only]

═══════════════════════════════════════════════════════

🔀 PULL REQUEST
──────────────────────
[If PR exists: state, URL, review status]
[If NO PR: Note absence and search results]

═══════════════════════════════════════════════════════

💡 RECOMMENDATION: [✅ KEEP | 🔄 REBASE | 🔄 RECREATE | 🗑️ DELETE]
═══════════════════════════════════════════════════════

[If KEEP:]
✅ KEEP - Continue Development

Reasons:
• $PRIMARY_REASON
• $SECONDARY_REASON

Actions:
1. Continue development
2. [If behind:] Sync: git rebase origin/dev
3. [If no PR:] Create PR when ready

[If REBASE:]
🔄 REBASE - Update and Continue

Reasons:
• Behind by $BEHIND_COUNT commits
• Work not yet merged
• Still needed

Commands:
git branch backup/$BRANCH_NAME
git rebase origin/dev $BRANCH_NAME
git push --force-with-lease origin $BRANCH_NAME

[If RECREATE WITH CHERRY-PICK:]
🔄 RECREATE - Cherry-pick to Clean Branch

Analysis:
✓ Real work exists: $AHEAD_COUNT commits with actual changes
✓ Far behind: $BEHIND_COUNT commits (>1000)
✓ PR closed due to: [conflicts/review issues/messy history]
✓ Multiple merge commits polluting history
✓ Ticket still active: [status]
✓ Rebase cost: PROHIBITIVE | Cherry-pick cost: MANAGEABLE

Strategy:
The branch has valuable work but too many merge commits and conflicts
make rebasing impractical. Use the /recreate-pr command to cherry-pick
actual work commits to a fresh branch from updated dev.

Recommended Command:
# 1. First, identify work commits (exclude merge commits)
git log --no-merges --oneline dev..$BRANCH_NAME

# 2. Review output and identify commit hashes with real work
#    (skip commits that are just merges or already in dev)

# 3. Use /recreate-pr command with PR number and commit hashes:
/recreate-pr $PR_NUMBER <commit-hash-1> <commit-hash-2> ...

# Example for PR #160 with 3 work commits:
/recreate-pr 160 7b0e16f4 16c4176f a1676c27

The /recreate-pr command will:
- Create backup of original branch
- Create clean branch from updated dev
- Cherry-pick specified commits
- Push new branch and create new PR
- Close old PR with reference to new one
- Delete old remote branch
- Update GitHub with comment about recreation

[If DELETE - Hopeless Divergence:]
🗑️ DELETE - Hopeless Divergence

Analysis:
✓ Far behind: $BEHIND_COUNT commits (>1000)
✓ Has "commits ahead": $AHEAD_COUNT (artifacts, not real work)
✓ Last activity: $DAYS_AGO days ago (STALE)
✓ No PR ever created
✓ Last commit: Merge from another branch
✓ Remote exists but not tracking
✓ Rebase cost: EXTREME | Recreate cost: LOW

Verdict:
The "$AHEAD_COUNT commits ahead" are divergence artifacts from:
- Branch created from old dev state
- Merge commits from other branches
- Git history divergence over time

NOT real work. Safe to delete.

Commands:
# Delete local and remote
git branch -D $BRANCH_NAME
git push origin --delete $BRANCH_NAME

# Recreate from fresh dev when work resumes
git checkout dev
git pull origin dev
git checkout -b $BRANCH_NAME

[If DELETE - Other scenarios:]
🗑️ DELETE - [Scenario Name]

[Show relevant scenario details and deletion commands]

═══════════════════════════════════════════════════════

📋 DECISION PATH
──────────────────
Example 1 (Recreate):
In Sprint/Active? YES → PR closed? YES → Far Behind (>1000)? YES →
Real work commits? YES → Multiple merges? YES →
🔄 RECREATE (Cherry-pick to clean branch)

Example 2 (Delete):
In Sprint? NO → Recent (<7d)? NO → Merged? NO →
Far Behind (>1000)? YES → Has PR? NO + Stale? YES →
🗑️ DELETE (Hopeless Divergence)
```

## Safety Validations

**NEVER DELETE if:**
- ✋ Ticket in current milestone
- ✋ Recent activity (< 7 days)
- ✋ Open PR with recent updates
- ✋ High/Highest priority ticket

**Exception:** Branches with "commits ahead" CAN be deleted if they match Hopeless Divergence (>1000 behind + no PR + >30d stale). The "commits ahead" are artifacts, not real work.

---

**NOTE:** This provides RECOMMENDATIONS only. Verify before executing deletions.
