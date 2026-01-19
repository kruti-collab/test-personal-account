---
description: Verify feature integrations and data flow between Discovery → Approved Config → CLI Sync. SPECS ARE THE GOLDEN STANDARD.
argument-hint: "[flow-name|all]"
---

# Verify Integration (Orchestrator)

You are the **ORCHESTRATOR** for integration flow verification. You verify that GAL features work together as a cohesive product.

## CRITICAL: Specs Are The Golden Standard

**ALWAYS read feature specs FIRST. Specs define what features MUST do and how they connect.**

```
docs/features/gal/convenience/
├── 03-auto-discovery.md   # Defines scan results, config counts
├── 04-approved-config.md  # Defines two-tier model, uses discovery data
├── 05-sync.md             # Defines CLI sync, pulls approved config
```

**Rules:**
1. Read specs BEFORE verification
2. Verify UI/API against specs
3. If mismatch → FIX UI/API/TEST, never the spec
4. Specs are immutable source of truth

## Arguments

- `$ARGUMENTS`: Optional flow name or "all"
  - `discovery-dashboard` - Discovery → Dashboard data flow
  - `discovery-approved` - Discovery → Approved Config connection
  - `approved-cli` - Approved Config → CLI Sync flow
  - `settings-all` - Settings data consistency across features
  - `all` (default) - Verify all integration flows

## Why This Matters

Features tested in isolation can all pass while the product is broken:
- Discovery shows 292 configs
- Approved Config can't see discovered repos
- CLI sync returns empty config

**Integration verification catches these gaps.**

## Phase 0: Read Feature Specs (ALWAYS FIRST)

**Before ANY verification, read the specs to understand what should be verified.**

```bash
# Read specs for the integration being verified
Read(docs/features/gal/convenience/03-auto-discovery.md)
Read(docs/features/gal/convenience/04-approved-config.md)
Read(docs/features/gal/convenience/05-sync.md)
```

### Extract Integration Points from Specs

From **03-auto-discovery.md**:
- Precondition: "Discovery scan must have completed" (for Approved Config)
- AC11: "Configs Found count must equal sum of all config type counts"
- Data flows TO: Approved Config (discovered repos), Dashboard (stats)

From **04-approved-config.md**:
- Precondition: "Discovery scan must have completed"
- Precondition: "At least one config must be discovered"
- AC6: "Each discovered repo shows 'Set Override' button"
- Data flows FROM: Discovery (repos), TO: CLI Sync (merged config)

From **05-sync.md**:
- Pulls approved config from API
- Returns merged config (base + override)
- Data flows FROM: Approved Config

**Use these spec requirements as your verification checklist.**

## Phase 1: Environment Setup (YOU DO THIS)

### 1a. Check servers
```bash
curl -s http://localhost:3000/health && echo "API_READY" || echo "API_NOT_RUNNING"
curl -s http://localhost:5173 &>/dev/null && echo "DASHBOARD_READY" || echo "DASHBOARD_NOT_RUNNING"
```

### 1b. Get auth token
```bash
# Token should be in localStorage after login
# Or use dev-token endpoint
curl -s http://localhost:3000/auth/dev-token \
  -H "Content-Type: application/json" \
  -d '{"githubId": "YOUR_TEST_USER_GITHUB_ID"}' | jq -r .token
```

## Phase 2: Spawn Integration Verifier

```
Task(
  subagent_type: "integration-flow-verifier",
  prompt: "Verify GAL feature integrations.

           Flow to verify: $ARGUMENTS (or 'all' if not specified)

           YOU MUST:
           1. Collect baseline data from API endpoints
           2. Navigate to each feature page using playwright-skill
           3. Extract displayed data values
           4. Cross-reference values across pages
           5. Report any DATA_INCONSISTENCY or FLOW_BROKEN issues

           Return structured report with:
           - API_BASELINE
           - PAGE_DATA
           - FLOW_CHECKS
           - ISSUES_FOUND
           - VERDICT"
)
```

## Phase 3: Handle Issues

Based on subagent report:

### DATA_INCONSISTENCY Issues
```
IF issue.type == "DATA_INCONSISTENCY":
    - Check if API returns different data for same resource
    - Check if UI is caching stale values
    - Check if different endpoints return different counts

    Typical fixes:
    - Add API call to refresh data after operations
    - Ensure consistent API response format
    - Fix calculation logic
```

### FLOW_BROKEN Issues
```
IF issue.type == "FLOW_BROKEN":
    - Feature A should connect to Feature B but doesn't
    - Missing API call between features
    - Data from Feature A not passed to Feature B

    Typical fixes:
    - Add API integration between features
    - Pass data through shared state/context
    - Add navigation links between features
```

### ORPHAN_DATA Issues
```
IF issue.type == "ORPHAN_DATA":
    - Data exists in API but not shown in UI
    - Data shown in one page but not where expected

    Typical fixes:
    - Add UI component to display data
    - Connect data to correct page
```

## Phase 4: Iteration (If Needed)

```
IF verdict != INTEGRATED AND fixes_possible:
    Apply fixes
    Re-run integration verifier
    Repeat until INTEGRATED or blocked

IF verdict == DISCONNECTED AND no_fixes_possible:
    Report blocked - needs architectural changes
```

## Output Format

```
═══════════════════════════════════════════════════════
INTEGRATION VERIFICATION COMPLETE
═══════════════════════════════════════════════════════

Flow: $ARGUMENTS

ITERATIONS: N
├─ Iteration 1: [issues found] → [fixes applied]
└─ Iteration N: INTEGRATED ✓

DATA CONSISTENCY:
├─ Organizations count: API=1, Dashboard=1, Settings=1 ✓
├─ Configs count: API=292, Discovery=292, Dashboard=292 ✓
└─ Config breakdown sum: 284 ≠ 292 ✗ (NEEDS FIX)

FLOW STATUS:
├─ Discovery → Dashboard: ✓ CONNECTED
├─ Discovery → Approved Config: ✓ CONNECTED
├─ Approved Config → CLI: ✓ CONNECTED
└─ Settings → All: ✓ CONSISTENT

ISSUES FIXED: N
├─ DATA_INCONSISTENCY: N
├─ FLOW_BROKEN: N
└─ ORPHAN_DATA: N

REMAINING ISSUES: N (if any)
├─ [issue details]
└─ [blocked reason]

RESULT: ✓ INTEGRATED / ⚠ PARTIALLY_INTEGRATED / ✗ DISCONNECTED
═══════════════════════════════════════════════════════
```

## GAL-Specific Integration Checks

### Discovery → Dashboard
```
□ Dashboard "Configs Found" == Discovery total configs
□ Dashboard "Organizations" == Discovery scanned orgs count
□ Dashboard updates after new scan
```

### Discovery → Approved Config
```
□ All scanned repos appear in Project Overrides list
□ "Set Override" available for each discovered repo
□ Repo config counts visible in override section
```

### Approved Config → CLI
```
□ "Preview Merged Config" shows base + override
□ CLI `/sync` endpoint returns same merged config
□ Version numbers match between UI and API
```

### Settings → All Features
```
□ GitHub connection status consistent across pages
□ Org names match everywhere
□ "X org connected" in sidebar matches actual count
```

## Example Usage

```bash
# Verify all integration flows
/verify-integration all

# Verify specific flow
/verify-integration discovery-approved

# Verify data consistency only
/verify-integration settings-all
```
