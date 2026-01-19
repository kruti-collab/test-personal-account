---
description: TDD loop - generate tests from spec, run them (should FAIL), then hand off to IMPLEMENT
argument-hint: "<feature-spec.md> [environment]"
handoffs:
  - label: "Next: IMPLEMENT (Step 4)"
    agent: sdlc:4-implement:run
    prompt: Implement code to make tests pass (TDD - Red → Green)
---

## MANDATORY: Initialize Progress Tracking

**Before ANY work, create TodoWrite:**

```javascript
TodoWrite([
  { content: "[P3] Verify Phase 2 (DESIGN) complete", status: "pending", activeForm: "Verifying plan.md exists" },
  { content: "[P3] Check prerequisites (GCP, services)", status: "pending", activeForm: "Checking prerequisites" },
  { content: "[P3] Parse feature spec", status: "pending", activeForm: "Parsing feature spec" },
  { content: "[P3] Determine test file path", status: "pending", activeForm: "Determining test file location" },
  { content: "[P3] Generate test file", status: "pending", activeForm: "Generating test file from spec" },
  { content: "[P3] Run tests (should FAIL - TDD RED)", status: "pending", activeForm: "Running tests (RED phase)" },
  { content: "[P3] Verify test failures expected", status: "pending", activeForm: "Verifying expected failures" },
  { content: "[P3] Run stress testing", status: "pending", activeForm: "Running stress tests" },
  { content: "[P3] Add edge case tests", status: "pending", activeForm: "Adding edge case tests" },
  { content: "[P3] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

**Rules (from official docs):**
- Initialize ALL todos upfront before starting work
- Mark `in_progress` BEFORE beginning each step
- Only ONE todo `in_progress` at a time
- Mark `completed` immediately when done

---

## Phase Gate Check (REQUIRED - FAIL FAST)

**CRITICAL: Verify Phase 2 (DESIGN) completed before proceeding.**

```bash
# Detect feature directory from branch name or spec path argument
BRANCH=$(git branch --show-current)
FEATURE_NUM=$(echo "$BRANCH" | grep -oE '^[0-9]+' || echo "")

if [ -n "$FEATURE_NUM" ]; then
  SPECS_DIR=$(find specs -maxdepth 1 -type d -name "${FEATURE_NUM}-*" 2>/dev/null | head -1)
fi

# Check for spec.md (from Phase 1)
SPEC_FILE=""
if [ -n "$SPECS_DIR" ]; then
  SPEC_FILE=$(find "$SPECS_DIR" -name "spec.md" -o -name "*-spec.md" 2>/dev/null | head -1)
fi

if [ -z "$SPEC_FILE" ] || [ ! -f "$SPEC_FILE" ]; then
  echo "❌ PHASE GATE FAILED: spec.md not found"
  echo ""
  echo "┌─────────────────────────────────────────────────────────────┐"
  echo "│ SDLC Phase Order Violation                                  │"
  echo "├─────────────────────────────────────────────────────────────┤"
  echo "│ Current Phase:  3-TEST                                      │"
  echo "│ Required Phase: 1-SPECIFY (not completed)                   │"
  echo "│ Missing:        specs/{feature}/spec.md                     │"
  echo "└─────────────────────────────────────────────────────────────┘"
  echo ""
  echo "Run this command first:"
  echo "  /sdlc:1-specify:run <feature description>"
  echo ""
  exit 1
fi

# Check for plan.md (from Phase 2)
PLAN_FILE=""
if [ -n "$SPECS_DIR" ]; then
  PLAN_FILE=$(find "$SPECS_DIR" -name "plan.md" -o -name "*-plan.md" 2>/dev/null | head -1)
fi

if [ -z "$PLAN_FILE" ] || [ ! -f "$PLAN_FILE" ]; then
  echo "❌ PHASE GATE FAILED: plan.md not found"
  echo ""
  echo "┌─────────────────────────────────────────────────────────────┐"
  echo "│ SDLC Phase Order Violation                                  │"
  echo "├─────────────────────────────────────────────────────────────┤"
  echo "│ Current Phase:  3-TEST                                      │"
  echo "│ Required Phase: 2-DESIGN (not completed)                    │"
  echo "│ Missing:        specs/{feature}/plan.md                     │"
  echo "└─────────────────────────────────────────────────────────────┘"
  echo ""
  echo "Run this command first:"
  echo "  /sdlc:2-design:run"
  echo ""
  echo "Why: Tests are generated from the design plan."
  echo "     Without a plan, we cannot determine acceptance criteria."
  exit 1
fi

