---
allowed-tools: Bash, Read, Grep, Glob
argument-hint: <workflow_type> <environment>
description: Comprehensive CI workflow evaluation to prevent failures before triggering workflows
handoffs:
  - label: "Next: MAINTAIN (Step 8)"
    agent: sdlc:8-maintain:run
    prompt: Monitor for bugs and handle maintenance
---

## MANDATORY: Initialize Progress Tracking

**Before ANY work, create TodoWrite:**

```javascript
TodoWrite([
  { content: "[P7] Verify Phase 6 (VERIFY) complete", status: "pending", activeForm: "Verifying PR merged" },
  { content: "[P7] Prime context", status: "pending", activeForm: "Loading deployment context" },
  { content: "[P7] Scan for deployment scripts", status: "pending", activeForm: "Scanning for deploy scripts" },
  { content: "[P7] Check infrastructure blockers", status: "pending", activeForm: "Checking for infra issues" },
  { content: "[P7] Execute deployment", status: "pending", activeForm: "Deploying to production" },
  { content: "[P7] Verify production deployment", status: "pending", activeForm: "Verifying production version" },
  { content: "[P7] Auto-close GitHub issue", status: "pending", activeForm: "Closing GitHub issue" },
  { content: "[P7] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

**Rules (from official docs):**
- Initialize ALL todos upfront before starting work
- Mark `in_progress` BEFORE beginning each step
- Only ONE todo `in_progress` at a time
- Mark `completed` immediately when done

---

## Phase Gate Check (REQUIRED - FAIL FAST)

**CRITICAL: Verify Phase 5 (REVIEW) completed - PR must be merged before deployment.**

```bash
# Check if PR is merged or we're on target branch
PR_STATE=$(gh pr view --json state --jq '.state' 2>/dev/null || echo "NONE")
CURRENT_BRANCH=$(git branch --show-current)
TARGET_BRANCH="staging"  # or main, depends on project

GATE_PASSED=false

# Check 1: PR is merged
if [ "$PR_STATE" = "MERGED" ]; then
  GATE_PASSED=true
  echo "✅ Phase Gate: PR merged"
fi

# Check 2: Already on target branch (post-merge)
if [ "$CURRENT_BRANCH" = "$TARGET_BRANCH" ] || [ "$CURRENT_BRANCH" = "main" ]; then
  GATE_PASSED=true
  echo "✅ Phase Gate: On target branch ($CURRENT_BRANCH)"
fi

# Check 3: Manual deployment (no PR workflow)
if [ "$1" = "--force" ] || [ "$1" = "--manual" ]; then
  GATE_PASSED=true
  echo "⚠️  Phase Gate: Manual override (--force/--manual)"
fi

