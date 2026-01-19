---
description: Step-by-step implementation guide for applying universal TODO structure to SDLC commands
---

# TodoWrite Implementation Guide

**For SDLC Phase Command Authors**

---

## Quick Start

### Step 1: Add TodoWrite Initialization

In your SDLC phase command, add this as the **first executable step** (after frontmatter and gates):

```markdown
## Initialize Progress Tracking

TodoWrite([
  { content: "[P5] Verify code changes exist", status: "pending", activeForm: "Verifying code changes" },
  { content: "[P5] Set up review worktree", status: "pending", activeForm: "Setting up review worktree" },
  # ... copy all steps from template
])
```

### Step 2: Update TodoWrite as Work Progresses

When starting a step:
```bash
# Starting step 2
TodoWrite([
  { content: "[P5] Verify code changes exist", status: "completed", activeForm: "Code changes verified" },
  { content: "[P5] Set up review worktree", status: "in_progress", activeForm: "Setting up review worktree" },
  # ... rest
])
```

When step completes:
```bash
TodoWrite([
  { content: "[P5] Set up review worktree", status: "completed", activeForm: "Worktree created at /tmp/..." },
  { content: "[P5] Rebase onto staging", status: "in_progress", activeForm: "Rebasing onto staging" },
  # ... rest
])
```

### Step 3: Document Skips

When a conditional step doesn't apply:
```bash
TodoWrite([
  { content: "[P6] MCP verification - SKIPPED: No MCP in feature", status: "skipped", activeForm: "Skipping MCP verification" },
  { content: "[P6] Approve PR", status: "in_progress", activeForm: "Approving PR" },
  # ... rest
])
```

### Step 4: End with Capture Learnings

Always invoke at session end:
```bash
Skill(skill: "capture-learnings")
```

---

## Integration Patterns

### Pattern 1: Simple Sequential Steps

**Use When**: Steps execute one after another with no branching.

```bash
# Start
TodoWrite([
  { content: "[P1] Parse input", status: "in_progress", activeForm: "Parsing feature description" },
  { content: "[P1] Create branch", status: "pending", activeForm: "Creating feature branch" },
  { content: "[P1] Generate spec", status: "pending", activeForm: "Generating specification" }
])

# Parse completes
TodoWrite([
  { content: "[P1] Parse input", status: "completed", activeForm: "Feature description parsed" },
  { content: "[P1] Create branch", status: "in_progress", activeForm: "Creating feature branch" },
  { content: "[P1] Generate spec", status: "pending", activeForm: "Generating specification" }
])

# Create branch completes
TodoWrite([
  { content: "[P1] Parse input", status: "completed", activeForm: "Feature description parsed" },
  { content: "[P1] Create branch", status: "completed", activeForm: "Branch created: 240-favicon-update" },
  { content: "[P1] Generate spec", status: "in_progress", activeForm: "Generating specification" }
])
```

### Pattern 2: Parallel Execution (Multiple Agents)

**Use When**: Multiple subagents run simultaneously.

```bash
# Start delegation
TodoWrite([
  { content: "[P5] Run tier-based code review", status: "in_progress", activeForm: "Dispatching 5 agents in parallel" }
])

# Launch all agents
Task(subagent_type: "general-purpose", prompt: "Logic review...")
Task(subagent_type: "solutions-architect", prompt: "Architecture review...")
Task(subagent_type: "security-researcher", prompt: "Security audit...")
Task(subagent_type: "general-purpose", prompt: "Performance analysis...")
Task(subagent_type: "qa-engineer", prompt: "Test quality review...")

# After all complete
TodoWrite([
  { content: "[P5] Run tier-based code review", status: "completed", activeForm: "Code review passed - all agents approved" }
])
```

### Pattern 3: Conditional Steps (If/Else)

**Use When**: Step may or may not execute based on conditions.

```bash
# Check condition
if [ "$IS_SENSITIVE" = true ]; then
  # Sensitive files detected
  TodoWrite([
    { content: "[P5] Analyze PR complexity", status: "completed", activeForm: "Tier 4 (sensitive files)" },
    { content: "[P5] Run tier-based code review", status: "in_progress", activeForm: "Dispatching 5 agents for Tier 4" }
  ])
else
  # Not sensitive, skip
  TodoWrite([
    { content: "[P5] Analyze PR complexity", status: "completed", activeForm: "Tier 2 (4 files, 200 lines)" },
    { content: "[P5] Run tier-based code review", status: "in_progress", activeForm: "Dispatching 2 agents for Tier 2" }
  ])
fi
```

