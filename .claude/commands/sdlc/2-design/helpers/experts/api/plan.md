---
allowed-tools: Read, Grep, Glob, Write, TodoWrite
description: Create implementation plans for API changes using expert knowledge
argument-hint: [task_description]
---

# API Expert - Plan Mode

Create a detailed implementation plan for API changes by leveraging accumulated expertise about the GAL backend architecture, patterns, and constraints.

## Variables

USER_TASK: $1
EXPERTISE_PATH: .claude/commands/experts/api/expertise.yaml

## Instructions

- This is a PLANNING task - DO NOT implement, only create a plan
- Plans must respect backward compatibility (see expertise.yaml patterns)
- Plans must follow security patterns from expertise
- Output plan to `logs/plans/api-plan-{timestamp}.md`

## Workflow

1. Read `EXPERTISE_PATH` to understand current API architecture
2. Validate expertise against codebase - note any outdated information
3. Analyze the `USER_TASK` requirements
4. Identify affected endpoints, services, and middleware
5. Create step-by-step implementation plan
6. Flag any breaking changes or security considerations

## Plan Template

Write plan to `logs/plans/api-plan-{timestamp}.md`:

```markdown
# API Implementation Plan

## Task
[USER_TASK description]

## Affected Components
- Endpoints: [list]
- Services: [list]
- Middleware: [list]

## Prerequisites
- [ ] [What must exist before implementation]

## Implementation Steps

### Step 1: [Description]
- File: `path/to/file.ts`
- Changes: [What to modify]
- Rationale: [Why]

### Step 2: [Description]
...

## Backward Compatibility
- [ ] No endpoints removed
- [ ] No field types changed
- [ ] New fields are optional

## Security Checklist
- [ ] Rate limiting applied
- [ ] Auth middleware used
- [ ] Errors sanitized

## Testing Plan
- [ ] Unit tests for new service methods
- [ ] Integration tests for new endpoints
- [ ] E2E tests updated if UI affected
```

## Report

- Path to generated plan file
- Summary of planned changes
- Any expertise gaps discovered (for self-improve)
