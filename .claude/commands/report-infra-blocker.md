---
description: Report infrastructure blocker and pause SDLC workflow
allowed-tools: Bash, Read, Write
---

# Report Infrastructure Blocker

When CI/CD fails due to infrastructure issues (not code issues), use this command to:
1. Search for existing infra issues
2. Create tracking issue in infra repo (if none exists)
3. Comment on PR with blocker status
4. Pause SDLC workflow gracefully

## Usage

```
/report-infra-blocker <description>
```

Example:
```
/report-infra-blocker Arc runners not using custom image with Claude CLI
```

## Step 1: Identify Current Context

```bash
# Get current PR number
PR_NUMBER=$(gh pr view --json number --jq '.number' 2>/dev/null || echo "")

# Get failing checks
if [ -n "$PR_NUMBER" ]; then
  FAILING_CHECKS=$(gh pr checks $PR_NUMBER --json name,state --jq '[.[] | select(.state == "FAILURE" or .state == "PENDING")] | map(.name) | join(", ")')
  echo "PR #$PR_NUMBER - Failing checks: $FAILING_CHECKS"
fi
```

## Step 2: Search for Existing Infra Issues

```bash
INFRA_REPO="Scheduler-Systems/scheduler-systems-infra"
SEARCH_TERM="$1"  # User-provided description

# Search for existing issues
EXISTING=$(gh issue list --repo "$INFRA_REPO" --search "$SEARCH_TERM" --state open --limit 5 --json number,title)

if [ "$(echo "$EXISTING" | jq 'length')" -gt 0 ]; then
  echo "Found existing infra issues:"
  echo "$EXISTING" | jq -r '.[] | "  #\(.number): \(.title)"'
  INFRA_ISSUE=$(echo "$EXISTING" | jq -r '.[0].number')
else
  echo "No existing issues found for: $SEARCH_TERM"
  INFRA_ISSUE=""
fi
```

## Step 3: Create Infra Issue (If None Exists)

```bash
if [ -z "$INFRA_ISSUE" ]; then
  # Create new infra issue
  INFRA_ISSUE=$(gh issue create --repo "$INFRA_REPO" \
    --title "[CI Blocker] $SEARCH_TERM" \
    --body "$(cat <<EOF
## Infrastructure Blocker

**Reported from**: GAL PR #$PR_NUMBER
**Failing Checks**: $FAILING_CHECKS

## Description

$SEARCH_TERM

## Impact

- PR blocked waiting for infrastructure fix
- SDLC workflow paused

## Resolution Required

Fix the infrastructure issue and notify when resolved.
EOF
)" --json number --jq '.number')

  echo "Created infra issue #$INFRA_ISSUE"
fi
```

## Step 4: Comment on PR

```bash
if [ -n "$PR_NUMBER" ]; then
  gh pr comment $PR_NUMBER --body "$(cat <<EOF
## ⏸️ Blocked on Infrastructure

This PR is blocked waiting for infrastructure fix.

**Failing Checks:**
$(echo "$FAILING_CHECKS" | tr ',' '\n' | sed 's/^/- /')

**Tracking Issue:** $INFRA_REPO#$INFRA_ISSUE

Once the infra issue is resolved, re-run the failed checks.
EOF
)"
  echo "Added blocker comment to PR #$PR_NUMBER"
fi
```

## Step 5: Output Summary

```bash
echo ""
echo "═══════════════════════════════════════════════"
echo "INFRA BLOCKER REPORTED"
echo "═══════════════════════════════════════════════"
echo "PR:          #$PR_NUMBER"
echo "Infra Issue: $INFRA_REPO#$INFRA_ISSUE"
echo "Status:      SDLC paused - waiting for infra fix"
echo "═══════════════════════════════════════════════"
echo ""
echo "Next steps:"
echo "1. Monitor infra issue for resolution"
echo "2. Once resolved, re-run failed checks: gh pr checks $PR_NUMBER --watch"
echo "3. Resume SDLC workflow"
```

## Common Infrastructure Issues

| Symptom | Likely Cause |
|---------|--------------|
| "Claude Code CLI not found" | Arc runners not using custom image |
| "Docker daemon not available" | DinD not configured on runners |
| "Cannot pull image" | GHCR permissions or imagePullSecrets |
| "Connection timeout" | Network/firewall issues |
| "Out of disk space" | Runner cleanup needed |
| "Rate limit exceeded" | API throttling |