### Pattern 4: Retry Loops

**Use When**: A step may fail and need retries.

```bash
RETRY_COUNT=0
MAX_RETRIES=3

while [ $RETRY_COUNT -lt $MAX_RETRIES ]; do
  RETRY_COUNT=$((RETRY_COUNT + 1))

  TodoWrite([
    { content: "[P4] Rebase onto staging (Attempt $RETRY_COUNT/$MAX_RETRIES)", status: "in_progress", activeForm: "Rebasing attempt $RETRY_COUNT" }
  ])

  if git rebase origin/staging; then
    TodoWrite([
      { content: "[P4] Rebase onto staging", status: "completed", activeForm: "Rebased successfully on attempt $RETRY_COUNT" }
    ])
    break
  else
    if [ $RETRY_COUNT -lt $MAX_RETRIES ]; then
      # Retry
      TodoWrite([
        { content: "[P4] Rebase onto staging (Attempt $RETRY_COUNT/$MAX_RETRIES) - FAILED, retrying...", status: "in_progress", activeForm: "Resolving conflicts for retry" }
      ])
      # Fix conflicts...
    else
      # Final failure
      TodoWrite([
        { content: "[P4] Rebase onto staging - BLOCKED: Conflicts after $MAX_RETRIES attempts", status: "blocked", activeForm: "Blocked by merge conflicts" }
      ])
      exit 1
    fi
  fi
done
```

### Pattern 5: Long-Running Waits (CI, Merge)

**Use When**: Waiting for external system (CI checks, merge, deployment).

```bash
# Initial
TodoWrite([
  { content: "[P5] Wait for CI checks", status: "in_progress", activeForm: "Waiting for CI - Starting (max 20 min)" }
])

TIMEOUT=1200  # 20 minutes
INTERVAL=60   # Check every 60 seconds
ELAPSED=0

while [ $ELAPSED -lt $TIMEOUT ]; do
  CHECKS=$(gh pr checks $PR_NUMBER --json state,conclusion)

  # Update TodoWrite every 60 seconds
  TodoWrite([
    { content: "[P5] Wait for CI checks", status: "in_progress", activeForm: "Waiting for CI - $ELAPSED / $TIMEOUT sec ($((TIMEOUT - ELAPSED)) remaining)" }
  ])

  # Check if complete
  if echo "$CHECKS" | jq 'all(.conclusion != null)' | grep -q true; then
    # All checks complete
    if echo "$CHECKS" | jq 'all(.conclusion == "success")' | grep -q true; then
      # All passed
      TodoWrite([
        { content: "[P5] Wait for CI checks", status: "completed", activeForm: "CI passed ($ELAPSED sec)" }
      ])
      break
    else
      # Some failed
      TodoWrite([
        { content: "[P5] Wait for CI checks", status: "blocked", activeForm: "CI failed after $ELAPSED sec" }
      ])
      exit 1
    fi
  fi

  sleep $INTERVAL
  ELAPSED=$((ELAPSED + INTERVAL))
done
```

### Pattern 6: Delegated Work (Subagent)

**Use When**: Work executed by subagent via Task tool.

```bash
# Before delegation
TodoWrite([
  { content: "[P4] Manual testing - DELEGATED", status: "pending", activeForm: "Queuing manual tests" }
])

# Start delegation
TodoWrite([
  { content: "[P4] Manual testing - DELEGATED to manual-tester", status: "in_progress", activeForm: "Delegating CLI and UI tests to manual-tester agent" }
])

Task(
  subagent_type: "manual-tester",
  prompt: "Test CLI commands and UI pages. Report any issues found."
)

# After delegation returns
TodoWrite([
  { content: "[P4] Manual testing - DELEGATED to manual-tester", status: "completed", activeForm: "Manual tests completed (no issues found)" }
])
```

### Pattern 7: Conditional Skip (Decision Gate)

**Use When**: Step is optional and decision made inline.

```bash
# Check if skip criteria met
if [ -z "$MCP_CONFIG" ]; then
  # No MCP config, skip verification
  TodoWrite([
    { content: "[P6] MCP verification - SKIPPED: No .mcp.json found", status: "skipped", activeForm: "Skipping MCP verification" },
    { content: "[P6] Approve PR", status: "in_progress", activeForm: "Approving pull request" }
  ])
else
  # MCP exists, run verification
  TodoWrite([
    { content: "[P6] MCP verification", status: "in_progress", activeForm: "Verifying MCP server" }
  ])

  # ... verify MCP ...

  TodoWrite([
    { content: "[P6] MCP verification", status: "completed", activeForm: "MCP verification passed" },
    { content: "[P6] Approve PR", status: "in_progress", activeForm: "Approving pull request" }
  ])
fi
```

