---
allowed-tools: Read, Grep, Glob, Write, Edit, Bash
description: Sync API expert mental model with actual codebase changes
argument-hint: [optional: specific_area_to_update]
---

# API Expert - Self-Improve Mode

Synchronize the expertise.yaml mental model with the actual codebase. This runs automatically after API changes to keep the expert knowledge current.

## Variables

FOCUS_AREA: $1 (optional - specific area to update, e.g., "auth", "billing", "endpoints")
EXPERTISE_PATH: .claude/commands/experts/api/expertise.yaml

## Instructions

- This is a LEARNING task - update expertise.yaml with discoveries
- Compare current expertise against actual code
- Add new patterns, endpoints, services discovered
- Remove or mark deprecated items
- Update last_updated timestamp

## Workflow

### Step 1: Load Current Expertise
Read `EXPERTISE_PATH` to understand what the expert currently "knows"

### Step 2: Scan Codebase for Reality
```bash
# Find all endpoints
grep -n "app\.\(get\|post\|put\|delete\|patch\)" apps/api/src/index.ts

# List all services
ls apps/api/src/services/

# Check auth middleware
grep -n "requireAuth\|requireOrgAccess\|requirePlan" apps/api/src/index.ts
```

### Step 3: Compare and Identify Gaps

For each section in expertise.yaml:
1. **Endpoints**: Are all endpoints listed? Any new ones? Any removed?
2. **Services**: Are all services documented? Any new ones?
3. **Auth**: Are auth flows accurate? Any new methods?
4. **Patterns**: Any new patterns discovered during recent work?

### Step 4: Update Expertise File

Use Edit tool to update `EXPERTISE_PATH`:
- Add newly discovered endpoints/services
- Update changed patterns
- Mark deprecated items
- Update `last_updated` field

### Step 5: Document Learning

If significant patterns discovered, also update:
- `.claude/expertise/domains/api-patterns.md`

## What to Update

| Section | How to Verify | Update If |
|---------|--------------|-----------|
| endpoints | grep app.get/post | New endpoint added |
| services | ls services/ | New service file |
| auth | grep requireAuth | New auth method |
| security | Read SECURITY.md | New security pattern |
| patterns | Recent git commits | New coding pattern |

## Report

- Summary of changes made to expertise.yaml
- List of new discoveries
- Any items that need human verification
- Timestamp of update
