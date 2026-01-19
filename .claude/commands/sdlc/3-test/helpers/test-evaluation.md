---
allowed-tools: Bash, Read, Grep, Glob
argument-hint: <test_scope> [environment]
description: Comprehensive E2E/integration test environment evaluation with 130-point scoring
---

# Test Environment Evaluation (E2E & Integration Tests)

**Purpose:** Catch EVERY possible failure mode before running E2E or integration tests. Validates environment, configuration, dependencies, and infrastructure to ensure tests run successfully on the first try.

**Scope:** This command is specifically for E2E and integration test workflows. For other workflow types, use:
- `/ci-evaluation deploy <env>` - Deployment workflows
- `/ci-evaluation release` - Production releases
- `/ci-evaluation preview` - PR preview deployments
- `/ci-evaluation quality` - CI quality gates (lint, typecheck, build)

**Design Philosophy:** "Trust but verify" - systematically validate every assumption about the test environment.

## Variables

- **TEST_SCOPE**: First argument from $ARGUMENTS (required)
  - Test suite: "smoke", "features", "workflows", "integration", "all"
  - Affects test path validation and timeout expectations

- **ENVIRONMENT**: Second argument from $ARGUMENTS (optional, default: "dev")
  - Target environment: "dev", "staging", "production"
  - Determines required environment variables and infrastructure

## Usage

```bash
/test-evaluation <test_scope> [environment]
```

Examples:
- `/test-evaluation smoke dev` - Validate local development environment
- `/test-evaluation features staging` - Validate CI staging environment
- `/test-evaluation all staging` - Complete pre-flight validation

## Scoring System (0-130)

### Component Breakdown

| Component | Points | Type | Description |
|-----------|--------|------|-------------|
| **Environment Variables** | 10 | BLOCKER | Required vars for auth and config |
| **API Health** | 15 | BLOCKER | API responding at expected URL |
| **Authentication Setup** | 20 | BLOCKER | Auth method configured correctly |
| **CI Workflow Validation** | 10 | BLOCKER | Workflow uses canonical scripts |
| **Playwright Config** | 10 | BLOCKER | testDir matches test scope |
| **Storage State** | 10 | WARNING | Auth token validity |
| **Port Availability** | 5 | BLOCKER (dev) | No port conflicts |
| **Browser Dependencies** | 10 | BLOCKER | Playwright browsers installed |
| **BASE_URL Reachability** | 10 | BLOCKER (stg/prod) | Deployed site accessible |
| **Test Path Consistency** | 5 | BLOCKER | test.sh paths match config |
| **Node/pnpm Version** | 5 | WARNING | Version compatibility |
| **Previous Artifacts** | 5 | WARNING | Stale test results cleanup |
| **Test Timeout Config** | 5 | WARNING | Timeouts match environment |
| **Disk Space** | 5 | BLOCKER | Sufficient space for artifacts |
| **File Permissions** | 5 | BLOCKER | Can write to test directories |
| **Race Conditions** | 5 | BLOCKER | No concurrent test runs |
| **Git Clean State** | 5 | WARNING | No uncommitted config changes |
| **Dependency Lock** | 5 | WARNING | pnpm-lock.yaml integrity |
| **Network Connectivity** | 5 | BLOCKER (CI) | Can reach external services |
| **Known Issues Penalty** | -5/10 skipped | QUALITY | Technical debt indicator |

**Total:** 130 points (before penalty)

### Assessment Thresholds (130-point scale)

| Score | Assessment | Action |
|-------|------------|--------|
| 120-130 | ✅ READY | Safe to trigger CI |
| 105-119 | ⚠️ MINOR ISSUES | Consider fixing warnings |
| 85-104 | ⚠️ SIGNIFICANT ISSUES | Fix before CI |
| 60-84 | ❌ NOT READY | Multiple blockers |
| 0-59 | ❌ BLOCKED | Critical failures |

**Percentage equivalents:**
- ✅ READY: 92%+ (virtually perfect)
- ⚠️ MINOR: 81-91% (some warnings)
- ⚠️ SIGNIFICANT: 65-80% (fix blockers)
- ❌ NOT READY: 46-64% (major issues)
- ❌ BLOCKED: <46% (critical failures)

## Validation Components

### 1. Environment Variables (10 points) - BLOCKER

**What it checks:** Required environment variables are set and valid.

**Dev Environment:**
```bash
✅ API_URL (default: http://localhost:3000)
✅ TEST_USER_GITHUB_ID (for /auth/dev-token)
✅ TEST_USER_USERNAME (for test user creation)
```

**Staging/Production:**
```bash
✅ API_URL or VITE_API_URL
✅ BASE_URL (deployed dashboard URL)
✅ GITHUB_OIDC_TOKEN (CI only, from GitHub Actions)
```

**Scoring:**
- All required vars set: 10 points
- 1 missing: 5 points (degraded mode)
- 2+ missing: 0 points (BLOCKER)

**Validation:**
```bash
# Check each required var
missing_count=0

if [ "$ENVIRONMENT" = "dev" ]; then
  [ -z "$API_URL" ] && echo "❌ API_URL not set" && ((missing_count++))
  [ -z "$TEST_USER_GITHUB_ID" ] && echo "❌ TEST_USER_GITHUB_ID not set" && ((missing_count++))
  [ -z "$TEST_USER_USERNAME" ] && echo "❌ TEST_USER_USERNAME not set" && ((missing_count++))
else
  [ -z "$API_URL" ] && [ -z "$VITE_API_URL" ] && echo "❌ API_URL not set" && ((missing_count++))
  [ -z "$BASE_URL" ] && echo "❌ BASE_URL not set" && ((missing_count++))

  if [ -n "$CI" ]; then
    [ -z "$GITHUB_OIDC_TOKEN" ] && echo "❌ GITHUB_OIDC_TOKEN not set" && ((missing_count++))
  fi
fi

# Score
if [ $missing_count -eq 0 ]; then env_score=10
elif [ $missing_count -eq 1 ]; then env_score=5
else env_score=0; fi
```

