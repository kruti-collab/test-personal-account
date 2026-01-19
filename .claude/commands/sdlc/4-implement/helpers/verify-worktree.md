---
description: ENFORCE worktree usage for implementation work to prevent multi-session conflicts
allowed-tools: Bash
---

# Verify Worktree Enforcement

**Purpose**: BLOCK implementation work if not in a dedicated worktree.

## Usage

```bash
# Run at start of Phase 4
/sdlc:4-implement:helpers:verify-worktree
```

## Why This Helper Exists

**Problem**: From GAL-334 learnings (2026-01-16):
- Multiple Claude sessions sharing main worktree → branch conflicts
- Implementation work in main worktree → session interruptions
- Forgotten worktree requirement → lost work from improper rebases

**Solution**: MANDATORY worktree check at Phase 4 entry.

**Benefits**:
- Enforces best practice automatically
- Prevents multi-session conflicts
- Clear error message with remediation steps
- Can be invoked standalone for verification

## Implementation

```bash
#!/bin/bash
set -e

echo "┌─────────────────────────────────────────────┐"
echo "│  WORKTREE ENFORCEMENT CHECK                 │"
echo "└─────────────────────────────────────────────┘"
echo ""

# Get absolute path to main repo
MAIN_REPO_PATH="/Users/karabil/Documents/scheduler-systems/products/gal"

# Get current working directory
CURRENT_DIR=$(pwd)

# Check if we're in the main worktree
if [ "$CURRENT_DIR" = "$MAIN_REPO_PATH" ]; then
  echo "❌ STOP: Implementation work detected in main worktree"
  echo ""
  echo "┌─────────────────────────────────────────────────────────────┐"
  echo "│ WORKTREE VIOLATION                                          │"
  echo "├─────────────────────────────────────────────────────────────┤"
  echo "│ Current Location: $CURRENT_DIR"
  echo "│ Issue: Implementation MUST use dedicated worktree          │"
  echo "│                                                             │"
  echo "│ Why This Matters:                                           │"
  echo "│ - Prevents multi-session branch conflicts                  │"
  echo "│ - Avoids lost work from improper rebases                   │"
  echo "│ - Isolates implementation from other sessions              │"
  echo "└─────────────────────────────────────────────────────────────┘"
  echo ""
  echo "REQUIRED: Create implementation worktree"
  echo ""

  # Get current branch for worktree creation
  BRANCH=$(git branch --show-current)

  echo "Recommended worktree setup:"
  echo "  IMPL_BRANCH=\"${BRANCH}-impl-$(date +%s)\""
  echo "  WORKTREE_PATH=\"/tmp/worktrees/\$IMPL_BRANCH\""
  echo "  git worktree add \"\$WORKTREE_PATH\" -b \"\$IMPL_BRANCH\""
  echo "  cd \"\$WORKTREE_PATH\""
  echo ""
  echo "Then re-run the implementation command."
  echo ""
  exit 1
fi

# Check if we're in a worktree at all
WORKTREE_DIR=$(git rev-parse --git-common-dir 2>/dev/null || echo "")

if [ -z "$WORKTREE_DIR" ]; then
  echo "⚠️  WARNING: Not in a git repository"
  echo "   This check cannot verify worktree status."
  exit 0
fi

# Check if this is a worktree (not the main repo)
if git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
  GIT_DIR=$(git rev-parse --git-dir)

  # Worktrees have .git as a file, not a directory
  if [ -f ".git" ]; then
    echo "✅ PASS: Running in dedicated worktree"
    echo ""
    echo "Worktree: $CURRENT_DIR"
    echo "Main Repo: $MAIN_REPO_PATH"
    echo ""
    echo "Benefits:"
    echo "  - Isolated from other Claude sessions"
    echo "  - Safe to rebase/switch branches"
    echo "  - No conflicts with main development"
    echo ""
    exit 0
  fi
fi

# Edge case: In a subdirectory of main repo
if [[ "$CURRENT_DIR" == "$MAIN_REPO_PATH"* ]]; then
  echo "❌ STOP: In subdirectory of main worktree"
  echo ""
  echo "Current: $CURRENT_DIR"
  echo "Main Repo: $MAIN_REPO_PATH"
  echo ""
  echo "This is still considered main worktree."
  echo "Create a dedicated worktree in /tmp/worktrees/ instead."
  exit 1
fi

# Fallback: Unknown state
echo "⚠️  WARNING: Cannot determine worktree status"
echo "   Current: $CURRENT_DIR"
echo "   Main: $MAIN_REPO_PATH"
echo ""
echo "Proceeding with caution - verify manually if needed."
exit 0
```

