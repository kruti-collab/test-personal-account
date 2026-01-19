---
allowed-tools: Task, Read, Glob, Grep, Bash, TodoWrite
argument-hint: [test-file-path] [--strict] [--layer GAL|TAL|SAL]
description: Verify E2E tests actually test functionality and aren't workarounds
---

# E2E Test Verification

Comprehensive verification that E2E tests actually validate functionality rather than just checking page renders or using workarounds. This command prevents "false green" test suites where tests pass but don't verify real behavior.

## Purpose

Prevent scenarios where:
- Tests only check if page loads (no functionality verification)
- Tests use `.toBeVisible()` without verifying content
- Tests mock actual API calls instead of testing integration
- Tests skip assertions with `.skip` or empty blocks
- Tests use hardcoded values instead of dynamic verification
- Tests pass because they test nothing

## Variables

- **TEST_FILE_PATH**: Path to test file or directory to verify (optional)
  - If omitted, verifies all E2E tests in project
  - Supports glob patterns: `e2e/**/*.spec.ts`

- **STRICT**: `--strict` flag for 100% coverage requirement (optional)
  - Default: 80% minimum coverage
  - Strict: 100% coverage for core features

- **LAYER**: `--layer GAL|TAL|SAL` to filter by product layer (optional)
  - GAL: Governance layer tests
  - TAL: Triggering layer tests
  - SAL: Sandboxed layer tests

## Usage

```bash
/verify-e2e-coverage                                    # Verify all E2E tests
/verify-e2e-coverage e2e/sprint-features.spec.ts       # Verify specific file
/verify-e2e-coverage --strict                           # Require 100% coverage
/verify-e2e-coverage --layer GAL                        # Only GAL tests
/verify-e2e-coverage e2e/ --strict --layer TAL         # Strict TAL verification
```

## Verification Phases

### Phase 1: Test Discovery & Classification

Spawn the `e2e-test-verifier` agent to:

1. **Discover all test files**
   ```bash
   find . -name "*.spec.ts" -o -name "*.test.ts" | grep -E "(e2e|integration)"
   ```

2. **Classify tests by layer**
   - Map test file paths to GAL/TAL/SAL layers
   - Identify which features each test covers
   - Cross-reference with layer-architecture.md

3. **Identify core vs substitution features**
   - Core features require 100% coverage
   - Substitution features require 80% coverage

### Phase 2: Anti-Pattern Detection

Analyze each test for workaround patterns:

#### 2.1 Empty Test Detection
```typescript
// BAD: Empty test body
test('should work', async () => {
  // TODO: implement
})

// BAD: No assertions
test('renders page', async ({ page }) => {
  await page.goto('/workflow')
})
```

#### 2.2 Weak Assertion Detection
```typescript
// BAD: Only checks page exists
await expect(page).toHaveURL(/workflow/)

// BAD: Only checks something is visible, not what
await expect(page.locator('div')).toBeVisible()

// GOOD: Checks specific content
await expect(page.getByRole('heading', { name: 'TAL Workflow Engine' })).toBeVisible()
await expect(page.getByText('gal trigger watch')).toBeVisible()
```

#### 2.3 Mock Abuse Detection
```typescript
// BAD: Mocking the thing being tested
jest.mock('../services/workflow', () => ({
  getWorkflow: () => ({ status: 'active' })
}))

// BAD: Skipping actual API calls
test.skip('calls real API', ...)

// GOOD: Using test fixtures, not mocks
await page.route('**/api/status', route => route.fulfill({ body: testFixture }))
```

#### 2.4 Hardcoded Value Detection
```typescript
// BAD: Checking hardcoded values
expect(result).toBe('expected')  // Where does 'expected' come from?

// GOOD: Checking computed/dynamic values
expect(result).toMatch(/GAL-\d+/)
expect(response.status).toBe(200)
```

#### 2.5 Skipped Test Detection
```typescript
// FLAG: Skipped tests
test.skip('important feature', ...)
test.todo('implement later', ...)
describe.skip('entire suite', ...)
```

### Phase 3: Coverage Analysis

For each feature defined in `layer-architecture.md`:

1. **Core Feature Coverage (100% required)**
   - Direct access test exists
   - Content verification test exists
   - Functionality test exists
   - CLI parity test exists (if applicable)
   - Integration test exists

2. **Substitution Feature Coverage (80% required)**
   - Direct access test exists
   - Content verification test exists
   - Basic functionality test exists

### Phase 4: Test Quality Scoring

