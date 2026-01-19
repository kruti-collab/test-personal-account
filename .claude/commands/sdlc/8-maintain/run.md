---
allowed-tools: Task, Bash, Write
argument-hint: [#XXX] | [manual_bug_description] | --post-deploy-check
description: Bug triage and CI-triggered post-merge maintenance verification
handoffs:
  - label: "New Feature: SPECIFY (Step 1)"
    agent: sdlc:1-specify:run
    prompt: Start new feature specification
---

## MANDATORY: Initialize Progress Tracking

**Before ANY work, create TodoWrite:**

```javascript
TodoWrite([
  { content: "[P8] Parse input (GitHub issue or bug description)", status: "pending", activeForm: "Parsing bug input" },
  { content: "[P8] Prime context", status: "pending", activeForm: "Loading maintenance context" },
  { content: "[P8] Classify bug (severity, component, platform)", status: "pending", activeForm: "Classifying bug" },
  { content: "[P8] Route to specialist agent", status: "pending", activeForm: "Routing to specialist" },
  { content: "[P8] Coordinate resolution", status: "pending", activeForm: "Coordinating bug resolution" },
  { content: "[P8] Update GitHub issue", status: "pending", activeForm: "Updating GitHub issue" },
  { content: "[P8] Post-deployment monitoring (if applicable)", status: "pending", activeForm: "Monitoring deployment" },
  { content: "[P8] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

**Rules (from official docs):**
- Initialize ALL todos upfront before starting work
- Mark `in_progress` BEFORE beginning each step
- Only ONE todo `in_progress` at a time
- Mark `completed` immediately when done

---

## Phase Gate Check (INFORMATIONAL)

**Note: MAINTAIN phase handles bugs and can run independently or after deployment.**

```bash
# This phase is different - it handles ongoing maintenance
# It can run at any time when bugs are reported

echo "╔══════════════════════════════════════════════════════════════╗"
echo "║ SDLC Phase 8: MAINTAIN                                       ║"
echo "╠══════════════════════════════════════════════════════════════╣"
echo "║ Purpose: Bug triage, production monitoring, maintenance      ║"
echo "║                                                              ║"
echo "║ This phase can run:                                          ║"
echo "║   • After deployment (standard flow)                         ║"
echo "║   • Anytime for production bugs                              ║"
echo "║   • Independent of feature development                       ║"
echo "║                                                              ║"
echo "║ Loops back to: SPECIFY (for new feature requests)            ║"
echo "╚══════════════════════════════════════════════════════════════╝"
echo ""

# Optional: Check if there's a deployed feature to maintain
BRANCH=$(git branch --show-current)

if [ "$BRANCH" = "staging" ] || [ "$BRANCH" = "main" ]; then
  echo "✅ On deployment branch: $BRANCH"
  echo "   Ready for production maintenance tasks."
else
  echo "ℹ️  On feature branch: $BRANCH"
  echo "   For feature-specific bugs, consider using /sdlc:3-test:run"
  echo "   For production bugs, this command is appropriate."
fi
echo ""
```

**MAINTAIN phase has flexible entry - can run after DEPLOY or independently for bugs.**

---

## Phase 0: Prime Context (REQUIRED - AUTO-EXECUTE)

**IMPORTANT: Execute the prime command FIRST before proceeding.**

```
Skill(skill: "sdlc:prime:7-maintain")
```

This loads maintenance context including:
- Documentation: `docs/sdlc/8-maintain/` structure
- Security: API security guidelines, Firestore rules
- Production state: releases, workflows, incidents
- Key patterns: Secret Manager, incident severity, rollback procedures

**DO NOT PROCEED until prime command completes.**

---

# Bug Triage

Comprehensive bug triage system that analyzes GitHub issues from GitHub, classifies bug severity and component areas, and delegates to appropriate engineering specialists for resolution.

## Usage

```bash
# Fetch and triage existing GitHub issue
/bug-triage #270

# Triage with manual bug description (fallback mode)
/bug-triage "Authentication failing on iOS production builds"
```

## What This Command Does

This slash command orchestrates a complete bug triage workflow:

1. **Ticket Analysis**: Fetches GitHub issue details from GitHub using GitHub CLI
2. **Bug Classification**: Categorizes by severity, component, platform, and urgency
3. **Specialist Routing**: Delegates to flutter-engineer, qa-engineer, or devops-engineer based on issue type
4. **Resolution Orchestration**: Coordinates from ticket analysis to PR using bug-resolver-orchestrator
5. **Fallback Support**: Handles manual descriptions when GitHub CLI is not available
6. **GitHub Issue Update**: If bug is tracked as GitHub issue, invoke `/update-gh-issue` to set fields:
   ```
   Skill(skill: "sdlc:8-maintain:helpers:update-gh-issue", args: "<issue-number>")
   ```
   Sets: assignee, milestone, project, type=Bug, priority, status

## Bug Classification Matrix

The triage system uses these classification criteria:

### Severity Levels
- **Critical**: App crashes, data loss, security vulnerabilities
- **High**: Major features broken, significant user impact
- **Medium**: Minor features broken, workarounds available
- **Low**: UI polish, performance improvements, edge cases

### Component Areas
- **Authentication**: Login, registration, session management
- **Core Features**: Scheduling, calendar, notifications
- **UI/UX**: Layout issues, responsive design, accessibility
- **Backend**: Firebase functions, data sync, API issues
- **Platform**: iOS/Android specific, web deployment
- **DevOps**: CI/CD, build issues, deployment problems

### Specialist Routing Logic
- **flutter-engineer**: Code bugs, UI issues, core feature problems
- **qa-engineer**: Test failures, regression analysis, coverage gaps
- **devops-engineer**: CI/CD issues, deployment problems, infrastructure
- **bug-resolver-orchestrator**: Complex multi-component issues requiring coordination

## Agent Delegation Strategy

The command uses **bug-resolver-orchestrator** as the primary coordination agent:

```markdown
Task: Analyze and triage bug from GitHub issue with comprehensive classification and specialist routing
Context: Bug triage request for #XXX with full GitHub integration and multi-specialist coordination
Requirements:
- **GITHUB INTEGRATION**: Use GitHub CLI (`gh issue view #XXX`) for ticket details
- **CLASSIFICATION**: Complete bug analysis (severity, component, platform impact)
- **ROUTING STRATEGY**: Determine appropriate specialist(s) for resolution
- **COORDINATION PLAN**: Create comprehensive resolution workflow
- **FALLBACK HANDLING**: Support manual bug descriptions when GitHub CLI unavailable
- **SMR COMPLIANCE**: Follow branch naming and commit message standards
Expected Deliverables:
- Detailed bug analysis and classification report
- Specialist assignment with clear context and requirements
- Complete resolution workflow plan
- Risk assessment and timeline estimates
```

## GitHub CLI Integration

The command uses GitHub CLI for ticket operations:

1. **Fetch Ticket Details**:
   ```bash
   gh issue view #XXX
   ```
   - Retrieves complete ticket information including description, comments, attachments
   - Extracts acceptance criteria and reproduction steps
   - Parses priority and status information

2. **Search for Related Issues**:
   ```bash
   gh issue list --search "project = SMR AND status != Done AND text ~ 'keywords'"
   ```
   - Searches for related bugs or duplicate issues
   - Identifies similar problems for context

3. **Error Handling**:
   - Falls back to manual description mode if GitHub CLI fails
   - Checks if GitHub CLI is installed and configured
   - Provides clear error messages and alternative approaches
   - Maintains triage capability without GitHub CLI dependency

## Example Scenarios

### High Priority Production Bug
```bash
/bug-triage #270
# Analysis: Authentication failures on iOS production
# Classification: Critical, Authentication, iOS Platform
# Routing: flutter-engineer (primary) + qa-engineer (regression testing)
# Workflow: Immediate fix → emergency testing → hotfix PR
```

### Complex Multi-Component Issue
```bash
/bug-triage #265
# Analysis: Calendar sync issues across web/mobile
# Classification: High, Core Features, Cross-Platform
# Routing: bug-resolver-orchestrator coordinates flutter-engineer + devops-engineer
# Workflow: Root cause analysis → multi-platform fixes → comprehensive testing
```

### UI Polish Request
```bash
/bug-triage #275
# Analysis: Button alignment issues on tablet layouts
# Classification: Low, UI/UX, Responsive Design
# Routing: flutter-engineer (UI specialist focus)
# Workflow: Standard development cycle with visual testing
```

### Fallback Mode Example
```bash
/bug-triage "Calendar notifications not working on Android 14"
# Manual classification when GitHub CLI unavailable
# Analysis: Platform-specific notification issue
# Routing: flutter-engineer + qa-engineer for Android testing
```

## Resolution Workflow Integration

The bug triage command integrates with existing project workflows:

### SMR Ticket Compliance
- Creates branches with #XXX naming convention
- Ensures commit messages follow "#XXX: description" format
- Updates GitHub issue status during resolution phases

### Multi-Specialist Coordination
- Launches agents in parallel for independent tasks
- Coordinates dependencies between frontend, backend, and testing work
- Provides complete context to each specialist

### Quality Assurance Integration
- Routes to qa-engineer for comprehensive test coverage
- Requires regression testing for critical/high severity bugs
- Ensures cross-platform validation for multi-platform issues

## Error Handling and Fallbacks

The command includes robust error handling:

### GitHub CLI Connection Failures
- Automatically switches to manual mode
- Prompts user for bug description and context
- Maintains full triage capability without external tools

### Specialist Assignment Failures
- Falls back to general-purpose agent with specialist context
- Provides clear instructions for manual specialist routing
- Maintains workflow continuity despite agent availability

### Invalid SMR Tickets
- Validates GitHub issue format before GitHub queries
- Provides helpful error messages for typos
- Suggests alternative approaches (manual description, ticket search)

## Command Implementation

Task: Orchestrate comprehensive bug triage workflow with GitHub integration and specialist routing

Context: User has reported a bug via GitHub issue number or manual description that needs professional triage and resolution assignment.

Specific Instructions for bug-resolver-orchestrator:

1. **Input Analysis and Validation**:
   - If #XXX format provided: Validate ticket format and proceed with GitHub integration
   - If manual description provided: Parse description for classification clues
   - Handle mixed input gracefully (e.g., "#270: additional context")

2. **GitHub CLI Integration Protocol** (when GitHub issue provided):
   - Use `gh issue view #XXX` to fetch complete ticket details
   - Parse output for description, priority, status, comments
   - Search for related issues: `gh issue list --search "project = SMR AND text ~ 'keywords'"`
   - **FALLBACK**: If GitHub CLI fails, request manual bug description and continue with manual mode

3. **Comprehensive Bug Classification**:
   - **Severity Assessment**: Critical/High/Medium/Low based on user impact and system stability
   - **Component Identification**: Authentication, Core Features, UI/UX, Backend, Platform, DevOps
   - **Platform Impact**: iOS, Android, Web, Cross-platform
   - **Urgency Evaluation**: Production impact, user base affected, business critical functions
   - **Complexity Analysis**: Simple fix, moderate investigation, complex multi-component issue

4. **Specialist Routing Decision**:
   - **flutter-engineer**: Code bugs, UI issues, core Flutter development tasks
   - **qa-engineer**: Test failures, regression analysis, quality validation
   - **devops-engineer**: CI/CD issues, deployment problems, infrastructure concerns
   - **bug-resolver-orchestrator**: Multi-component issues requiring coordination
   - **Multiple specialists**: Complex bugs requiring parallel workstreams

5. **Resolution Workflow Planning**:
   - Create detailed resolution plan with phases and dependencies
   - Estimate timeline based on severity and complexity
   - Identify risks and potential blockers
   - Plan testing strategy (unit, integration, E2E as appropriate)
   - Consider rollback/hotfix strategies for critical issues

6. **Specialist Delegation with Context**:
   - Provide complete bug context to assigned specialist(s)
   - Include GitHub issue details, reproduction steps, acceptance criteria
   - Specify expected deliverables and testing requirements
   - Set clear timeline expectations based on severity
   - **SIMULTANEOUS LAUNCH**: Use multiple Task calls for parallel workstreams

7. **Coordination and Tracking**:
   - Monitor progress across all assigned specialists
   - Coordinate dependencies between parallel workstreams
   - Provide status updates and escalation as needed
   - Ensure SMR workflow compliance (branching, commits, PR process)

Expected Response Format:
- **Bug Analysis Summary**: Severity, component, platform impact
- **Classification Matrix**: Complete categorization with reasoning
- **Specialist Assignment**: Who is handling what with clear rationale
- **Resolution Workflow**: Detailed plan with phases and timelines
- **Risk Assessment**: Potential blockers and mitigation strategies
- **Next Steps**: Clear actions for user and assigned specialists

Use all available tools including GitHub CLI, Task delegation, and Write operations for comprehensive bug triage and resolution coordination.

---

## CI-Triggered Post-Merge Maintenance

**When invoked with `--post-deploy-check`, this command runs automated maintenance verification.**

This mode is designed for CI/CD execution via `.github/workflows/post-merge-maintenance.yml`:
- Runs daily for 3 days after PR merges to staging
- Uses tmux for interactive session management
- Executes via Claude Code in headless mode

### CI Usage

```bash
# Invoked by GitHub Actions workflow:
/sdlc:8-maintain:run --post-deploy-check

# With context from CI environment:
# - PR_NUMBER: The merged PR being monitored
# - DAYS_SINCE_MERGE: Day 1, 2, or 3 of monitoring
# - STAGING_API_URL: API endpoint to test
# - STAGING_DASHBOARD_URL: Dashboard to verify
```

### Post-Deploy Check Workflow

When `--post-deploy-check` is detected:

1. **Parse CI Environment Variables**:
   - Extract PR context (number, title, branch, days since merge)
   - Determine test scope based on PR changes

2. **Delegate to Manual Tester**:
   ```
   Task(
     subagent_type: "manual-tester",
     prompt: "CI-triggered post-merge maintenance check..."
   )
   ```

3. **Report Results**:
   - Output structured result for CI parsing
   - Create GitHub issue if bugs found
   - Update workflow summary

### Environment Variable Detection

```bash
# If these variables are set, we're in CI mode:
if [ -n "$PR_NUMBER" ] && [ -n "$STAGING_API_URL" ]; then
  echo "CI mode detected - running automated post-merge maintenance"
  # Skip interactive prompts, use structured output
fi
```

---

## Post-Deployment Monitoring

**After DEPLOY phase completes, run periodic manual testing to catch deployment-specific issues.**

### Monitoring Schedule

| Time After Deploy | Test Scope | Purpose |
|-------------------|------------|---------|
| Immediately | Smoke tests | Verify deployment succeeded |
| 15 minutes | Core flows | Catch startup/cache issues |
| 1 hour | Full manual test | Catch delayed failures |
| 24 hours | Comprehensive | Verify stability over time |
| Days 1-3 | CI-triggered | Automated daily maintenance via workflow |

### Periodic Manual Testing

**Use manual-tester agent for systematic post-deployment verification:**

```
Task(
  subagent_type: "manual-tester",
  prompt: "Post-deployment verification for $ENVIRONMENT:

  1. Run smoke tests:
     - Dashboard loads (/get-started, /docs, /settings)
     - CLI commands work (gal --version, gal auth status)
     - API health check (curl $API_URL/health)

  2. Test core user flows:
     - GitHub OAuth login
     - Organization selection
     - Config sync (gal sync --pull --demo)

  3. Check for deployment-specific issues:
     - Cache invalidation (new features visible?)
     - Environment variables (correct API URLs?)
     - Static asset loading (images, fonts)

  4. Report findings:
     - Create issues for new bugs
     - Update deploy ticket with status
     - Take screenshots as evidence

  Environment: $ENVIRONMENT
  Deploy Time: $DEPLOY_TIME
  Version: $VERSION"
)
```

### Automated Monitoring Triggers

For critical deployments, schedule follow-up tests:

```bash
# Schedule 15-minute follow-up
echo "/sdlc:8-maintain:run --post-deploy-check" | at now + 15 minutes

# Or use a manual reminder
echo "⏰ Reminder: Run post-deployment check in 15 minutes"
echo "   Command: /sdlc:8-maintain:run --post-deploy-check"
```

### Post-Deployment Issue Handling

If issues are found during monitoring:

1. **Severity Assessment**: Is this a rollback-worthy issue?
2. **Quick Fix vs Rollback**: Can it be fixed forward or need to revert?
3. **Communication**: Update deploy ticket and notify stakeholders
4. **Root Cause**: Why wasn't this caught in review phase?

---

## MANDATORY: Capture Learnings (AUTO-EXECUTE)

**DO NOT end without invoking:**

```
Skill(skill: "capture-learnings")
```

This captures:
- Process deviations that occurred
- Manual interventions from user
- Improvements to agentic layer

**Update TodoWrite when complete:**
```javascript
TodoWrite([
  // ... previous steps as completed ...
  { content: "[P8] Capture learnings", status: "completed", activeForm: "Learnings captured" }
])
```