---
description: Capture session learnings as separate agentic improvement PR
allowed-tools: Bash, Read, Write, Edit, Glob
---

# Capture Learnings

Automates the agentic layer improvement workflow by:
1. Analyzing the conversation for **manual interventions** (user corrections)
2. Detecting uncommitted `.claude/` changes
3. Creating a separate PR with improvements

## Step 1: Analyze Conversation for Manual Interventions

**CRITICAL: Review the conversation history for patterns where the user had to manually correct the agent.**

Look for these intervention patterns:
- User saying "you cannot...", "you should have...", "I had to tell you..."
- User reminding about workflow steps that were skipped
- User correcting agent behavior mid-flow
- Questions that indicate confusion about the workflow

### Common Intervention Categories

| Category | Example User Message | Improvement Target |
|----------|---------------------|-------------------|
| **Workflow Order** | "you need to rebase first" | Add explicit ordering to command |
| **Blocking Gates** | "you cannot continue if CI fails" | Add STOP instruction before gate |
| **Missing Skills** | "you didn't invoke the skill" | Add explicit skill invocation |
| **Premature Actions** | "don't do X until Y" | Add prerequisite checks |

### Analyze This Session

Review the conversation and identify:

1. **What manual interventions occurred?**
   - List each time the user had to correct the agent
   - Note the exact user message

2. **What should have happened automatically?**
   - What workflow step was missed?
   - What instruction was unclear?

3. **How to prevent this in the future?**
   - What command/rule needs updating?
   - What explicit instruction should be added?

## Step 2: Check for Uncommitted Changes

```bash
# Check for uncommitted .claude/ changes
CHANGES=$(git status --porcelain .claude/ CLAUDE.md 2>/dev/null | grep -v "^?" | wc -l | tr -d ' ')

if [ "$CHANGES" = "0" ]; then
  echo "No uncommitted .claude/ or CLAUDE.md changes detected"
  echo ""
  echo "Based on conversation analysis, you may need to:"
  echo "1. Edit the relevant command/rule files"
  echo "2. Re-run /capture-learnings after making changes"
fi
```

## Step 3: Document Learnings

Before creating the PR, document:

```markdown
## Session Learnings

### Manual Interventions Detected
1. [User message] → [What was missed] → [Fix applied]
2. ...

### Files Updated
- `.claude/commands/...` - [Description of change]
- `.claude/rules/...` - [Description of change]

### Prevention Strategy
- Added explicit STOP instruction before [gate]
- Added prerequisite check for [condition]
- Made workflow order explicit: [step A] → [step B] → [step C]
```

## Step 4: Create Improvement PR (Using Worktree)

**CRITICAL: Use a worktree to avoid interrupting other sessions on the main worktree.**

```bash
# Get current date for branch name
DATE=$(date +%Y%m%d)
BRANCH="agentic/sdlc-improvements-$DATE"

# Check if branch exists, add suffix if needed
SUFFIX=""
COUNT=1
while git rev-parse --verify "$BRANCH$SUFFIX" >/dev/null 2>&1; do
  SUFFIX="-$COUNT"
  COUNT=$((COUNT + 1))
done
BRANCH="$BRANCH$SUFFIX"

# Save current directory and list of changed files
ORIGINAL_DIR=$(pwd)
CHANGED_FILES=$(git status --porcelain .claude/ CLAUDE.md | grep -v "^?" | awk '{print $2}')

# Create worktree for agentic PR (avoids interrupting other sessions)
WORKTREE="/tmp/worktrees/agentic-$(date +%s)"
git worktree add "$WORKTREE" -b "$BRANCH" origin/staging

# Copy changed files to worktree
for file in $CHANGED_FILES; do
  cp "$ORIGINAL_DIR/$file" "$WORKTREE/$file"
done

# Work in worktree
cd "$WORKTREE"
git add .claude/ CLAUDE.md

# Show what will be committed
echo ""
echo "Changes to be committed:"
git diff --cached --stat
```

After reviewing changes, commit and create PR:

```bash
# Commit with [Agentic] prefix and learnings documentation
git commit -m "[Agentic] $DESCRIPTION [skip ci]

## Session Learnings
$LEARNINGS_DOCUMENTATION"

# Push and create PR
git push origin "$BRANCH"

gh pr create --base staging \
  --title "[Agentic] $DESCRIPTION" \
  --body "## Agentic Layer Improvement

### Session Learnings

#### Manual Interventions Detected
$INTERVENTIONS_LIST

#### Prevention Strategy
$PREVENTION_STRATEGY

### Changes
$(git diff --stat origin/staging)

### Files Modified
$(git diff --name-only origin/staging | sed 's/^/- /')

---
*This PR was created by analyzing session conversations for improvement opportunities.*"

# Return to original directory and clean up worktree
cd "$ORIGINAL_DIR"
git worktree remove "$WORKTREE" --force 2>/dev/null || true
```

## Usage

Run at any point during a session when:

1. **User had to manually correct agent behavior** - Analyze what went wrong
2. **You've made improvements to `.claude/` files** - Capture the changes
3. **End of SDLC session** - Standard checkpoint for learnings

### Intervention Analysis Checklist

Before creating the PR, ensure you've:

- [ ] Reviewed conversation for user corrections/interventions
- [ ] Identified the root cause (missing instruction, unclear ordering, etc.)
- [ ] Made specific edits to prevent recurrence
- [ ] Documented the learning in the PR description
- [ ] **META-CHECK**: Updated the command that triggered this session (see below)

## Meta-Learning: Update Triggering Commands

**CRITICAL (Learning from 2026-01-18): Always check if the slash command that started this session should also be updated.**

When capturing learnings, ask:
1. **What command triggered this session?** (e.g., `/prune-branches`, `/implement-feature`)
2. **Did any learning apply to that command's workflow?**
3. **Should that command be updated to prevent the issue?**

### Example

Session started with `/prune-branches` → found stash with forgotten work → learning: `/prune-branches` should prompt about stashes, not just report count.

**Files to potentially update:**
- The triggering command itself
- Related rules that the command should reference
- `/capture-learnings` itself (this file) if the capture process had issues
