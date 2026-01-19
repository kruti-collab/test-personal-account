---
allowed-tools: Read, Grep, Glob, Write, Edit, Bash
description: Sync E2E expert mental model with actual test codebase changes
argument-hint: [optional: specific_area_to_update]
---

# E2E Expert - Self-Improve Mode

Synchronize the expertise.yaml mental model with the actual E2E test codebase. This runs automatically after test changes to keep the expert knowledge current.

## Variables

FOCUS_AREA: $1 (optional - specific area to update, e.g., "smoke", "auth", "patterns")
EXPERTISE_PATH: .claude/commands/experts/e2e/expertise.yaml

## Instructions

- This is a LEARNING task - update expertise.yaml with discoveries
- Compare current expertise against actual test code
- Add new patterns, test files, utilities discovered
- Update skipped tests tracking
- Update last_updated timestamp

## Workflow

### Step 1: Load Current Expertise
Read `EXPERTISE_PATH` to understand what the expert currently "knows"

### Step 2: Scan Codebase for Reality
```bash
# Find all test files
find apps/dashboard/e2e -name "*.spec.ts" | wc -l

# Check test organization
ls -la apps/dashboard/e2e/ui/

# Find any new patterns in tests
grep -r "page.waitFor" apps/dashboard/e2e/ui/ --include="*.ts" | head -10

# Check skipped tests
grep -c "test.skip" apps/dashboard/e2e/ui/**/*.ts 2>/dev/null || echo "0"

# List page objects
ls apps/dashboard/e2e/pages/ 2>/dev/null
```

### Step 3: Compare and Identify Gaps

For each section in expertise.yaml:
1. **Structure**: Are all test directories listed? Any new ones?
2. **Patterns**: Any new testing patterns discovered?
3. **Workflows**: Are CI workflow references accurate?
4. **Skipped Tests**: Is the count up to date?

### Step 4: Update Expertise File

Use Edit tool to update `EXPERTISE_PATH`:
- Add newly discovered test files/patterns
- Update test counts and durations
- Mark deprecated patterns
- Update `last_updated` field

### Step 5: Document Learning

If significant patterns discovered, also update:
- `.claude/expertise/domains/e2e-patterns.md`
- `apps/dashboard/e2e/SKIPPED_TESTS.md` if skips changed

## What to Update

| Section | How to Verify | Update If |
|---------|--------------|-----------|
| structure | ls e2e/ui/ | New test directory |
| patterns | grep common patterns | New testing pattern |
| workflows | ls .github/workflows/e2e* | New workflow file |
| skipped_tests | grep test.skip | Skip count changed |
| auth | Read global-setup.ts | Auth flow changed |

## Report

- Summary of changes made to expertise.yaml
- List of new discoveries
- Updated test counts
- Any items that need human verification