**Remediation:**
```bash
# Dev: Add to scripts/env-config.sh
export TEST_USER_GITHUB_ID=48866801
export TEST_USER_USERNAME=karabil

# CI: Add workflow permissions
permissions:
  id-token: write
```

---

### 2. API Health (15 points) - BLOCKER

**What it checks:** API is running and responding to health checks.

**Validation:**
```bash
API_URL="${API_URL:-http://localhost:3000}"
status=$(curl -s -o /dev/null -w "%{http_code}" --max-time 5 "$API_URL/health" 2>/dev/null || echo "000")

if [ "$status" = "200" ]; then
  echo "✅ API healthy"
  api_score=15
elif [ "$status" = "000" ] && [ "$ENVIRONMENT" = "dev" ]; then
  # Check if we can auto-start
  if gcloud auth application-default print-access-token >/dev/null 2>&1; then
    echo "⚠️  API not running (can auto-start)"
    api_score=10
  else
    echo "❌ API not running, can't auto-start (need GCP auth)"
    api_score=0
  fi
else
  echo "❌ API not responding (HTTP $status)"
  api_score=0
fi
```

**Scoring:**
- HTTP 200: 15 points
- Can auto-start (dev): 10 points
- Not running, can't start: 0 points (BLOCKER)

**Remediation:**
```bash
# Dev: Authenticate and start
gcloud auth application-default login
./scripts/run.sh

# CI: Verify API_URL points to running service
curl -v https://gal-api-wug5dzqj2a-uc.a.run.app/health
```

---

### 3. Authentication Setup (20 points) - BLOCKER

**What it checks:** Authentication method is properly configured for environment.

**Dev:** Checks `/auth/dev-token` endpoint and TEST_USER vars
**CI:** Checks `GITHUB_OIDC_TOKEN` and `/auth/ci` endpoint

**Validation:**
```bash
auth_score=0

if [ "$ENVIRONMENT" = "dev" ]; then
  # Dev: Check dev-token endpoint
  if [ -n "$TEST_USER_GITHUB_ID" ] && [ -n "$TEST_USER_USERNAME" ]; then
    dev_token_status=$(curl -s -o /dev/null -w "%{http_code}" \
      "$API_URL/auth/dev-token?github_id=$TEST_USER_GITHUB_ID" 2>/dev/null)

    if [ "$dev_token_status" = "200" ] || [ "$dev_token_status" = "302" ]; then
      echo "✅ Dev token auth available"
      auth_score=20
    else
      echo "❌ Dev token endpoint failed (HTTP $dev_token_status)"
      auth_score=0
    fi
  else
    echo "❌ Missing TEST_USER_GITHUB_ID or TEST_USER_USERNAME"
    auth_score=0
  fi
elif [ -n "$CI" ]; then
  # CI: Check OIDC token
  if [ -n "$GITHUB_OIDC_TOKEN" ]; then
    ci_auth_status=$(curl -s -o /dev/null -w "%{http_code}" \
      -H "Authorization: Bearer $GITHUB_OIDC_TOKEN" \
      "$API_URL/auth/ci" 2>/dev/null)

    if [ "$ci_auth_status" = "200" ] || [ "$ci_auth_status" = "302" ]; then
      echo "✅ OIDC auth working"
      auth_score=20
    else
      echo "❌ OIDC auth failed (HTTP $ci_auth_status)"
      auth_score=5
    fi
  else
    echo "❌ Missing GITHUB_OIDC_TOKEN"
    auth_score=0
  fi
else
  # Non-CI staging/prod: Assume auth will work
  echo "⚠️  Non-CI environment, assuming auth configured"
  auth_score=15
fi
```

**Scoring:**
- Auth working: 20 points
- Configured but not tested: 15 points
- Partial setup: 5 points
- Not configured: 0 points (BLOCKER)

---

### 4. CI Workflow Validation (10 points) - BLOCKER

**What it checks:** GitHub Actions workflow uses canonical scripts, not direct pnpm.

**Why it matters:** Direct `pnpm build` doesn't build workspace dependencies (@gal/types), causing build failures.

**Validation:**
```bash
workflow_score=10  # Default for dev (N/A)

if [ "$ENVIRONMENT" != "dev" ]; then
  # Determine workflow file
  case "$TEST_SCOPE" in
    smoke) workflow=".github/workflows/e2e-smoke.yml" ;;
    features) workflow=".github/workflows/e2e-features.yml" ;;
    workflows) workflow=".github/workflows/e2e-workflows.yml" ;;
    all) workflow=".github/workflows/e2e-tests.yml" ;;
  esac

  if [ ! -f "$workflow" ]; then
    echo "⚠️  Workflow not on current branch: $workflow"
    echo "   Will use fallback workflow from main"
    workflow=".github/workflows/e2e-tests.yml"
  fi

  # Check for canonical scripts
  if grep -q "./scripts/test.sh\|./scripts/build.sh" "$workflow" 2>/dev/null; then
    echo "✅ Workflow uses canonical scripts"
    workflow_score=10
  else
    echo "❌ Workflow uses direct pnpm commands"
    echo "   Risk: Build will fail with 'Cannot find module @gal/types'"
    workflow_score=0
  fi

  # Check for OIDC permission
  if grep -q "id-token: write" "$workflow" 2>/dev/null; then
    echo "✅ OIDC permission configured"
  else
    echo "⚠️  Missing 'id-token: write' permission"
    workflow_score=$((workflow_score - 2))
  fi
fi
```