if [ "$GATE_PASSED" = false ]; then
  echo "❌ PHASE GATE FAILED: PR not merged"
  echo ""
  echo "┌─────────────────────────────────────────────────────────────┐"
  echo "│ SDLC Phase Order Violation                                  │"
  echo "├─────────────────────────────────────────────────────────────┤"
  echo "│ Current Phase:  7-DEPLOY                                    │"
  echo "│ Required Phase: 6-VERIFY (not completed)                    │"
  echo "│ Missing:        PR must be merged before deployment         │"
  echo "└─────────────────────────────────────────────────────────────┘"
  echo ""
  echo "Current PR state: $PR_STATE"
  echo "Current branch: $CURRENT_BRANCH"
  echo ""
  echo "SDLC flow: ... → IMPLEMENT → REVIEW → DEPLOY"
  echo ""
  echo "Next steps:"
  if [ "$PR_STATE" = "OPEN" ]; then
    echo "  PR #\$PR_NUMBER is still open - checking why..."
    echo ""

    # Check review decision and auto-approve status
    REVIEW_DECISION=\$(gh pr view \$PR_NUMBER --json reviewDecision --jq '.reviewDecision' 2>/dev/null || echo "UNKNOWN")
    PR_LABELS=\$(gh pr view \$PR_NUMBER --json labels --jq '.labels[] | select(.name | startswith("risk:") or . == "auto-approve-eligible") | .name' 2>/dev/null || echo "")

    echo "📊 PR Review Status:"
    echo "   Review Required: \$REVIEW_DECISION"
    echo "   Risk Assessment: \$(echo \"\$PR_LABELS\" | grep \"risk:\" || echo \"Not assessed\")"
    echo "   Auto-Approve Eligible: \$(echo \"\$PR_LABELS\" | grep \"auto-approve-eligible\" >/dev/null && echo \"Yes\" || echo \"No\")"
    echo ""

    if echo "\$PR_LABELS" | grep -q "risk:high"; then
      echo "🛡️ **Auto-Approve Analysis:**"
      echo "   This PR contains high-risk changes (Docker, infrastructure, secrets)"
      echo "   GAL governance system requires manual maintainer review for security"
      echo "   Auto-merge is enabled but waiting for required approval"
      echo ""
      echo "⏳ **Waiting for:** Maintainer review approval"
      echo "🔄 **Monitor with:** gh pr view \$PR_NUMBER --json reviewDecision"
    elif echo "\$PR_LABELS" | grep -q "risk:medium"; then
      echo "🛡️ **Auto-Approve Analysis:**"
      echo "   This PR has medium-risk changes requiring manual review"
      echo "   Auto-merge is enabled but waiting for maintainer approval"
      echo ""
      echo "⏳ **Waiting for:** 1 maintainer review"
    elif echo "\$PR_LABELS" | grep -q "auto-approve-eligible"; then
      echo "🛡️ **Auto-Approve Analysis:**"
      echo "   This PR is eligible for auto-approval but may still be processing"
      echo "   Check if governance workflow completed successfully"
      echo ""
      echo "🔄 **Check status:** gh run list --workflow=\"pr-governance.yml\" --limit 3"
    else
      echo "⚠️  **Review Status Unclear** - Check PR governance workflow"
    fi
    echo ""
    echo "💡 **GAL Auto-Approve System:**"
    echo "   - Low risk (docs, UI): Auto-approved immediately"
    echo "   - Medium risk (CLI, deps): 1 maintainer review required"
    echo "   - High risk (Docker, secrets): 2 maintainer reviews required"
    echo ""
    echo "🔄 **Next Actions:**"
    echo "   1. Wait for required review approval (if high/medium risk)"
    echo "   2. Auto-merge will complete automatically when approved"
    echo "   3. Then run: /sdlc:7-deploy:run \$PR_NUMBER"
  elif [ "$PR_STATE" = "NONE" ]; then
    echo "  1. Create PR: /sdlc:5-review:run"
    echo "  2. Get PR approved and merged"
    echo "  3. Then run /sdlc:7-deploy:run"
  else
    echo "  PR state: $PR_STATE"
    echo "  Ensure PR is merged before deploying."
  fi
  echo ""
  echo "Override (use with caution):"
  echo "  /sdlc:7-deploy:run --force"
  exit 1
fi

