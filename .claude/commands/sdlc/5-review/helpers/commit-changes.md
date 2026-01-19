---
allowed-tools: Task
description: Initiate scrum-master agent to analyze git status and commit changes following SMR workflow protocols
---

# Commit Changes

Delegates to the scrum-master agent to analyze the current git repository state and commit changes following proper GitHub issue workflow and pre-commit protocols.

## Usage

```bash
/commit-changes [commit_message_hint]
```

## What This Command Does

This slash command:
1. **Launches the scrum-master agent** with git analysis context
2. **Uses git_status_agent_prompt.md** for comprehensive repository analysis
3. **Creates logical commit chunks** based on functionality and file relationships
4. **Detects multi-branch scenarios** (mixed GitHub issue scope)
5. **Orchestrates chunked commit workflows** (multiple focused commits per branch)
6. **Follows GitHub issue workflow** (branch naming, commit formatting)
7. **Applies pre-commit protocols** (file classification, staging recommendations)
8. **REQUESTS HUMAN AUTHORIZATION** for complete workflow before execution
9. **Creates multiple focused commits** with #XXX: prefix (logical chunks)
10. **Handles stash cleanup** and push operations automatically

## Agent Delegation

The command delegates to the **scrum-master** agent with the following context:

```markdown
Task: Analyze git status and orchestrate complete commit workflow following SMR protocols
Context: User wants to commit current changes with proper GitHub issue coordination and multi-branch handling
Requirements:
- Use /git_status agent prompt for comprehensive repository analysis
- **DETECT MIXED SCOPE**: Identify if files belong to different GitHub issues
- **CREATE LOGICAL CHUNKS**: Group related files into focused, atomic commits
- **PLAN CHUNKED WORKFLOW**: Design multiple commits per branch based on functionality
- **PRESENT COMPLETE WORKFLOW PLAN**: Show all branches, commits chunks, and operations upfront
- **SINGLE AUTHORIZATION GATE**: Get approval for entire workflow before execution
- **EXECUTE SYSTEMATICALLY**: Handle stash → branch → chunked commits → cleanup operations
- **STASH MANAGEMENT**: Track and auto-cleanup stashes after successful commits
- Create multiple #XXX-formatted commit messages (#XXX: focused descriptions)
- **PUSH COORDINATION**: Handle push operations for all affected branches when requested
- Ensure clean git workflow compliance throughout
Expected Deliverables: 
- Repository analysis summary with scope detection
- Complete workflow plan presented to user for approval
- User confirmation received before executing workflow
- All commits completed with proper SMR format (only after approval)
- Stash cleanup completed automatically
- Clean git status across all affected branches
```

## SMR Workflow Integration