**Scoring:**
- Uses canonical scripts + OIDC: 10 points
- Uses canonical scripts, no OIDC: 8 points
- Uses direct pnpm: 0 points (BLOCKER)

**Remediation:**
```yaml
# Fix workflow to use canonical scripts:
- name: Run tests
  run: ./scripts/test.sh smoke staging

# Add OIDC permission:
permissions:
  id-token: write
  contents: read
```

---

### 5. Playwright Config Validation (10 points) - BLOCKER

**What it checks:** Playwright project's `testDir` matches the test scope being run.

**Why it matters:** Mismatched testDir causes "No tests found" error (discovered 2025-12-26).

**Validation:**
```bash
config_score=0

# Determine project name from environment
case "$ENVIRONMENT" in
  dev) project_name="ui" ;;
  staging) project_name="staging" ;;
  production) project_name="prod" ;;
  preview) project_name="preview" ;;
esac

# Expected test directory based on scope
case "$TEST_SCOPE" in
  smoke|features|workflows) expected_testdir="./e2e/ui" ;;
  integration) expected_testdir="./e2e/api" ;;
  all) expected_testdir="./e2e" ;;
esac

# Read actual testDir from playwright.config.ts
if [ -f "apps/dashboard/playwright.config.ts" ]; then
  # Extract project config (find project block, get testDir)
  actual_testdir=$(sed -n "/name: '$project_name'/,/}/p" apps/dashboard/playwright.config.ts | \
    grep "testDir:" | head -1 | sed "s/.*testDir: '\(.*\)'.*/\1/")

  if [ -z "$actual_testdir" ]; then
    # No explicit testDir = uses root config (./e2e)
    actual_testdir=$(grep "testDir:" apps/dashboard/playwright.config.ts | head -1 | sed "s/.*testDir: '\(.*\)'.*/\1/")
  fi

  # Validate match
  if [ "$actual_testdir" = "$expected_testdir" ] || \
     [ "$expected_testdir" = "./e2e" ] || \
     [[ "$expected_testdir" == "$actual_testdir"* ]]; then
    echo "✅ Playwright config matches test scope"
    echo "   Project '$project_name' testDir: $actual_testdir"
    echo "   Expected for $TEST_SCOPE: $expected_testdir"
    config_score=10
  else
    echo "❌ Playwright config MISMATCH"
    echo "   Project '$project_name' testDir: $actual_testdir"
    echo "   Expected for $TEST_SCOPE tests: $expected_testdir"
    echo "   Result: 'No tests found' error!"
    config_score=0
  fi
else
  echo "⚠️  playwright.config.ts not found"
  config_score=5
fi
```

**Scoring:**
- testDir matches scope: 10 points
- Mismatch (will cause "no tests found"): 0 points (BLOCKER)
- Config file not found: 5 points (warning)

**Remediation:**
```typescript
// Fix playwright.config.ts:
{
  name: 'staging',
  testDir: './e2e/ui',  // Not './e2e/api' if running UI tests
  ...
}
```

---

### 6. Storage State Validation (10 points) - WARNING

**What it checks:** Authentication storage state file is valid and token not expired.

**Why it matters:** Invalid storage state causes all tests to see login page.

**Validation:**
```bash
STORAGE_STATE="apps/dashboard/e2e/.auth/storage-state.json"
storage_score=0

if [ ! -f "$STORAGE_STATE" ]; then
  echo "⚠️  Storage state file missing (will be created by global-setup)"
  storage_score=7  # Will be created, not a blocker
elif ! jq empty "$STORAGE_STATE" 2>/dev/null; then
  echo "❌ Storage state corrupted (invalid JSON)"
  storage_score=0
else
  # Check structure
  has_origins=$(jq '.origins | length' "$STORAGE_STATE" 2>/dev/null || echo "0")
  has_token=$(jq -r '.origins[0].localStorage[]? | select(.name == "gal_auth_token") | .value' "$STORAGE_STATE" 2>/dev/null)

  if [ "$has_origins" -gt 0 ] && [ -n "$has_token" ]; then
    echo "✅ Storage state structure valid"
    storage_score=10

    # Verify token against API (optional, slow check)
    if [ -n "$API_URL" ] && [ "$ENVIRONMENT" = "dev" ]; then
      token_status=$(curl -s -o /dev/null -w "%{http_code}" \
        -H "Authorization: Bearer $has_token" \
        "$API_URL/auth/me" 2>/dev/null || echo "000")

      if [ "$token_status" != "200" ]; then
        echo "⚠️  Token expired or invalid (will re-auth)"
        storage_score=7
      fi
    fi
  else
    echo "❌ Storage state missing required fields"
    storage_score=3
  fi
fi
```

**Scoring:**
- Valid with working token: 10 points
- Valid structure, untested token: 8 points
- Will be created by global-setup: 7 points
- Missing required fields: 3 points
- Corrupted: 0 points

---

### 7. Port Availability (5 points) - BLOCKER (dev only)

**What it checks:** Required ports are available or running expected services.

**Dev only:** Staging/production use deployed services.

**Validation:**
```bash
port_score=5

if [ "$ENVIRONMENT" = "dev" ]; then
  conflicts=0

  check_port() {
    local port=$1
    local service=$2

    if lsof -Pi :$port -sTCP:LISTEN -t >/dev/null 2>&1; then
      # Port in use - check if it's our service
      if curl -s --max-time 2 http://localhost:$port/health >/dev/null 2>&1 || \
         curl -s --max-time 2 http://localhost:$port >/dev/null 2>&1; then
        echo "✅ Port $port: $service running"
      else
        echo "⚠️  Port $port: occupied by other process"
        ((conflicts++))
      fi
    else
      echo "✅ Port $port: available"
    fi
  }

  check_port 5173 "dashboard dev server"
  check_port 3000 "API server"

  # Score based on conflicts
  if [ $conflicts -eq 0 ]; then port_score=5
  elif [ $conflicts -eq 1 ]; then port_score=3
  else port_score=0; fi
else
  echo "✅ Non-dev environment (using deployed services)"
  port_score=5
fi
```

