---
description: Add or update PR description with context from commits and changes
argument-hint: "[pr-number]"
---

# Describe PR

Generate or update a PR description with meaningful context derived from commits, file changes, and associated GitHub issue.

## Variables

- **PR_NUMBER**: First argument from $ARGUMENTS (optional)
  - If provided, work with the specified PR number
  - If not provided, work with the current branch's PR

## Steps to perform:

### Phase 1: Gather Information

1. **Get PR information:**
   - If PR_NUMBER provided: Use `gh pr view $PR_NUMBER --json number,title,body,headRefName,commits`
   - If not provided: Use `gh pr view --json number,title,body,headRefName,commits`
   - Extract current description (if any)

2. **Extract GitHub issue:**
   - Get ticket ID from PR title or branch name (e.g., #123)
   - Use `gh issue view <ISSUE-ID>` to fetch:
     - Issue summary
     - Description
     - Acceptance criteria
     - Issue type (Story, Task, Bug, etc.)

3. **Analyze commits:**
   - Get all commits in the PR
   - Extract commit messages
   - Identify patterns:
     - Feature additions
     - Bug fixes
     - Refactoring
     - Documentation
     - Tests

4. **Analyze file changes:**
   - Use `gh pr view --json files` to get changed files
   - Categorize by type:
     - Source code (lib/)
     - Tests (test/)
     - Documentation (docs/, README.md)
     - Configuration (.github/, .claude/, etc.)
     - Build/CI (.github/workflows/, cloudbuild.yaml)

### Phase 2: Generate Description

5. **Create structured description:**

```markdown
## Summary
[Brief 1-2 sentence summary from GitHub issue]

## Changes Made
[Organized list of changes by category]

### Features
- [Feature 1 from commits]
- [Feature 2 from commits]

### Bug Fixes
- [Fix 1 from commits]
- [Fix 2 from commits]

### Refactoring
- [Refactoring 1]

### Documentation
- [Doc changes]

### Tests
- [Test additions/updates]

## Files Changed
- **Source Code**: X files
- **Tests**: Y files
- **Documentation**: Z files
- **Configuration**: N files

## GitHub Ticket
[ISSUE-ID]: [Issue Summary]

**Acceptance Criteria:**
- [ ] [Criterion 1 from GitHub]
- [ ] [Criterion 2 from GitHub]

## Testing
[Suggest testing approach based on changes]

---
🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Phase 3: Update PR Automatically

6. **Update PR description:**
   - Automatically update using: `gh pr edit $PR_NUMBER --body-file /tmp/pr-description.md`
   - If PR already has description, replace it entirely with the new comprehensive description
   - Show confirmation message after successful update

7. **Output summary:**
   - Display: "✅ Updated PR #[NUMBER] description"
   - Show preview of the first few lines
   - Provide GitHub link to the PR

## Special Cases

### If no GitHub issue found:
- Generate description without GitHub section
- Base it purely on commits and file changes

### If PR is draft:
- Note in description that PR is in draft status
- Add "🚧 Draft PR" badge at the top of description

### If update fails:
- Display error message from gh CLI
- Suggest checking permissions or PR status

## Output Format

```
Updating PR Description
========================

PR #[NUMBER]: [Title]
Branch: [branch-name] → [base-branch]
GitHub: [ISSUE-ID] ([Type]) [if found]

Generating comprehensive description from 9 commits and 26 files...

✅ Updated PR #[NUMBER] description successfully!

View PR: https://github.com/[org]/[repo]/pull/[NUMBER]
```

**IMPORTANT:**
- Automatically update PR without asking for confirmation
- Generate comprehensive, well-structured descriptions
- Always replace existing description (new one is more comprehensive)
