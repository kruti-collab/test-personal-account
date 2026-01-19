---
description: Recreate PR from clean base (wrong base or messy merge history)
argument-hint: "[pr-number] [commits-to-keep...]"
---

# Recreate PR

Recreate a PR cleanly when the original branch is unrecoverable due to wrong base or excessive merge commits.

## Use Cases

This command is for scenarios where:

**Scenario 1: Wrong Base Branch**
- Branch was created from a stale/wrong commit
- Branch includes already-merged work from other tickets
- PR has 100+ unrelated commits (like #322 with 590 instead of 1)

**Scenario 2: Messy Merge History**
- Branch has real work but is too far behind dev (1000+ commits)
- Multiple merge commits polluting history
- PR closed due to conflicts or review issues
- Rebase is impractical but work needs to be preserved

**Common trait:** Simpler to recreate than to clean up

## Variables

- **PR_NUMBER**: First argument from $ARGUMENTS (required)
  - The PR number to recreate

- **COMMITS_TO_KEEP**: Remaining arguments (required)
  - Space-separated list of commit hashes to cherry-pick
  - Example: `a98082c3` or `a98082c3 b1d4c6b7`

## Steps to perform:

### Phase 1: Analysis & Validation

1. **Get PR information:**
   ```bash
   gh pr view $PR_NUMBER --json number,title,body,headRefName,baseRefName
   ```
   - Extract branch name, base branch, title, description

2. **Validate commits exist:**
   ```bash
   for commit in $COMMITS_TO_KEEP; do
     git rev-parse --verify $commit
   done
   ```
   - Exit with error if any commit doesn't exist

3. **Show what will be kept:**
   ```bash
   for commit in $COMMITS_TO_KEEP; do
     git show $commit --stat
   done
   ```
   - Display each commit's changes for user review

### Phase 2: User Confirmation

4. **Present plan to user:**
   ```
   Recreate PR Plan
   ================

   Original PR: #$PR_NUMBER
   Branch: $HEAD_BRANCH
   Base: $BASE_BRANCH

   Commits to keep ($COMMIT_COUNT):
   - $COMMIT_1: [message]
   - $COMMIT_2: [message]

   Actions:
   1. Fetch latest base branch
   2. Create new branch: $HEAD_BRANCH-clean from $BASE_BRANCH
   3. Cherry-pick: $COMMITS_TO_KEEP
   4. Push new branch
   5. Create new PR with same title/description
   6. Close old PR #$PR_NUMBER
   7. Delete old remote branch $HEAD_BRANCH

   Note: Old commits remain accessible via git reflog and commit hashes.

   Proceed? (yes/no):
   ```

5. **Wait for explicit approval:**
   - Only proceed if user types "yes"
   - Exit if "no" or any other response

### Phase 3: Execution

6. **Fetch latest base:**
   ```bash
   git fetch origin $BASE_BRANCH
   ```

7. **Create clean branch:**
   ```bash
   git checkout -b $HEAD_BRANCH-clean origin/$BASE_BRANCH
   ```

8. **Cherry-pick commits:**
   ```bash
   for commit in $COMMITS_TO_KEEP; do
     git cherry-pick $commit
     if [ $? -ne 0 ]; then
       echo "ERROR: Cherry-pick failed for $commit"
       echo "Resolve conflicts and run: git cherry-pick --continue"
       exit 1
     fi
   done
   ```
   - Stop on conflict, give user instructions

9. **Push new branch:**
    ```bash
    git push -u origin $HEAD_BRANCH-clean
    ```

10. **Create new PR:**
    ```bash
    gh pr create --base $BASE_BRANCH \
      --head $HEAD_BRANCH-clean \
      --title "$PR_TITLE" \
      --body "$PR_BODY

---

**Note**: Recreated from PR #$PR_NUMBER due to unrecoverable branch state.
Original branch had $OLD_COMMIT_COUNT commits, kept $COMMIT_COUNT relevant commits.
Old commits remain accessible via git reflog.

**Reason for recreation**: [wrong base branch | messy merge history]"
    ```

11. **Close old PR:**
    ```bash
    gh pr close $PR_NUMBER --comment "Closing - recreated as #$NEW_PR_NUMBER due to unrecoverable branch state.

Original branch had $OLD_COMMIT_COUNT commits (including merges/unrelated work).
New clean branch has $COMMIT_COUNT relevant commits cherry-picked from fresh base.

**Reason**: [Wrong base branch | Too many merge commits and conflicts]

New clean PR: #$NEW_PR_NUMBER

Note: Old commits remain in git history and reflog if recovery is needed."
    ```

12. **Delete old branch:**
    ```bash
    git push origin --delete $HEAD_BRANCH
    ```
    - Deletes remote branch only
    - Old commits remain accessible via reflog and commit hashes

### Phase 4: Report

13. **Display summary:**
    ```
    ✅ PR Recreation Complete
    ========================

    Old PR: #$PR_NUMBER (closed)
    New PR: #$NEW_PR_NUMBER

    Branch: $HEAD_BRANCH-clean
    Base: $BASE_BRANCH
    Commits: $COMMIT_COUNT

    Actions taken:
    - ✅ Fetched latest base branch
    - ✅ Created clean branch from $BASE_BRANCH
    - ✅ Cherry-picked $COMMIT_COUNT commits
    - ✅ Pushed new branch
    - ✅ Created new PR #$NEW_PR_NUMBER
    - ✅ Closed old PR #$PR_NUMBER
    - ✅ Deleted old remote branch

    Recovery options (if needed):
    - Old commits: git reflog
    - Restore from hash: git checkout <commit-hash>

    Next steps:
    - Review new PR: gh pr view $NEW_PR_NUMBER
    - Clean up local: git branch -D $HEAD_BRANCH (if exists)
    ```

## Error Handling

### If cherry-pick fails:
1. Stop execution
2. Show conflict details
3. Provide resolution commands:
   ```bash
   # Resolve conflicts manually, then:
   git cherry-pick --continue

   # Or abort and restore:
   git cherry-pick --abort
   git checkout devops
   git branch -D $HEAD_BRANCH-clean
   ```

### If PR creation fails:
1. Branch is pushed but PR not created
2. User can manually create PR:
   ```bash
   gh pr create --base $BASE_BRANCH --head $HEAD_BRANCH-clean
   ```

### Rollback if needed:
```bash
# Find old commit hash from reflog or closed PR
git reflog | grep $HEAD_BRANCH
# or: gh pr view $PR_NUMBER

# Restore from commit hash
git checkout <old-commit-hash>
git checkout -b $HEAD_BRANCH-restored
git push -f origin $HEAD_BRANCH-restored:$HEAD_BRANCH

# Reopen old PR (manual via GitHub UI)

# Delete new branch
git push origin --delete $HEAD_BRANCH-clean
git branch -D $HEAD_BRANCH-clean
```

## Safety Features

- ✅ Validates all commits exist before starting
- ✅ Shows preview of what will be kept
- ✅ Requires explicit user confirmation
- ✅ Stops on cherry-pick conflicts
- ✅ Preserves old PR as closed (not deleted)
- ✅ Links new PR to old PR for traceability
- ✅ Old commits remain in git history (reflog + commit hashes)

## Example Usage

```bash
# Recreate PR #249 with only 1 commit
/recreate-pr 249 a98082c3

# Recreate with multiple commits
/recreate-pr 250 b1d4c6b7 8328d83a
```

## When NOT to use this command

Use `/split-pr` instead if:
- You want to keep the original PR
- Changes should be split into multiple PRs
- Original branch has reasonable number of commits (<50)

Use manual cherry-pick if:
- You need fine-grained control over each commit
- Commits need modification during recreation
- Complex conflict resolution expected
