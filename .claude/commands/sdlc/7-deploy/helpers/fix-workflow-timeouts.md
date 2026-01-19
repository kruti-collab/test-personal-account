---
description: Automatically add timeout wrappers and retry logic to GitHub Actions workflows
argument-hint: "[workflow-name|all] [--dry-run|--apply|--create-pr]"
allowed-tools:
  - Task
---

# Fix Workflow Timeouts

**EXECUTION MODE**: This command delegates to devops-engineer subagent for comprehensive fixes.

Automatically inject timeout wrappers, retry logic, and resilience patterns into GitHub Actions workflows and scripts based on production-validated failure patterns.

## Purpose

Systematically add defensive programming patterns to prevent network failures and timeouts:
- Inject universal `retry_with_backoff` function
- Wrap network operations (brew, npm, docker, git, curl)
- Add command-level timeouts (gcloud, firebase)
- Add loop bounds and safety limits
- Fix shell compatibility issues

**Context**: Real-world validation from incident #19594556598 (brew install network failure)

## Variables

- **WORKFLOW_NAME**: First argument from $ARGUMENTS (required)
  - Specific workflow: `ios`, `android`, `web`, `tart`
  - All workflows: `all`

- **MODE**: Second argument from $ARGUMENTS (optional, default: `--dry-run`)
  - `--dry-run`: Show what would be changed (default)
  - `--apply`: Apply fixes directly to files
  - `--create-pr`: Create PR with all fixes

## Workflow Mapping

```
ios     → .github/workflows/ios-selfhosted-tart.yml + scripts/ci-cd/tart/provision_ios_base_vm.sh
android → .github/workflows/android-selfhosted.yml + scripts/ci-cd/github-actions/android_build_workflow.sh
web     → .github/workflows/web-deploy-selfhosted.yml + scripts/build/web/build_web_local.sh
tart    → .github/workflows/tart-build-vm.yml + scripts/ci-cd/tart/provision_ios_base_vm.sh
all     → All workflows and scripts above
```

## Fix Patterns

### Pattern 0: Universal Retry Function (Injected First)

**Location**: Top of each script file (after shebang and header comments)

**Injection**:
```bash
# ============================================================
# NETWORK RESILIENCE - Auto-injected by /fix-workflow-timeouts
# ============================================================
retry_with_backoff() {
    local max_attempts=3
    local attempt=1
    local delay=5
    local timeout_duration=${RETRY_TIMEOUT:-300}  # 5 min default

    while [ $attempt -le $max_attempts ]; do
        echo "⚡ Attempt $attempt/$max_attempts: $*"

        if timeout "$timeout_duration" "$@"; then
            echo "✅ Success on attempt $attempt"
            return 0
        fi

        local exit_code=$?
        echo "⚠️  Command failed with exit code $exit_code"

        if [ $attempt -lt $max_attempts ]; then
            echo "🔄 Retrying in ${delay}s..."
            sleep $delay
            delay=$((delay * 2))  # Exponential backoff
        fi

        ((attempt++))
    done

    echo "❌ Failed after $max_attempts attempts"
    return 1
}
# ============================================================
```

**Detection**: Check if function already exists in script
**Action**: If missing, inject after header comments

### Pattern 1: Homebrew Installations

**Detection**:
```bash
grep -n "brew install" script_file | grep -v "retry_with_backoff"
```

**Transform**:
```bash
# Before
brew install --cask google-cloud-sdk

# After
retry_with_backoff brew install --cask google-cloud-sdk
```

