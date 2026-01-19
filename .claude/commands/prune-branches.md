---
description: Systematic git branch cleanup and pruning for clean repository state
argument-hint: "[--dry-run] [--aggressive] [--include-stashes]"
allowed-tools: Bash, Read, Write
---

# 🧹 Git Branch Pruning - Keep Repository Clean

**Purpose**: Systematically clean up merged branches, stale remotes, old worktrees, and optional stashes to maintain a clean git graph.

## Usage

```bash
# Preview what would be cleaned (recommended first run)
/prune-branches --dry-run

# Clean merged branches and stale remotes
/prune-branches

# Aggressive cleanup including old stashes and worktrees
/prune-branches --aggressive

# Include stash cleanup (requires manual confirmation)
/prune-branches --include-stashes
```

## Implementation

### Step 1: Parse Arguments and Setup

```bash
DRY_RUN=false
AGGRESSIVE=false
INCLUDE_STASHES=false

for arg in $ARGUMENTS; do
  if [ "$arg" = "--dry-run" ]; then
    DRY_RUN=true
  elif [ "$arg" = "--aggressive" ]; then
    AGGRESSIVE=true
  elif [ "$arg" = "--include-stashes" ]; then
    INCLUDE_STASHES=true
  fi
done

echo "🧹 Git Branch Pruning Tool"
echo "═══════════════════════════════════════════════════════════"
echo ""

if [ "$DRY_RUN" = true ]; then
  echo "🔍 DRY RUN MODE - No changes will be made"
  echo ""
fi
```

### Step 2: Fetch and Prune Remote References

```bash
echo "📡 FETCHING & PRUNING REMOTE REFERENCES"
echo "──────────────────────────────────────────────────────────"

if [ "$DRY_RUN" = true ]; then
  echo "Would run: git fetch --prune --prune-tags origin"
  echo "Would run: git remote prune origin"
else
  echo "Fetching latest refs and pruning deleted remotes..."
  git fetch --prune --prune-tags origin
  git remote prune origin
  echo "✅ Remote references updated and pruned"
fi

echo ""
```

### Step 3: Identify Merged Branches

```bash
echo "🔍 ANALYZING LOCAL BRANCHES"
echo "──────────────────────────────────────────────────────────"

CURRENT_BRANCH=$(git branch --show-current)
echo "Current branch: $CURRENT_BRANCH"

# Get list of merged branches (excluding main/staging/current)
MERGED_BRANCHES=$(git branch --merged | grep -v '^\*' | grep -v 'main\|staging' | sed 's/^[ ]*//g')

if [ -n "$MERGED_BRANCHES" ]; then
  echo ""
  echo "📋 Branches merged into current branch:"
  for branch in $MERGED_BRANCHES; do
    # Check if PR exists and is merged
    PR_STATE=$(gh pr list --state all --head "$branch" --json state --jq '.[0].state // "NO_PR"' 2>/dev/null || echo "NO_PR")

    if [ "$PR_STATE" = "MERGED" ]; then
      echo "  🟢 $branch (PR merged)"
    elif [ "$PR_STATE" = "CLOSED" ]; then
      echo "  🟡 $branch (PR closed without merge)"
    else
      echo "  🔵 $branch (no PR or open PR)"
    fi
  done
else
  echo "✅ No merged branches to clean up"
fi

echo ""
```

### Step 4: Clean Up Merged Branches

```bash
echo "🧹 CLEANING MERGED BRANCHES"
echo "──────────────────────────────────────────────────────────"

CLEANED_COUNT=0

for branch in $MERGED_BRANCHES; do
  # Only delete branches with merged PRs for safety
  PR_STATE=$(gh pr list --state all --head "$branch" --json state --jq '.[0].state // "NO_PR"' 2>/dev/null || echo "NO_PR")

  if [ "$PR_STATE" = "MERGED" ]; then
    if [ "$DRY_RUN" = true ]; then
      echo "Would delete: $branch (PR merged)"
    else
      echo "Deleting merged branch: $branch"
      git branch -D "$branch" 2>/dev/null || echo "  ⚠️ Could not delete $branch (may not exist locally)"
    fi
    CLEANED_COUNT=$((CLEANED_COUNT + 1))
  fi
done

if [ "$CLEANED_COUNT" -gt 0 ]; then
  if [ "$DRY_RUN" = true ]; then
    echo "Would clean $CLEANED_COUNT merged branches"
  else
    echo "✅ Cleaned $CLEANED_COUNT merged branches"
  fi
else
  echo "✅ No merged branches needed cleanup"
fi

echo ""
```

### Step 5: Clean Stale Remote-Tracking Branches

```bash
echo "🌐 CLEANING STALE REMOTE-TRACKING BRANCHES"
echo "──────────────────────────────────────────────────────────"

# Find remote branches that no longer exist
STALE_REMOTES=$(git branch -r --merged | grep 'origin/' | grep -v 'origin/main\|origin/staging' | sed 's|^[ ]*origin/||g')

STALE_COUNT=0
for remote_branch in $STALE_REMOTES; do
  # Check if the remote branch actually exists
  if ! git ls-remote --exit-code --heads origin "$remote_branch" >/dev/null 2>&1; then
    if [ "$DRY_RUN" = true ]; then
      echo "Would remove stale remote tracking: origin/$remote_branch"
    else
      echo "Removing stale remote tracking: origin/$remote_branch"
      git branch -dr "origin/$remote_branch" 2>/dev/null || echo "  ⚠️ Could not remove origin/$remote_branch"
    fi
    STALE_COUNT=$((STALE_COUNT + 1))
  fi
done

if [ "$STALE_COUNT" -gt 0 ]; then
  if [ "$DRY_RUN" = true ]; then
    echo "Would clean $STALE_COUNT stale remote-tracking branches"
  else
    echo "✅ Cleaned $STALE_COUNT stale remote-tracking branches"
  fi
else
  echo "✅ No stale remote-tracking branches found"
fi

echo ""
```

