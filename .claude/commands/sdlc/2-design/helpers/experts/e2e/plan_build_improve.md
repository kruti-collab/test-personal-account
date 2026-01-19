---
allowed-tools: Task, Read, Write, TodoWrite
description: Complete E2E test implementation cycle - plan with expertise, build tests, then self-improve
argument-hint: [task_description]
---

# E2E Expert - Plan, Build, Improve

This workflow orchestrates a complete E2E test implementation cycle by chaining three specialized commands: expertise-informed planning, building tests from the plan, and self-improving the expertise based on changes made.

## Variables

USER_PROMPT: $1
HUMAN_IN_THE_LOOP: $2 or true if not specified

## Instructions

- This is an ORCHESTRATION workflow - it spawns subagents for each phase
- Phase 1 (Plan) creates an expertise-informed test plan
- Phase 2 (Build) writes the tests following the plan
- Phase 3 (Self-Improve) updates expertise with learnings
- If HUMAN_IN_THE_LOOP is true, pause for approval between phases

## Workflow

### Step 1: Create Plan

Spawn a subagent to run the planning command:

```
Task(
  subagent_type: "qa-engineer",
  prompt: "Run SlashCommand('/experts:e2e:plan [USER_PROMPT]'). Return the path to the generated plan file."
)
```

Replace `[USER_PROMPT]` with the actual user request.

Use TaskOutput to get `path_to_plan` before proceeding.

**If HUMAN_IN_THE_LOOP**: Show plan and ask for approval before continuing.

### Step 2: Build Tests

Spawn a subagent to implement the test plan:

```
Task(
  subagent_type: "qa-engineer",
  prompt: "Read the plan at [path_to_plan]. Write the E2E tests following all patterns from the plan. Run tests locally with './scripts/test.sh [scope] dev-local' to verify they pass."
)
```

Use TaskOutput to confirm tests pass and get list of files created.

**If HUMAN_IN_THE_LOOP**: Show test results and ask for approval before continuing.

### Step 3: Self-Improve

Spawn a subagent to update the expertise:

```
Task(
  subagent_type: "general-purpose",
  prompt: "Run SlashCommand('/experts:e2e:self-improve'). E2E tests were just added/modified. Sync the expertise.yaml with the new test patterns."
)
```

## Error Handling

- If Plan fails: Report error, do not proceed
- If Build fails (tests don't pass): Report failure details, do not proceed to self-improve
- If Self-Improve fails: Report warning, tests are complete but expertise not updated

## Report

After all phases complete:

```markdown
## E2E Expert - Execution Summary

### Task
[USER_PROMPT]

### Phase 1: Plan
- Status: [success/failed]
- Plan file: [path]
- Test scope: [smoke/features/workflows]

### Phase 2: Build
- Status: [success/failed]
- Tests created: [count]
- Local run: [pass/fail]

### Phase 3: Self-Improve
- Status: [success/failed]
- Expertise updates: [list]

### CI Integration
To run in CI:
\`\`\`bash
git push origin [branch]
gh workflow run [workflow].yml --ref [branch]
\`\`\`

### Next Steps
- [Any remaining manual tasks]
```