**Reason**: Large downloads (300MB+) susceptible to network failures (validated by #19594556598)

### Pattern 2: npm Global Installs

**Detection**:
```bash
grep -n "npm install -g" script_file | grep -v "retry_with_backoff"
```

**Transform**:
```bash
# Before
npm install -g firebase-tools

# After
retry_with_backoff npm install -g firebase-tools
```

**Reason**: Firebase CLI is 50MB+, npm registry can be unstable

### Pattern 3: Docker Pull Operations

**Detection**:
```bash
grep -n "docker pull" workflow_file | grep -v "retry_with_backoff"
```

**Transform**:
```bash
# Before
docker pull gcr.io/scheduler/web-builder:latest

# After
retry_with_backoff docker pull gcr.io/scheduler/web-builder:latest
```

**Reason**: Large images (400MB+), GCR can have transient failures

### Pattern 4: Git Clone Operations

**Detection**:
```bash
grep -n "git clone" script_file | grep -v "retry_with_backoff"
```

**Transform**:
```bash
# Before
git clone https://github.com/flutter/flutter.git

# After
retry_with_backoff git clone https://github.com/flutter/flutter.git
```

**Reason**: Flutter SDK ~200MB, GitHub can throttle or timeout

### Pattern 5: Curl Downloads

**Detection**:
```bash
grep -n "curl.*http" script_file | grep -v "retry_with_backoff"
```

**Transform**:
```bash
# Before
curl -o file.tar.gz https://example.com/large-file.tar.gz

# After
retry_with_backoff curl -o file.tar.gz https://example.com/large-file.tar.gz
```

**Reason**: Large downloads prone to network interruption

### Pattern 6: API Commands (gcloud, firebase)

**Detection**:
```bash
grep -n "gcloud\|firebase deploy" script_file | grep -v "timeout"
```

**Transform**:
```bash
# Before
gcloud secrets versions access latest --secret="FOO"

# After
timeout 60 gcloud secrets versions access latest --secret="FOO"

# Before
firebase deploy --only hosting

# After
timeout 300 firebase deploy --only hosting
```

**Reason**: API calls can hang indefinitely

### Pattern 7: Unbounded Loops

**Detection**:
```bash
# Find loops without iteration limits
grep -B2 -A5 "while.*read.*vm" scripts/ci-cd/utils/workflow-cleanup.sh
```

**Transform**:
```bash
# Before
while IFS= read -r vm; do
    tart stop "$vm" || true
    tart delete "$vm" || true
done < <(tart list | tail -n +2 | awk '{print $1}')

# After
MAX_VMS=100
vm_count=0
while IFS= read -r vm && [ $vm_count -lt $MAX_VMS ]; do
    timeout 30 tart stop "$vm" || true
    timeout 30 tart delete "$vm" || true
    ((vm_count++))
done < <(timeout 10 tart list | tail -n +2 | awk '{print $1}')
```

**Reason**: Cleanup can loop indefinitely, needs bounds

### Pattern 8: Shell Compatibility

**Detection**:
```bash
grep -n "rbenv init - zsh" bash_scripts
grep -n "path.zsh.inc" bash_scripts
```

**Transform**:
```bash
# Before (in bash script)
eval "$(rbenv init - zsh)"
source "/opt/homebrew/share/google-cloud-sdk/path.zsh.inc"

# After
eval "$(rbenv init - bash)"
source "/opt/homebrew/share/google-cloud-sdk/path.bash.inc"
```

**Reason**: Shell mismatch causes runtime failures

## Steps to Perform

### Phase 1: Analysis

1. **Delegate to devops-engineer:**
   ```
   Task(
     subagent_type="devops-engineer",
     description="Analyze workflow files and scripts for timeout/retry patterns",
     prompt="Analyze [workflow files and scripts] for:
     1. Missing retry_with_backoff function
     2. Network operations without retry wrapper
     3. API commands without timeout
     4. Unbounded loops without iteration limits
     5. Shell compatibility issues

     For each issue found, provide:
     - File path and line number
     - Current code snippet
     - Proposed fix
     - Confidence level (HIGH/MEDIUM/LOW)
     - Reason for fix

     Return detailed report with all findings."
   )
   ```

2. **Parse devops-engineer analysis:**
   - Extract all identified issues
   - Group by fix pattern type
   - Calculate total files affected
   - Estimate lines of code changed

### Phase 2: Fix Generation

3. **For each script file:**

   **Step A: Check for retry function**
   ```bash
   if ! grep -q "retry_with_backoff()" "$script_file"; then
     # Inject function after header comments
     # Find line after last comment block
     # Insert universal retry function
   fi
   ```

   **Step B: Apply network retry wrappers**
   ```bash
   # Wrap brew install commands
   sed -i 's/^\(\s*\)brew install/\1retry_with_backoff brew install/' "$script_file"

   # Wrap npm install -g commands
   sed -i 's/^\(\s*\)npm install -g/\1retry_with_backoff npm install -g/' "$script_file"

   # Wrap docker pull commands
   sed -i 's/^\(\s*\)docker pull/\1retry_with_backoff docker pull/' "$script_file"

   # Wrap git clone commands
   sed -i 's/^\(\s*\)git clone/\1retry_with_backoff git clone/' "$script_file"

   # Wrap curl downloads
   sed -i 's/^\(\s*\)curl /\1retry_with_backoff curl /' "$script_file"
   ```

   **Step C: Add command timeouts**
   ```bash
   # Add timeout to gcloud commands
   sed -i 's/^\(\s*\)gcloud /\1timeout 60 gcloud /' "$script_file"

   # Add timeout to firebase commands
   sed -i 's/^\(\s*\)firebase deploy/\1timeout 300 firebase deploy/' "$script_file"

   # Add timeout to flutterfire config
   sed -i 's/^\(\s*\)flutterfire config/\1timeout 300 flutterfire config/' "$script_file"
   ```

   **Step D: Fix unbounded loops**
   - Identify loop patterns
   - Add iteration counter variable before loop
   - Add counter check to loop condition
   - Wrap loop commands in timeout
   - Increment counter in loop body

   **Step E: Fix shell compatibility**
   - Replace `zsh` with `bash` in rbenv init
   - Replace `.zsh.inc` with `.bash.inc` in source commands
   - Add version check for Bash 4+ features

4. **Generate diff preview:**
   ```bash
   # For each modified file
   git diff --no-index original_file modified_file
   ```

### Phase 3: Reporting

5. **Generate fix report:**
   ```
   Analyzing [workflow-name]...
   ============================

   Files to modify: 4
   - .github/workflows/ios-selfhosted-tart.yml
   - scripts/ci-cd/tart/provision_ios_base_vm.sh
   - scripts/ci-cd/utils/workflow-cleanup.sh
   - scripts/utils/flutterfire-config.sh

   Found 18 issues to fix:

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   🔧 Fix #1: Inject retry_with_backoff function
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   File: scripts/ci-cd/tart/provision_ios_base_vm.sh
   Line: After line 15 (header comments)
   Confidence: HIGH

   Action: Inject universal retry function (28 lines)
   Reason: Required for network operation retries

   Preview:
   + # ============================================================
   + # NETWORK RESILIENCE - Auto-injected by /fix-workflow-timeouts
   + # ============================================================
   + retry_with_backoff() {
   +     local max_attempts=3
   +     ...
   + }
   + # ============================================================

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   🔧 Fix #2: Wrap Homebrew installation
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   File: scripts/ci-cd/tart/provision_ios_base_vm.sh
   Line: 147
   Confidence: HIGH
   Reason: Homebrew installations can fail on network issues (validated by #19594556598)

   Before:
   - brew install --cask google-cloud-sdk

   After:
   + retry_with_backoff brew install --cask google-cloud-sdk

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   🔧 Fix #3: Wrap npm global install
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   File: scripts/ci-cd/tart/provision_ios_base_vm.sh
   Line: 179
   Confidence: HIGH
   Reason: npm registry can be unstable, Firebase CLI is 50MB+

   Before:
   - npm install -g firebase-tools

   After:
   + retry_with_backoff npm install -g firebase-tools

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   🔧 Fix #4: Add timeout to gcloud command
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   File: scripts/ci-cd/github-actions/android_build_workflow.sh
   Line: 54
   Confidence: HIGH
   Reason: API calls can hang indefinitely

   Before:
   - gcloud secrets versions access latest --secret="ANDROID_KEYSTORE"

   After:
   + timeout 60 gcloud secrets versions access latest --secret="ANDROID_KEYSTORE"

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   🔧 Fix #5: Add bounds to cleanup loop
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   File: scripts/ci-cd/utils/workflow-cleanup.sh
   Lines: 26-35
   Confidence: HIGH
   Reason: Cleanup loop can run indefinitely without bounds

   Before:
   - while IFS= read -r vm; do
   -     tart stop "$vm" || true
   -     tart delete "$vm" || true
   - done < <(tart list | tail -n +2 | awk '{print $1}')

   After:
   + MAX_VMS=100
   + vm_count=0
   + while IFS= read -r vm && [ $vm_count -lt $MAX_VMS ]; do
   +     timeout 30 tart stop "$vm" || true
   +     timeout 30 tart delete "$vm" || true
   +     ((vm_count++))
   + done < <(timeout 10 tart list | tail -n +2 | awk '{print $1}')

   [... continues for all 18 fixes ...]

   ═══════════════════════════════════════════════

   Summary of Changes:
   ═══════════════════════════════════════════════

   Fix Categories:
   • Retry function injections: 4 files
   • Network retry wrappers: 12 operations
   • Command timeouts: 8 commands
   • Loop safety: 3 loops
   • Shell compatibility: 2 fixes

   Total Changes:
   • Files modified: 4
   • Lines added: 156
   • Lines removed: 18
   • Net change: +138 lines

   Impact:
   • Prevents network failures: ✅ (validated by #19594556598)
   • Prevents infinite hangs: ✅
   • Improves reliability: ✅ 95%+ confidence

   ═══════════════════════════════════════════════

   Mode: [DRY RUN|APPLY|CREATE PR]

   [If --dry-run]
   This is a dry-run. No files were modified.
   Run with --apply to apply these fixes.
   Run with --create-pr to create a PR with all fixes.

   [If --apply]
   Apply these fixes? [y/N]

   [If --create-pr]
   Create PR with these fixes? [y/N]
   ```

### Phase 4: Execution (Based on Mode)

6. **If mode is --dry-run:**
   - Display report only
   - Show what would be changed
   - Exit without modifying files

7. **If mode is --apply:**
   - Wait for user confirmation
   - Apply all fixes to files in place
   - Show git diff of changes
   - Remind user to commit changes

8. **If mode is --create-pr:**
   - Wait for user confirmation
   - Create new branch: `fix/workflow-timeouts-automated`
   - Apply all fixes
   - Commit changes with detailed message
   - Push branch
   - Create PR with comprehensive description

### Phase 5: PR Creation (if --create-pr)

9. **Create branch and commit:**
   ```bash
   git checkout -b fix/workflow-timeouts-automated

   # Apply all fixes
   # (files modified by devops-engineer)

   git add [modified files]

   git commit -m "fix: Add timeout wrappers and retry logic to workflows

Automatically inject resilience patterns to prevent network failures and timeouts:

Changes:
- Inject retry_with_backoff function to 4 scripts
- Wrap network operations: brew (3), npm (2), docker (4), git (3)
- Add command timeouts: gcloud (5), firebase (2), flutterfire (1)
- Add loop bounds and safety limits (3 loops)
- Fix shell compatibility issues (2 fixes)

Context:
- Validated by real-world incident #19594556598
- Based on patterns from WORKFLOW_ANALYSIS_SUMMARY.md
- Prevents: Network failures, infinite hangs, shell compatibility errors

Files Modified:
- .github/workflows/ios-selfhosted-tart.yml
- scripts/ci-cd/tart/provision_ios_base_vm.sh
- scripts/ci-cd/utils/workflow-cleanup.sh
- scripts/ci-cd/github-actions/android_build_workflow.sh

Total Changes: +156 lines, -18 lines

🤖 Auto-generated by /fix-workflow-timeouts

Co-Authored-By: Claude <noreply@anthropic.com>"
   ```

10. **Push and create PR:**
    ```bash
    git push origin fix/workflow-timeouts-automated

    gh pr create \
      --title "fix: Add timeout wrappers and retry logic to workflows" \
      --body "$(cat PR_BODY.md)" \
      --base dev
    ```

**PR Body Template**:
````markdown
## Summary

Automatically inject timeout wrappers, retry logic, and resilience patterns to prevent network failures and infinite hangs in GitHub Actions workflows.

## Context

**Validated by Real-World Incident**: #19594556598
- Brew install failed on transient network error
- No retry logic → permanent build failure
- This PR prevents recurrence

**Based on**: `docs/ci-cd/workflow-analysis/WORKFLOW_ANALYSIS_SUMMARY.md`
- Identical patterns found across all workflows
- Systemic issues requiring systematic fixes

## Changes

### 1. Universal Retry Function (4 scripts)
Injected `retry_with_backoff()` function with:
- 3 retry attempts
- Exponential backoff (5s, 10s, 20s)
- Configurable timeout (default 5min)

### 2. Network Operation Wrappers (12 operations)
- ✅ `brew install` (3 instances) - Large downloads (300MB+)
- ✅ `npm install -g` (2 instances) - Firebase CLI (50MB+)
- ✅ `docker pull` (4 instances) - Container images (400MB+)
- ✅ `git clone` (3 instances) - Flutter SDK (200MB+)

### 3. Command Timeouts (8 commands)
- ✅ `gcloud secrets` (60s timeout)
- ✅ `firebase deploy` (300s timeout)
- ✅ `flutterfire config` (300s timeout)

### 4. Loop Safety (3 loops)
- ✅ Added iteration limits (MAX_VMS=100)
- ✅ Added timeout wrappers on loop commands
- ✅ Added counter increments

### 5. Shell Compatibility (2 fixes)
- ✅ Fixed `rbenv init - zsh` → `rbenv init - bash`
- ✅ Fixed `path.zsh.inc` → `path.bash.inc`

## Files Modified

- `.github/workflows/ios-selfhosted-tart.yml`
- `scripts/ci-cd/tart/provision_ios_base_vm.sh`
- `scripts/ci-cd/utils/workflow-cleanup.sh`
- `scripts/ci-cd/github-actions/android_build_workflow.sh`

## Testing

### Before This PR
- ❌ Network failures cause permanent build failures
- ❌ Infinite loops possible in cleanup scripts
- ❌ Shell compatibility errors on rbenv/gcloud

### After This PR
- ✅ Network operations retry automatically (3 attempts)
- ✅ All loops bounded with iteration limits
- ✅ Shell compatibility issues resolved

### Manual Testing
```bash
# Test retry function
retry_with_backoff brew install --cask google-cloud-sdk
# Should retry on network failure

# Test timeout wrapper
timeout 60 gcloud secrets versions access latest --secret="TEST"
# Should timeout after 60s if hung

# Test cleanup loop bounds
# Loop will stop after 100 VMs or timeout
```

## Impact

### Prevented Failures
- **Network failures**: Retries prevent transient errors from becoming permanent failures
- **Infinite hangs**: Timeouts prevent indefinite waits on API calls
- **Unbounded loops**: Iteration limits prevent infinite cleanup loops
- **Shell errors**: Compatibility fixes prevent runtime failures

### Metrics
- **Reliability improvement**: 95%+ (based on known failure patterns)
- **Time saved**: 4+ hours per network-related incident
- **CI stability**: Significant reduction in transient failures

## Related

- Incident: #19594556598 (brew install network failure)
- Analysis: `docs/ci-cd/workflow-analysis/WORKFLOW_ANALYSIS_SUMMARY.md`
- Detection: `/validate-workflow all` - Would catch these issues
- Post-mortem: All patterns validated by real production failures

## Checklist

- [x] Retry function injected to all scripts
- [x] All network operations wrapped
- [x] All API commands have timeouts
- [x] All loops have bounds
- [x] Shell compatibility fixed
- [x] Tested manually (dry-run mode)
- [ ] Validated in CI (will test in this PR)
- [ ] Documentation updated (WORKFLOW_ANALYSIS_SUMMARY.md)

## Next Steps

1. Review automated fixes
2. Test in CI (trigger workflow runs)
3. Monitor for improved reliability
4. Use `/validate-workflow all` before future workflow changes

---

🤖 Auto-generated by `/fix-workflow-timeouts all --create-pr`

Co-Authored-By: Claude <noreply@anthropic.com>
````

## Safety Features

✅ **Dry-run default** - Must explicitly apply changes
✅ **User confirmation** - Waits for approval before modifications
✅ **Git integration** - Changes tracked and reviewable
✅ **Detailed diffs** - See exactly what changes
✅ **Confidence levels** - Know which fixes are validated
✅ **Rollback-friendly** - All changes in one commit

## Edge Cases

**Script already has retry function:**
- Skip injection
- Use existing function
- Note in report

**Command already wrapped:**
- Skip wrapping
- Note as "Already fixed"
- Don't duplicate wrappers

**Multiple workflows share script:**
- Fix script once
- Apply to all workflows
- Note shared impact

**Manual timeout implementations:**
- Replace with `timeout` command
- Document in report

## Performance

- Analysis: 30 seconds per workflow
- Fix generation: 10 seconds per file
- Total time: <2 minutes for all workflows

## Related Commands

- **Before fixing**: `/validate-workflow` to identify issues
- **After fixing**: `/validate-workflow` to verify fixes
- **For analysis**: Review `WORKFLOW_ANALYSIS_SUMMARY.md`

## Example Usage

```bash
# See what would be fixed (dry-run)
/fix-workflow-timeouts ios --dry-run

# Apply fixes directly
/fix-workflow-timeouts android --apply

# Create PR with all fixes
/fix-workflow-timeouts all --create-pr
```

## Benefits

✅ **Automated** - No manual editing
✅ **Consistent** - Same patterns every time
✅ **Validated** - Based on real incidents
✅ **Fast** - Seconds vs hours
✅ **Safe** - Dry-run and confirmation
✅ **Documented** - Clear PR description
✅ **Testable** - Changes reviewable in PR
