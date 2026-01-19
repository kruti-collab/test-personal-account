---
description: Systematic workflow for implementing DevOps features in unified devops branch
argument-hint: "[feature-description]"
---

# Implement DevOps Feature

Systematic workflow for implementing DevOps features, tools, and infrastructure improvements following the unified devops branch strategy.

## Workflow Overview

This command guides you through:
1. Analyzing the feature request
2. Finding or creating appropriate GitHub issue under #258 Epic
3. Implementing changes in `devops` branch
4. Committing with proper ticket reference
5. Updating GitHub with progress

## Variables

- **FEATURE_DESCRIPTION**: First argument from $ARGUMENTS (optional)
  - Brief description of what needs to be implemented
  - If not provided, will ask user for details

## Steps to perform:

### Phase 1: Feature Analysis

1. **Get feature details:**
   - If FEATURE_DESCRIPTION provided: Use it as starting point
   - If not provided: Ask user: "What DevOps feature needs to be implemented?"
   - Ask clarifying questions:
     - What problem does this solve?
     - What files/systems will be affected?
     - Is this a new feature or enhancement?

2. **Categorize feature:**
   - **Slash commands** (.claude/commands/)
   - **GitHub Actions** (.github/workflows/)
   - **Documentation** (.github/, docs/)
   - **Agents** (.claude/agents/)
   - **Hooks** (.claude/hooks/)
   - **Build/Deploy** (scripts/, cloudbuild.yaml)
   - **Infrastructure** (Docker, DevContainer, etc.)

### Phase 2: Ticket Management

3. **Search for existing ticket:**
   ```bash
   gh issue list --search "project = SMR AND status != Done AND summary ~ 'keyword'"
   ```
   - Search based on feature keywords
   - Check Epic #258 for related tickets

4. **Decision: Use existing or create new:**
   - Show found tickets to user
   - Ask: "Use existing ticket or create new?"

5. **If creating new ticket:**
   ```bash
   gh issue create \
     --type Task \
     --project SMR \
     --summary "[Feature summary]" \
     --priority Medium
   ```
   - Link to Epic #258:
     ```bash
     # gh project item-add #258 #XXX
     ```

6. **If using existing ticket:**
   - Confirm ticket number with user
   - Verify it's not Done
   - Move to "In Progress" if not already:
     ```bash
     gh issue edit #XXX "In Progress"
     ```

### Phase 3: Environment Setup

7. **Verify devops branch:**
   ```bash
   git checkout devops
   git pull origin devops
   ```
   - Ensure working directory is clean
   - Ensure latest changes pulled

8. **Show current context:**
   ```
   Current Environment
   ===================

   Branch: devops
   Ticket: #XXX
   Epic: #258 (DevOps)
   Working in unified DevOps branch

   No new branch needed - work directly in devops.
   ```

### Phase 4: Implementation

9. **Guide implementation:**
   - Based on category, suggest structure:

   **For Slash Commands:**
   ```markdown
   Create: .claude/commands/[category]/[command-name].md
   Include:
   - Frontmatter (description, argument-hint)
   - Clear phases/steps
   - Error handling
   - Examples
   ```

   **For GitHub Actions:**
   ```yaml
   Create: .github/workflows/[workflow-name].yml
   Include:
   - Clear triggers
   - Job names
   - Security considerations
   - Documentation link
   ```

   **For Documentation:**
   ```markdown
   Update: README.md, CONTRIBUTING.md, or docs/
   Include:
   - Clear examples
   - Setup instructions
   - Troubleshooting
   ```

10. **Implementation checklist:**
    - [ ] Files created/modified
    - [ ] Documentation updated
    - [ ] Examples included
    - [ ] Tested locally (if applicable)

### Phase 5: Commit & Push

11. **Review changes:**
    ```bash
    git status
    git diff
    ```
    - Show user what's changed

12. **Stage files:**
    ```bash
    git add [files]
    ```
    - Only stage relevant files

13. **Create commit message:**
    ```
    #XXX: [Brief summary]

    [Detailed description of changes]

    - [Change 1]
    - [Change 2]

    🤖 Generated with [Claude Code](https://claude.com/claude-code)

    Co-Authored-By: Claude <noreply@anthropic.com>
    ```

14. **Commit:**
    ```bash
    git commit -m "$(cat commit-message.txt)"
    ```

15. **Push to devops:**
    ```bash
    git push origin devops
    ```

### Phase 6: GitHub Update

16. **Add comment to GitHub:**
    ```bash
    gh issue comment #XXX "Implemented [feature name]

Changes:
- [File 1]: [description]
- [File 2]: [description]

Commit: [commit-hash]

Status: Ready for testing"
    ```

17. **Ask about status:**
    - "Is this feature complete or in progress?"
    - If complete: Move to "Done"
      ```bash
      gh issue edit #XXX "Done"
      ```
    - If in progress: Keep in "In Progress"

### Phase 7: Summary

18. **Display completion report:**
    ```
    ✅ DevOps Feature Implementation Complete
    ========================================

    Ticket: #XXX
    Feature: [Feature name]
    Branch: devops
    Commit: [hash]

    Files Changed:
    - [file1]
    - [file2]

    GitHub Status: [In Progress/Done]

    Next Steps:
    - Feature is now in devops branch
    - Will be included in next devops → dev merge
    - Test the feature: [testing instructions]

    Commands for reference:
    - View commit: git show [hash]
    - View ticket: gh issue view #XXX
    - Check branch: git log devops --oneline -5
    ```

## Special Cases

### If feature requires multiple commits:
- Keep ticket "In Progress"
- Add comment for each significant commit
- Move to "Done" only when fully complete

### If feature is enhancement to existing ticket:
- Reopen ticket if Done:
  ```bash
  gh issue edit #XXX "In Progress"
  ```
- Add comment explaining enhancement

### If feature requires restart:
- Add note in commit message:
  ```
  ⚠️ REQUIRES CLAUDE CODE RESTART
  Changes to .claude/ configuration
  ```

## Integration with Other Commands

This command works alongside:
- **After /review-pr-scope**: Implement fixes for identified issues
- **Before /describe-pr**: Implement feature, then describe in PR
- **With /check-branch-health**: Ensure devops stays healthy

## Example Usage

```bash
# With description
/implement-devops-feature "Add slash command for cleaning stale branches"

# Without description (interactive)
/implement-devops-feature
> What feature? "Automated deployment rollback"
```

## Benefits

✅ **Systematic**: Consistent workflow every time
✅ **Tracked**: Always linked to GitHub issue
✅ **Documented**: Proper commit messages and comments
✅ **Unified**: Works directly in devops branch
✅ **Visible**: Progress tracked in GitHub
✅ **Complete**: From idea to commit in one workflow

## When NOT to use this command

- For non-DevOps features (use regular feature workflow)
- For emergency hotfixes (use manual approach)
- For experimental changes (create separate branch first)