echo ""
echo "Phase Gate passed. Ready for deployment."
```

**If gate fails → STOP. PR must be reviewed and merged before deployment.**

---

## Phase 0: Prime Context (REQUIRED - AUTO-EXECUTE)

**IMPORTANT: Execute the prime command FIRST before proceeding.**

```
Skill(skill: "sdlc:prime:6-deploy")
```

This loads deployment context including:
- Documentation: `docs/sdlc/7-deploy/` and deployment flow
- CI workflows: GitHub Actions configuration
- Environment config: scripts and variables
- CI state: recent runs, workflow list, PR checks

**DO NOT PROCEED until prime command completes.**

---

# CI Workflow Evaluation (Comprehensive)

**Purpose:** Catch EVERY possible failure mode before triggering CI workflows. Validates environment, configuration, dependencies, and prerequisites specific to each workflow type.

**Design Philosophy:** "Fail fast, fail clear" - systematically validate every workflow requirement before wasting CI resources.

## Variables

- **WORKFLOW_TYPE**: First argument from $ARGUMENTS (required)
  - Workflow category: "test", "deploy", "release", "preview", "quality"
  - Determines which validation components to run

- **ENVIRONMENT**: Second argument from $ARGUMENTS (optional, default: based on workflow type)
  - Target environment: "dev", "staging", "production", "preview"
  - Auto-determined for some workflow types

## Usage

```bash
/ci-evaluation <workflow_type> [environment]
```

Examples:
- `/ci-evaluation deploy staging` - Validate before triggering orchestrate-staging.yml
- `/ci-evaluation release production` - Validate before production release
- `/ci-evaluation preview` - Validate before PR preview deployment
- `/ci-evaluation quality` - Validate before ci.yml quality gates

## Workflow Type Mappings

| Type | Environment | Workflows Covered | Auto-triggers? |
|------|-------------|-------------------|----------------|
| **test** | staging | e2e-smoke, e2e-features, e2e-workflows, integration-tests | Yes (on push) |
| **deploy** | staging | orchestrate-staging.yml | Yes (on push to main) |
| **deploy** | production | deploy-api, deploy-dashboard, deploy-website, deploy-mcp | No (manual) |
| **release** | production | orchestrate-release.yml | No (manual) |
| **preview** | preview | orchestrate-preview.yml | Yes (on PR) |
| **quality** | N/A | ci.yml, pr-governance.yml | Yes (on PR) |

## Scoring System by Workflow Type

### Test Workflows (130 points)
See `test-evaluation.md` for complete breakdown.

### Deploy Workflows (100 points)

| Component | Points | Type | Description |
|-----------|--------|------|-------------|
| **Environment Variables** | 15 | BLOCKER | Deployment credentials and config |
| **GCP Authentication** | 20 | BLOCKER | Workload identity and service accounts |
| **Build Artifacts** | 15 | BLOCKER | Components built and ready |
| **Deployment Targets** | 15 | BLOCKER | Cloud Run services exist |
| **Secrets Availability** | 15 | BLOCKER | GCP Secret Manager access |
| **Docker** | 10 | BLOCKER | Docker daemon and registry auth |
| **Git State** | 5 | WARNING | Clean state, no uncommitted changes |
| **Previous Deployments** | 5 | WARNING | No stale deployments in progress |

**Total:** 100 points

### Release Workflows (120 points)

| Component | Points | Type | Description |
|-----------|--------|------|-------------|
| **Version Tag** | 20 | BLOCKER | Valid semver tag |
| **Changelog** | 15 | BLOCKER | CHANGELOG.md updated |
| **All Tests Passing** | 20 | BLOCKER | Latest E2E/integration tests green |
| **Staging Deployment** | 15 | BLOCKER | Staging matches release commit |
| **GCP Authentication** | 15 | BLOCKER | Production deployment credentials |
| **NPM Authentication** | 15 | BLOCKER | NPM token for CLI publishing |
| **Git State** | 10 | BLOCKER | On main, no uncommitted changes |
| **Previous Release** | 10 | WARNING | No ongoing release workflows |

**Total:** 120 points

### Preview Workflows (90 points)

| Component | Points | Type | Description |
|-----------|--------|------|-------------|
| **PR State** | 15 | BLOCKER | PR open, not draft, no conflicts |
| **Base Branch** | 15 | BLOCKER | Targeting correct branch (main) |
| **Build Artifacts** | 15 | BLOCKER | Components build successfully |
| **Firebase Channels** | 15 | BLOCKER | Preview channel available |
| **GCP Authentication** | 15 | BLOCKER | Preview deployment credentials |
| **CI Checks** | 10 | WARNING | Other PR checks passing |
| **Size Limits** | 5 | WARNING | Bundle size within limits |

**Total:** 90 points

### Quality Workflows (80 points)

| Component | Points | Type | Description |
|-----------|--------|------|-------------|
| **Node/pnpm Version** | 10 | BLOCKER | Correct runtime versions |
| **Dependencies** | 20 | BLOCKER | pnpm-lock.yaml in sync |
| **TypeScript** | 20 | BLOCKER | No type errors |
| **Linting** | 15 | WARNING | ESLint passes |
| **Build** | 15 | BLOCKER | All components build |

**Total:** 80 points

## Infrastructure Blocker Check

**Before deployment, check for infrastructure issues that would cause failures.**

```bash
# Check recent workflow runs for infra-related failures
RECENT_RUNS=$(gh run list --workflow deploy.yml --limit 3 --json conclusion,name,databaseId)
FAILED_RUNS=$(echo "$RECENT_RUNS" | jq '[.[] | select(.conclusion == "failure")]')