Score each test on a 0-100 scale:

| Criteria | Points |
|----------|--------|
| Has meaningful assertions | 20 |
| Tests actual functionality | 25 |
| No empty blocks | 10 |
| No skipped tests | 10 |
| Tests error scenarios | 15 |
| Tests edge cases | 10 |
| CLI parity (if applicable) | 10 |

**Minimum passing score**:
- Core features: 80/100
- Substitution features: 60/100

### Phase 5: Report Generation

Generate comprehensive verification report:

```markdown
## E2E Test Verification Report

### Summary
- Total Tests: XX
- Verified: XX (XX%)
- Anti-Patterns Found: XX
- Coverage Score: XX%

### Layer Coverage

#### GAL (Governance Layer)
| Feature | Type | Coverage | Score | Status |
|---------|------|----------|-------|--------|
| Policy Enforcement | Core | 100% | 85/100 | PASS |
| Security Scanning | Core | 100% | 90/100 | PASS |
| Auto-Fix Generation | Sub | 80% | 65/100 | PASS |

#### TAL (Triggering Layer)
| Feature | Type | Coverage | Score | Status |
|---------|------|----------|-------|--------|
| Workflow Engine | Core | 100% | 88/100 | PASS |
| Time Tracking | Sub | 75% | 55/100 | WARN |

#### SAL (Sandboxed Layer)
| Feature | Type | Coverage | Score | Status |
|---------|------|----------|-------|--------|
| Sandbox Execution | Core | 100% | 92/100 | PASS |
| Maintenance | Sub | 85% | 70/100 | PASS |

### Anti-Patterns Detected

#### Empty Tests (X found)
- `e2e/workflow.spec.ts:45` - Empty test body
- `e2e/quality.spec.ts:78` - No assertions

#### Weak Assertions (X found)
- `e2e/tests.spec.ts:23` - Only checks URL, not content
- `e2e/time.spec.ts:56` - Generic `toBeVisible()` without context

#### Skipped Tests (X found)
- `e2e/protection.spec.ts:12` - `test.skip('security rules')`

### Recommendations

1. **Critical**: Add assertions to empty tests
2. **High**: Replace weak assertions with specific content checks
3. **Medium**: Unskip or remove skipped tests
4. **Low**: Add error scenario tests

### Verification Result: [PASS/FAIL]
```

## Agent Integration

This command spawns the `e2e-test-verifier` agent:

```markdown
**Agent Task:**
"Verify E2E test coverage for $TEST_FILE_PATH.

Cross-reference tests against layer-architecture.md to:
1. Identify which layer/feature each test covers
2. Check if test is core or substitution feature
3. Verify coverage meets requirements (100% core, 80% sub)
4. Detect anti-patterns: empty tests, weak assertions, mocks, skips
5. Generate quality score for each test
6. Produce comprehensive verification report

Flag any tests that appear to 'pass' without actually verifying functionality.
STRICT mode: $STRICT
LAYER filter: $LAYER"
```

## Success Criteria

### PASS
- All core features have 100% test coverage
- All substitution features have 80%+ coverage
- No empty or skipped tests in core features
- Test quality score >= 80 for core, >= 60 for substitution
- No critical anti-patterns detected

### FAIL
- Any core feature below 100% coverage
- Multiple substitution features below 80%
- Empty tests found in core features
- Skipped tests without justification
- Critical anti-patterns detected

## Direct Execution Instructions

1. **Parse arguments** from $ARGUMENTS
2. **Read features doc** from `docs/features.md` (Feature Classification section)
3. **Discover test files** matching pattern or all E2E tests
4. **Spawn e2e-test-verifier agent** for deep analysis
5. **Analyze each test** for anti-patterns
6. **Calculate coverage** against layer requirements
7. **Score test quality** on 0-100 scale
8. **Generate report** with PASS/FAIL verdict
9. **Output recommendations** for improvements

## Example Output

```
E2E Test Verification Complete

Layer Coverage:
- GAL: 95% (7/7 core, 4/5 substitution)
- TAL: 100% (5/5 core, 5/5 substitution)
- SAL: 88% (6/6 core, 3/5 substitution)

Anti-Patterns:
- 0 empty tests
- 2 weak assertions (non-blocking)
- 1 skipped test (substitution feature, allowed)

Quality Score: 84/100

Result: PASS - E2E tests verify actual functionality
```

This command ensures E2E tests provide real value and catch actual bugs, not just pass because they test nothing.
