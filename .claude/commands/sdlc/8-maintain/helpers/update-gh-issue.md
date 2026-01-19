---
description: Configure GitHub issue fields (assignee, milestone, project, type, priority, linked PR)
argument-hint: "<issue-number>"
---

# Update GitHub Issue Fields

Configure all fields for a GitHub issue: assignee, milestone, project board, issue type, priority, status, and linked PR.

## Arguments

- `$ARGUMENTS`: GitHub issue number to update

## Workflow

Execute these steps in order:

### Step 1: Detect Repository and Issue

```bash
# Get current repo
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
ISSUE_NUM=$ARGUMENTS

if [ -z "$ISSUE_NUM" ]; then
  echo "❌ Usage: /update-gh-issue <issue-number>"
  exit 1
fi

# Fetch issue details
ISSUE_DATA=$(gh issue view $ISSUE_NUM --json number,title,assignees,labels,milestone,projectItems,body,url)
echo "📋 Issue #$ISSUE_NUM: $(echo $ISSUE_DATA | jq -r '.title')"
```

### Step 2: Check for Linked PRs (Development field)

```bash
# Find PRs that reference this issue
LINKED_PRS=$(gh pr list --search "closes:#$ISSUE_NUM OR fixes:#$ISSUE_NUM OR resolves:#$ISSUE_NUM" --json number,url,state --jq '.')
echo "🔗 Linked PRs: $(echo $LINKED_PRS | jq -r 'map("#\(.number) (\(.state))") | join(", ") // "None"')"
```

### Step 3: Set Assignee

```bash
# Assign current user if not assigned
CURRENT_ASSIGNEES=$(echo $ISSUE_DATA | jq -r '.assignees[].login // empty')
if [ -z "$CURRENT_ASSIGNEES" ]; then
  gh issue edit $ISSUE_NUM --add-assignee @me
  echo "✅ Assigned to: @me"
else
  echo "👤 Already assigned to: $CURRENT_ASSIGNEES"
fi
```

### Step 4: Set Milestone (sync with GAL)

```bash
# Get GAL milestone as reference
GAL_MILESTONE=$(gh api /repos/Scheduler-Systems/gal/milestones --jq '.[0].title' 2>/dev/null || echo "")

# Check if milestone exists in current repo, create if not
CURRENT_MILESTONE=$(echo $ISSUE_DATA | jq -r '.milestone.title // empty')
if [ -z "$CURRENT_MILESTONE" ] && [ -n "$GAL_MILESTONE" ]; then
  # Create milestone if it doesn't exist
  gh api /repos/${REPO}/milestones --jq ".[] | select(.title==\"$GAL_MILESTONE\") | .title" 2>/dev/null || \
    gh api -X POST /repos/${REPO}/milestones -f title="$GAL_MILESTONE" 2>/dev/null

  gh issue edit $ISSUE_NUM --milestone "$GAL_MILESTONE"
  echo "✅ Milestone: $GAL_MILESTONE"
else
  echo "🏁 Milestone: ${CURRENT_MILESTONE:-Not set}"
fi
```

### Step 5: Add to Project Board

Determine project based on repository:

| Repository | Project Name | Project Number |
|------------|--------------|----------------|
| scheduler-systems-infra | Infrastructure | 9 |
| gal | GAL - Governance Agentic Layer | 5 |
| tal | TAL (Triggering Agentic Layer) | 14 |
| scheduler-systems-web | Company Website | 7 |
| Scheduler | Scheduler App | 4 |
| lobali | Lobali App | 10 |

```bash
# Map repo to project number
REPO_NAME=$(echo $REPO | cut -d'/' -f2)
case $REPO_NAME in
  scheduler-systems-infra) PROJECT_NUM=9 ;;
  gal) PROJECT_NUM=5 ;;
  tal) PROJECT_NUM=14 ;;
  scheduler-systems-web) PROJECT_NUM=7 ;;
  Scheduler) PROJECT_NUM=4 ;;
  lobali) PROJECT_NUM=10 ;;
  *) PROJECT_NUM="" ;;
esac

if [ -n "$PROJECT_NUM" ]; then
  ISSUE_URL=$(echo $ISSUE_DATA | jq -r '.url')
  gh project item-add $PROJECT_NUM --owner Scheduler-Systems --url "$ISSUE_URL" 2>/dev/null || true
  echo "✅ Added to project #$PROJECT_NUM"
fi
```

### Step 6: Set Project Fields (Type, Priority, Status)

Use GraphQL to set project-specific fields:

```bash
# Get project item ID
ITEM_ID=$(gh project item-list $PROJECT_NUM --owner Scheduler-Systems --format json | \
  jq -r ".items[] | select(.content.number == $ISSUE_NUM and .content.repository == \"$REPO_NAME\") | .id")

if [ -n "$ITEM_ID" ]; then
  # Project IDs and Field IDs (Infrastructure project example)
  PROJECT_ID="PVT_kwDODVjuFs4BLvId"

  # Field IDs
  STATUS_FIELD="PVTSSF_lADODVjuFs4BLvIdzg7OJ0k"
  PRIORITY_FIELD="PVTSSF_lADODVjuFs4BLvIdzg7OJ4o"
  TYPE_FIELD="PVTSSF_lADODVjuFs4BLvIdzg7OJ4s"

  # Option IDs
  # Status: Todo=f75ad846, In Progress=47fc9ee4, Done=98236657
  # Priority: Highest=e0b47d9e, High=b435a1b0, Medium=ba35b2b2, Low=55e88b6c
  # Type: Story=0cd05eb3, Task=c00eb75f, Bug=4d57c5e5
fi
```

### Step 7: Infer Type from Labels

```bash
# Auto-detect type from labels
LABELS=$(echo $ISSUE_DATA | jq -r '.labels[].name' | tr '\n' ' ')
if [[ "$LABELS" == *"bug"* ]]; then
  TYPE_OPTION="4d57c5e5"  # Bug (project field)
  TYPE_NAME="Bug"
  NATIVE_TYPE_NAME="Bug"
elif [[ "$LABELS" == *"enhancement"* ]] || [[ "$LABELS" == *"feature"* ]]; then
  TYPE_OPTION="0cd05eb3"  # Story (project field)
  TYPE_NAME="Story"
  NATIVE_TYPE_NAME="Feature"
elif [[ "$LABELS" == *"story"* ]]; then
  TYPE_OPTION="0cd05eb3"  # Story (project field)
  TYPE_NAME="Story"
  NATIVE_TYPE_NAME="Feature"
else
  TYPE_OPTION="c00eb75f"  # Task (project field)
  TYPE_NAME="Task"
  NATIVE_TYPE_NAME="Task"
fi
echo "📝 Type: $TYPE_NAME (from labels)"
```

### Step 7b: Set Native GitHub Issue Type

GitHub has two type systems: **Project field** (custom) and **Native Issue Type** (sidebar).
This step sets the native type on the issue itself.

```bash
# Get issue node ID for GraphQL
ISSUE_NODE_ID=$(gh api graphql -f query="
query {
  repository(owner: \"Scheduler-Systems\", name: \"$REPO_NAME\") {
    issue(number: $ISSUE_NUM) { id }
  }
}" --jq '.data.repository.issue.id')

# Get native issue type ID from repo
NATIVE_TYPE_ID=$(gh api graphql -f query="
query {
  repository(owner: \"Scheduler-Systems\", name: \"$REPO_NAME\") {
    issueTypes(first: 10) {
      nodes { id name }
    }
  }
}" --jq ".data.repository.issueTypes.nodes[] | select(.name==\"$NATIVE_TYPE_NAME\") | .id")

# Set native issue type
if [ -n "$NATIVE_TYPE_ID" ] && [ -n "$ISSUE_NODE_ID" ]; then
  gh api graphql -f query="
  mutation {
    updateIssue(input: {
      id: \"$ISSUE_NODE_ID\"
      issueTypeId: \"$NATIVE_TYPE_ID\"
    }) {
      issue { issueType { name } }
    }
  }"
  echo "✅ Native Type: $NATIVE_TYPE_NAME"
else
  echo "⚠️ Native issue types not configured for this repo"
fi
```

### Step 8: Set Status Based on PR

```bash
# If there's an open PR, set to "In Progress"
HAS_OPEN_PR=$(echo $LINKED_PRS | jq -r 'map(select(.state=="OPEN")) | length')
if [ "$HAS_OPEN_PR" -gt 0 ]; then
  STATUS_OPTION="47fc9ee4"  # In Progress
  STATUS_NAME="In Progress"
else
  STATUS_OPTION="f75ad846"  # Todo
  STATUS_NAME="Todo"
fi
echo "📊 Status: $STATUS_NAME"
```

### Step 9: Apply Project Field Updates via GraphQL

```bash
# Update Type
gh api graphql -f query="
mutation {
  updateProjectV2ItemFieldValue(input: {
    projectId: \"$PROJECT_ID\"
    itemId: \"$ITEM_ID\"
    fieldId: \"$TYPE_FIELD\"
    value: { singleSelectOptionId: \"$TYPE_OPTION\" }
  }) { projectV2Item { id } }
}"

# Update Status
gh api graphql -f query="
mutation {
  updateProjectV2ItemFieldValue(input: {
    projectId: \"$PROJECT_ID\"
    itemId: \"$ITEM_ID\"
    fieldId: \"$STATUS_FIELD\"
    value: { singleSelectOptionId: \"$STATUS_OPTION\" }
  }) { projectV2Item { id } }
}"

echo "✅ Project fields updated"
```

