# Prime DEPLOY - Load Deployment Context

Prime context window with everything needed for CI/CD and deployment work.

## Step 1: Read Deployment Documentation

Read these files to understand deployment process:

1. Read `docs/sdlc/7-deploy/README.md`
2. Read `docs/sdlc/7-deploy/operations/CI-CD.md` (if exists)
3. Read `docs/sdlc/7-deploy/operations/DEPLOYMENT.md` (if exists)
4. Read `docs/architecture/DEPLOYMENT_FLOW.md`
5. Read `.claude/rules/ci-workflow.md`
6. Read `.claude/rules/deployment.md`

## Step 2: Read CI/CD Configuration

Read workflow and deployment files:

1. Read `.github/workflows/orchestrate-staging.yml`
2. Read `.github/workflows/orchestrate-preview.yml` (if exists)
3. Read `.github/workflows/deploy.yml` (if exists)
4. Read `scripts/deploy-api.sh`
5. Read `scripts/deploy-dashboard.sh`
6. Read `scripts/env-config.sh`

## Step 3: Check CI/CD Status

```bash
# Recent workflow runs
gh run list --limit 10

# Check current branch CI status
BRANCH=$(git branch --show-current)
gh pr checks 2>/dev/null || echo "No PR checks"

# List available workflows
gh workflow list

# Check deployment status
gh release list --limit 5 2>/dev/null || echo "No releases found"
```

## Step 4: Check Environment Config

```bash
# Firebase config
cat firebase.json | head -30

# Environment scripts
ls -la scripts/deploy-*.sh
ls -la scripts/env-*.sh

# Check for environment files
ls -la .env* 2>/dev/null || echo "No .env files"
```

## Step 5: Read Build Configuration

1. Read `apps/dashboard/vite.config.ts`
2. Read `apps/api/package.json` (scripts section)
3. Read `package.json` (root scripts)

## Deployment Patterns (Memorize These)

### Environment URLs

| Env | Dashboard | API |
|-----|-----------|-----|
| Dev-Local | localhost:5173 | localhost:3000 |
| Dev-CI | gal-staging-dashboard--pr-XX.web.app | staging API |
| Staging | gal-staging-dashboard.web.app | gal-api-staging |
| Production | app.gal.run | gal-api |

### Deployment Flow

```
Feature branch → PR to staging → Preview deployment
                      ↓
              Merge to staging
                      ↓
              Auto-PR to main → Staging deployment
                      ↓
              Merge to main
                      ↓
              Manual release → Production
```

### CI Commands

```bash
# Push without CI (debugging)
git commit -m "[GAL-XXX] Fix [skip ci]"
git push

# Trigger specific workflow
gh workflow run e2e-dashboard-smoke.yml --ref $(git branch --show-current)

# Monitor workflow
gh run watch <run_id>

# Cancel accidental run
gh run cancel <run_id>

# View failed logs
gh run view <run_id> --log-failed
```

### Deployment Commands

```bash
# Local development
./scripts/run.sh

# Build for environment
./scripts/build.sh dashboard staging
./scripts/build.sh api production

# Manual deployment
./scripts/deploy-dashboard.sh staging
./scripts/deploy-api.sh staging

# Production release
gh workflow run orchestrate-release.yml -f version=1.0.0
```

### CI Safety Rules

1. **Never trigger full pipeline when tests failing**
2. **Always `[skip ci]` during debugging**
3. **Cancel accidental runs immediately**
4. **Test locally before pushing**

## CRITICAL RULES (MUST FOLLOW)

### CI Check Protocol
**NEVER approve or merge a PR until ALL CI checks pass.** Don't run tests locally while CI runs - wait for CI. Don't use `--admin` to bypass failures - fix them first.

### Fix Failures On The Spot
When discovering a bug or failure during SDLC workflow: fix immediately while having context. Don't skip past failures. Never use `--admin` bypass.

### GitHub Issue Linkage
**ALWAYS use "Closes #X" (not "Relates to") in PR descriptions for auto-close.**

## After Priming

You are now ready to run:
- `/sdlc/deploy/run` - Execute deployment
- `/sdlc/deploy/helpers/ci-pipeline` - Debug CI pipeline
- `/sdlc/deploy/helpers/validate-workflow` - Validate workflow config

## Report After Priming

Summarize:
1. Current branch and deployment target
2. Recent workflow run status
3. Any failing workflows
4. Ready to proceed with deployment