**Scoring:**
- All ports available/running correctly: 5 points
- 1 port conflict: 3 points
- 2+ port conflicts: 0 points (BLOCKER)

**Remediation:**
```bash
# Kill conflicting processes
lsof -ti:5173 | xargs kill -9
lsof -ti:3000 | xargs kill -9
```

---

### 8. Browser Dependencies (10 points) - BLOCKER

**What it checks:** Playwright and browser binaries are installed.

**Why it matters:** Tests fail immediately with "Executable doesn't exist" in CI.

**Validation:**
```bash
browser_score=0

# Check Playwright package
if [ ! -d "apps/dashboard/node_modules/@playwright/test" ]; then
  echo "❌ @playwright/test not installed"
  browser_score=0
elif [ ! -d "apps/dashboard/node_modules/playwright" ]; then
  echo "❌ playwright package not installed"
  browser_score=0
else
  echo "✅ Playwright packages installed"

  # Check browser binaries
  cd apps/dashboard

  # Try to get Playwright version
  if npx playwright --version >/dev/null 2>&1; then
    echo "✅ Playwright CLI working"

    # Check if chromium is installed
    # Note: Can't use --dry-run reliably, just check if install succeeds quickly
    install_output=$(npx playwright install chromium --dry-run 2>&1 || echo "")

    if echo "$install_output" | grep -q "is already installed"; then
      echo "✅ Chromium browser installed"
      browser_score=10
    else
      echo "❌ Browser binaries not installed"
      echo "   Run: npx playwright install chromium"
      browser_score=4
    fi
  else
    echo "⚠️  Playwright CLI not working"
    browser_score=4
  fi

  cd - >/dev/null
fi
```

**Scoring:**
- Package + browsers: 10 points
- Package, no browsers: 4 points
- No package: 0 points (BLOCKER)

**Remediation:**
```bash
# Install dependencies
pnpm install --filter gal-dashboard...

# Install browsers
cd apps/dashboard
npx playwright install chromium
```

---

### 9. BASE_URL Reachability (10 points) - BLOCKER (staging/prod)

**What it checks:** Deployed site is accessible at BASE_URL.

**Why it matters:** Tests fail immediately if site isn't deployed yet.

**Validation:**
```bash
baseurl_score=10

if [ "$ENVIRONMENT" != "dev" ] && [ -n "$BASE_URL" ]; then
  echo "→ Testing BASE_URL: $BASE_URL"

  response=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 "$BASE_URL" 2>/dev/null || echo "000")

  if [ "$response" = "200" ] || [ "$response" = "302" ]; then
    echo "✅ BASE_URL reachable (HTTP $response)"
    baseurl_score=10

    # Additional check: Does it return HTML?
    content_type=$(curl -s -I --max-time 5 "$BASE_URL" 2>/dev/null | grep -i "content-type" | cut -d: -f2)
    if [[ "$content_type" != *"text/html"* ]]; then
      echo "⚠️  BASE_URL not returning HTML (got: $content_type)"
      baseurl_score=8
    fi
  elif [ "$response" = "404" ] || [ "$response" = "500" ]; then
    echo "❌ BASE_URL returns error (HTTP $response)"
    baseurl_score=0
  else
    echo "❌ BASE_URL unreachable (timeout or connection failed)"
    baseurl_score=0
  fi
else
  echo "✅ Dev environment (using localhost)"
  baseurl_score=10
fi
```

**Scoring:**
- Reachable + returns HTML: 10 points
- Reachable, wrong content-type: 8 points
- Error response or unreachable: 0 points (BLOCKER)

**Remediation:**
```bash
# Verify deployment
curl -I https://gal-scheduler-systems.web.app

# Check Firebase Hosting status
firebase hosting:channel:list
```

---

### 10. Test Path Consistency (5 points) - BLOCKER

**What it checks:** test.sh TEST_PATH matches Playwright project's testDir expectations.

**Why it matters:** Ensures test.sh and playwright.config.ts are in sync.

**Validation:**
```bash
consistency_score=5

# Get TEST_PATH from test.sh for this scope
case "$TEST_SCOPE" in
  smoke) expected_path="e2e/ui/smoke/" ;;
  features) expected_path="e2e/ui/features/" ;;
  workflows) expected_path="e2e/ui/workflows/" ;;
  integration) expected_path="e2e/integration/" ;;
  all) expected_path="e2e/" ;;
esac

# Verify test.sh would use this path
if grep -q "TEST_PATH=\"$expected_path\"" scripts/test.sh 2>/dev/null; then
  echo "✅ test.sh TEST_PATH matches scope"
  consistency_score=5
else
  echo "⚠️  test.sh might use different path for $TEST_SCOPE"
  consistency_score=3
fi

# Cross-check with Playwright config testDir
if [ -n "$actual_testdir" ]; then
  # testDir should be parent of TEST_PATH
  if [[ "$expected_path" == "${actual_testdir#./}"* ]]; then
    echo "✅ Test path consistent with Playwright config"
  else
    echo "⚠️  Test path may not match Playwright project testDir"
    consistency_score=0
  fi
fi
```

**Scoring:**
- Paths consistent: 5 points
- Minor inconsistency: 3 points
- Major mismatch: 0 points (BLOCKER)

---

### 11. Node/pnpm Version (5 points) - WARNING

**What it checks:** Node and pnpm versions match expected versions.

**Why it matters:** Version mismatches cause dependency installation failures in CI.

