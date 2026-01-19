---
description: Verify feature specs are complete with proper integration points, API definitions, and match implementation
argument-hint: "[spec-file|all]"
---

# Verify Specs (Orchestrator)

You are the **ORCHESTRATOR** for spec verification. You ensure feature specs are complete and properly define how features connect.

## Why This Exists

Incomplete specs → Broken integrations:
- "Precondition: Discovery must complete" without defining the API
- Frontend expects `api.getDiscoveredRepos()` but it doesn't exist
- Features work in isolation but not together

## Arguments

- `$ARGUMENTS`: Spec file or "all"
  - `all` - Verify all convenience tier specs
  - `03-auto-discovery.md` - Verify specific spec
  - `discovery` - Shorthand for discovery spec

## Phase 1: Identify Specs to Verify

```bash
# List all convenience tier specs
ls docs/features/gal/convenience/*.md
```

Expected specs for Convenience tier:
- 01-dashboard.md
- 02-github-auth.md
- 03-auto-discovery.md
- 04-approved-config.md
- 05-sync.md
- 06-settings.md
- 07-billing.md
- 08-docs.md

## Phase 2: Spawn Spec Verifier

```
Task(
  subagent_type: "spec-verifier",
  prompt: "Verify feature specs are complete.

           Specs to verify: $ARGUMENTS

           YOU MUST:
           1. Read each spec file
           2. Check required sections exist
           3. Build dependency graph from Integration Points
           4. Verify each connection is defined on BOTH ends
           5. Check implementation matches spec (API exists)

           Return structured report with:
           - SPECS_ANALYZED
           - DEPENDENCY_GRAPH
           - ISSUES_FOUND
           - VERDICT"
)
```

## Phase 3: Handle Issues

Based on subagent report:

### MISSING_SECTION Issues
```
Fix: Add the missing section to the spec
- User Story
- Acceptance Criteria
- Data Requirements
- Preconditions
- Integration Points
```

### UNDEFINED_INTEGRATION Issues
```
Fix: Add Integration Points section defining:
- What data flows IN (RECEIVES)
- What data flows OUT (PROVIDES)
- API endpoints for each
- Which ACs depend on each connection
```

### API_MISMATCH Issues
```
Fix options:
1. Add missing API endpoint to apps/api/src/index.ts
2. Add missing frontend method to apps/dashboard/src/lib/api.ts
3. Update spec if endpoint name is wrong
```

## Required Spec Sections

Every feature spec must have:

```markdown
# Feature Name

## User Story
As a [role], I want [goal], so that [benefit].

## Acceptance Criteria
1. [Testable requirement]
2. [Testable requirement]
...

## Data Requirements
### API Response
interface ...

## Preconditions
- [What must be true]

## Integration Points
### [Feature] → This Feature (RECEIVES)
| Data | API Endpoint | Required For |

### This Feature → [Feature] (PROVIDES)
| Data | API Endpoint | Required For |
```

## Convenience Tier Dependency Graph

```
┌─────────────────────────────────────────────────────────────────┐
│                  GAL CONVENIENCE DATA FLOW                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  GitHub Auth ────────────────┐                                  │
│       │                      │                                  │
│       ▼                      ▼                                  │
│  ┌──────────┐          ┌──────────┐                            │
│  │ Settings │◄────────►│ Dashboard│                            │
│  │ (orgs)   │          │ (stats)  │                            │
│  └────┬─────┘          └────┬─────┘                            │
│       │                     │                                   │
│       └─────────┬───────────┘                                   │
│                 │                                               │
│                 ▼                                               │
│          ┌───────────┐                                         │
│          │ Discovery │ ◄── Scan GitHub repos                   │
│          │ (configs) │                                         │
│          └─────┬─────┘                                         │
│                │                                               │
│    ┌───────────┴───────────┐                                   │
│    │                       │                                   │
│    ▼                       ▼                                   │
│  ┌──────────────┐    ┌──────────┐                             │
│  │ Approved     │───►│ CLI Sync │                             │
│  │ Config       │    │ (pull)   │                             │
│  │ (overrides)  │    └──────────┘                             │
│  └──────────────┘                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Required Integration Points:
1. GitHub Auth → Settings (provides: user, org access)
2. Settings → Dashboard (provides: org connection status)
3. Discovery → Dashboard (provides: config counts)
4. Discovery → Approved Config (provides: discovered repos) ← CRITICAL
5. Approved Config → CLI Sync (provides: merged config)
```

## Output Format

```
═══════════════════════════════════════════════════════
SPEC VERIFICATION COMPLETE
═══════════════════════════════════════════════════════

Specs: $ARGUMENTS

SPECS ANALYZED: N
├── [spec] ✓/⚠/✗
└── ...

INTEGRATION COVERAGE:
├── Discovery → Approved Config: DEFINED/MISSING
├── Approved Config → CLI Sync: DEFINED/MISSING
└── ...

ISSUES FIXED: N
├── MISSING_SECTION: N
├── UNDEFINED_INTEGRATION: N
└── API_MISMATCH: N

VERDICT: ✓ COMPLETE / ⚠ PARTIAL / ✗ INCOMPLETE
═══════════════════════════════════════════════════════
```

## Example Usage

```bash
# Verify all specs
/testing:verify-specs all

# Verify specific spec
/testing:verify-specs 03-auto-discovery.md

# Verify by feature name
/testing:verify-specs discovery
```
