# SDLC Status Update Helper

Helper function for updating PR and issue status with SDLC phase labels.

## Usage

```bash
# Source the helper function
update_sdlc_status() {
  local PR_NUM=$1
  local PHASE=$2      # "5-review", "6-verify", "7-deploy"
  local STATUS=$3     # "in-progress", "passed", "blocked", "complete"

  # Remove all SDLC labels first
  gh pr edit $PR_NUM --remove-label "sdlc:5-review-in-progress,sdlc:5-review-passed,sdlc:5-review-blocked,sdlc:6-verify-in-progress,sdlc:6-verify-passed,sdlc:6-verify-blocked,sdlc:7-deploy-in-progress,sdlc:7-deploy-complete" 2>/dev/null || true

  # Add new label
  gh pr edit $PR_NUM --add-label "sdlc:${PHASE}-${STATUS}" 2>/dev/null || true

  # Update linked issue if exists
  LINKED_ISSUE=$(gh pr view $PR_NUM --json body --jq '.body' 2>/dev/null | grep -oE 'Closes #[0-9]+|Fixes #[0-9]+|Resolves #[0-9]+' | grep -oE '[0-9]+' | head -1)
  if [ -n "$LINKED_ISSUE" ]; then
    gh issue comment $LINKED_ISSUE --body "**SDLC Update**: Phase ${PHASE} - ${STATUS}" 2>/dev/null || true
  fi

  echo "📊 SDLC Status: ${PHASE} → ${STATUS}"
}
```

## Available Labels

| Label | Phase | Status |
|-------|-------|--------|
| `sdlc:5-review-in-progress` | 5-REVIEW | Code review underway |
| `sdlc:5-review-passed` | 5-REVIEW | Code review passed |
| `sdlc:5-review-blocked` | 5-REVIEW | Code review blocked |
| `sdlc:6-verify-in-progress` | 6-VERIFY | Manual testing underway |
| `sdlc:6-verify-passed` | 6-VERIFY | Verification complete |
| `sdlc:6-verify-blocked` | 6-VERIFY | Verification blocked |
| `sdlc:7-deploy-in-progress` | 7-DEPLOY | Deployment underway |
| `sdlc:7-deploy-complete` | 7-DEPLOY | Deployed to production |

## Example Calls

```bash
# Phase 5: Review
update_sdlc_status 511 "5-review" "in-progress"  # Start review
update_sdlc_status 511 "5-review" "passed"       # Review passed
update_sdlc_status 511 "5-review" "blocked"      # Review blocked

# Phase 6: Verify
update_sdlc_status 511 "6-verify" "in-progress"  # Start verification
update_sdlc_status 511 "6-verify" "passed"       # Verification passed
update_sdlc_status 511 "6-verify" "blocked"      # Verification blocked

# Phase 7: Deploy
update_sdlc_status 511 "7-deploy" "in-progress"  # Deployment started
update_sdlc_status 511 "7-deploy" "complete"     # Deployment complete
```
