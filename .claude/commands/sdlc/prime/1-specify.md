# Prime SPECIFY - Load Specification Context

Prime context window with everything needed for feature specification work.

## Step 1: Read Core Documentation

Read these files to understand specification requirements:

1. Read `docs/sdlc/1-specify/README.md`
2. Read `docs/sdlc/1-specify/features/README.md`
3. Read `openspec/templates/spec-template.md` (if exists)
4. Read `openspec/project.md` (if exists)

## Step 2: Scan Existing Specs

```bash
# List all feature specs to understand patterns
find docs/features -name "*.md" -type f 2>/dev/null | head -30

# List any active spec work
find . -path "*/openspec/*" -name "spec.md" 2>/dev/null | head -10
```

## Step 3: Read Example Specs

Read 2-3 existing feature specs to understand the format:

1. Read `docs/features/gal/convenience/01-auto-discovery.md` (if exists)
2. Read `docs/features/gal/enforcement/01-approval-workflows.md` (if exists)

## Step 4: Check Current Work

```bash
# Current branch
git branch --show-current

# Any uncommitted spec changes
git status --short | grep -E "docs/features|openspec"

# Recent spec commits
git log --oneline -5 -- "docs/features/" "openspec/"
```

## Specification Rules (Memorize These)

### The Golden Rule
**Specs define WHAT users need, not HOW to build it.**

### Required Sections
- **Overview**: Problem being solved
- **User Stories**: As a [role], I want [X] so that [Y]
- **Acceptance Criteria**: Testable, measurable outcomes
- **Success Metrics**: Quantifiable goals

### What NOT to Include
- Technology choices (React, Postgres, etc.)
- API specifications
- Database schemas
- Implementation timelines

### Spec Quality Checklist
- [ ] Written for non-technical stakeholders
- [ ] Every AC is testable
- [ ] No implementation details
- [ ] Success criteria are measurable
- [ ] User value is clear

## CRITICAL RULES (MUST FOLLOW)

### CI Check Protocol
**NEVER approve or merge a PR until ALL CI checks pass.** Don't run tests locally while CI runs - wait for CI. Don't use `--admin` to bypass failures - fix them first.

### Fix Failures On The Spot
When discovering a bug or failure during SDLC workflow: fix immediately while having context. Don't skip past failures. Never use `--admin` bypass.

### GitHub Issue Linkage
**ALWAYS use "Closes #X" (not "Relates to") in PR descriptions for auto-close.**

## After Priming

You are now ready to run:
- `/sdlc/specify/run [feature description]` - Create new spec
- `/sdlc/specify/helpers/clarify` - Clarify requirements
- `/sdlc/specify/helpers/proposal` - Create change proposal

## Report After Priming

Summarize:
1. Current branch and spec-related changes
2. Number of existing feature specs found
3. Template availability
4. Ready to proceed with specification work
