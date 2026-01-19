---
description: Universal TodoWrite structure template for all SDLC phase commands
---

# Universal TODO Structure for SDLC Phase Commands

**Version**: 1.0.0
**Effective Date**: 2026-01-17
**Applies To**: All `/sdlc:[N]:run` commands
**Status**: Proposed Standard

## Overview

This document defines a standardized TodoWrite structure that all SDLC phase commands (`/sdlc:1-specify`, `/sdlc:2-design`, `/sdlc:3-test`, `/sdlc:4-implement`, `/sdlc:5-review`, `/sdlc:6-verify`, `/sdlc:7-deploy`) must follow.

### Why This Matters

1. **Progress Visibility**: Users can see exactly where work stands at any moment
2. **Context Preservation**: TodoWrite survives context compaction (persists in system reminders)
3. **Skip Documentation**: Clear rationale when steps are skipped
4. **Parallel Tracking**: Supports subagent delegation while tracking overall progress
5. **CI Wait Tracking**: Shows progress during long-running operations
6. **Consistent UX**: Users see same structure across all SDLC phases

### Key Design Decisions

- **1:1 Mapping**: Each major workflow step → 1 todo
- **Atomic Actions**: Each todo completable in a single operation
- **Standard Verbs**: Consistent naming across all phases
- **Conditional Logic**: Inline skip reason documentation
- **Status Tracking**: Three states (pending → in_progress → completed/skipped)
- **Active Forms**: Human-readable progress descriptions

---

## Structure

### 1. PHASE INITIALIZATION (MANDATORY FIRST)

Every SDLC phase command **MUST** create TodoWrite before any work begins.

#### Template

```javascript
TodoWrite([
  // Phase Initialization
  {
    content: "[Phase N] Verify prerequisites",
    status: "pending",
    activeForm: "Verifying prerequisites"
  },

  // Core Steps (N steps specific to phase)
  {
    content: "[Phase N] Step 1: {Description}",
    status: "pending",
    activeForm: "{Active verb phrase}"
  },
  {
    content: "[Phase N] Step 2: {Description}",
    status: "pending",
    activeForm: "{Active verb phrase}"
  },
  // ... more steps

  // Cleanup & Learnings
  {
    content: "[Phase N] Cleanup & finalization",
    status: "pending",
    activeForm: "Cleaning up resources"
  },
  {
    content: "[Phase N] Capture learnings",
    status: "pending",
    activeForm: "Capturing session learnings"
  }
])
```

#### Status Values

| Status | Meaning | When to Use |
|--------|---------|------------|
| `pending` | Not started | Initial state for all todos |
| `in_progress` | Currently executing | When starting a step |
| `completed` | Successfully finished | When step succeeds |
| `skipped` | Not applicable (documented reason) | When step is conditional and not needed |
| `blocked` | Cannot proceed (documented reason) | When step fails blocking other steps |

#### Active Form Verb Tenses

Use **present continuous** (-ing form) for human-readable progress:

| Activity | Active Form |
|----------|-------------|
| Running tests | "Running E2E tests" |
| Waiting for CI | "Waiting for CI checks" |
| Rebasing code | "Rebasing onto staging" |
| Analyzing PR | "Analyzing PR complexity" |
| Delegating work | "Delegating to qa-engineer" |
| Reviewing code | "Running tier-based code review" |
| Verifying deployment | "Verifying production deployment" |

**Anti-pattern**: "Check CI" (unclear) → **Better**: "Waiting for CI checks to pass"

---

## Phase-Specific Templates

### PHASE 1: SPECIFY

