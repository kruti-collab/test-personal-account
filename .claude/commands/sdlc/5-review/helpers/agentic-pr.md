---
description: Create a dedicated PR for agentic layer improvements discovered during implementation
---

# Create Agentic Layer PR

Creates a separate PR for Claude Code configuration improvements, keeping them isolated from feature work.

## When to Use

- After capturing learnings from sdlc:4-implement
- When user selects "Create separate PR" option
- For significant agentic layer improvements that should be reviewed independently

## Process

### Step 1: Stash Feature Changes

```bash
# Save current work
git stash push -m "feature-work-in-progress"

# Note current branch
FEATURE_BRANCH=$(git branch --show-current)
```

### Step 2: Create Agentic Branch

```bash
# Create new branch from main/staging
git checkout staging
git pull origin staging
git checkout -b agentic/improvements-$(date +%Y%m%d)
```

### Step 3: Apply Agentic Changes Only

For each improvement from the capture-learnings output:

1. **SDLC Commands**: Edit files in `.claude/commands/sdlc/`
2. **Context Rules**: Edit files in `.claude/rules/`
3. **Agent Definitions**: Edit files in `.claude/agents/`
4. **Hooks**: Edit files in `.claude/hooks/`

### Step 4: Create Commit

```bash
git add .claude/
git commit -m "$(cat <<'EOF'
[Agentic] Improve SDLC workflow automation

- Add auto-task generation to sdlc:2-design
- Add OAuth testing guidelines to sdlc:3-test
- Add pre-commit learnings capture to sdlc:4-implement

Discovered during: [FEATURE_BRANCH]
Session improvements: [N] friction points addressed

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

### Step 5: Create PR

```bash
gh pr create \
  --title "[Agentic] SDLC workflow improvements" \
  --body "$(cat <<'EOF'
## Summary

Improvements to Claude Code SDLC workflow discovered during feature implementation.

## Changes

| File | Change |
|------|--------|
| `.claude/commands/sdlc/2-design/run.md` | Auto-generate tasks.md |
| `.claude/commands/sdlc/3-test/run.md` | OAuth testing guidelines |
| `.claude/commands/sdlc/4-implement/run.md` | Pre-commit learnings |
| `.claude/commands/sdlc/4-implement/helpers/capture-learnings.md` | New helper |
| `.claude/commands/sdlc/5-review/helpers/agentic-pr.md` | New helper |

## Session Context

Discovered during: `FEATURE_BRANCH`

## Friction Points Addressed

1. User had to remind agent to generate tasks.md
2. OAuth testing used cached credentials
3. No mechanism to capture session learnings

## Testing

- [ ] Verified on next SDLC session
- [ ] Commands invoke correctly
- [ ] No syntax errors

---
Generated with [Claude Code](https://claude.com/claude-code)
EOF
)" \
  --base staging
```

### Step 6: Return to Feature Branch

```bash
# Return to feature work
git checkout $FEATURE_BRANCH
git stash pop
```

### Step 7: Report

```
## Agentic PR Created

PR: [URL from gh pr create output]
Branch: agentic/improvements-YYYYMMDD

Feature branch restored. Continue with feature implementation.
```

## PR Naming Convention

| Type | Branch | PR Title |
|------|--------|----------|
| SDLC improvements | `agentic/sdlc-*` | `[Agentic] SDLC: ...` |
| Rule updates | `agentic/rules-*` | `[Agentic] Rules: ...` |
| Agent definitions | `agentic/agents-*` | `[Agentic] Agents: ...` |
| Mixed | `agentic/improvements-*` | `[Agentic] Mixed improvements` |

## Important Notes

1. **Isolation**: Agentic PRs should be independent of feature PRs
2. **Review**: These PRs affect all Claude Code sessions - review carefully
3. **Testing**: Validate on next session before merging
4. **Documentation**: Include context about discovery session