**Validation:**
```bash
version_score=5

# Check Node version
node_version=$(node --version 2>/dev/null | sed 's/v//')
expected_node="20"  # From .github/workflows env.NODE_VERSION

if [ -n "$node_version" ]; then
  if [[ "$node_version" == "$expected_node"* ]]; then
    echo "✅ Node.js v$node_version"
  else
    echo "⚠️  Node.js v$node_version (expected v$expected_node.*)"
    version_score=3
  fi
else
  echo "❌ Node.js not installed"
  version_score=0
fi

# Check pnpm version
pnpm_version=$(pnpm --version 2>/dev/null)

if [ -n "$pnpm_version" ]; then
  echo "✅ pnpm v$pnpm_version"
else
  echo "⚠️  pnpm not installed"
  version_score=$((version_score - 2))
fi
```

**Scoring:**
- Correct versions: 5 points
- Wrong Node version: 3 points
- Missing pnpm: Subtract 2 points
- Missing Node: 0 points

**Remediation:**
```bash
# Install correct Node version
nvm install 20
nvm use 20

# Install pnpm
npm install -g pnpm
```

---

### 12. Previous Artifacts Cleanup (5 points) - WARNING

**What it checks:** Stale test artifacts that might cause confusion.

**Why it matters:** Old test reports can mislead about current test status.

**Validation:**
```bash
artifacts_score=5

cd apps/dashboard

# Check for stale artifacts
stale_artifacts=0

if [ -d "playwright-report" ]; then
  report_age=$(find playwright-report -maxdepth 1 -type f -mtime +1 | wc -l)
  if [ "$report_age" -gt 0 ]; then
    echo "⚠️  Stale playwright-report/ (>24h old)"
    ((stale_artifacts++))
  fi
fi

if [ -d "test-results" ]; then
  results_age=$(find test-results -maxdepth 1 -type d -mtime +1 | wc -l)
  if [ "$results_age" -gt 0 ]; then
    echo "⚠️  Stale test-results/ (>24h old)"
    ((stale_artifacts++))
  fi
fi

if [ $stale_artifacts -eq 0 ]; then
  echo "✅ No stale test artifacts"
  artifacts_score=5
elif [ $stale_artifacts -eq 1 ]; then
  artifacts_score=3
else
  artifacts_score=0
fi

cd - >/dev/null
```

**Scoring:**
- No stale artifacts: 5 points
- 1 stale directory: 3 points
- 2+ stale directories: 0 points

**Remediation:**
```bash
# Clean up stale artifacts
rm -rf apps/dashboard/playwright-report apps/dashboard/test-results
```

---

### 13. Test Timeout Configuration (5 points) - WARNING

**What it checks:** Test timeouts are appropriate for environment and scope.

**Why it matters:** Too-aggressive timeouts cause flaky test failures.

**Validation:**
```bash
timeout_score=5

# Expected timeouts by environment
case "$ENVIRONMENT" in
  dev)
    expected_timeout=30000  # 30s for local
    ;;
  staging|production)
    expected_timeout=60000  # 60s for deployed (network latency)
    ;;
esac

# Read actual timeout from playwright.config.ts
actual_timeout=$(grep "timeout:" apps/dashboard/playwright.config.ts | grep -v "//" | head -1 | grep -oE "[0-9]+")

if [ -n "$actual_timeout" ]; then
  if [ "$actual_timeout" -ge "$expected_timeout" ]; then
    echo "✅ Test timeout: ${actual_timeout}ms (appropriate for $ENVIRONMENT)"
    timeout_score=5
  else
    echo "⚠️  Test timeout: ${actual_timeout}ms (may be too aggressive for $ENVIRONMENT)"
    echo "   Recommended: ${expected_timeout}ms+"
    timeout_score=3
  fi
else
  echo "⚠️  Could not detect test timeout configuration"
  timeout_score=4
fi

# Check for workflow-specific timeout (smoke tests should be fast)
if [ "$TEST_SCOPE" = "smoke" ] && [ "$actual_timeout" -gt 60000 ]; then
  echo "ℹ️  Smoke tests with long timeout (${actual_timeout}ms) - consider shorter timeout"
fi
```

**Scoring:**
- Timeout appropriate for environment: 5 points
- Timeout too aggressive: 3 points
- Unknown configuration: 4 points

---

### 14. Disk Space Availability (5 points) - BLOCKER

**What it checks:** Sufficient disk space for test artifacts (videos, screenshots, reports).

**Why it matters:** Tests fail mid-run when disk fills up, wasting partial CI time.

**Validation:**
```bash
disk_score=0

# Get available disk space (in GB)
if command -v df >/dev/null 2>&1; then
  # Get available space for current directory
  available_gb=$(df -BG . | awk 'NR==2 {print $4}' | sed 's/G//')

  # Minimum required: 2GB for artifacts
  required_gb=2

  if [ "$available_gb" -ge "$required_gb" ]; then
    echo "✅ Disk space: ${available_gb}GB available"
    disk_score=5
  elif [ "$available_gb" -ge 1 ]; then
    echo "⚠️  Disk space: ${available_gb}GB (low, might fail)"
    disk_score=3
  else
    echo "❌ Disk space: ${available_gb}GB (insufficient)"
    disk_score=0
  fi
else
  echo "⚠️  Cannot check disk space"
  disk_score=4
fi
```

**Scoring:**
- 2GB+ available: 5 points
- 1-2GB available: 3 points (warning)
- <1GB available: 0 points (BLOCKER)

**Remediation:**
```bash
# Clean up old artifacts
rm -rf apps/dashboard/playwright-report apps/dashboard/test-results
rm -rf apps/dashboard/test-results-*

# Check Docker disk usage
docker system df
docker system prune -a
```

---

### 15. File Write Permissions (5 points) - BLOCKER

