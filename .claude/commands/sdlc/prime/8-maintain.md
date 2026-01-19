# Prime MAINTAIN - Load Maintenance Context

Prime context window with everything needed for software maintenance and support.

## Step 1: Read Maintenance Documentation

Read these files to understand maintenance:

1. Read `docs/sdlc/8-maintain/README.md`
2. Read `docs/sdlc/8-maintain/operations/SECRETS.md` (if exists)
3. Read `docs/sdlc/8-maintain/operations/FEATURE_FLAGS.md` (if exists)
4. Read `docs/sdlc/8-maintain/operations/MONITORING.md` (if exists)
5. Read `docs/sdlc/8-maintain/security/SECURITY-POLICY.md` (if exists)

## Step 2: Read Security Documentation

1. Read `apps/api/SECURITY.md`
2. Read `docs/security/SECURITY-POLICY.md` (if exists)
3. Read `firestore.rules`
4. Read `storage.rules`

## Step 3: Check Production Status

```bash
# Recent deployments
gh release list --limit 5

# Recent production workflows
gh run list --workflow=orchestrate-release.yml --limit 5 2>/dev/null

# Check for any incidents
gh issue list --label=incident --limit 5 2>/dev/null || echo "No incident label or issues"
```

## Step 4: Check Logs and Monitoring

```bash
# Local logs directory
ls -la logs/ 2>/dev/null || echo "No logs directory"

# Firebase logs
ls -la firestore-debug.log 2>/dev/null

# Check any error patterns in recent logs
tail -50 logs/*.log 2>/dev/null | grep -i error | head -10 || echo "No recent errors in logs"
```

## Step 5: Read Maintenance Scripts

1. Read `scripts/verify-deployment.sh` (if exists)
2. Read `scripts/load-env.sh`
3. Read `scripts/env-config.sh`

## Maintenance Patterns (Memorize These)

### Environment Management

| Environment | Purpose | Access |
|-------------|---------|--------|
| Dev-Local | Development | All devs |
| Dev-CI | PR testing | Automated |
| Staging | Integration | Team leads |
| Production | Live service | Ops team |

### Secrets Management

```bash
# Never hardcode - use GCP Secret Manager
gcloud secrets list --project=gal-api

# Access secret in code
const secret = await secretManager.accessSecret('secret-name')

# Rotate secrets
gcloud secrets versions add SECRET_NAME --data-file=new-secret.txt
```

### Incident Response

1. **Detect** - Monitoring alerts
2. **Triage** - Assess severity (P0-P3)
3. **Communicate** - Status page update
4. **Investigate** - Root cause analysis
5. **Mitigate** - Immediate fix
6. **Resolve** - Permanent fix
7. **Review** - Post-incident review

### Severity Levels

| Level | Description | Response Time |
|-------|-------------|---------------|
| P0 | Service down | Immediate |
| P1 | Major feature broken | < 1 hour |
| P2 | Minor feature broken | < 4 hours |
| P3 | Cosmetic/minor | Next sprint |

### Monitoring Commands

```bash
# Check API health
curl https://api.gal.run/health

# Check dashboard
curl -I https://app.gal.run

# Firebase status
firebase functions:log --only api

# GCP logs
gcloud logging read "resource.type=cloud_run_revision" --limit=50
```

### Rollback Procedures

```bash
# Rollback dashboard to previous version
firebase hosting:clone gal-staging-dashboard:live gal-staging-dashboard:rollback
firebase hosting:channel:deploy rollback

# Rollback API (Cloud Run)
gcloud run services update-traffic gal-api --to-revisions=PREVIOUS_REVISION=100
```

### Security Checklist

- [ ] All secrets in Secret Manager
- [ ] Firestore rules up to date
- [ ] No exposed API keys
- [ ] Auth tokens expire properly
- [ ] Rate limiting active
- [ ] CORS properly configured

## CRITICAL RULES (MUST FOLLOW)

### CI Check Protocol
**NEVER approve or merge a PR until ALL CI checks pass.** Don't run tests locally while CI runs - wait for CI. Don't use `--admin` to bypass failures - fix them first.

### Fix Failures On The Spot
When discovering a bug or failure during SDLC workflow: fix immediately while having context. Don't skip past failures. Never use `--admin` bypass.

### GitHub Issue Linkage
**ALWAYS use "Closes #X" (not "Relates to") in PR descriptions for auto-close.**

## After Priming

You are now ready to run:
- `/sdlc/8-maintain/run` - Triage maintenance issues
- `/sdlc/8-maintain/helpers/analyze-branch` - Analyze branch health
- `/sdlc/8-maintain/helpers/review-branches` - Review stale branches

## Report After Priming

Summarize:
1. Recent deployment status
2. Any active incidents
3. Security configuration status
4. Ready to proceed with maintenance work
