---
description: Update GitHub issue fields from PR data using gh CLI
argument-hint: "[pr-number]"
allowed-tools: Bash, Task
---

# Update Ticket

Update GitHub issue fields (labels, story points, assignee, project status) from GitHub PR data using gh CLI.

## Variables

- **PR_NUMBER**: First argument from $ARGUMENTS (optional)
  - If provided, work with the specified PR number
  - If not provided, work with the current branch's PR

## Usage

```bash
# Update ticket for current branch's PR
/update-ticket

# Update ticket for specific PR
/update-ticket 326
```

## What This Command Does

This command:
1. **Fetches PR data** from GitHub using gh CLI
2. **Extracts issue number** from PR title/branch/body
3. **Gets issue details** from GitHub
4. **Prompts user** for field updates (labels, story points, project)
5. **Updates GitHub fields** using gh CLI
6. **Verifies changes** and reports results

## Tool Selection

**Why gh CLI:**
- Direct GitHub API integration
- No browser automation needed
- Reliable field updates
- Fast execution
- Simple error handling

## Phase 1: Extract PR and Issue Information

**Get PR details:**

```bash
unset GITHUB_TOKEN

# Get PR number
if [ -z "$1" ]; then
  PR_NUMBER=$(gh pr view --json number --jq '.number' 2>/dev/null)
else
  PR_NUMBER=$1
fi

if [ -z "$PR_NUMBER" ]; then
  echo "No PR found. Provide PR number or run from branch with open PR."
  exit 1
fi

# Fetch PR data
PR_DATA=$(gh pr view $PR_NUMBER --json number,title,headRefName,body,createdAt)

PR_TITLE=$(echo "$PR_DATA" | jq -r '.title')
BRANCH_NAME=$(echo "$PR_DATA" | jq -r '.headRefName')
PR_BODY=$(echo "$PR_DATA" | jq -r '.body // ""')
CREATED_AT=$(echo "$PR_DATA" | jq -r '.createdAt')

echo "PR #$PR_NUMBER: $PR_TITLE"
echo "Branch: $BRANCH_NAME"
```

**Extract issue number:**

```bash
# Try from PR body first (Closes #XXX format)
ISSUE_NUMBER=$(echo "$PR_BODY" | grep -oE 'Closes #[0-9]+|Fixes #[0-9]+|Resolves #[0-9]+' | grep -oE '[0-9]+' | head -1)

# Fallback to title (e.g., "[#123] Description" or "#123: Description")
if [ -z "$ISSUE_NUMBER" ]; then
  ISSUE_NUMBER=$(echo "$PR_TITLE" | grep -oE '#[0-9]+|\[#[0-9]+\]' | grep -oE '[0-9]+' | head -1)
fi

# Fallback to branch name (e.g., "123-feature-name")
if [ -z "$ISSUE_NUMBER" ]; then
  ISSUE_NUMBER=$(echo "$BRANCH_NAME" | grep -oE '^[0-9]+' | head -1)
fi

if [ -z "$ISSUE_NUMBER" ]; then
  echo "No GitHub issue found in PR body, title, or branch name"
  exit 1
fi

echo "GitHub Issue: #$ISSUE_NUMBER"
```

## Phase 2: Fetch Current Issue State

**Fetch issue details:**

```bash
ISSUE_DATA=$(gh issue view $ISSUE_NUMBER --json number,title,state,labels,assignees,projectItems)

ISSUE_TITLE=$(echo "$ISSUE_DATA" | jq -r '.title')
ISSUE_STATE=$(echo "$ISSUE_DATA" | jq -r '.state')
CURRENT_LABELS=$(echo "$ISSUE_DATA" | jq -r '.labels[].name' | tr '\n' ',' | sed 's/,$//')
CURRENT_ASSIGNEES=$(echo "$ISSUE_DATA" | jq -r '.assignees[].login' | tr '\n' ',' | sed 's/,$//')

# Extract story points from labels (sp:X format)
CURRENT_STORY_POINTS=$(echo "$CURRENT_LABELS" | grep -oE 'sp:[0-9]+' | grep -oE '[0-9]+' | head -1)
```

**Display current state:**

```bash
echo ""
echo "Current Issue State:"
echo "  Title: $ISSUE_TITLE"
echo "  State: $ISSUE_STATE"
echo "  Labels: ${CURRENT_LABELS:-None}"
echo "  Story Points: ${CURRENT_STORY_POINTS:-Not set}"
echo "  Assignees: ${CURRENT_ASSIGNEES:-None}"
echo ""
```

## Phase 3: Interactive Field Updates

**Prompt user for updates:**

```bash
echo "Update GitHub Issue Fields (press Enter to skip):"
echo ""

# Story Points (via labels)
echo "Story Points (1, 2, 3, 5, 8, 13):"
read -p "Story Points [${CURRENT_STORY_POINTS}]: " NEW_STORY_POINTS
STORY_POINTS_UPDATE=${NEW_STORY_POINTS:-$CURRENT_STORY_POINTS}

# Status labels
echo ""
echo "Status labels: status:todo, status:in-progress, status:done, status:blocked"
read -p "Add status label (or Enter to skip): " STATUS_LABEL

# Priority labels
echo ""
echo "Priority labels: P0, P1, P2, P3"
read -p "Add priority label (or Enter to skip): " PRIORITY_LABEL

# Assignee
echo ""
read -p "Assign to (GitHub username, or Enter to skip): " NEW_ASSIGNEE
```

## Phase 4: Update GitHub Issue

**Execute updates:**