echo "✅ Phase Gate: spec.md found at $SPEC_FILE"
echo "✅ Phase Gate: plan.md found at $PLAN_FILE"
```

**If gate fails → STOP. Do NOT proceed. User must complete prior phases first.**

---

## Arguments

- `$1`: Feature spec path (e.g., `specs/240-favicon-update/spec.md`)
- `$2`: Environment (default: `dev-local`, options: `dev-local`, `staging-preview`, `staging`)

---

## PREREQUISITES (Auto-Check and Start)

**Execute these checks BEFORE running tests. Fix issues automatically.**

### Step 1: Check GCP Authentication

```bash
gcloud auth application-default print-access-token &>/dev/null && echo "GCP_AUTH_OK" || echo "GCP_AUTH_NEEDED"
```

**If GCP_AUTH_NEEDED → STOP and tell user:**
```
❌ GCP authentication required.
Run: gcloud auth application-default login
Then retry this command.
```

### Step 2: Check Services Running

```bash
curl -s http://localhost:3000/health && echo "API_READY" || echo "API_NOT_RUNNING"
curl -s http://localhost:5173 &>/dev/null && echo "DASHBOARD_READY" || echo "DASHBOARD_NOT_RUNNING"
```

**If API_NOT_RUNNING or DASHBOARD_NOT_RUNNING → Start services:**
```bash
make dev &
sleep 20
# Re-check until ready
curl -s http://localhost:3000/health && echo "API_READY" || echo "API_STILL_NOT_RUNNING"
```

### Step 3: Verify Both Ready

Only proceed when BOTH are ready:
- ✅ API: http://localhost:3000/health returns OK
- ✅ Dashboard: http://localhost:5173 responds

### Test Commands Reference

| Command | Purpose |
|---------|---------|
| `make dev` | Start all local services |
| `make test-smoke` | Run smoke tests |
| `make test` | Run all tests |
| `make test-smoke ARGS="--grep Feature"` | Run specific tests |

---

## PHASE 0: Generate Tests from Spec (TDD - Write Tests First)

**This is the TDD workflow: Tests are written BEFORE implementation.**

### Step 1: Parse the spec file

Read the feature spec to extract:
- Feature name
- Acceptance criteria (AC)
- User scenarios
- Functional requirements

### Step 2: Determine test file path

```
SPEC_PATH = $1
FEATURE_NAME = extract from spec (e.g., "favicon-update" from "specs/240-favicon-update/spec.md")
TEST_FILE = apps/dashboard/e2e/ui/smoke/{feature-name}.spec.ts
```

### Step 3: Check if test file exists

```bash
if [ -f "$TEST_FILE" ]; then
  echo "TEST_EXISTS"
else
  echo "TEST_MISSING"
fi
```

### Step 4: If TEST_MISSING → Generate test file

**Create test file based on spec's acceptance criteria:**

```typescript
/**
 * {Feature Name} E2E Tests
 *
 * Feature: {Feature Name} (GitHub Issue #{issue})
 * Spec: {spec_path}
 *
 * Acceptance Criteria:
 * - AC1: {from spec}
 * - AC2: {from spec}
 * - AC3: {from spec}
 */

import { test, expect } from '../../fixtures/test-with-mocks'

test.describe('{Feature Name} - Visual Verification', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/')
    await page.waitForLoadState('domcontentloaded')

    // Login redirect guard (mandatory)
    if (page.url().includes('/login')) {
      console.log('Redirected to login - skipping test (auth state issue)')
      return
    }
  })

  // Generate one test per acceptance criterion
  test('AC1: {acceptance criterion from spec}', async ({ page }) => {
    // Test implementation based on spec requirement
  })

  test('AC2: {acceptance criterion from spec}', async ({ page }) => {
    // Test implementation based on spec requirement
  })
})
```

### Step 5: Report test file created

```
TEST FILE: {path}
ACCEPTANCE CRITERIA COVERED: {count}
STATUS: Ready to run (tests should FAIL - no implementation yet)
```

---

## PHASE 1: Run Tests (Should FAIL)

**TDD: Tests must FAIL first because implementation doesn't exist yet.**

### 1a. Check servers running

```bash
curl -s http://localhost:3000/health && echo "API_READY" || echo "API_NOT_RUNNING"
curl -s http://localhost:5173 &>/dev/null && echo "DASHBOARD_READY" || echo "DASHBOARD_NOT_RUNNING"
```

### 1b. Run the test

```bash
# Run specific test file
make test-smoke ARGS="--grep {feature_name}"

