---
allowed-tools: Read, Grep, Glob, Write, TodoWrite
description: Create implementation plans for E2E test changes using expert knowledge
argument-hint: [task_description]
---

# E2E Expert - Plan Mode

Create a detailed implementation plan for E2E test changes by leveraging accumulated expertise about the GAL testing architecture, patterns, and constraints.

## Variables

USER_TASK: $1
EXPERTISE_PATH: .claude/commands/experts/e2e/expertise.yaml

## Instructions

- This is a PLANNING task - DO NOT implement, only create a plan
- Plans must follow critical patterns from expertise (login guard, no OR-logic, proper waits)
- Plans must consider CI workflow integration
- Output plan to `logs/plans/e2e-plan-{timestamp}.md`

## Workflow

1. Read `EXPERTISE_PATH` to understand current E2E architecture
2. Validate expertise against codebase - note any outdated information
3. Analyze the `USER_TASK` requirements
4. Identify affected test files, page objects, and utilities
5. Create step-by-step implementation plan
6. Flag any patterns that must be followed

## Plan Template

Write plan to `logs/plans/e2e-plan-{timestamp}.md`:

```markdown
# E2E Test Implementation Plan

## Task
[USER_TASK description]

## Test Scope
- Type: [smoke/features/workflows]
- Environment: [dev-local/staging-preview/staging]
- Expected Duration: [estimate]

## Affected Files
- Tests: [list]
- Page Objects: [list]
- Utilities: [list]

## Implementation Steps

### Step 1: [Description]
- File: `path/to/test.spec.ts`
- Changes: [What to add/modify]
- Pattern: [Which expertise pattern applies]

### Step 2: [Description]
...

## Required Patterns Checklist
- [ ] Login redirect guard included
- [ ] No OR-logic in assertions
- [ ] Proper wait patterns used
- [ ] .filter() syntax (not :has-text)
- [ ] Headless mode default

## CI Workflow
- Workflow file: [which one]
- Manual trigger: `gh workflow run [workflow] --ref [branch]`

## Validation Steps
- [ ] Run locally: `./scripts/test.sh [scope] dev-local`
- [ ] Check skipped tests updated if needed
```

## Report

- Path to generated plan file
- Summary of planned test changes
- Any expertise gaps discovered (for self-improve)