if [ "$(echo "$FAILED_RUNS" | jq 'length')" -gt 0 ]; then
  echo "⚠️  Recent deployment failures detected"

  # Check for infra keywords in logs
  RUN_ID=$(echo "$FAILED_RUNS" | jq -r '.[0].databaseId')
  LOGS=$(gh run view $RUN_ID --log-failed 2>/dev/null | head -100)

  INFRA_KEYWORDS="Docker daemon|CLI not found|not installed|ImagePull|connection timeout|rate limit|out of disk|cannot connect"
  if echo "$LOGS" | grep -iE "$INFRA_KEYWORDS" > /dev/null; then
    echo "🚧 INFRASTRUCTURE ISSUE DETECTED"
    echo ""
    echo "Run: /report-infra-blocker <description>"
  fi
fi
```

**If infra issue detected:**
```
/report-infra-blocker <description of the issue>
```

This pauses SDLC, creates infra tracking issue, and prevents wasted deploy attempts.

## Assessment Thresholds

### Deploy/Release (100-120 point scales)
| Score | Assessment | Action |
|-------|------------|--------|
| 95%+ | ✅ READY | Safe to deploy/release |
| 85-94% | ⚠️ MINOR ISSUES | Review warnings |
| 70-84% | ⚠️ SIGNIFICANT ISSUES | Fix before deploy |
| <70% | ❌ BLOCKED | Critical failures |

### Preview/Quality (80-90 point scales)
| Score | Assessment | Action |
|-------|------------|--------|
| 90%+ | ✅ READY | Safe to proceed |
| 75-89% | ⚠️ MINOR ISSUES | Review warnings |
| 60-74% | ⚠️ SIGNIFICANT ISSUES | Fix first |
| <60% | ❌ BLOCKED | Critical failures |

## Validation Components

### Deploy Workflows

#### 1. Environment Variables (15 points) - BLOCKER

**Staging:**
```bash
✅ GCP_PROJECT_ID (scheduler-systems-dev)
✅ STAGING_API_URL
✅ STAGING_DASHBOARD_URL
```

**Production:**
```bash
✅ GCP_PROJECT_ID (scheduler-systems-prod)
✅ PROD_API_URL
✅ PROD_DASHBOARD_URL
✅ FIREBASE_TOKEN (for hosting deploy)
```

**Validation:**
```bash
env_score=0
missing_count=0

if [ "$ENVIRONMENT" = "staging" ]; then
  [ -z "$GCP_PROJECT_ID" ] && echo "❌ GCP_PROJECT_ID not set" && ((missing_count++))
  [ -z "$STAGING_API_URL" ] && echo "❌ STAGING_API_URL not set" && ((missing_count++))
elif [ "$ENVIRONMENT" = "production" ]; then
  [ -z "$GCP_PROJECT_ID" ] && echo "❌ GCP_PROJECT_ID not set" && ((missing_count++))
  [ -z "$PROD_API_URL" ] && echo "❌ PROD_API_URL not set" && ((missing_count++))
  [ -z "$FIREBASE_TOKEN" ] && echo "❌ FIREBASE_TOKEN not set" && ((missing_count++))