### Step 10: Prompt for Priority

```bash
echo ""
echo "Set Priority:"
echo "  1) Highest"
echo "  2) High"
echo "  3) Medium"
echo "  4) Low"
read -p "Priority [3]: " PRIORITY_CHOICE

case ${PRIORITY_CHOICE:-3} in
  1) PRIORITY_OPTION="e0b47d9e"; PRIORITY_NAME="Highest" ;;
  2) PRIORITY_OPTION="b435a1b0"; PRIORITY_NAME="High" ;;
  3) PRIORITY_OPTION="ba35b2b2"; PRIORITY_NAME="Medium" ;;
  4) PRIORITY_OPTION="55e88b6c"; PRIORITY_NAME="Low" ;;
esac

# Update Priority
gh api graphql -f query="
mutation {
  updateProjectV2ItemFieldValue(input: {
    projectId: \"$PROJECT_ID\"
    itemId: \"$ITEM_ID\"
    fieldId: \"$PRIORITY_FIELD\"
    value: { singleSelectOptionId: \"$PRIORITY_OPTION\" }
  }) { projectV2Item { id } }
}"

echo "✅ Priority: $PRIORITY_NAME"
```

### Step 11: Summary

```bash
echo ""
echo "═══════════════════════════════════════"
echo "✅ Issue #$ISSUE_NUM Updated"
echo "═══════════════════════════════════════"
echo "  Assignee:   $(gh issue view $ISSUE_NUM --json assignees -q '.assignees[0].login')"
echo "  Milestone:  $(gh issue view $ISSUE_NUM --json milestone -q '.milestone.title // "Not set"')"
echo "  Project:    #$PROJECT_NUM"
echo "  Type:       $TYPE_NAME"
echo "  Priority:   $PRIORITY_NAME"
echo "  Status:     $STATUS_NAME"
echo "  Linked PRs: $(echo $LINKED_PRS | jq -r 'map("#\(.number)") | join(", ") // "None"')"
echo ""
echo "View: $(echo $ISSUE_DATA | jq -r '.url')"
```

## Project Field IDs Reference

### Infrastructure (Project #9)
```
Project ID: PVT_kwDODVjuFs4BLvId

Status (PVTSSF_lADODVjuFs4BLvIdzg7OJ0k):
  - Todo: f75ad846
  - In Progress: 47fc9ee4
  - Done: 98236657

Priority (PVTSSF_lADODVjuFs4BLvIdzg7OJ4o):
  - Highest: e0b47d9e
  - High: b435a1b0
  - Medium: ba35b2b2
  - Low: 55e88b6c

Issue Type (PVTSSF_lADODVjuFs4BLvIdzg7OJ4s) - Project field:
  - Story: 0cd05eb3
  - Task: c00eb75f
  - Bug: 4d57c5e5
```

### Native GitHub Issue Types

These are the **native issue types** shown in the issue sidebar (different from project fields).
Query for any repo:

```bash
gh api graphql -f query='
query {
  repository(owner: "Scheduler-Systems", name: "REPO_NAME") {
    issueTypes(first: 10) {
      nodes { id name description }
    }
  }
}'
```

Example IDs (scheduler-systems-infra):
```
Task: IT_kwDODVjuFs4Bq_A9
Bug: IT_kwDODVjuFs4Bq_A-
Feature: IT_kwDODVjuFs4Bq_A_
```

### GAL (Project #5)
Query field IDs with:
```bash
gh project field-list 5 --owner Scheduler-Systems
```

## When to Use

- After creating a new GitHub issue
- When triaging issues
- Before starting work on an issue
- After PR is opened (to update status)

## Integration with SDLC

This command should be invoked by:
- `/sdlc:7-maintain/run` - After bug triage, update issue fields (type=Bug, priority, status=Todo)
- `/sdlc:4-implement/run` - When starting implementation, update status to "In Progress"
- `/sdlc:5-review/run` - After PR is created, status updates automatically via linked PR detection

### Auto-invocation Example

In SDLC commands, add after issue/ticket identification:

```markdown
## Update GitHub Issue Fields

After identifying the issue, invoke:

Skill(skill: "sdlc:7-maintain:helpers:update-gh-issue", args: "<issue-number>")

This ensures:
- Issue is assigned to current user
- Milestone synced with GAL
- Added to correct project board
- Type/Priority/Status fields set
- Linked PRs detected for status updates
```
