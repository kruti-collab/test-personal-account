---
description: Implement Flutter features from GitHub issues or descriptions
argument-hint: "[#XXX or feature-description]"
allowed-tools: Task
---

# Implement Feature

Comprehensive workflow for implementing Flutter features from GitHub GitHub issues or feature descriptions. This command handles the complete development lifecycle from requirements gathering to code deployment.

## Variables

- **INPUT**: First argument from $ARGUMENTS (required)
  - Can be either:
    - **GitHub issue number** (e.g., "#242", "#198") - Fetches requirements from GitHub
    - **Feature description** (e.g., "Add dark mode toggle") - Interactive ticket search/creation
  - Used to determine workflow path and fetch implementation requirements

## Usage

```bash
# With GitHub issue number (recommended)
/implement-feature #242

# With feature description (interactive mode)
/implement-feature "Add OAuth authentication flow"
/implement-feature "Implement calendar sync with Google Calendar"
```

## Workflow Overview

This command orchestrates:
1. **Input Analysis**: Detect if input is GitHub issue or description
2. **Requirement Gathering**: Fetch GitHub issue details using GitHub CLI
3. **Implementation Delegation**: Route to flutter-engineer specialist for feature development
4. **Test Generation**: Route to qa-engineer specialist for Patrol integration tests
5. **Testing and Validation**: Ensure code quality and comprehensive test coverage
6. **Git Workflow**: Proper branching, commits, and PR preparation

## Phase 1: Input Analysis and Validation

**Detect Input Type:**

```
If INPUT matches "#\d+":
  → SMR Ticket Mode: Use GitHub CLI to fetch requirements
Else:
  → Description Mode: Search for existing tickets or create new
```

**Validation Checks:**
- GitHub issue format: Must match `#XXX` pattern
- Feature description: Must be clear and actionable
- If invalid: Prompt user for clarification

## Phase 2: GitHub Integration (GitHub CLI)

**GitHub CLI Protocol**:

1. **Fetch Ticket Details** (if #XXX provided):
   ```bash
   gh issue view #XXX
   ```
   - Extract: summary, description, acceptance criteria, comments, priority
   - Parse ticket details from CLI output

2. **Search for Existing Tickets** (if description provided):
   ```bash
   gh issue list --search "project = SMR AND status != Done AND summary ~ 'keywords'"
   ```
   - Search based on feature keywords from description
   - Present found tickets to user for selection
   - If multiple matches: Ask user to choose
   - If no matches: Offer to create new ticket

3. **Create New Ticket** (if needed):
   ```bash
   gh issue create \
     --type Task \
     --project SMR \
     --summary "Feature summary" \
     --priority Medium
   ```
   - Get summary and priority from user if not specified
   - Return new ticket number (#XXX)

4. **Update Ticket Status**:
   ```bash
   gh issue edit #XXX "In Progress"
   ```
   - Move ticket to "In Progress" if not already
   - Handle case where ticket is already in progress

## Phase 3: Implementation Delegation

**Delegate to flutter-engineer specialist:**

```markdown
Task: Implement Flutter feature from GitHub GitHub issue with complete development lifecycle

Context: User has requested implementation of feature from #XXX ticket. The ticket has been fetched from GitHub and validated.

SMR Ticket Details:
- **Ticket**: #XXX
- **Summary**: [GitHub summary]
- **Description**: [Full description from GitHub]
- **Acceptance Criteria**: [Criteria from ticket]
- **Priority**: [High/Medium/Low]

Specific Instructions for flutter-engineer:

1. **Requirement Analysis**:
   - Review all ticket details above
   - Identify affected components and dependencies
   - Plan implementation approach using /think or /think-hard if needed

2. **Branch Management** (CRITICAL):
   - **Determine base branch based on feature type**:
     * App features (Flutter code, UI, business logic): Branch from `dev`
     * DevOps features (CI/CD, workflows, agents, tooling): Branch from `devops`
   - **Check if #XXX branch exists**:
     ```bash
     git fetch origin
     git branch -a | grep "#XXX" || echo "Branch doesn't exist"
     ```
   - **If branch exists**: Checkout and pull latest
     ```bash
     git checkout #XXX-brief-description
     git pull origin #XXX-brief-description
     ```
   - **If branch doesn't exist**: Create from appropriate base
     ```bash
     # For app features:
     git checkout -b #XXX-brief-description dev

     # For DevOps features:
     git checkout -b #XXX-brief-description devops
     ```
   - Ensure clean working directory before starting
   - **Update GitHub Issue Status** (if tracked as GitHub issue):
     ```
     Skill(skill: "sdlc:7-maintain:helpers:update-gh-issue", args: "<issue-number>")
     ```
     This sets status to "In Progress" when work begins.

3. **Codebase Analysis**:
   - Study existing Provider patterns in lib/providers/
   - Review related widgets and screens
   - Identify reusable components
   - Check flavor configurations

4. **Implementation Requirements**:
   - Use Provider pattern for state management (MANDATORY)
   - Follow existing project structure and conventions
   - Implement proper error handling and loading states
   - Ensure flavor-specific configurations are handled
   - Add proper documentation for complex logic

5. **Testing Protocol (MANDATORY)**:
   - Write unit tests for Provider classes
   - Write widget tests for UI components
   - Run app locally to manually test:
     * Web: ./scripts/run_web_local.sh
     * Mobile: ./scripts/run_mobile.sh dev
   - Navigate to feature and test all interactions
   - Test edge cases and error states
   - Test with dev accounts (shaypanuilov@gmail.com / 123456)

6. **Pre-Commit Protocol (CRITICAL)**:
   - Run: git status --porcelain
   - Classify ALL files (code vs artifacts vs generated)
   - Update .gitignore for test artifacts if needed
   - Stage ONLY intended code changes
   - Verify clean status after staging

7. **Commit and Push**:
   - Commit message format: "#XXX: [descriptive summary]"
   - Include detailed description of changes
   - Add Claude Code attribution
   - **Push to #XXX feature branch** (NOT to dev or devops directly):
     ```bash
     git push origin #XXX-brief-description
     ```

Expected Deliverables:
- Complete feature implementation matching acceptance criteria
- Comprehensive test coverage (unit + widget tests)
- Manual testing verification report
- Clean git history with proper commit messages
- Feature branch ready for PR creation

Use all available tools including Read, Write, Edit, MultiEdit, Glob, Grep, Bash, and TodoWrite for comprehensive feature implementation.
```

## Phase 4: Comprehensive Test Generation

**Delegate to qa-engineer specialist for Patrol integration tests:**

```markdown
Task: Generate comprehensive Patrol integration tests for #XXX feature

Context: flutter-engineer has completed feature implementation. Now generate comprehensive Patrol-based integration tests following official documentation.

SMR Ticket: #XXX
Feature: [Feature name]
Implementation Details:
- Files changed: [list from flutter-engineer]
- Components: [list of components to test]
- User flows: [key user journeys to test]

Specific Instructions for qa-engineer:

1. **Review Patrol Documentation**:
   - Read patrol-docs/README.md for comprehensive patterns
   - Check patrol-docs/EXAMPLES.md for similar test scenarios
   - Reference patrol-docs/QUICK_START.md for syntax

2. **Test Planning**:
   - Identify key user journeys from acceptance criteria
   - Plan test scenarios covering:
     * Happy path (successful flow)
     * Error cases (validation, network failures)
     * Edge cases (empty states, boundaries)
     * Native interactions (if applicable: permissions, notifications)

3. **Write Patrol Tests** (MANDATORY):
   - Use patrolTest() function exclusively
   - Follow Patrol's $ syntax for widget interactions
   - Organize tests in integration_test/features/[feature-name]/
   - Include native interactions if feature requires permissions/notifications

4. **Test Structure**:
   ```dart
   // integration_test/features/[feature]/[feature]_test.dart
   import 'package:patrol/patrol.dart';
   import 'package:scheduler/main.dart';

   void main() {
     group('[Feature Name]', () {
       patrolTest('should [expected behavior]', ($) async {
         await $.pumpWidgetAndSettle(MyApp());

         // Test implementation using Patrol patterns
         await $(#widgetKey).tap();
         await $(#result).waitUntilVisible();

         expect($(#result).visible, true);
       });
     });
   }
   ```

5. **Execute Tests**:
   - Run: patrol test integration_test/features/[feature]/
   - Verify all tests pass
   - Fix any failures before reporting completion

6. **Report Format**:
   ```markdown
   ## QA Testing Complete - #XXX

   ### Patrol Tests Created:
   - integration_test/features/[feature]/[test].dart

   ### Test Coverage:
   ✅ X Patrol integration tests
   ✅ Y test scenarios covered
   ✅ All tests passing

   ### Test Scenarios:
   - [Scenario 1]: Happy path
   - [Scenario 2]: Error handling
   - [Scenario 3]: Edge cases

   ### Commands to Run:
   patrol test integration_test/features/[feature]/
   patrol develop integration_test/features/[feature]/[test].dart (for development)
   ```

Expected Deliverables:
- Patrol integration tests in proper directory structure
- All tests passing (verified with patrol test command)
- Test report with coverage details
- Clear instructions for running tests

Use patrol-docs/ as primary reference for all test patterns and syntax.
```

## Phase 5: Post-Implementation Actions

**After flutter-engineer AND qa-engineer complete:**

1. **Verify Deliverables**:
   - Check all acceptance criteria met
   - Confirm unit/widget tests written and passing (flutter-engineer)
   - Confirm Patrol integration tests written and passing (qa-engineer)
   - Verify manual testing completed
   - Review commit message formatting

2. **GitHub Update**:
   ```bash
   gh issue comment #XXX "Implemented [feature name]

   Changes:
   - [File 1]: [description]
   - [File 2]: [description]

   Testing:
   ✅ Unit tests passing (flutter-engineer)
   ✅ Widget tests passing (flutter-engineer)
   ✅ Patrol integration tests passing (qa-engineer)
   ✅ Manual testing verified

   Commit: [commit-hash]
   Branch: [branch-name]

   Status: Ready for PR review"
   ```

3. **Next Steps Guidance**:
   ```
   Display to user:
   ✅ Feature Implementation Complete

   Ticket: #XXX
   Feature: [Feature name]
   Branch: [branch-name]
   Commit: [hash]

   Files Changed:
   - [list of files from flutter-engineer]
   - [list of test files from qa-engineer]

   Testing Status:
   ✅ Unit tests passing (flutter-engineer)
   ✅ Widget tests passing (flutter-engineer)
   ✅ Patrol integration tests passing (qa-engineer)
   ✅ Manual testing verified

   Test Commands:
   - Patrol tests: patrol test integration_test/features/[feature]/
   - Unit tests: flutter test test/unit/[feature]/
   - Manual test: ./scripts/run_web_local.sh

   Next Steps:
   1. Review implementation locally
   2. Run all tests to verify
   3. Create PR: gh pr create --base dev --head [branch]
   4. Or use: /describe-pr to auto-generate PR description

   Commands for reference:
   - Test locally: ./scripts/run_web_local.sh
   - View commit: git show [hash]
   - View ticket: [GitHub URL]
   ```

## Error Handling and Fallbacks

### GitHub CLI Connection Failures
- Attempt GitHub CLI commands first
- If fails: Check if GitHub CLI is installed and configured
- Fallback: Ask user for ticket details manually
- Provide workflow to continue without GitHub integration

### Invalid Ticket Numbers
- Validate #XXX format before GitHub CLI calls
- If invalid: Prompt for correction or feature description
- Suggest similar ticket numbers if typo detected

### Implementation Failures
- If flutter-engineer encounters blockers, report to user
- Provide clear error context and suggested resolutions
- Maintain ticket in "In Progress" state until resolved

### Missing Requirements
- If ticket lacks acceptance criteria, prompt user for clarification
- Suggest adding details to GitHub issue before proceeding
- Can proceed with user-provided requirements if urgent

## Integration with Other Commands

This command works alongside:
- **Before**: `/think` or `/think-hard` for architectural planning
- **After**: `/describe-pr` for PR creation and description
- **Testing**: `/feature-generate-tests` for comprehensive test coverage
- **Quality**: `/review-pr-scope` to verify PR scope matches ticket

## Special Considerations

### Multi-Platform Features
- Specify platform requirements in delegation
- Ensure flutter-engineer tests on all required platforms
- Web primary, but iOS/Android if ticket specifies

### Complex Features
- Use `/think-harder` or `/ultrathink` for architectural analysis first
- Consider breaking into multiple subtasks
- May require multiple agents for different aspects

### Dependencies on Other Features
- Check if dependent tickets are complete
- Identify integration points early
- Coordinate with other feature branches if needed

### Security-Sensitive Features
- Involve security-researcher for review if needed
- Ensure proper authentication/authorization checks
- Validate input sanitization and data protection

## Success Criteria

Feature implementation is **complete** when:
1. ✅ All acceptance criteria from GitHub issue met
2. ✅ Provider pattern used correctly for state management
3. ✅ Comprehensive test coverage (unit + widget)
4. ✅ Manual testing verified on target platform(s)
5. ✅ Clean git history with proper #XXX commit messages
6. ✅ Code follows project conventions and best practices
7. ✅ Feature branch ready for PR creation
8. ✅ GitHub issue updated with implementation details

## Direct Execution Instructions

1. **Parse INPUT**: Determine if GitHub issue or description
2. **Fetch Ticket Details**: Use GitHub CLI (`gh issue view #XXX`)
3. **Search/Create Ticket**: Use GitHub CLI if description provided
4. **Delegate to flutter-engineer**: Launch Task with complete context for implementation
5. **Monitor flutter-engineer**: Wait for implementation completion
6. **Delegate to qa-engineer**: Launch Task with implementation details for Patrol test generation
7. **Monitor qa-engineer**: Wait for test generation completion
8. **Update GitHub**: Add comment using GitHub CLI with implementation AND testing summary
9. **Report to User**: Display completion summary including both implementation and test results

## Benefits

✅ **Flexible Input**: Works with ticket numbers or descriptions
✅ **GitHub CLI Integration**: Uses GitHub CLI for consistent ticket management
✅ **Complete Workflow**: From requirements to deployable code
✅ **Quality Assurance**: Mandatory testing and validation
✅ **Best Practices**: Follows Provider pattern and project conventions
✅ **Clear Delegation**: Specialized agent handles implementation details
✅ **Tracked Progress**: GitHub issue maintained throughout workflow

## When NOT to Use This Command

- (No special cases - all features use this command now)
- For bug fixes (use `/bug-triage` instead)
- For experimental prototypes (manual approach better)
- For emergency hotfixes (use expedited manual process)
- When ticket requirements are incomplete (gather requirements first)