fi

if [ $missing_count -eq 0 ]; then env_score=15
elif [ $missing_count -eq 1 ]; then env_score=10
else env_score=0; fi
```

---

#### 2. GCP Authentication (20 points) - BLOCKER

**What it checks:** GCP workload identity configured, can access Cloud Run and Secret Manager.

**Validation:**
```bash
gcp_score=0

# Check if running in GitHub Actions
if [ -n "$CI" ] && [ -n "$GITHUB_ACTIONS" ]; then
  # Check workload identity environment vars
  if [ -n "$GCP_WORKLOAD_IDENTITY_PROVIDER" ] && [ -n "$GCP_SERVICE_ACCOUNT" ]; then
    echo "✅ Workload identity configured"
    gcp_score=10

    # Try to authenticate (would be done by workflow)
    if gcloud auth list --filter=status:ACTIVE 2>/dev/null | grep -q "@"; then
      echo "✅ GCP authenticated"
      gcp_score=20

      # Test Secret Manager access
      if gcloud secrets list --project="$GCP_PROJECT_ID" >/dev/null 2>&1; then
        echo "✅ Secret Manager access verified"
      else
        echo "⚠️  Cannot access Secret Manager"
        gcp_score=15
      fi
    else
      echo "⚠️  GCP authentication pending"
    fi
  else
    echo "❌ Workload identity not configured"
    gcp_score=0
  fi
else
  # Local: Check application default credentials
  if gcloud auth application-default print-access-token >/dev/null 2>&1; then
    echo "✅ Application default credentials active"
    gcp_score=20
  else
    echo "❌ GCP authentication required"
    echo "   Run: gcloud auth application-default login"
    gcp_score=0
  fi
fi
```

**Scoring:**
- Authenticated + Secret Manager access: 20 points
- Authenticated, no Secret Manager: 15 points
- Configured but not authenticated: 10 points
- Not configured: 0 points (BLOCKER)

---

#### 3. Build Artifacts (15 points) - BLOCKER

**What it checks:** All components required for deployment are built.

**Validation:**
```bash
build_score=0

# Components to check based on workflow
case "$WORKFLOW_TYPE" in
  deploy)
    if [ "$ENVIRONMENT" = "staging" ]; then
      components=("api" "dashboard" "website")
    else
      components=("api" "dashboard" "website" "mcp" "cli")
    fi
    ;;
  release)
    components=("api" "dashboard" "website" "mcp" "cli")
    ;;
esac

missing_builds=0

for component in "${components[@]}"; do
  case "$component" in
    api)
      if [ -f "apps/api/dist/index.js" ]; then
        echo "✅ API built"
      else
        echo "❌ API not built"
        ((missing_builds++))
      fi
      ;;
    dashboard)
      if [ -d "apps/dashboard/dist" ] && [ -f "apps/dashboard/dist/index.html" ]; then
        echo "✅ Dashboard built"
      else
        echo "❌ Dashboard not built"
        ((missing_builds++))
      fi
      ;;
    website)
      if [ -d "apps/website/dist" ] && [ -f "apps/website/dist/index.html" ]; then
        echo "✅ Website built"
      else
        echo "❌ Website not built"
        ((missing_builds++))
      fi
      ;;
    mcp)
      if [ -f "apps/mcp/dist/index.js" ]; then
        echo "✅ MCP built"
      else
        echo "❌ MCP not built"
        ((missing_builds++))
      fi
      ;;
    cli)
      if [ -f "apps/cli/dist/index.js" ]; then
        echo "✅ CLI built"
      else
        echo "❌ CLI not built"
        ((missing_builds++))
      fi
      ;;
  esac
done

