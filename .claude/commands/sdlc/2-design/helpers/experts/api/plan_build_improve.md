---
allowed-tools: Task, Read, Write, TodoWrite
description: Complete API implementation cycle - plan with expertise, build, then self-improve
argument-hint: [task_description]
---

# API Expert - Plan, Build, Improve

This workflow orchestrates a complete API implementation cycle by chaining three specialized commands: expertise-informed planning, building from the plan, and self-improving the expertise based on changes made.

## Variables

USER_PROMPT: $1
HUMAN_IN_THE_LOOP: $2 or true if not specified

## Instructions

- This is an ORCHESTRATION workflow - it spawns subagents for each phase
- Phase 1 (Plan) creates an expertise-informed implementation plan
- Phase 2 (Build) implements the plan following TDD
- Phase 3 (Self-Improve) updates expertise with learnings
- If HUMAN_IN_THE_LOOP is true, pause for approval between phases

## Workflow

### Step 1: Create Plan

Spawn a subagent to run the planning command:

```
Task(
  subagent_type: "general-purpose",
  prompt: "Run SlashCommand('/experts:api:plan [USER_PROMPT]'). Return the path to the generated plan file."
)
```

Replace `[USER_PROMPT]` with the actual user request.

Use TaskOutput to get `path_to_plan` before proceeding.

**If HUMAN_IN_THE_LOOP**: Show plan and ask for approval before continuing.

### Step 2: Build

Spawn a subagent to implement the plan:

```
Task(
  subagent_type: "general-purpose",
  prompt: "Read the plan at [path_to_plan]. Implement each step following TDD - write failing tests first, then implement. Follow all security patterns from the plan."
)
```

Use TaskOutput to confirm build completion and get `git_diff` of changes.

**If HUMAN_IN_THE_LOOP**: Show changes and ask for approval before continuing.

### Step 3: Self-Improve

Spawn a subagent to update the expertise:

```
Task(
  subagent_type: "general-purpose",
  prompt: "Run SlashCommand('/experts:api:self-improve'). The API was just modified. Sync the expertise.yaml with the new codebase state."
)
```

## Error Handling

- If Plan fails: Report error, do not proceed
- If Build fails: Report error, suggest manual intervention, still run Self-Improve to capture partial learnings
- If Self-Improve fails: Report warning, changes are complete but expertise not updated

## Report

After all phases complete:

```markdown
## API Expert - Execution Summary

### Task
[USER_PROMPT]

### Phase 1: Plan
- Status: [success/failed]
- Plan file: [path]

### Phase 2: Build
- Status: [success/failed]
- Files changed: [count]
- Tests: [pass/fail count]

### Phase 3: Self-Improve
- Status: [success/failed]
- Expertise updates: [list]

### Next Steps
- [Any remaining manual tasks]
```