# Or run all smoke tests
make test-smoke
```

### 1c. Expected Result

- **Tests FAIL** → Correct! Proceed to IMPLEMENT phase
- **Tests PASS** → Either implementation already exists, or tests are too weak

### 1d. Report

```
═══════════════════════════════════════════════════════
TEST PHASE COMPLETE: {Feature Name}
═══════════════════════════════════════════════════════

Spec: $1
Test: {test_file}

TEST GENERATION:
├─ Acceptance Criteria: {count}
├─ Tests Generated: {count}
└─ Test File: {path}

TEST EXECUTION:
├─ Status: FAILED (expected - TDD Red phase)
├─ Failures: {count}
└─ Ready for: IMPLEMENT phase

NEXT STEP: /sdlc:4-implement:run
  → Write minimum code to make tests PASS (TDD Green phase)
═══════════════════════════════════════════════════════
```

---

## PHASE 2: Verification (After Implementation)

**Only run this phase AFTER implementation is complete.**

# Verify Feature (Orchestrator with Checkpoints)

You are the **ORCHESTRATOR**. You execute phases directly, validate subagent work, apply fixes, and **ITERATE UNTIL COMPLETION**.

**CRITICAL: DO NOT STOP until verdict == PRODUCTION-REAL. Keep iterating.**

## Zero Bugs Methodology

This workflow implements mathematically grounded approaches to software correctness:

### Principles (from Zero Bugs Research)

1. **Specs are immutable truth** - Correctness is defined by specification. If UI doesn't match spec → fix UI, not spec.
2. **Testing shows presence, not absence** - Dijkstra's principle. We use multi-layer verification to increase confidence.
3. **Mutation-aware testing** - Flag tests that might miss defects (blind passes, weak assertions).
4. **Multi-environment progression** - Verify locally first, then escalate: `dev-local → staging-preview → staging → production`.
5. **Correctness by construction** - Spec → Test → Code (in that order, always).

### Environment Progression

| Environment | When | Command | What's Validated |
|-------------|------|---------|------------------|
| **dev-local** | First | `make test-smoke` | Logic, UI, auth flow |
| **staging-preview** | After local passes | `gh workflow run e2e-dashboard-*.yml` | CI compatibility, preview deployment |
| **staging** | After PR merge | Auto-triggered | Real API, production-like |
| **production** | After release | Smoke tests only | Live verification |

**Rule: Never skip environments. Fix failures at each level before progressing.**

## Architecture

```
ORCHESTRATOR (this slash command - main Claude)
├── PHASE 1: Environment Setup (YOU do this directly)
├── PHASE 2: Verification (spawn subagent - ANALYZE ONLY)
├── PHASE 3: Apply Fixes (YOU fix issues from report)
├── PHASE 4: Re-verify (spawn subagent again)
└── REPEAT until PRODUCTION-REAL or max iterations
```

**KEY PRINCIPLE: Subagent reports. You fix. You re-verify. You don't stop.**

---

## PHASE 1: Environment Setup (DO THIS DIRECTLY)

**You (main Claude) execute this phase. Do NOT delegate.**

### 1a. Check servers running

```bash
curl -s http://localhost:3000/health && echo "API_READY" || echo "API_NOT_RUNNING"
curl -s http://localhost:5173 &>/dev/null && echo "DASHBOARD_READY" || echo "DASHBOARD_NOT_RUNNING"
```

### 1b. If no servers → start services

```bash
make stop 2>/dev/null || true
sleep 2
make dev &
sleep 15
```

### 1c. Check auth state

```bash
AUTH_FILE="apps/dashboard/e2e/.auth/storage-state.json"
if [ -f "$AUTH_FILE" ]; then
  AGE_MINUTES=$(( ($(date +%s) - $(stat -f %m "$AUTH_FILE")) / 60 ))
  echo "Auth: $AGE_MINUTES min old"
  [ $AGE_MINUTES -gt 55 ] && echo "AUTH_EXPIRED" || echo "AUTH_VALID"
else
  echo "AUTH_MISSING"
fi
```

**NOTE:** Auth is handled automatically by `global-setup.ts` via `/auth/dev-token` endpoint.
No interactive OAuth needed - tests auto-authenticate using configured TEST_USER.

### 1d. OAuth Flow Testing (SPECIAL CASE)

**When testing OAuth/authentication flows specifically:**

1. **Clear browser state for clean testing**:
   - Use Playwright MCP with fresh browser context
   - Clear all cookies and localStorage before OAuth tests
   - Example Playwright command:
     ```typescript
     await context.clearCookies();
     await page.evaluate(() => localStorage.clear());
     ```

2. **Watch for cached credentials**:
   - If redirect goes straight to dashboard → cached auth, not real OAuth
   - Must see GitHub login page for true OAuth flow testing
   - Check browser DevTools → Application → Cookies

3. **Use Playwright MCP for browser automation**:
   ```
   mcp__playwright__browser_navigate to login page
   mcp__playwright__browser_snapshot for state verification
   mcp__playwright__browser_click for interactions
   ```

4. **Run servers in background**:
   ```bash
   make dev &
   sleep 15
   ```
   Then use Playwright MCP for browser testing.

### 1e. CHECKPOINT: Environment Ready (FAIL FAST)

**STOP AND FIX if ANY check fails:**

```
□ API responding (localhost:3000/health)
  → If not: Run `make dev` and wait 15 seconds