total_components=${#components[@]}
built_components=$((total_components - missing_builds))

# Score proportionally
build_score=$((15 * built_components / total_components))
```

**Scoring:**
- All components built: 15 points
- Some missing: Proportional (e.g., 3/4 = 11 points)
- None built: 0 points (BLOCKER)

**Remediation:**
```bash
# Build all components for staging
./scripts/build.sh all staging

# Build all components for production
./scripts/build.sh all production
```

---

#### 4. Deployment Targets (15 points) - BLOCKER

**What it checks:** Cloud Run services exist and are accessible.

**Validation:**
```bash
targets_score=0

# Services to check
case "$ENVIRONMENT" in
  staging)
    services=("gal-api-staging" "gal-dashboard-staging")
    ;;
  production)
    services=("gal-api" "gal-dashboard")
    ;;
esac

missing_services=0

for service in "${services[@]}"; do
  if gcloud run services describe "$service" \
      --region=us-central1 \
      --project="$GCP_PROJECT_ID" \
      >/dev/null 2>&1; then
    echo "✅ Service exists: $service"
  else
    echo "❌ Service not found: $service"
    echo "   Create with: gcloud run services create $service ..."
    ((missing_services++))
  fi
done

total_services=${#services[@]}
existing_services=$((total_services - missing_services))

# Score proportionally
targets_score=$((15 * existing_services / total_services))
```

**Scoring:**
- All services exist: 15 points
- Some missing: Proportional
- None exist: 0 points (BLOCKER)

---

#### 5. Secrets Availability (15 points) - BLOCKER

**What it checks:** Required secrets exist in GCP Secret Manager.

**Validation:**
```bash
secrets_score=0

# Required secrets
required_secrets=(
  "GITHUB_APP_PRIVATE_KEY"
  "GITHUB_CLIENT_ID"
  "GITHUB_CLIENT_SECRET"
  "JWT_SECRET"
  "SESSION_SECRET"
)

missing_secrets=0

for secret in "${required_secrets[@]}"; do
  if gcloud secrets versions access latest \
      --secret="$secret" \
      --project="$GCP_PROJECT_ID" \
      >/dev/null 2>&1; then
    echo "✅ Secret exists: $secret"
  else
    echo "❌ Secret missing: $secret"
    ((missing_secrets++))
  fi
done

total_secrets=${#required_secrets[@]}
existing_secrets=$((total_secrets - missing_secrets))

# Score proportionally
secrets_score=$((15 * existing_secrets / total_secrets))
```

**Scoring:**
- All secrets available: 15 points
- Some missing: Proportional
- Critical secrets missing: 0 points (BLOCKER)

---

#### 6. Docker Dependencies (10 points) - BLOCKER

**What it checks:** Docker daemon running and authenticated to registry.

**Validation:**
```bash
docker_score=0

# Check Docker daemon
if docker info >/dev/null 2>&1; then
  echo "✅ Docker daemon running"
  docker_score=5

  # Check GCR authentication
  if gcloud auth configure-docker us-central1-docker.pkg.dev >/dev/null 2>&1; then
    echo "✅ GCR authenticated"
    docker_score=10
  else
    echo "⚠️  GCR not authenticated"
    echo "   Run: gcloud auth configure-docker"
  fi
else
  echo "❌ Docker daemon not running"
  docker_score=0
fi
```

**Scoring:**
- Docker + GCR auth: 10 points
- Docker only: 5 points
- No Docker: 0 points (BLOCKER)

---

### Release Workflows

#### 1. Version Tag (20 points) - BLOCKER

**What it checks:** Valid semver tag exists and matches expected pattern.

**Validation:**
```bash
tag_score=0

# Get current branch/tag
current_ref=$(git describe --tags --exact-match 2>/dev/null || echo "")

if [ -n "$current_ref" ]; then
  # Validate semver format (v1.2.3 or v1.2.3-beta.4)
  if [[ "$current_ref" =~ ^v[0-9]+\.[0-9]+\.[0-9]+(-[a-z]+\.[0-9]+)?$ ]]; then
    echo "✅ Valid version tag: $current_ref"
    tag_score=20

    # Check if tag is annotated
    if git describe --tags --exact-match --abbrev=0 2>/dev/null | grep -q "^$current_ref$"; then
      echo "✅ Annotated tag"
    else
      echo "⚠️  Lightweight tag (use annotated tags for releases)"
      tag_score=15
    fi
  else
    echo "❌ Invalid version format: $current_ref"
    echo "   Expected: vX.Y.Z or vX.Y.Z-beta.N"
    tag_score=0
  fi
else
  echo "❌ No version tag found"
  echo "   Create tag: git tag -a v1.0.0 -m 'Release v1.0.0'"
  tag_score=0
fi
```

**Scoring:**
- Valid annotated tag: 20 points
- Valid lightweight tag: 15 points
- Invalid or missing: 0 points (BLOCKER)

---

#### 2. Changelog (15 points) - BLOCKER

**What it checks:** CHANGELOG.md updated for current version.

**Validation:**
```bash
changelog_score=0

if [ -f "CHANGELOG.md" ]; then
  # Extract version from tag (remove 'v' prefix)
  version="${current_ref#v}"

  # Check if version appears in changelog
  if grep -q "## \[$version\]" CHANGELOG.md || grep -q "## $version" CHANGELOG.md; then
    echo "✅ CHANGELOG.md updated for $version"
    changelog_score=15

    # Check for unreleased section
    if grep -q "## \[Unreleased\]" CHANGELOG.md; then
      unreleased_content=$(sed -n '/## \[Unreleased\]/,/## \[/p' CHANGELOG.md | tail -n +2 | head -n -1)
      if [ -n "$unreleased_content" ] && [ "$(echo "$unreleased_content" | wc -w)" -gt 10 ]; then
        echo "⚠️  Unreleased section has content (should be moved to $version)"
        changelog_score=10
      fi
    fi
  else
    echo "❌ CHANGELOG.md not updated for $version"
    changelog_score=0
  fi
else
  echo "❌ CHANGELOG.md not found"
  changelog_score=0
fi
```

**Scoring:**
- Changelog updated, unreleased clean: 15 points
- Changelog updated, unreleased has content: 10 points
- Not updated: 0 points (BLOCKER)

---

#### 3. All Tests Passing (20 points) - BLOCKER

**What it checks:** Latest E2E and integration tests passed on current commit.

**Validation:**
```bash
tests_score=0

# Get current commit SHA
current_sha=$(git rev-parse HEAD)

# Check latest workflow runs for this commit
if command -v gh >/dev/null 2>&1; then
  # Check E2E tests
  e2e_status=$(gh run list --commit="$current_sha" --workflow=e2e-smoke.yml --json conclusion --jq '.[0].conclusion' 2>/dev/null || echo "")

  if [ "$e2e_status" = "success" ]; then
    echo "✅ E2E smoke tests passed"
    tests_score=10
  elif [ "$e2e_status" = "failure" ]; then
    echo "❌ E2E smoke tests failed"
    tests_score=0
  else
    echo "⚠️  E2E smoke tests not run on this commit"
    tests_score=5
  fi

  # Check integration tests
  integration_status=$(gh run list --commit="$current_sha" --workflow=integration-tests.yml --json conclusion --jq '.[0].conclusion' 2>/dev/null || echo "")

  if [ "$integration_status" = "success" ]; then
    echo "✅ Integration tests passed"
    tests_score=$((tests_score + 10))
  elif [ "$integration_status" = "failure" ]; then
    echo "❌ Integration tests failed"
    tests_score=0
  else
    echo "⚠️  Integration tests not run on this commit"
    tests_score=$((tests_score + 5))
  fi
else
  echo "⚠️  Cannot verify test status (gh CLI not available)"
  tests_score=10
fi
```

**Scoring:**
- All tests passed: 20 points
- Tests not run (warning): 10 points
- Any tests failed: 0 points (BLOCKER)

---

## Direct Execution Instructions

```bash
#!/bin/bash
# CI Workflow Evaluation

set -e

# Parse arguments
WORKFLOW_TYPE="${1:-test}"
ENVIRONMENT="${2:-}"

# Auto-determine environment if not provided
if [ -z "$ENVIRONMENT" ]; then
  case "$WORKFLOW_TYPE" in
    deploy) ENVIRONMENT="staging" ;;
    release) ENVIRONMENT="production" ;;
    preview) ENVIRONMENT="preview" ;;
    quality) ENVIRONMENT="dev" ;;
    test) ENVIRONMENT="dev" ;;
  esac
