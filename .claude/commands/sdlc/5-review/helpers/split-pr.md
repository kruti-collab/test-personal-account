---
description: Split unrelated changes from current PR into a new PR
argument-hint: "[pr-number] [target-branch]"
allowed-tools: Bash, Read, Write, Grep, Glob
---

# Split PR

Split unrelated changes from the current PR into a new separate PR.

**Philosophy**: When a PR has scope issues, split it immediately and continue SDLC on the simpler one first.

## Variables

- **PR_NUMBER**: First argument from $ARGUMENTS (optional)
  - If provided, work with the specified PR number
  - If not provided, work with the current branch's PR

- **TARGET_BRANCH**: Second argument from $ARGUMENTS (optional, defaults to 'staging')
  - The base branch for the new PR
  - Common values: 'staging', 'main', 'dev'

## Step 1: Analyze PR Scope

```bash
# Get PR details
PR_NUMBER=${1:-$(gh pr view --json number --jq '.number' 2>/dev/null)}
PR_DATA=$(gh pr view $PR_NUMBER --json title,body,files,commits)

# List changed files grouped by likely ticket/feature
echo "$PR_DATA" | jq -r '.files[].path' | sort

# List commits to identify separate work streams
gh pr view $PR_NUMBER --json commits --jq '.commits[] | "\(.oid[0:7]) \(.messageHeadline)"'
```

**Identify Split Points**:
- Different GitHub issue prefixes (GAL-336 vs GAL-335)
- Unrelated directory paths (cli/ vs scripts/)
- Different commit authors/dates
- Logically independent changes

## Step 2: Plan the Split

Determine:
1. **Primary PR** (simpler, ship first): Which changes are smaller/simpler?
2. **Secondary PR** (new PR): Which changes are larger/more complex?
3. **GitHub issues**: Which ticket does each belong to?

Example:
```
Primary PR (keep in current):
  - scripts/load-env.sh (GAL-336 - OAuth fix)

Secondary PR (split out):
  - apps/cli/src/services/BackgroundAgentManager.ts (GAL-335)
  - apps/cli/src/commands/agent-*.ts (GAL-335)
  - docs/features/background-agents.md (GAL-335)
```

## Step 3: Execute Split

### 3.1 Create Backup
```bash
ORIGINAL_BRANCH=$(git branch --show-current)
git branch backup-${ORIGINAL_BRANCH}-$(date +%Y%m%d%H%M%S)
```

### 3.2 Create New Branch for Secondary Changes
```bash
TARGET_BRANCH="${2:-staging}"
NEW_BRANCH="feature/GAL-XXX-split-from-${ORIGINAL_BRANCH}"

git checkout $TARGET_BRANCH
git pull origin $TARGET_BRANCH
git checkout -b $NEW_BRANCH
```

### 3.3 Cherry-pick or Copy Secondary Changes
```bash
# Option A: If changes are in separate commits
git cherry-pick <commit-hash-1> <commit-hash-2>

# Option B: If changes are mixed in commits
git checkout $ORIGINAL_BRANCH -- path/to/file1 path/to/file2
git add .
git commit -m "[GAL-XXX] Split from PR #$PR_NUMBER: Description"
```

### 3.4 Push New Branch and Create PR
```bash
git push -u origin $NEW_BRANCH

gh pr create --base $TARGET_BRANCH --head $NEW_BRANCH \
  --title "[GAL-XXX] Description (split from #$PR_NUMBER)" \
  --body "$(cat <<'EOF'
## Summary

Split from PR #$PR_NUMBER to separate concerns.

## Changes

- [List changes moved to this PR]

## Original PR

Related to #$PR_NUMBER

---
🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### 3.5 Clean Original Branch
```bash
git checkout $ORIGINAL_BRANCH

# Remove secondary changes from original
git checkout $TARGET_BRANCH -- path/to/file1 path/to/file2

# Or if files were entirely new, just delete them
git rm path/to/new-file1 path/to/new-file2

git add .
git commit -m "[GAL-YYY] Remove unrelated changes (moved to PR #NEW_PR)"

# Force push with lease (safer than --force)
git push --force-with-lease origin $ORIGINAL_BRANCH
```

## Step 4: Update GitHub Issues

**CRITICAL**: Keep issue tracking in sync when splitting PRs.

```bash
# Get the GitHub/GitHub issue numbers from commits
ORIGINAL_TICKET=$(echo "$PR_DATA" | jq -r '.title' | grep -oE 'GAL-[0-9]+' | head -1)
SPLIT_TICKET=$(echo "GAL-XXX") # The ticket for split changes

# Update original PR description if needed
gh pr edit $PR_NUMBER --title "[${ORIGINAL_TICKET}] Updated title after split"

# Add cross-reference comments
gh pr comment $PR_NUMBER --body "Split unrelated changes to PR #$NEW_PR_NUMBER (${SPLIT_TICKET})"
gh pr comment $NEW_PR_NUMBER --body "Split from PR #$PR_NUMBER to separate ${SPLIT_TICKET} work"
```

## Step 5: Continue SDLC

After split:

1. **Primary PR**: Continue review in current session
   ```bash
   /sdlc/5-review/run $PR_NUMBER
   ```

2. **Secondary PR**: Document for new session
   ```
   New session needed for PR #$NEW_PR_NUMBER
   Run: /sdlc/5-review/run $NEW_PR_NUMBER
   ```

## Safety Checks

- ✅ ALWAYS create backup branch before destructive changes
- ✅ Use `--force-with-lease` instead of `--force`
- ✅ Verify no uncommitted changes before starting
- ✅ Update both PRs with cross-references
- ✅ Update GitHub issues to reflect the split

## Output Summary

After split, report:

```markdown
## PR Split Complete

### Original PR (#$PR_NUMBER)
- Title: [Updated title]
- Ticket: GAL-XXX
- Changes remaining: [count] files
- Status: Ready for continued review

### New PR (#$NEW_PR_NUMBER)
- Title: [New title]
- Ticket: GAL-YYY
- Changes moved: [count] files
- Status: Needs separate review session

### Next Steps
1. Continue review of PR #$PR_NUMBER in this session
2. Open new session for PR #$NEW_PR_NUMBER
```