---

## Common Mistakes to Avoid

### ❌ Mistake 1: Forgetting Initial TodoWrite

```bash
# WRONG - skips TodoWrite initialization
echo "Starting review..."
# ... do work ...
```

**CORRECT:**
```bash
# RIGHT - initialize immediately
TodoWrite([
  { content: "[P5] Verify code changes exist", status: "pending", activeForm: "Verifying code changes" },
  # ... all steps
])

echo "Starting review..."
# ... do work ...
```

### ❌ Mistake 2: Using Wrong Status Values

```bash
# WRONG
{ content: "[P5] Test step", status: "working", activeForm: "Testing" }
```

**CORRECT:**
```bash
# RIGHT - only use: pending, in_progress, completed, skipped, blocked
{ content: "[P5] Test step", status: "in_progress", activeForm: "Testing" }
```

### ❌ Mistake 3: Forgetting activeForm Updates

```bash
# WRONG - activeForm never changes
{ content: "[P5] Wait for CI", status: "in_progress", activeForm: "Waiting for CI" }
# 10 minutes pass...
# Still shows "Waiting for CI" with no progress indicator
```

**CORRECT:**
```bash
# RIGHT - update activeForm with progress
{ content: "[P5] Wait for CI", status: "in_progress", activeForm: "Waiting for CI - 5 min / 20 min (15 remaining)" }
```

### ❌ Mistake 4: Not Updating When Step Completes

```bash
# WRONG - never marks step as completed
TodoWrite([
  { content: "[P5] Rebase onto staging", status: "in_progress", activeForm: "Rebasing" }
])

# ... rebase completes ...
# No TodoWrite update, step still shows as in_progress!
```

**CORRECT:**
```bash
TodoWrite([
  { content: "[P5] Rebase onto staging", status: "in_progress", activeForm: "Rebasing" }
])

# ... rebase completes ...

TodoWrite([
  { content: "[P5] Rebase onto staging", status: "completed", activeForm: "Rebased successfully" },
  { content: "[P5] Wait for CI checks", status: "in_progress", activeForm: "Waiting for CI checks" }
])
```

### ❌ Mistake 5: Skipping without Documentation

```bash
# WRONG - step just disappears
# ... test code that might skip step ...
# Result: User sees 7 steps completed, 1 missing - confused why
```

**CORRECT:**
```bash
if [ "$IS_SKIPPABLE" = true ]; then
  TodoWrite([
    { content: "[P6] MCP verification - SKIPPED: No MCP in scope", status: "skipped", activeForm: "Skipping MCP verification" }
  ])
else
  # ... do verification ...
fi
```

### ❌ Mistake 6: Using Past Tense in activeForm

```bash
# WRONG - use past tense when step is completed (user sees "Completed step")
{ content: "[P5] Rebase", status: "in_progress", activeForm: "Rebased successfully" }
```

**CORRECT:**
```bash
# RIGHT - in_progress uses continuous form
{ content: "[P5] Rebase", status: "in_progress", activeForm: "Rebasing onto staging" }

# And when complete, change status
{ content: "[P5] Rebase", status: "completed", activeForm: "Rebased successfully" }
```

### ❌ Mistake 7: Missing Capture Learnings

```bash
# WRONG - ends phase without capture-learnings
echo "Phase complete!"
exit 0
```

**CORRECT:**
```bash
# RIGHT - always invoke capture-learnings
TodoWrite([
  { content: "[P5] Capture learnings", status: "in_progress", activeForm: "Capturing session learnings" }
])

Skill(skill: "capture-learnings")

TodoWrite([
  { content: "[P5] Capture learnings", status: "completed", activeForm: "Learnings captured" }
])
```

---

## Testing Your Implementation

### Checklist Before Merging

- [ ] TodoWrite initializes in first executable code
- [ ] All major steps have corresponding todos
- [ ] Status updates happen at step start (in_progress)
- [ ] Status updates happen at step end (completed/skipped)
- [ ] ActiveForm uses present continuous (-ing) verb
- [ ] Skipped steps document reason in content field
- [ ] Delegated work clearly marked (e.g., "DELEGATED to manual-tester")
- [ ] Long waits update progress periodically
- [ ] Final step is capture-learnings
- [ ] No pending todos at session end (except genuinely blocked with reason)

