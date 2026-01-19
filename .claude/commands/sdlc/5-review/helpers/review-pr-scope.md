---
description: Review if PR contains changes unrelated to the GitHub issue
argument-hint: "[pr-number]"
---

# Review PR Scope

Analyze the current PR to identify changes that may be unrelated to the associated GitHub issue, map unrelated changes to existing or new GitHub issues, and create an actionable execution plan.

## Variables

- **PR_NUMBER**: First argument from $ARGUMENTS (optional)
  - If provided, analyze the specified PR number
  - If not provided, analyze the current branch's PR using `gh pr view`

## Steps to perform:

### Phase 1: Analysis

1. **Get PR information:**
   - If PR_NUMBER is provided: Use `gh pr view $PR_NUMBER --json number,title,body,files,headRefName`
   - If not provided: Use `gh pr view --json number,title,body,files,headRefName` to get the current PR details
   - Extract the GitHub issue ID from the PR title or branch name (e.g., #123)

2. **Fetch GitHub issue details:**
   - Use `gh issue view <ISSUE-ID>` to fetch the GitHub issue information
   - Extract the issue summary, description, and acceptance criteria from the CLI output
   - If it's a subtask, get the parent story context

3. **Analyze PR changes:**
   - Review all changed files using `gh pr view --json files`
   - Categorize changes into logical groups (e.g., CI/CD, DevContainer, Security, Testing, Documentation)
   - Compare each category against the GitHub issue scope

4. **Identify unrelated changes:**
   - List files/changes that appear unrelated to the GitHub issue
   - Categorize them (e.g., CI/CD infrastructure, developer tooling, security, documentation)
   - Calculate the percentage of changes that seem unrelated

### Phase 2: Ticket Mapping

5. **Search for existing GitHub issues:**
   - For each category of unrelated changes, search GitHub for existing tickets
   - Use JQL queries like: `project = SMR AND (summary ~ 'keyword1' OR summary ~ 'keyword2')`
   - Check tickets mentioned in PR description or commit messages
   - Example searches:
     - Cloud Build/CI/CD: `summary ~ 'Cloud Build' OR summary ~ 'deployment' OR summary ~ 'CI/CD'`
     - DevContainer: `summary ~ 'DevContainer' OR summary ~ 'development environment'`
     - Claude Code: `summary ~ 'Claude' OR summary ~ 'hooks' OR summary ~ 'MCP'`
     - Security: `summary ~ 'security' OR summary ~ 'certificate' OR summary ~ 'keystore'`

6. **Map changes to tickets:**
   - For each unrelated category, determine:
     - **EXISTING TICKET**: If a matching ticket exists, note the ticket ID
     - **NEW TICKET NEEDED**: If no matching ticket exists, draft a ticket summary
   - Create a mapping table showing which files belong to which ticket

### Phase 3: Execution Plan

7. **Generate execution plan:**
   - Create a detailed plan showing:
     - Which files stay in the current PR (related to original ticket)
     - Which files move to existing tickets (with ticket IDs)
     - Which files require new tickets (with proposed ticket summaries)
   - Include specific commands to execute the plan
   - Present this plan to the user for approval

8. **Wait for user approval:**
   - DO NOT execute the plan automatically
   - Present the plan clearly and wait for user to approve or request corrections
   - If corrections needed, iterate on the analysis

## Output format:

```
PR Scope Review Report
======================

PR Number: #[NUMBER]
Branch: [BRANCH_NAME]
GitHub Issue: [ISSUE-ID]
Issue Summary: [Summary from GitHub]

SCOPE ANALYSIS:
===============

✅ Related to [ISSUE-ID] (X files, Y lines):
- [file1]: [description]
- [file2]: [description]

❌ Unrelated to [ISSUE-ID] (A files, B lines):

Category 1: [Category Name] (N files, M lines)
- [file3]: [description]
- [file4]: [description]
Reason: [why unrelated]

[Repeat for each category...]

TICKET MAPPING:
===============

✅ Existing Tickets Found:
- **#XXX** - [Ticket Summary]
  Files (N):
  - [file1]
  - [file2]

🆕 New Tickets Needed:
- **[Proposed Summary]**
  Description: [Brief description]
  Files (N):
  - [file3]
  - [file4]

EXECUTION PLAN:
===============

If approved, I will execute the following:

1. **Keep in current PR #[NUMBER] (#[ORIGINAL]):**
   - [file1]
   - [file2]
   Total: X files

2. **Move to #XXX ([Existing Ticket Summary]):**
   - Create new branch: #XXX-[descriptive-name]
   - Cherry-pick files:
     - [file3]
     - [file4]
   - Create new PR
   Total: Y files

3. **Create new ticket & PR:**
   - Create GitHub issue: "[Proposed Summary]"
   - Create branch: #[NEW]-[descriptive-name]
   - Cherry-pick files:
     - [file5]
     - [file6]
   - Create new PR
   Total: Z files

COMMANDS TO EXECUTE:
====================

[Specific git and gh commands that will be run]

APPROVAL REQUIRED:
==================

Please review this plan and respond with:
- "approve" - Execute the plan as-is
- "correct [details]" - Adjust the analysis/mapping
- "cancel" - Cancel the operation

Do you approve this execution plan?
```

**IMPORTANT:** After generating the report, STOP and wait for user approval. Do NOT execute any file moves, branch creates, or PR operations until explicitly approved.