fi

# Validate workflow type
if [[ ! "$WORKFLOW_TYPE" =~ ^(test|deploy|release|preview|quality)$ ]]; then
  echo "❌ Invalid workflow type: $WORKFLOW_TYPE"
  echo "Valid: test, deploy, release, preview, quality"
  exit 1
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  CI WORKFLOW EVALUATION"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "Workflow Type: $WORKFLOW_TYPE"
echo "Environment: $ENVIRONMENT"
echo ""

# Initialize scoring
total_score=0
max_score=0

# Run appropriate validation based on workflow type
case "$WORKFLOW_TYPE" in
  test)
    echo "→ For test workflows, use: /test-evaluation <scope> $ENVIRONMENT"
    exit 0
    ;;

  deploy)
    max_score=100
    # Run deploy validation components
    # [Components 1-6 from above]
    ;;

  release)
    max_score=120
    # Run release validation components
    # [Components 1-3 from above + deploy components]
    ;;

  preview)
    max_score=90
    # Run preview validation components
    ;;

  quality)
    max_score=80
    # Run quality validation components
    ;;
esac

# Calculate final score
percentage=$((total_score * 100 / max_score))

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  FINAL SCORE: $total_score/$max_score ($percentage%)"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Assessment
if [ $percentage -ge 95 ]; then
  echo "✅ READY - Safe to trigger workflow"
  exit 0