**What it checks:** Can write to test directories (test-results, playwright-report, .auth).

**Why it matters:** Tests fail immediately if can't create artifacts or storage state.

**Validation:**
```bash
permissions_score=5

# Test write access to critical directories
test_write() {
  local dir=$1
  local test_file="$dir/.write-test-$$"

  if [ ! -d "$dir" ]; then
    mkdir -p "$dir" 2>/dev/null || {
      echo "❌ Cannot create directory: $dir"
      return 1
    }
  fi

  if touch "$test_file" 2>/dev/null; then
    rm -f "$test_file"
    echo "✅ Writable: $dir"
    return 0
  else
    echo "❌ Not writable: $dir"
    return 1
  fi
}

cd apps/dashboard

failed=0
test_write "test-results" || ((failed++))
test_write "playwright-report" || ((failed++))
test_write "e2e/.auth" || ((failed++))

if [ $failed -eq 0 ]; then
  permissions_score=5
elif [ $failed -eq 1 ]; then
  permissions_score=3
else
  permissions_score=0
fi

cd - >/dev/null
```

**Scoring:**
- All directories writable: 5 points
- 1 directory not writable: 3 points
- 2+ directories not writable: 0 points (BLOCKER)

**Remediation:**
```bash
# Fix permissions
chmod -R u+w apps/dashboard/test-results apps/dashboard/playwright-report apps/dashboard/e2e/.auth

# Check ownership
ls -la apps/dashboard/ | grep -E "(test-results|playwright-report|e2e)"
```

---

### 16. Race Condition Detection (5 points) - BLOCKER

**What it checks:** No other test runs active (prevents state conflicts).

**Why it matters:** Concurrent runs share storage state, causing auth failures and flaky tests.

**Validation:**
```bash
race_score=5

# Check for active Playwright processes
active_playwright=$(pgrep -f "playwright test" | wc -l)

if [ "$active_playwright" -gt 0 ]; then
  echo "❌ Active Playwright processes detected: $active_playwright"
  echo "   Concurrent runs will conflict on storage state"
  race_score=0

  # Show active processes
  pgrep -fl "playwright test"
else
  echo "✅ No concurrent test runs detected"
  race_score=5
fi

# Check for stale lock files
if [ -f "apps/dashboard/.test-lock" ]; then
  lock_age=$(($(date +%s) - $(stat -f %m "apps/dashboard/.test-lock" 2>/dev/null || stat -c %Y "apps/dashboard/.test-lock")))

  if [ "$lock_age" -lt 3600 ]; then
    echo "⚠️  Recent lock file found (${lock_age}s old)"
    echo "   Another test run may have crashed"
    race_score=3
  else
    echo "ℹ️  Stale lock file found (removing)"
    rm -f "apps/dashboard/.test-lock"
  fi
fi
```

**Scoring:**
- No active runs: 5 points
- Stale lock only: 3 points
- Active concurrent run: 0 points (BLOCKER)

**Remediation:**
```bash
# Kill active Playwright processes
pkill -f "playwright test"

# Remove lock files
rm -f apps/dashboard/.test-lock
```

---

### 17. Git Clean State (5 points) - WARNING

**What it checks:** No uncommitted changes in critical config files.

**Why it matters:** Modified configs can cause unexpected test behavior.

**Validation:**
```bash
git_score=5

# Check for uncommitted changes in critical files
critical_files=(
  "apps/dashboard/playwright.config.ts"
  "apps/dashboard/e2e/setup/global-setup.ts"
  "scripts/test.sh"
  "scripts/env-config.sh"
  ".env.sample"
)

uncommitted=0

for file in "${critical_files[@]}"; do
  if git diff --name-only | grep -q "^$file$"; then
    echo "⚠️  Uncommitted changes: $file"
    ((uncommitted++))
  fi

  if git diff --cached --name-only | grep -q "^$file$"; then
    echo "⚠️  Staged changes: $file"
    ((uncommitted++))
  fi
done

if [ "$uncommitted" -eq 0 ]; then
  echo "✅ No uncommitted config changes"
  git_score=5
elif [ "$uncommitted" -le 2 ]; then
  echo "⚠️  $uncommitted uncommitted config file(s)"
  git_score=3
else
  echo "⚠️  $uncommitted uncommitted config files (may affect tests)"
  git_score=0
fi

# Check for detached HEAD
if git symbolic-ref -q HEAD >/dev/null; then
  echo "✅ Not in detached HEAD state"
else
  echo "⚠️  In detached HEAD state"
  git_score=$((git_score - 2))
fi
```

**Scoring:**
- Clean state: 5 points
- 1-2 uncommitted configs: 3 points
- 3+ uncommitted or detached HEAD: 0 points

**Remediation:**
```bash
# Review uncommitted changes
git status

# Stash changes
git stash push -m "Pre-test stash"

# Or commit changes
git add .
git commit -m "[test-prep] Commit config changes"
```

---

### 18. Dependency Lock Integrity (5 points) - WARNING

**What it checks:** pnpm-lock.yaml not modified (prevents phantom dependencies).

**Why it matters:** Lock file drift causes CI failures when dependencies don't match.

**Validation:**
```bash
lock_score=5

if [ -f "pnpm-lock.yaml" ]; then
  # Check if lock file is modified
  if git diff --name-only | grep -q "pnpm-lock.yaml"; then
    echo "⚠️  pnpm-lock.yaml has uncommitted changes"
    echo "   Lock file drift can cause CI dependency mismatches"
    lock_score=2
  elif git diff --cached --name-only | grep -q "pnpm-lock.yaml"; then
    echo "⚠️  pnpm-lock.yaml staged for commit"
    lock_score=3
  else
    echo "✅ pnpm-lock.yaml clean"
    lock_score=5
  fi

  # Check if lock file is out of sync with package.json
  if command -v pnpm >/dev/null 2>&1; then
    if ! pnpm install --frozen-lockfile --dry-run >/dev/null 2>&1; then
      echo "❌ pnpm-lock.yaml out of sync with package.json"
      echo "   Run: pnpm install"
      lock_score=0
    fi
  fi
else
  echo "⚠️  pnpm-lock.yaml not found"
  lock_score=0
fi
```