### Step 6: Aggressive Cleanup (Optional)

```bash
if [ "$AGGRESSIVE" = true ]; then
  echo "⚡ AGGRESSIVE CLEANUP MODE"
  echo "──────────────────────────────────────────────────────────"

  # Clean up worktrees
  echo "🗂️ Cleaning stale worktrees..."
  if [ "$DRY_RUN" = true ]; then
    echo "Would run: git worktree prune"
    if [ -d "/tmp/worktrees" ]; then
      echo "Would clean: /tmp/worktrees/* (old session worktrees)"
    fi
  else
    git worktree prune
    if [ -d "/tmp/worktrees" ]; then
      echo "Cleaning old session worktrees..."
      find /tmp/worktrees -maxdepth 1 -type d -mtime +7 -exec rm -rf {} \; 2>/dev/null || true
      echo "✅ Cleaned old worktrees (>7 days)"
    fi
  fi

  # Clean up untracked files
  echo ""
  echo "🗑️ Cleaning untracked files..."
  if [ "$DRY_RUN" = true ]; then
    echo "Would run: git clean -fd"
  else
    git clean -fd
    echo "✅ Cleaned untracked files and directories"
  fi

  echo ""
fi
```

### Step 7: Stash Cleanup (Optional with Confirmation)

```bash
if [ "$INCLUDE_STASHES" = true ]; then
  echo "💾 STASH CLEANUP"
  echo "──────────────────────────────────────────────────────────"

  STASH_COUNT=$(git stash list | wc -l)

  if [ "$STASH_COUNT" -gt 0 ]; then
    echo "Found $STASH_COUNT stashes:"
    git stash list --oneline
    echo ""

    if [ "$DRY_RUN" = true ]; then
      echo "Would prompt for stash cleanup confirmation"
    else
      echo "⚠️ Stashes contain potentially important work!"
      echo "Review each stash before deletion. Clean them up manually if safe:"
      echo "  git stash show stash@{N} --stat"
      echo "  git stash drop stash@{N}  # if safe to delete"
      echo "  git stash clear          # delete ALL (dangerous!)"
    fi
  else
    echo "✅ No stashes found"
  fi

  echo ""
fi
```

### Step 8: Summary Report

```bash
echo "📊 CLEANUP SUMMARY"
echo "═══════════════════════════════════════════════════════════"

if [ "$DRY_RUN" = true ]; then
  echo "🔍 DRY RUN COMPLETED - No changes were made"
else
  echo "🧹 CLEANUP COMPLETED"
fi

echo ""
echo "📈 Git Repository Health:"

# Branch counts
LOCAL_BRANCHES=$(git branch | wc -l)
REMOTE_BRANCHES=$(git branch -r | wc -l)
STASH_COUNT=$(git stash list | wc -l)

echo "  📋 Local branches: $LOCAL_BRANCHES"
echo "  🌐 Remote tracking: $REMOTE_BRANCHES"
echo "  💾 Stashes: $STASH_COUNT"

if [ -d "/tmp/worktrees" ]; then
  WORKTREE_COUNT=$(find /tmp/worktrees -maxdepth 1 -type d | wc -l)
  echo "  🗂️ Session worktrees: $((WORKTREE_COUNT - 1))"
fi

echo ""
echo "💡 RECOMMENDATIONS:"

if [ "$LOCAL_BRANCHES" -gt 20 ]; then
  echo "  ⚠️ High branch count ($LOCAL_BRANCHES) - consider more frequent pruning"
fi

if [ "$STASH_COUNT" -gt 0 ]; then
  echo ""
  echo "  ⚠️ STASH REVIEW NEEDED ($STASH_COUNT stashes)"
  echo "  Stashes may contain forgotten uncommitted work!"
  echo ""
  git stash list | head -5
  if [ "$STASH_COUNT" -gt 5 ]; then
    echo "  ... and $((STASH_COUNT - 5)) more"
  fi
  echo ""
  echo "  To inspect: git stash show stash@{N} --stat"
  echo "  To apply:   git stash pop stash@{N}"
  echo "  To clean:   /prune-branches --include-stashes"
fi

echo ""
echo "  ✅ Run 'git log --oneline --graph' to see clean history"
echo "  💡 Add to periodic maintenance: /prune-branches"

echo ""
```

## Integration with SDLC

### Post-Merge Cleanup Hook

Add to SDLC Phase 6 post-merge actions:

```bash
# After PR merge completion
echo "🧹 Running post-merge branch cleanup..."
/prune-branches --dry-run

# Show what would be cleaned
echo "💡 To clean up your local branches, run:"
echo "   /prune-branches"
```

### Periodic Maintenance

Recommended frequency:
- **Weekly**: `/prune-branches`
- **Monthly**: `/prune-branches --aggressive`
- **As needed**: `/prune-branches --include-stashes` (with caution)

## Safety Features

| Safety Measure | Description |
|----------------|-------------|
| **PR Verification** | Only deletes branches with merged PRs |
| **Protected Branches** | Never touches main/staging branches |
| **Dry Run Mode** | Preview changes before executing |
| **Stash Protection** | Requires explicit flag and confirmation |
| **Worktree Age Check** | Only removes worktrees >7 days old |

## Examples

```bash
# Safe first run - see what would be cleaned
/prune-branches --dry-run

# Normal cleanup after sprint
/prune-branches

# Deep clean before release
/prune-branches --aggressive

# Complete maintenance (careful!)
/prune-branches --aggressive --include-stashes
```

This keeps your git repository clean and performant by removing the clutter of old branches while preserving important work.