```javascript
TodoWrite([
  { content: "[P1] Verify prerequisites", status: "pending", activeForm: "Verifying prerequisites" },
  { content: "[P1] Parse input & extract issue number", status: "pending", activeForm: "Parsing feature description" },
  { content: "[P1] Check for existing branches", status: "pending", activeForm: "Checking existing branches" },
  { content: "[P1] Create feature branch & directory", status: "pending", activeForm: "Creating feature branch" },
  { content: "[P1] Generate specification", status: "pending", activeForm: "Generating feature spec" },
  { content: "[P1] Create quality checklists", status: "pending", activeForm: "Creating quality checklists" },
  { content: "[P1] Update GitHub issue", status: "pending", activeForm: "Updating GitHub issue" },
  { content: "[P1] Commit & push", status: "pending", activeForm: "Committing specification" },
  { content: "[P1] Cleanup", status: "pending", activeForm: "Cleaning up" },
  { content: "[P1] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

### PHASE 2: DESIGN

```javascript
TodoWrite([
  { content: "[P2] Verify Phase 1 (SPECIFY) complete", status: "pending", activeForm: "Verifying spec.md exists" },
  { content: "[P2] Load spec & project context", status: "pending", activeForm: "Loading feature context" },
  { content: "[P2] Generate research tasks", status: "pending", activeForm: "Dispatching research agents" },
  { content: "[P2] Create data model", status: "pending", activeForm: "Creating data-model.md" },
  { content: "[P2] Generate API contracts", status: "pending", activeForm: "Generating API contracts" },
  { content: "[P2] Update agent context", status: "pending", activeForm: "Updating agent context" },
  { content: "[P2] Generate implementation plan", status: "pending", activeForm: "Creating implementation plan" },
  { content: "[P2] Auto-generate tasks", status: "pending", activeForm: "Generating tasks.md" },
  { content: "[P2] Commit & push", status: "pending", activeForm: "Committing design artifacts" },
  { content: "[P2] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

### PHASE 3: TEST

```javascript
TodoWrite([
  { content: "[P3] Verify Phase 2 (DESIGN) complete", status: "pending", activeForm: "Verifying plan.md exists" },
  { content: "[P3] Check prerequisites (GCP, services)", status: "pending", activeForm: "Checking prerequisites" },
  { content: "[P3] Parse feature spec", status: "pending", activeForm: "Parsing feature spec" },
  { content: "[P3] Generate test file", status: "pending", activeForm: "Generating test file from spec" },
  { content: "[P3] Run tests (should FAIL)", status: "pending", activeForm: "Running tests (RED phase)" },
  { content: "[P3] Verify test failures expected", status: "pending", activeForm: "Verifying test failures" },
  { content: "[P3] Run stress testing", status: "pending", activeForm: "Running stress tests" },
  { content: "[P3] Add edge case tests", status: "pending", activeForm: "Adding edge case tests" },
  { content: "[P3] Commit test files", status: "pending", activeForm: "Committing test files" },
  { content: "[P3] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

### PHASE 4: IMPLEMENT

```javascript
TodoWrite([
  { content: "[P4] Verify Phase 3 (TEST) complete", status: "pending", activeForm: "Verifying test files exist" },
  { content: "[P4] Set up implementation worktree", status: "pending", activeForm: "Setting up worktree" },
  { content: "[P4] Load tasks & implementation plan", status: "pending", activeForm: "Loading implementation tasks" },
  { content: "[P4] Execute implementation tasks", status: "pending", activeForm: "Implementing features" },
  { content: "[P4] Verify tests PASS (GREEN phase)", status: "pending", activeForm: "Running full test suite" },
  { content: "[P4] Run manual CLI testing", status: "pending", activeForm: "Delegating CLI testing to manual-tester" },
  { content: "[P4] Run manual UI testing", status: "pending", activeForm: "Delegating UI testing to manual-tester" },
  { content: "[P4] Rebase onto staging", status: "pending", activeForm: "Rebasing onto staging" },
  { content: "[P4] Commit & create PR", status: "pending", activeForm: "Creating pull request" },
  { content: "[P4] Update GitHub issue", status: "pending", activeForm: "Updating GitHub issue status" },
  { content: "[P4] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

### PHASE 5: REVIEW

```javascript
TodoWrite([
  { content: "[P5] Verify code changes exist", status: "pending", activeForm: "Verifying code changes" },
  { content: "[P5] Set up review worktree", status: "pending", activeForm: "Setting up review worktree" },
  { content: "[P5] Rebase onto staging", status: "pending", activeForm: "Rebasing onto staging" },
  { content: "[P5] Wait for CI checks", status: "pending", activeForm: "Waiting for CI checks to pass" },
  { content: "[P5] Analyze PR complexity & tier", status: "pending", activeForm: "Analyzing PR complexity" },
  { content: "[P5] Run tier-based code review", status: "pending", activeForm: "Running tier-based agent review" },
  { content: "[P5] Check code review gate", status: "pending", activeForm: "Checking code review gate" },
  { content: "[P5] Clean up review worktree", status: "pending", activeForm: "Cleaning up worktree" },
  { content: "[P5] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

### PHASE 6: VERIFY

```javascript
TodoWrite([
  { content: "[P6] Verify Phase 5 (REVIEW) passed", status: "pending", activeForm: "Verifying code review gate" },
  { content: "[P6] Check preview deployment", status: "pending", activeForm: "Verifying preview deployment" },
  { content: "[P6] Delegate CLI testing", status: "pending", activeForm: "Running CLI tests via manual-tester" },
  { content: "[P6] Delegate UI testing", status: "pending", activeForm: "Running UI tests via manual-tester" },
  { content: "[P6] MCP verification (if applicable)", status: "pending", activeForm: "Verifying MCP integration" },
  { content: "[P6] Approve PR", status: "pending", activeForm: "Approving pull request" },
  { content: "[P6] Enable auto-merge", status: "pending", activeForm: "Enabling auto-merge" },
  { content: "[P6] Monitor merge", status: "pending", activeForm: "Monitoring PR merge" },
  { content: "[P6] Clean up worktree", status: "pending", activeForm: "Cleaning up resources" },
  { content: "[P6] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

### PHASE 7: DEPLOY

```javascript
TodoWrite([
  { content: "[P7] Verify Phase 6 (VERIFY) complete", status: "pending", activeForm: "Verifying PR merged" },
  { content: "[P7] Scan for deployment scripts", status: "pending", activeForm: "Scanning for deploy scripts" },
  { content: "[P7] Execute deployment", status: "pending", activeForm: "Deploying to production" },
  { content: "[P7] Verify production deployment", status: "pending", activeForm: "Verifying production version" },
  { content: "[P7] Auto-close GitHub issue", status: "pending", activeForm: "Closing GitHub issue" },
  { content: "[P7] Document deployment status", status: "pending", activeForm: "Recording deployment status" },
  { content: "[P7] Cleanup", status: "pending", activeForm: "Cleaning up" },
  { content: "[P7] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

---

## Conditional Step Handling

### Pattern: Optional Steps

When a step is optional based on conditions, document the skip reason inline:

```javascript
{
  content: "[P5] Manual CLI testing - SKIPPED: Test-only PR",
  status: "skipped",
  activeForm: "Skipping CLI testing"
}
```

### Pattern: Subagent Delegation

When delegating to a subagent, clearly mark it:

```javascript
{
  content: "[P4] Manual testing - DELEGATED to manual-tester",
  status: "in_progress",
  activeForm: "Running manual tests via manual-tester agent"
}
```

### Pattern: CI Wait States

For long-running waits, show explicit progress:

```javascript
{
  content: "[P5] Wait for CI checks (⏳ 0/1200s elapsed)",
  status: "in_progress",
  activeForm: "Waiting for CI checks - 15 minutes remaining"
}
```

Update the todo periodically during waits:

```bash
# After 60 seconds
TodoWrite([
  { content: "[P5] Wait for CI checks (⏳ 60/1200s elapsed)", status: "in_progress", activeForm: "Waiting for CI checks - 14 minutes remaining" }
])

# After 120 seconds
TodoWrite([
  { content: "[P5] Wait for CI checks (⏳ 120/1200s elapsed)", status: "in_progress", activeForm: "Waiting for CI checks - 13 minutes remaining" }
])
```

### Pattern: Retry Loops

When retrying failed steps:

```javascript
{
  content: "[P4] Fix CI failures & retry (Attempt 2/3)",
  status: "in_progress",
  activeForm: "Fixing CI failures and re-running tests"
}
```

---

## Implementation Rules

### Rule 1: Initialize at Phase Start

**MANDATORY**: Create TodoWrite on first execution of phase command.

```bash
# At start of /sdlc:5-review:run
TodoWrite([
  { content: "[P5] Verify code changes exist", status: "pending", activeForm: "Verifying code changes" },
  { content: "[P5] Set up review worktree", status: "pending", activeForm: "Setting up review worktree" },
  # ... all steps
])
```

### Rule 2: Mark as in_progress When Starting

When beginning a step, immediately mark it as in_progress:

```bash
# Starting Step 2
TodoWrite([
  { content: "[P5] Verify code changes exist", status: "completed", activeForm: "Verified code changes" },
  { content: "[P5] Set up review worktree", status: "in_progress", activeForm: "Setting up review worktree" },
  # ... rest with original status
])
```

### Rule 3: Mark as completed When Finished

When step succeeds:

```bash
# Finished Step 2
TodoWrite([
  { content: "[P5] Verify code changes exist", status: "completed", activeForm: "Verified code changes" },
  { content: "[P5] Set up review worktree", status: "completed", activeForm: "Worktree ready" },
  { content: "[P5] Rebase onto staging", status: "in_progress", activeForm: "Rebasing onto staging" },
  # ... rest
])
```

### Rule 4: Document Skips Immediately

When skipping a step, immediately mark it as skipped with reason:

```bash
# Skipping optional step
TodoWrite([
  { content: "[P6] MCP verification - SKIPPED: No MCP integration", status: "skipped", activeForm: "Skipping MCP verification" },
  { content: "[P6] Approve PR", status: "in_progress", activeForm: "Approving pull request" },
  # ... rest
])
```

### Rule 5: ALWAYS End with Capture Learnings

Every phase must end by invoking capture-learnings:

```javascript
{
  content: "[P5] Capture learnings",
  status: "in_progress",
  activeForm: "Capturing session learnings"
}

// After invoke
{
  content: "[P5] Capture learnings",
  status: "completed",
  activeForm: "Learnings captured"
}
```

---

## Naming Conventions

### Content Field

Format: `[Phase N] {Verb} {Object} - {Optional Context}`

**Good Examples:**
- `[P1] Generate specification`
- `[P5] Wait for CI checks`
- `[P6] Delegate CLI testing to manual-tester`
- `[P4] Fix CI failures & retry (Attempt 2/3)`

**Anti-patterns:**
- ❌ `Parse input` (too vague, missing phase)
- ❌ `[P1] Step 1` (no description)
- ❌ `[P5] Do code review` (too generic)

### Active Form Field

Format: `{Present Continuous Verb} {Object Details}`

**Good Examples:**
- `"Verifying prerequisites"`
- `"Waiting for CI checks to pass"`
- `"Running tier-based code review"`
- `"Delegating to manual-tester agent"`

**Anti-patterns:**
- ❌ `"Check CI"` (not continuous)
- ❌ `"Verified"` (past tense, indicates completed already)
- ❌ `"CI check"` (not action-oriented)

---

## Subagent Coordination

### Delegating Work Pattern

When delegating to a subagent, update todos to show delegation:

```javascript
// Before delegation
{
  content: "[P4] Manual testing",
  status: "pending",
  activeForm: "Running manual tests"
}

// Start delegation
{
  content: "[P4] Manual testing - DELEGATED to manual-tester",
  status: "in_progress",
  activeForm: "Delegating CLI/UI tests to manual-tester agent"
}

Task(
  subagent_type: "manual-tester",
  prompt: "Test CLI and UI..."
)

// After delegation returns
{
  content: "[P4] Manual testing - DELEGATED to manual-tester",
  status: "completed",
  activeForm: "Manual testing complete (from subagent)"
}
```

### Multi-Agent Orchestration

For phases with multiple parallel agents:

```javascript
{
  content: "[P5] Run tier-based code review (5 agents in parallel)",
  status: "in_progress",
  activeForm: "Running code review via: logic-analyst, architect, security-researcher, performance-analyst, qa-engineer"
}

// Track individual agents
Task(subagent_type: "general-purpose", prompt: "Logic review for PR #123")
Task(subagent_type: "solutions-architect", prompt: "Architecture review for PR #123")
Task(subagent_type: "security-researcher", prompt: "Security audit for PR #123")
Task(subagent_type: "general-purpose", prompt: "Performance analysis for PR #123")
Task(subagent_type: "qa-engineer", prompt: "Test quality review for PR #123")

// After all complete
{
  content: "[P5] Run tier-based code review (5 agents in parallel)",
  status: "completed",
  activeForm: "Code review complete (all agents passed)"
}
```

---

## CI/Long-Running Operation Tracking

### Pattern: Wait with Timeout Tracking

```javascript
// Initial
{
  content: "[P5] Wait for CI checks",
  status: "in_progress",
  activeForm: "Waiting for CI - Starting (max 20 min)"
}

// Every 60 seconds, update with elapsed time
// After 60s
{
  content: "[P5] Wait for CI checks",
  status: "in_progress",
  activeForm: "Waiting for CI - 1 min / 20 min (19 remaining)"
}

// After 120s
{
  content: "[P5] Wait for CI checks",
  status: "in_progress",
  activeForm: "Waiting for CI - 2 min / 20 min (18 remaining)"
}

// After CI completes
{
  content: "[P5] Wait for CI checks",
  status: "completed",
  activeForm: "CI checks passed (7 min 45 sec)"
}
```

### Pattern: Blocking Waits During Merge

```javascript
// Initial
{
  content: "[P6] Monitor PR merge",
  status: "in_progress",
  activeForm: "Auto-merge enabled - monitoring for merge (max 5 min)"
}

// Every 30 seconds
{
  content: "[P6] Monitor PR merge",
  status: "in_progress",
  activeForm: "Monitoring merge - 30s / 300s (270 remaining)"
}

// When merged
{
  content: "[P6] Monitor PR merge",
  status: "completed",
  activeForm: "PR merged successfully (2 min 15 sec)"
}
```

---

## Skip Decision Gate Pattern

For phases with conditional requirements (e.g., Phase 6 MCP verification):

```markdown
## ⛔ BLOCKING GATE: Skip Eligibility Check

Before marking MCP verification as SKIPPED, verify:

| Requirement | Status | Notes |
|-------------|--------|-------|
| MCP config exists (.mcp.json) | N/A | Not applicable - no MCP in this feature |
| MCP verification in spec | N/A | Not in feature requirements |
| MCP described in plan.md | N/A | Not relevant to feature scope |

**Decision**: MCP verification is NOT applicable to this feature.

```javascript
TodoWrite([
  {
    content: "[P6] MCP verification - SKIPPED: No MCP in feature scope",
    status: "skipped",
    activeForm: "Skipping MCP verification"
  }
])
```

---

## Error Handling

### When a Step Fails

```javascript
// Step failed
{
  content: "[P4] Commit & create PR - FAILED: Merge conflicts",
  status: "blocked",
  activeForm: "Blocked by merge conflicts - needs manual resolution"
}

// After fixing
{
  content: "[P4] Commit & create PR - RETRYING (Attempt 2)",
  status: "in_progress",
  activeForm: "Retrying after resolving conflicts"
}

// After success
{
  content: "[P4] Commit & create PR",
  status: "completed",
  activeForm: "PR created successfully"
}
```

### When a Step Cannot Proceed

```javascript
// Blocked by prerequisite
{
  content: "[P5] Analyze PR complexity",
  status: "blocked",
  activeForm: "Blocked - Phase 4 (IMPLEMENT) incomplete"
}
```

---

## Examples by Phase

### Phase 5 (REVIEW) Complete Example

```javascript
TodoWrite([
  { content: "[P5] Verify code changes exist", status: "completed", activeForm: "Verified code changes" },
  { content: "[P5] Set up review worktree", status: "completed", activeForm: "Worktree ready at /tmp/worktrees/review-527-oauth-preview-auth" },
  { content: "[P5] Rebase onto staging", status: "completed", activeForm: "Rebased successfully" },
  { content: "[P5] Wait for CI checks", status: "completed", activeForm: "CI passed (5 min 30 sec)" },
  { content: "[P5] Analyze PR complexity & tier", status: "completed", activeForm: "Tier 3 determined (11 files, 780 lines)" },
  { content: "[P5] Run tier-based code review", status: "completed", activeForm: "Code review passed (all agents passed)" },
  { content: "[P5] Check code review gate", status: "completed", activeForm: "Gate passed - ready for Phase 6" },
  { content: "[P5] Clean up review worktree", status: "completed", activeForm: "Worktree cleaned up" },
  { content: "[P5] Capture learnings", status: "completed", activeForm: "Learnings captured in PR #535" }
])
```

### Phase 4 (IMPLEMENT) with Skipped Step

```javascript
TodoWrite([
  { content: "[P4] Verify Phase 3 (TEST) complete", status: "completed", activeForm: "Test files found" },
  { content: "[P4] Set up implementation worktree", status: "completed", activeForm: "Worktree ready" },
  { content: "[P4] Load tasks & implementation plan", status: "completed", activeForm: "Tasks loaded (12 tasks)" },
  { content: "[P4] Execute implementation tasks", status: "completed", activeForm: "All tasks complete" },
  { content: "[P4] Verify tests PASS", status: "completed", activeForm: "All tests passing (test suite: 156 passed)" },
  { content: "[P4] Run manual CLI testing - DELEGATED", status: "completed", activeForm: "Manual CLI tests passed" },
  { content: "[P4] Run manual UI testing - DELEGATED", status: "completed", activeForm: "Manual UI tests passed" },
  { content: "[P4] Rebase onto staging", status: "completed", activeForm: "Rebased cleanly" },
  { content: "[P4] Commit & create PR", status: "completed", activeForm: "PR #527 created" },
  { content: "[P4] Update GitHub issue", status: "completed", activeForm: "Issue #527 updated" },
  { content: "[P4] Capture learnings", status: "in_progress", activeForm: "Capturing learnings" }
])
```

---

## Enforcement

### For Command Authors

**EVERY SDLC phase command MUST:**

1. ✅ Initialize TodoWrite in first executable code block
2. ✅ Use the phase-specific template provided above
3. ✅ Update status as each step progresses
4. ✅ Document all skips with reason
5. ✅ Mark each step in_progress when starting
6. ✅ Mark each step completed when finished
7. ✅ End with capture-learnings invocation

### For Operators (Users)

**EVERY SDLC phase execution SHOULD:**

1. ✅ Check TodoWrite for current progress
2. ✅ Reference TodoWrite when context-switching
3. ✅ Document additional insights in capture-learnings
4. ✅ Reference TodoWrite in /clear command to preserve context

---

## Future Enhancements

### Possible Extensions (Post-1.0)

1. **Nested Todos**: Support sub-steps for complex phases
2. **Metrics Capture**: Track timing data (elapsed time per phase)
3. **Dependency Tracking**: Mark blocking/blocked relationships
4. **Rollback Information**: Store recovery instructions for failed steps
5. **Auto-Reporting**: Generate phase completion reports from todos

---

## Migration Path

### For Existing Commands

Apply this template incrementatively:

1. **Phase 5 (REVIEW)** - PRIORITY 1 (uses TodoWrite, good test case)
2. **Phase 6 (VERIFY)** - PRIORITY 1 (manual testing visibility)
3. **Phase 4 (IMPLEMENT)** - PRIORITY 2 (long-running, needs progress tracking)
4. **Phase 3 (TEST)** - PRIORITY 2 (CI/wait tracking)
5. **Phase 1 (SPECIFY)** - PRIORITY 3
6. **Phase 2 (DESIGN)** - PRIORITY 3
7. **Phase 7 (DEPLOY)** - PRIORITY 3

### For New Commands

Use this template immediately for all new SDLC phase commands.

---

## Validation Checklist

Before declaring compliance, verify:

- [ ] Phase initialization creates TodoWrite (first step of command)
- [ ] All major steps map to todos (no orphaned logic)
- [ ] Each todo has 3-field structure (content, status, activeForm)
- [ ] Status follows rules: pending → in_progress → completed/skipped
- [ ] ActiveForm uses present continuous verb tense
- [ ] Conditional steps document skip reason inline
- [ ] Subagent delegation clearly marked
- [ ] Final step is /capture-learnings
- [ ] Content field follows naming convention `[PN] {Verb} {Object}`
- [ ] No todos with status=pending at session end (except blocked with reason)