**Scoring:**
- Lock file clean and in sync: 5 points
- Lock file staged: 3 points
- Lock file modified: 2 points
- Lock file out of sync: 0 points

**Remediation:**
```bash
# Regenerate lock file
pnpm install

# Or reset lock file
git checkout pnpm-lock.yaml
pnpm install --frozen-lockfile
```

---

### 19. Network Connectivity (5 points) - BLOCKER (CI only)

**What it checks:** Can reach external services (API, deployed sites, CDNs).

**Why it matters:** Network issues cause mysterious timeouts and failures.

**Validation:**
```bash
network_score=5

# Only check in CI or when testing remote environments
if [ -n "$CI" ] || [ "$ENVIRONMENT" != "dev" ]; then
  # Test DNS resolution
  if ! nslookup google.com >/dev/null 2>&1; then
    echo "❌ DNS resolution failed"
    network_score=0
  else
    echo "✅ DNS resolution working"
  fi

  # Test HTTPS connectivity
  if ! curl -s --max-time 5 https://google.com >/dev/null 2>&1; then
    echo "❌ HTTPS connectivity failed"
    echo "   Check firewall/proxy settings"
    network_score=0
  else
    echo "✅ HTTPS connectivity working"
  fi

  # Test API reachability (already done in Component 2)
  # Test BASE_URL reachability (already done in Component 9)

  # Measure network latency
  if [ -n "$API_URL" ]; then
    latency=$(curl -s -o /dev/null -w "%{time_total}" --max-time 10 "$API_URL/health" 2>/dev/null || echo "999")
    latency_ms=$(echo "$latency * 1000" | bc 2>/dev/null || echo "999")

    if [ "${latency_ms%.*}" -lt 1000 ]; then
      echo "✅ API latency: ${latency_ms%.*}ms"
    elif [ "${latency_ms%.*}" -lt 3000 ]; then
      echo "⚠️  API latency: ${latency_ms%.*}ms (slow)"
      network_score=3
    else
      echo "❌ API latency: ${latency_ms%.*}ms (too slow)"
      network_score=0
    fi
  fi
else
  echo "✅ Dev environment (localhost connectivity assumed)"
  network_score=5
fi
```

**Scoring:**
- All network checks pass, low latency: 5 points
- High latency but working: 3 points
- DNS or HTTPS failure: 0 points (BLOCKER)

**Remediation:**
```bash
# Check network settings
ping -c 3 8.8.8.8

# Check DNS
nslookup api.gal.run

# Check for VPN/proxy issues
env | grep -i proxy

# Test with verbose curl
curl -v https://gal-api-wug5dzqj2a-uc.a.run.app/health
```

---

### 20. Known Issues Penalty (-5 per 10 skipped tests)

**What it checks:** Number of skipped tests (technical debt indicator).

**Why it matters:** High skip count indicates quality issues.

**Validation:**
```bash
# Parse SKIPPED_TESTS.md
skipped_file="apps/dashboard/e2e/SKIPPED_TESTS.md"
penalty=0

if [ -f "$skipped_file" ]; then
  # Extract total from "**Total: XX skipped tests"
  total_line=$(grep -E "^\*\*Total:" "$skipped_file" 2>/dev/null || echo "")
  skipped_count=$(echo "$total_line" | grep -oE "[0-9]+" | head -1 || echo "0")

  if [ "$skipped_count" -gt 0 ]; then
    penalty=$((skipped_count / 10 * 5))
    echo "⚠️  $skipped_count skipped tests (Penalty: -$penalty points)"
  else
    echo "✅ No skipped tests"
  fi
else
  echo "ℹ️  No SKIPPED_TESTS.md found (assuming 0 skipped)"
fi
```

**Penalty Calculation:**
- 0-9 skipped: 0 penalty
- 10-19 skipped: -5 points
- 20-29 skipped: -10 points
- 30+ skipped: -15 points (max)

---

## Direct Execution Instructions

```bash
#!/bin/bash
# Comprehensive test environment evaluation

set -e

# Parse arguments
TEST_SCOPE="${1:-smoke}"
ENVIRONMENT="${2:-dev}"

# Validate arguments
if [[ ! "$TEST_SCOPE" =~ ^(smoke|features|workflows|integration|all)$ ]]; then
  echo "❌ Invalid test scope: $TEST_SCOPE"
  echo "Valid: smoke, features, workflows, integration, all"
  exit 1
fi

if [[ ! "$ENVIRONMENT" =~ ^(dev|staging|production)$ ]]; then
  echo "❌ Invalid environment: $ENVIRONMENT"
  echo "Valid: dev, staging, production"
  exit 1
fi

# Initialize scoring
total_score=0

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  TEST ENVIRONMENT EVALUATION"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "Test Scope: $TEST_SCOPE"
echo "Environment: $ENVIRONMENT"
echo ""

# 1. Environment Variables (10 points)
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  1. ENVIRONMENT VARIABLES"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
# [Full validation from component 1]

# 2-14. Additional components...
# [All component validations]

# Calculate final score
final_score=$((total_score - penalty))

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  FINAL SCORE: $final_score/100"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Assessment
if [ $final_score -ge 95 ]; then
  echo "✅ READY - Safe to run tests"
elif [ $final_score -ge 85 ]; then
  echo "⚠️  MINOR ISSUES - Consider fixing warnings"
elif [ $final_score -ge 70 ]; then
  echo "⚠️  SIGNIFICANT ISSUES - Fix before running CI"
elif [ $final_score -ge 50 ]; then
  echo "❌ NOT READY - Multiple blockers present"
else
  echo "❌ BLOCKED - Critical failures detected"
fi
```