□ Dashboard running (localhost:5173)
  → If not: Check if `make dev` started both services

□ Auth state exists (storage-state.json)
  → If not: global-setup will auto-authenticate via dev-token
  → If dev-token fails: Check TEST_USER_GITHUB_ID in .env
```

**DO NOT PROCEED TO PHASE 2 if environment check fails. Fix first.**

---

## PHASE 2: Verification (SPAWN SUBAGENT)

**Subagent ANALYZES and REPORTS only. It does NOT fix issues.**

```
Task(
  subagent_type: "e2e-test-verifier",
  prompt: "Verify E2E test against feature spec.

           Test file: $1
           Feature spec: $2

           YOU MUST:
           - Analyze the test and spec
           - Run the test headed
           - Report ALL issues found
           - DO NOT fix anything - just report

           REQUIRED OUTPUT FORMAT:
           [structured report with ENVIRONMENT, SPEC_VALIDATION, HYGIENE_CHECK,
            TEST_EXECUTION, AC_COVERAGE, ISSUES_FOUND, VERDICT]"
)
```

---

## PHASE 3: Apply Fixes (YOU DO THIS - DO NOT SKIP)

**When subagent reports issues, YOU apply ALL fixes immediately.**

```
FOR each issue in ISSUES_FOUND:
    IF issue.type == "MISSING_AC":
        → **WRITE NEW TEST** using suggested_fix as template
        → This is NOT optional - you must add the test code
    ELIF issue.type == "WEAK_ASSERTION":
        → Replace weak assertion with strong one
    ELIF issue.type == "BLIND_PASS":
        → Add real data verification
    ELIF issue.type == "DATA_MISMATCH":
        → Add API call to fetch real data
        → Compare UI values against API response
        → CRITICAL: Must verify UI matches API, not just visibility
    ELIF issue.type == "SELECTOR_BUG":
        → Fix selector (exact: true, .first(), specific class)
    ELIF issue.type == "SPEC_GAP":
        → Update spec to include API validation requirement
        → Then update test to validate API vs UI
    ELIF issue.type == "HYGIENE":
        → Fix test structure
    ELIF issue.type == "SKIPPED_TEST":
        → Note for later (can't fix if UI not implemented)

    Edit(
      file_path: issue.location.split(':')[0],
      old_string: issue.current_code,
      new_string: issue.suggested_fix
    )
```

**IMPORTANT: Apply as many fixes as possible. Only skip if:**
- UI feature not implemented (skipped tests)
- Fix would break other tests
- Suggested fix is incomplete

**For UI/SPEC mismatches:**
- **Spec is ALWAYS correct** - specs are immutable source of truth
- If UI doesn't match spec → **FIX THE UI**, never the spec
- Only exception: SPEC_GAP issues (adding missing data accuracy requirements)

---

## PHASE 4: Iteration Loop (CONTINUE UNTIL DONE)

**NO MAXIMUM ITERATIONS. Continue until PRODUCTION-REAL or blocked.**

```
iteration = 0

WHILE verdict != PRODUCTION-REAL:
    iteration++

    IF iteration > 1:
        LOG "Iteration {iteration}: Re-verifying after fixes..."

    result = spawn_subagent()  // Subagent analyzes and reports

    IF result.verdict == "PRODUCTION-REAL" AND result.issue_count == 0:
        → OUTPUT final_report(iteration)
        → EXIT SUCCESS

    ELIF result.issue_count > 0:
        // Apply ALL fixes
        fixes_applied = 0
        FOR each issue in result.ISSUES_FOUND:
            IF can_fix(issue):
                apply_fix(issue)
                fixes_applied++

        IF fixes_applied == 0:
            → OUTPUT blocked_report(result, "No fixable issues remaining")
            → EXIT BLOCKED

        // Continue to next iteration
        CONTINUE

    ELIF checkpoint_1_fails (spec incomplete):
        → OUTPUT blocked_report("Feature spec incomplete")
        → EXIT BLOCKED

    ELIF test_failed AND no_issues_reported:
        → Investigate failure manually
        → Apply fix
        → CONTINUE

// Should never reach here
OUTPUT error_report("Unexpected state")
```

---

## Exit Conditions

| Condition | Action |
|-----------|--------|
| `verdict == PRODUCTION-REAL && issue_count == 0` | EXIT SUCCESS |
| `spec incomplete` | EXIT BLOCKED - user must fix spec |
| `UI not implemented` (skipped tests) | EXIT PARTIAL - note what's missing |
| `fixes_applied == 0 && issues > 0` | EXIT BLOCKED - can't proceed |

---

## Final Output Format

```
═══════════════════════════════════════════════════════
VERIFICATION COMPLETE: [Feature Name]
═══════════════════════════════════════════════════════

Test: $1
Spec: $2

ITERATIONS: N
├─ Iteration 1: [issues found] → [fixes applied]
├─ Iteration 2: [issues found] → [fixes applied]
└─ Iteration N: PRODUCTION-REAL ✓

FIXES APPLIED: N total
├─ MISSING_AC: N
├─ WEAK_ASSERTION: N
├─ BLIND_PASS: N
├─ DATA_MISMATCH: N
├─ SPEC_GAP: N
└─ HYGIENE: N

FINAL CHECKPOINTS:
├─ [✓] Spec complete
├─ [✓] Hygiene clean
├─ [✓] Test executed
├─ [✓] Test passed
├─ [✓] AC coverage 100%
└─ [✓] Verdict: PRODUCTION-REAL

RESULT: ✓ SUCCESS (N iterations, M fixes)
═══════════════════════════════════════════════════════
```

---

## Summary of Responsibilities

| Actor | Does | Does NOT |
|-------|------|----------|
| **Subagent** | Analyze, detect issues, report | Fix code, make decisions, stop workflow |
| **Orchestrator (you)** | Apply fixes, validate, iterate until done | Stop prematurely, ask user for permission |

---

## Mutation-Aware Testing

**Would this test catch a defect if we introduced one?**

When analyzing tests, ask yourself: "If I changed the code to be wrong, would this test fail?"

### Red Flags (Tests that might miss defects)

| Pattern | Problem | Fix |
|---------|---------|-----|
| `expect(element).toBeVisible()` | Won't catch wrong text | Add `.toContainText()` |
| `expect(count).toBeDefined()` | Won't catch zero values | Use `.toBeGreaterThan(0)` |
| `expect(list.length).toBe(N)` | Hardcoded, won't catch API changes | Compare to API response |
| OR-logic assertions | Passes if either works | Test each condition separately |
| `waitForTimeout(N)` | Hides race conditions | Use explicit waits |

### Mutation Examples

**Would this test catch each mutation?**

| Mutation | Test Should | If Test Passes Anyway |
|----------|-------------|----------------------|
| Return empty array from API | Fail with count check | Test is BLIND_PASS |
| Return wrong user name | Fail with text check | Test is WEAK |
| Return 404 from endpoint | Fail with error handling | Test is INCOMPLETE |
| Remove required field | Fail with field assertion | Test is STRONG |

**Rule: A test that doesn't fail when the code is wrong provides false confidence.**

## Example Usage

```bash
# TDD: Generate tests from spec, run them (should FAIL)
/sdlc:3-test:run specs/240-favicon-update/spec.md

# With explicit environment
/sdlc:3-test:run specs/240-favicon-update/spec.md staging

# For existing feature specs
/sdlc:3-test:run docs/features/gal/convenience/auto-discovery.md
```

**TDD Flow:**
1. Run `/sdlc:3-test:run {spec}` → Generates tests, runs them (FAIL expected)
2. Run `/sdlc:4-implement:run` → Write code to make tests PASS
3. Tests now PASS → Feature complete

---

## MANDATORY: Capture Learnings (AUTO-EXECUTE)

**DO NOT end without invoking:**

```
Skill(skill: "capture-learnings")
```

This step is REQUIRED when ANY of these occurred:
- User had to manually correct agent behavior
- Workflow improvements were discovered
- New patterns/anti-patterns were identified
- `.claude/` files were modified during the session

This captures:
- Process deviations that occurred
- Manual interventions from user
- Improvements to agentic layer

**Update TodoWrite when complete:**
```javascript
TodoWrite([
  // ... previous steps as completed ...
  { content: "[P3] Capture learnings", status: "completed", activeForm: "Learnings captured" }
])
```

**When to invoke:**
1. After committing feature changes
2. Before ending SDLC phase
3. When user provides workflow corrections