```bash
UPDATE_CMDS=""

# Update story points label (remove old, add new)
if [ -n "$STORY_POINTS_UPDATE" ] && [ "$STORY_POINTS_UPDATE" != "$CURRENT_STORY_POINTS" ]; then
  # Remove old story points label if exists
  if [ -n "$CURRENT_STORY_POINTS" ]; then
    gh issue edit $ISSUE_NUMBER --remove-label "sp:$CURRENT_STORY_POINTS" 2>/dev/null || true
  fi
  # Add new story points label
  gh issue edit $ISSUE_NUMBER --add-label "sp:$STORY_POINTS_UPDATE"
  echo "Updated story points: $CURRENT_STORY_POINTS -> $STORY_POINTS_UPDATE"
fi

# Add status label
if [ -n "$STATUS_LABEL" ]; then
  gh issue edit $ISSUE_NUMBER --add-label "$STATUS_LABEL"
  echo "Added label: $STATUS_LABEL"
fi

# Add priority label
if [ -n "$PRIORITY_LABEL" ]; then
  gh issue edit $ISSUE_NUMBER --add-label "$PRIORITY_LABEL"
  echo "Added label: $PRIORITY_LABEL"
fi

# Update assignee
if [ -n "$NEW_ASSIGNEE" ]; then
  gh issue edit $ISSUE_NUMBER --add-assignee "$NEW_ASSIGNEE"
  echo "Assigned to: $NEW_ASSIGNEE"
fi
```

## Phase 5: Verification and Reporting

**Fetch updated issue:**

```bash
UPDATED_DATA=$(gh issue view $ISSUE_NUMBER --json labels,assignees)
UPDATED_LABELS=$(echo "$UPDATED_DATA" | jq -r '.labels[].name' | tr '\n' ',' | sed 's/,$//')
UPDATED_ASSIGNEES=$(echo "$UPDATED_DATA" | jq -r '.assignees[].login' | tr '\n' ',' | sed 's/,$//')
```

**Display results:**

```bash
echo ""
echo "Issue Update Complete"
echo ""
echo "Issue: #$ISSUE_NUMBER - $ISSUE_TITLE"
echo "PR: #$PR_NUMBER - $PR_TITLE"
echo ""
echo "Updated Fields:"
echo "  Labels: $UPDATED_LABELS"
echo "  Assignees: $UPDATED_ASSIGNEES"
echo ""
echo "View issue: https://github.com/$(gh repo view --json nameWithOwner --jq '.nameWithOwner')/issues/$ISSUE_NUMBER"
echo ""
```

## Project Board Updates (Optional)

**If using GitHub Projects v2:**

```bash
# Get project ID (replace with your project number)
PROJECT_NUMBER=1
PROJECT_ID=$(gh project list --owner "$(gh repo view --json owner --jq '.owner.login')" --format json | jq -r ".projects[] | select(.number == $PROJECT_NUMBER) | .id")

if [ -n "$PROJECT_ID" ]; then
  # Add issue to project
  gh project item-add $PROJECT_NUMBER --owner "$(gh repo view --json owner --jq '.owner.login')" --url "$(gh issue view $ISSUE_NUMBER --json url --jq '.url')"

  # Update sprint field (if configured)
  # gh project item-edit --project-id $PROJECT_ID --id $ITEM_ID --field-id $SPRINT_FIELD_ID --single-select-option-id $SPRINT_OPTION_ID
fi
```

## Error Handling

### Issue Not Found

```
If gh issue view returns error:
  -> Error: "Issue #$ISSUE_NUMBER not found"
  -> Suggest: Verify issue number exists
  -> Exit: Cannot update non-existent issue
```

### Label Creation

```bash
# If label doesn't exist, create it
if ! gh label list | grep -q "sp:$STORY_POINTS_UPDATE"; then
  gh label create "sp:$STORY_POINTS_UPDATE" --color "0052CC" --description "Story points: $STORY_POINTS_UPDATE"
fi
```

### Permission Issues

```
If gh issue edit fails with permission error:
  -> Error: "Insufficient permissions to edit issue"
  -> Suggest: Verify you have write access to the repository
  -> Check: gh auth status
```

## Example Workflows

### Simple Story Points Update

```bash
/update-ticket 326

# Output:
PR #326: [#265] Implement advanced scheduling
Branch: 265-advanced-scheduling
GitHub Issue: #265

Current Issue State:
  Title: Implement advanced scheduling
  State: OPEN
  Labels: enhancement
  Story Points: Not set
  Assignees: None

Update GitHub Issue Fields (press Enter to skip):

Story Points (1, 2, 3, 5, 8, 13):
Story Points []: 5

Issue Update Complete

Updated Fields:
  Labels: enhancement,sp:5

View issue: https://github.com/org/repo/issues/265
```

### Complete Issue Setup

```bash
/update-ticket

# User enters:
Story Points: 8
Status label: status:in-progress
Priority label: P1
Assignee: developer-username

# Output:
Issue Update Complete

Updated Fields:
  Labels: enhancement,sp:8,status:in-progress,P1
  Assignees: developer-username
```

## Benefits

- **Simple**: Direct gh CLI calls
- **Fast**: No browser automation
- **Reliable**: GitHub API is stable
- **Flexible**: Skip any field by pressing Enter
- **Verified**: Fetches updated issue to confirm success

## When NOT to Use This Command

- **Bulk issue updates**: Use GitHub bulk edit or API scripts
- **Complex project board operations**: Use GitHub Projects UI
- **Automated CI updates**: Use GitHub Actions with gh CLI

## Direct Execution Instructions

1. **Extract**: Get PR number and fetch PR data with gh CLI
2. **Identify**: Extract issue number from PR body/title/branch
3. **Fetch**: Get current issue state with gh issue view
4. **Prompt**: Interactive user input for field updates
5. **Update**: gh issue edit calls per field
6. **Verify**: Fetch updated issue to confirm changes
7. **Report**: Display summary with updated values