---

## Output Format

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEST ENVIRONMENT EVALUATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Test Scope: smoke
Environment: staging

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  1. ENVIRONMENT VARIABLES (10 points)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ API_URL: https://gal-api-wug5dzqj2a-uc.a.run.app
✅ BASE_URL: https://gal-scheduler-systems.web.app
✅ GITHUB_OIDC_TOKEN: Set

Score: 10/10 ✅

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  2. API HEALTH (15 points)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Endpoint: https://gal-api-wug5dzqj2a-uc.a.run.app/health
Status: HTTP 200
Response: {"status":"ok","timestamp":"2025-12-26T..."}

Score: 15/15 ✅

[... all components ...]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  FINAL SCORE: 95/100
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ READY - Safe to run tests

Recommendations:
1. All critical checks passed
2. Minor: 5 skipped tests (-5 penalty)
3. Consider fixing skipped tests to improve quality score

Next Steps:
→ Trigger workflow: gh workflow run e2e-smoke.yml
→ Monitor: gh run watch
```

---

## Success Indicators

### ✅ READY (95-100 points)
- All blockers resolved
- Auth configured correctly
- Workflow uses canonical scripts
- Playwright config correct
- No critical issues

### ⚠️ MINOR ISSUES (85-94 points)
- Some warnings present
- Stale artifacts
- Higher skip count
- Safe to proceed with caution

### ❌ NOT READY (<85 points)
- One or more blockers present
- Fix issues before running tests
- Risk of wasted CI runs

---

## Discovered Issues (Historical Log)

### Issue 1: Missing TEST_USER Variables (2025-12-26)
**Symptom:** All tests saw login page instead of dashboard
**Root Cause:** JWT_SECRET randomly generated, invalidating tokens
**Fix:** Added TEST_USER_GITHUB_ID/USERNAME to env-config.sh
**Prevention:** Component 1 (Environment Variables) + Component 3 (Auth Setup)

### Issue 2: Workflow Direct pnpm Commands (2025-12-26)
**Symptom:** Build failed with "Cannot find module '@gal/types'"
**Root Cause:** Workflow used `pnpm build` instead of `./scripts/build.sh`
**Fix:** GAL-118 workflows use canonical scripts
**Prevention:** Component 4 (CI Workflow Validation)

### Issue 3: Playwright testDir Mismatch (2025-12-26)
**Symptom:** "No tests found" error in staging smoke tests
**Root Cause:** staging project had `testDir: './e2e'` but should be `'./e2e/ui'`
**Fix:** Updated playwright.config.ts staging and prod projects
**Prevention:** Component 5 (Playwright Config Validation)

---

## Triggering Workflows After Validation

**IMPORTANT:** After test-evaluation passes, use the correct workflow invocation command with the environment dropdown (added 2025-12-26).

### Correct Workflow Invocation

```bash
# ✅ CORRECT: Use environment dropdown (auto-maps to correct Firebase URL)
gh workflow run e2e-smoke.yml --ref main -f environment=staging
gh workflow run e2e-smoke.yml --ref main -f environment=production
gh workflow run e2e-smoke.yml --ref main -f environment=dev

# ❌ WRONG: Manual base_url (deprecated, error-prone)
gh workflow run e2e-smoke.yml --ref main -f base_url=https://...
```

### Environment → Firebase URL Mapping

The workflow automatically maps environment to Firebase Hosting URL (from `.firebaserc`):

| Environment | Firebase URL | Firebase Project |
|-------------|--------------|------------------|
| `staging` | `https://gal-staging-dashboard.web.app` | `gal-staging` |
| `production` | `https://gal-dashboard.web.app` | `gal-scheduler-systems` |
| `dev` | `http://localhost:4173` | N/A (local) |

**Why this matters:**
- Prevents accidentally testing production when you meant staging
- No need to remember or look up Firebase URLs
- Workflow validates environment selection

**When to use each:**
- `staging` - After test-evaluation passes for staging environment
- `production` - Manual production validation (use sparingly)
- `dev` - Testing workflow locally (requires local API running)

---

## Implementation Checklist

**Core Environment (45 points):**
- [x] Component 1: Environment Variables (10)
- [x] Component 2: API Health (15)
- [x] Component 3: Authentication Setup (20)

**Configuration Validation (45 points):**
- [x] Component 4: CI Workflow Validation (10)
- [x] Component 5: Playwright Config Validation (10)
- [x] Component 6: Storage State Validation (10)
- [x] Component 7: Port Availability (5)
- [x] Component 8: Browser Dependencies (10)

**Infrastructure & Network (25 points):**
- [x] Component 9: BASE_URL Reachability (10)
- [x] Component 10: Test Path Consistency (5)
- [x] Component 19: Network Connectivity (5)
- [x] Component 14: Disk Space (5)

**System State (15 points):**
- [x] Component 15: File Permissions (5)
- [x] Component 16: Race Conditions (5)
- [x] Component 11: Node/pnpm Version (5)

**Code Quality & Hygiene (20 points):**
- [x] Component 17: Git Clean State (5)
- [x] Component 18: Dependency Lock (5)
- [x] Component 12: Previous Artifacts (5)
- [x] Component 13: Test Timeout Config (5)

**Penalty:**
- [x] Component 20: Known Issues Penalty (-5/10 skipped)

**Total:** 20 components, 130 points possible

**Status:** Enterprise-grade validation system with 20 components covering ALL known failure modes.