## Integration with Phase 4

Phase 4 run.md should start with:

```markdown
---
description: Execute implementation plan...
---

## Step 0: Worktree Enforcement (MANDATORY FIRST STEP)

⛔ CRITICAL: MUST verify worktree usage before ANY implementation work.

Skill(skill: "sdlc:4-implement:helpers:verify-worktree")

If check fails → STOP → Create worktree → Re-run implementation.
If check passes → Continue to Phase Gate Check.
```

## Standalone Usage

Useful for manual verification:

```bash
# Before starting implementation
/sdlc:4-implement:helpers:verify-worktree

# After cd to worktree (sanity check)
/sdlc:4-implement:helpers:verify-worktree
```

## Output Example (Violation)

```
┌─────────────────────────────────────────────┐
│  WORKTREE ENFORCEMENT CHECK                 │
└─────────────────────────────────────────────┘

❌ STOP: Implementation work detected in main worktree

┌─────────────────────────────────────────────────────────────┐
│ WORKTREE VIOLATION                                          │
├─────────────────────────────────────────────────────────────┤
│ Current Location: /Users/karabil/.../products/gal           │
│ Issue: Implementation MUST use dedicated worktree          │
│                                                             │
│ Why This Matters:                                           │
│ - Prevents multi-session branch conflicts                  │
│ - Avoids lost work from improper rebases                   │
│ - Isolates implementation from other sessions              │
└─────────────────────────────────────────────────────────────┘

REQUIRED: Create implementation worktree

Recommended worktree setup:
  IMPL_BRANCH="527-oauth-preview-auth-impl-1705522800"
  WORKTREE_PATH="/tmp/worktrees/$IMPL_BRANCH"
  git worktree add "$WORKTREE_PATH" -b "$IMPL_BRANCH"
  cd "$WORKTREE_PATH"

Then re-run the implementation command.
```

## Output Example (Pass)

```
┌─────────────────────────────────────────────┐
│  WORKTREE ENFORCEMENT CHECK                 │
└─────────────────────────────────────────────┘

✅ PASS: Running in dedicated worktree

Worktree: /tmp/worktrees/527-oauth-preview-auth-impl-1705522800
Main Repo: /Users/karabil/Documents/scheduler-systems/products/gal

Benefits:
  - Isolated from other Claude sessions
  - Safe to rebase/switch branches
  - No conflicts with main development
```

## What This Prevents

| Without Worktree | With Worktree |
|------------------|---------------|
| Session A rebases → interrupts Session B | Each session has isolated workspace |
| Branch switch → files change under Session B | Branches switch independently |
| Lost uncommitted work from conflicts | Work preserved in separate directory |
| Multi-session race conditions | No conflicts possible |

## Worktree Creation Pattern

**Recommended pattern for implementation:**

```bash
# At start of Phase 4
BRANCH=$(git branch --show-current)
IMPL_BRANCH="${BRANCH}-impl-$(date +%s)"
WORKTREE_PATH="/tmp/worktrees/$IMPL_BRANCH"

# Create and enter worktree
git worktree add "$WORKTREE_PATH" -b "$IMPL_BRANCH"
cd "$WORKTREE_PATH"

# Verify (should pass)
/sdlc:4-implement:helpers:verify-worktree

# Now safe to implement
/sdlc:4-implement:run
```

## Cleanup After Implementation

When Phase 4 completes:

```bash
# Merge or push changes first
git push origin $IMPL_BRANCH

# Return to main repo
cd /Users/karabil/Documents/scheduler-systems/products/gal

# Remove worktree
git worktree remove "/tmp/worktrees/$IMPL_BRANCH"
```

## Alternative: Allow Main Worktree (NOT RECOMMENDED)

If you MUST bypass this check (emergency only):

```bash
# Override environment variable
export ALLOW_MAIN_WORKTREE=true

# Run implementation (bypasses check)
/sdlc:4-implement:run
```

**Use with extreme caution. Document the reason.**

## Learning Source

This check enforces the learnings from:
- `.claude/rules/git-workflow.md` - Multi-Session Isolation section
- `.claude/rules/sdlc-automation.md` - Worktree enforcement for implementation

**Key Insight**: The problem GAL-334 revealed was that multiple sessions sharing the main worktree caused frequent interruptions. Mandatory worktree usage prevents this entirely.
