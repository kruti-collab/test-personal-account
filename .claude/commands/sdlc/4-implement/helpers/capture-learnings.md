---
description: Capture agentic layer improvements from implementation session before commit
---

# Capture Session Learnings

Before committing implementation changes, analyze the session for agentic layer improvements.

## When to Use

Invoke this helper automatically at the end of sdlc:4-implement, before any commit.

## Process

### Step 1: Analyze Session Friction

Review the session and identify:

1. **User corrections needed** - Where did the user have to redirect the agent?
2. **Missing automation** - What should have happened automatically?
3. **Pattern discoveries** - What new patterns emerged during implementation?
4. **Tool misuse** - Were the wrong tools used for a task?

### Step 2: Categorize Learnings

| Category | File Location | Example |
|----------|---------------|---------|
| SDLC command improvements | `.claude/commands/sdlc/*/` | Add auto-task generation |
| Context rules | `.claude/rules/` | Add OAuth testing patterns |
| Agent definitions | `.claude/agents/` | Update agent capabilities |
| Hook additions | `.claude/hooks/` | Pre-commit validation |

### Step 3: Generate Improvement List

For each learning, create an entry:

```yaml
- type: [command|rule|agent|hook]
  file: path/to/file.md
  change: Brief description of the change
  rationale: Why this helps future sessions
  priority: [high|medium|low]
```

### Step 4: Ask User for Approval

Present findings in this format:

```
## Agentic Layer Improvements Identified

During this implementation, the following improvements were identified:

| # | Type | Change | Priority |
|---|------|--------|----------|
| 1 | command | Add auto-tasks.md to sdlc:2-design | high |
| 2 | rule | Add OAuth clean-slate testing pattern | medium |
| 3 | command | Separate test/implement phases | high |

Would you like to:
1. Create a separate PR for these agentic improvements
2. Include in the current feature PR
3. Skip for now (record in backlog)
```

### Step 5: Execute Based on User Choice

**Option 1: Separate PR**
- Invoke: `Skill(skill: "sdlc:5-review:helpers:agentic-pr")`
- Creates dedicated PR for agentic layer changes

**Option 2: Include in Current PR**
- Apply changes to current branch
- Include in feature commit

**Option 3: Skip**
- Write learnings to `specs/FEATURE_DIR/learnings.md`
- Create GitHub issue for future work

## Output Format

```
## Session Learnings Report

### Friction Points Identified: N
### Improvements Proposed: N

| Improvement | Applied | Location |
|-------------|---------|----------|
| Auto-generate tasks.md | Yes | .claude/commands/sdlc/2-design/run.md |
| OAuth testing guidelines | Yes | .claude/commands/sdlc/3-test/run.md |

### Next Session Benefit
These changes will prevent N user corrections in similar future sessions.
```

## Key Principle

**Continuous Improvement**: Every implementation session should leave the agentic layer slightly better than it found it.