The scrum-master will:
- **Detect GitHub issue** from branch name (e.g., #250)
- **Analyze file changes** and classify as code/test artifacts/generated files
- **Apply gitignore updates** for test artifacts if needed
- **Create commit message** following project standards
- **Coordinate with GitHub** if ticket updates are needed

## Example Scenarios

### Basic Commit
```bash
/commit-with-scrum-master
# Scrum-master analyzes current changes and commits with auto-generated SMR message
```

### Commit with Hint
```bash
/commit-with-scrum-master "Add GitHub MCP tools to subagents"
# Scrum-master incorporates hint into #XXX-formatted commit message
```

### Complex Multi-File Changes (Single SMR)
```bash
/commit-changes
# Scrum-master classifies all files, updates .gitignore, stages appropriately
# Results in clean commit with only intended code changes
```

### Mixed Scope Multi-Branch Scenario
```bash
/commit-changes
# Scrum-master detects mixed GitHub issues (e.g., #243 + #250)
# Plans complete workflow: stash → commit #243 → switch → commit #250
# Presents entire workflow plan for single approval
# Executes systematically with automatic stash cleanup
```

## Agent Execution

When executed, this command will:

1. **Launch scrum-master agent** using Task tool
2. **Provide comprehensive context** about commit requirements
3. **Reference git_status_agent_prompt.md** for structured analysis
4. **Ensure SMR compliance** throughout the workflow
5. **Return summary** of committed changes and git status

The scrum-master agent has access to:
- Full git analysis capabilities (status, diff, history)
- GitHub issue detection and formatting
- File classification and staging protocols
- GitHub MCP tools for PR coordination (if needed)
- GitHub CLI for GitHub issue management

This command streamlines the commit process while ensuring all project standards and GitHub issue workflows are properly followed.

## Command Implementation

Task: Analyze git status and commit changes with SMR workflow compliance

Context: User wants to commit current repository changes following proper GitHub issue protocols and pre-commit standards.

Specific Instructions for scrum-master:
1. FIRST: Use the git_status_agent_prompt.md for comprehensive repository analysis:
   - Run /git_status to get structured repository state analysis
   - Identify current branch and GitHub issue context
   - Classify all staged/unstaged/untracked files appropriately

2. SMR Ticket Analysis:
   - Extract GitHub issue number from branch name (e.g., #250)
   - Ensure commit message follows #XXX: format
   - Consider if GitHub issue status needs updating

3. Mixed Scope Detection and Chunking Strategy:
   - **DETECT MIXED SCOPE**: Identify files belonging to different GitHub issues
   - **CREATE LOGICAL CHUNKS**: Group related files into focused, atomic commits:
     * Configuration changes (Dockerfile, devcontainer.json, configs)
     * New features/commands (new .md files, new scripts)
     * Documentation updates (README, AI docs, help files)
     * Cleanup operations (deletions, .gitignore updates)
     * Security/firewall changes (firewall scripts, security configs)
   - **PLAN CHUNKED WORKFLOW**: Design multiple focused commits per branch
   - **SINGLE vs MULTI-BRANCH**: Choose appropriate workflow strategy based on scope analysis

4. Pre-Commit Protocol Execution:
   - Classify files: code changes vs test artifacts vs generated files
   - Update .gitignore for any test artifacts (playwright-report, test-results, *.xml)
   - Stage only intended code changes and documentation per GitHub issue
   - **PRESENT COMPLETE WORKFLOW PLAN TO USER** (not just staging plan)
   - **WAIT FOR EXPLICIT USER APPROVAL** for entire workflow before proceeding

5. Human Authorization Gate (Enhanced):
   - Show user complete workflow plan including all branches and operations
   - Display ALL proposed commit messages for review
   - For multi-branch: Show stash operations and cleanup procedures  
   - Request explicit confirmation: "Proceed with this complete workflow? (y/n)"
   - Only continue after receiving user approval for entire process

6. Chunked Workflow Execution (Only After Approval):
   - **SINGLE BRANCH**: Execute multiple logical commits:
     * Commit 1: Configuration changes (Dockerfile, devcontainer.json)
     * Commit 2: New features/commands (new slash commands, scripts)
     * Commit 3: Documentation updates (README, AI docs)
     * Commit 4: Cleanup operations (file deletions, .gitignore)
     * Commit 5: Security/firewall changes (firewall scripts, security configs)
   - **MULTI-BRANCH**: Execute complete chunked workflow:
     * Stash files for other GitHub issues with descriptive messages
     * Create multiple focused commits for current GitHub issue
     * Switch to other SMR branches (create if needed)
     * Restore stashes and create chunked commits for respective branches
     * **AUTO-CLEANUP**: Drop stashes after successful commits
   - Follow project's commit message standards throughout
   - Ensure proper SMR formatting for all commits

7. Post-Workflow Validation:
   - Verify all commits succeeded across branches
   - Report final git status for all affected branches
   - Provide summary of all committed changes with branch mapping
   - **PUSH COORDINATION**: Handle push operations when requested by user
   - Suggest next steps (PR creation, branch management, etc.)
   - Confirm stash cleanup completed successfully

Expected Response Format:
- Repository State Summary with Mixed Scope Detection
- SMR Ticket Analysis and Mapping
- Complete Chunked Workflow Plan (Single or Multi-Branch)
- File Classification Results per SMR Ticket with Logical Groupings
- All Commit Chunk Details (messages, hashes, files, branches)
- Commit Sequence Summary (chronological order of all commits)
- Stash Management Summary (if applicable)
- Next Steps Recommendations (PR creation, push operations)

Use all available tools including git analysis, file classification, and GitHub/GitHub CLI as needed for complete SMR workflow compliance.