### Manual Test

Run your phase command and verify:

1. **At start**: TodoWrite shows all steps as pending
2. **During execution**: Current step shows in_progress, previous steps show completed
3. **On skip**: Skipped step shows skipped with reason visible
4. **On error**: Failed step shows blocked with reason
5. **At end**: All steps completed/skipped, capture-learnings invoked

---

## Examples

### Full Phase 5 (REVIEW) Example

```markdown
## Initialize Progress Tracking

TodoWrite([
  { content: "[P5] Verify code changes exist", status: "pending", activeForm: "Verifying code changes" },
  { content: "[P5] Set up review worktree", status: "pending", activeForm: "Setting up review worktree" },
  { content: "[P5] Rebase onto staging", status: "pending", activeForm: "Rebasing onto staging" },
  { content: "[P5] Wait for CI checks", status: "pending", activeForm: "Waiting for CI checks" },
  { content: "[P5] Analyze PR complexity & tier", status: "pending", activeForm: "Analyzing PR complexity" },
  { content: "[P5] Run tier-based code review", status: "pending", activeForm: "Running code review agents" },
  { content: "[P5] Check code review gate", status: "pending", activeForm: "Checking review gate" },
  { content: "[P5] Clean up worktree", status: "pending", activeForm: "Cleaning up resources" },
  { content: "[P5] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

Then as work proceeds:

```bash
# Starting verification
TodoWrite([
  { content: "[P5] Verify code changes exist", status: "in_progress", activeForm: "Verifying code changes" },
  # ... rest
])

# Verification complete
TodoWrite([
  { content: "[P5] Verify code changes exist", status: "completed", activeForm: "2 uncommitted files and 3 commits ahead found" },
  { content: "[P5] Set up review worktree", status: "in_progress", activeForm: "Setting up review worktree" },
  # ... rest
])

# ... (more updates as work progresses) ...

# Eventually all complete
TodoWrite([
  { content: "[P5] Verify code changes exist", status: "completed", activeForm: "2 uncommitted files and 3 commits ahead" },
  { content: "[P5] Set up review worktree", status: "completed", activeForm: "Worktree: /tmp/worktrees/review-527-oauth" },
  { content: "[P5] Rebase onto staging", status: "completed", activeForm: "Rebased cleanly" },
  { content: "[P5] Wait for CI checks", status: "completed", activeForm: "All 12 checks passed (8 min)" },
  { content: "[P5] Analyze PR complexity & tier", status: "completed", activeForm: "Tier 3 (11 files, 680 lines)" },
  { content: "[P5] Run tier-based code review", status: "completed", activeForm: "Code review passed (all agents)" },
  { content: "[P5] Check code review gate", status: "completed", activeForm: "Gate passed - ready for Phase 6" },
  { content: "[P5] Clean up worktree", status: "completed", activeForm: "Worktree deleted" },
  { content: "[P5] Capture learnings", status: "in_progress", activeForm: "Capturing learnings" }
])

# End with capture-learnings
Skill(skill: "capture-learnings")
```

---

## FAQs

**Q: What if a step is really complex with sub-steps?**
A: Create separate todos for major sub-steps. TodoWrite doesn't support nesting in v1.0.

**Q: Should I update TodoWrite after EVERY small operation?**
A: No, only at meaningful checkpoints. Excessive updates clutter history. Update when: starting a new step, finishing a step, or significant progress milestone reached.

**Q: Can I skip TodoWrite for simple phases?**
A: No. Every phase command must initialize TodoWrite for visibility and context preservation across compactions.

**Q: What if a step has no clear end (like "waiting")?**
A: Update periodically (every 60-120 seconds) with elapsed time. Show: `"Waiting for CI - 5 min / 20 min (15 remaining)"`.

**Q: How do I show a step that partially succeeded?**
A: Use status=completed with details in activeForm about what succeeded. If partially failed, use status=blocked with failure reason.

**Q: What's the difference between "skipped" and "completed"?**
A: `skipped` = step doesn't apply to this execution (e.g., no MCP to verify). `completed` = step executed successfully.

---

## Support & Questions

For questions about TodoWrite implementation:

1. Check `UNIVERSAL_TODO_STRUCTURE.md` for detailed patterns
2. Reference existing Phase 5 (REVIEW) command for working example
3. Submit questions in SDLC phase command discussions
