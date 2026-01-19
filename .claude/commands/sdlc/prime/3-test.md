# Prime TEST - Load Testing Context

Prime context window with everything needed for E2E and unit testing work.

## Step 1: Read Testing Documentation

Read these files to understand testing strategy:

1. Read `docs/sdlc/3-test/README.md`
2. Read `docs/sdlc/3-test/operations/TESTING.md` (if exists)
3. Read `.claude/rules/e2e-testing.md`
4. Read `.claude/rules/tdd.md`

## Step 2: Read Test Configuration

Read Playwright and test setup files:

1. Read `apps/dashboard/playwright.config.ts`
2. Read `apps/dashboard/e2e/setup/global-setup.ts`
3. Read `apps/dashboard/e2e/SKIPPED_TESTS.md` (if exists)
4. Read `scripts/test.sh`

## Step 3: Scan Test Structure

```bash
# E2E test structure
ls -la apps/dashboard/e2e/
ls -la apps/dashboard/e2e/ui/smoke/ 2>/dev/null
ls -la apps/dashboard/e2e/ui/features/ 2>/dev/null

# Unit test structure
ls -la apps/api/tests/
ls -la packages/core/tests/ 2>/dev/null

# Auth state
ls -la apps/dashboard/e2e/.auth/ 2>/dev/null
```

## Step 4: Read Example Tests

Read 2-3 existing tests to understand patterns:

1. Read one smoke test from `apps/dashboard/e2e/ui/smoke/`
2. Read one feature test from `apps/dashboard/e2e/ui/features/`
3. Read one API test from `apps/api/tests/`

## Step 5: Check Test Environment

```bash
# Check if servers are running
curl -s http://localhost:3000/health 2>/dev/null && echo "API: ✓ Running" || echo "API: ✗ Not running"
curl -s http://localhost:5173 2>/dev/null && echo "Dashboard: ✓ Running" || echo "Dashboard: ✗ Not running"

# Auth state age
AUTH_FILE="apps/dashboard/e2e/.auth/storage-state.json"
if [ -f "$AUTH_FILE" ]; then
  AGE_MINUTES=$(( ($(date +%s) - $(stat -f %m "$AUTH_FILE" 2>/dev/null || stat -c %Y "$AUTH_FILE" 2>/dev/null)) / 60 ))
  echo "Auth state: ${AGE_MINUTES}min old"
else
  echo "Auth state: Missing (will be created on test run)"
fi

# Current branch
git branch --show-current
```

## Testing Patterns (Memorize These)

### Mandatory Login Guard

```typescript
await page.goto('/path')
if (page.url().includes('/login')) {
  console.log('Auth issue - skipping')
  return
}
```

### Correct Locators (Playwright)

```typescript
// ✅ Good - filter, not :has-text()
page.locator('h1').filter({ hasText: 'Title' })

// ✅ Good - role-based
page.getByRole('button', { name: 'Submit' })

// ✅ Good - test IDs
page.getByTestId('submit-button')

// ❌ Bad - fragile CSS
page.locator('.btn-primary.mt-4')
```

### Proper Waits

```typescript
// ✅ Good - wait for URL
await page.waitForURL('**/dashboard**')

// ✅ Good - auto-retrying assertion
await expect(element).toBeVisible()

// ❌ Bad - hardcoded timeout
await page.waitForTimeout(3000)
```

### Test Commands

```bash
# Local testing
./scripts/test.sh smoke dev-local
./scripts/test.sh features dev-local --headed

# Single test file
pnpm --filter dashboard test:e2e apps/dashboard/e2e/ui/smoke/navigation.spec.ts

# CI testing
gh workflow run e2e-dashboard-smoke.yml --ref $(git branch --show-current)
```

### Flaky Test Debugging

1. Run headed: `--headed`
2. Add slowMo: `--slow-mo 500`
3. Check auth state freshness
4. Look for race conditions
5. Add explicit waits for network

## CRITICAL RULES (MUST FOLLOW)

### CI Check Protocol
**NEVER approve or merge a PR until ALL CI checks pass.** Don't run tests locally while CI runs - wait for CI. Don't use `--admin` to bypass failures - fix them first.

### Fix Failures On The Spot
When discovering a bug or failure during SDLC workflow: fix immediately while having context. Don't skip past failures. Never use `--admin` bypass.

### GitHub Issue Linkage
**ALWAYS use "Closes #X" (not "Relates to") in PR descriptions for auto-close.**

## After Priming

You are now ready to run:
- `/sdlc/test/run [test-file] [spec-file]` - Verify E2E test
- `/sdlc/test/helpers/test-debug` - Debug failing test
- `/sdlc/test/helpers/feature-generate-tests` - Generate tests from spec

## Report After Priming

Summarize:
1. Current branch and test-related changes
2. Test environment status (API, Dashboard, Auth)
3. Number of skipped tests and reasons
4. Ready to proceed with testing work