elif [ $percentage -ge 85 ]; then
  echo "⚠️  MINOR ISSUES - Review warnings before proceeding"
  exit 0
elif [ $percentage -ge 70 ]; then
  echo "⚠️  SIGNIFICANT ISSUES - Fix before triggering workflow"
  exit 1
else
  echo "❌ BLOCKED - Critical failures detected"
  exit 1
fi
```

---

## Integration with Existing Commands

This command complements `test-evaluation.md`:

- **test-evaluation**: Specialized for E2E/integration test workflows (130 points, 20 components)
- **ci-evaluation**: General for deployment, release, preview, quality workflows (80-120 points)

**Workflow:**
1. Tests: Use `/test-evaluation smoke staging`
2. Deployments: Use `/ci-evaluation deploy staging`
3. Releases: Use `/ci-evaluation release production`
4. Previews: Use `/ci-evaluation preview`
5. Quality: Use `/ci-evaluation quality`

---

## Success Indicators

### ✅ READY (95%+)
- All blockers resolved
- Authentication configured
- Build artifacts present
- Deployment targets exist
- No critical issues

### ⚠️ MINOR ISSUES (85-94%)
- Some warnings present
- Non-critical configs missing
- Safe to proceed with caution

### ❌ BLOCKED (<85%)
- One or more blockers present
- Fix issues before triggering
- Risk of failed deployment/release

---

## MANDATORY: Capture Learnings (AUTO-EXECUTE)

**DO NOT end without invoking:**

```
Skill(skill: "capture-learnings")
```

This captures:
- Process deviations that occurred
- Manual interventions from user
- Improvements to agentic layer

**Update TodoWrite when complete:**
```javascript
TodoWrite([
  // ... previous steps as completed ...
  { content: "[P7] Capture learnings", status: "completed", activeForm: "Learnings captured" }
])
```
