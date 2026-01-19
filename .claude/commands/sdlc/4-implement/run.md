---
description: Execute the implementation plan by processing and executing all tasks defined in tasks.md
handoffs:
  - label: "Next: REVIEW (Step 5)"
    agent: sdlc:5-review:run
    prompt: Review the PR created at end of implement phase
---

## MANDATORY: Initialize Progress Tracking

**Before ANY work, create TodoWrite:**

```javascript
TodoWrite([
  { content: "[P4] Verify Phase 3 (TEST) complete", status: "pending", activeForm: "Verifying test files exist" },
  { content: "[P4] Verify worktree usage", status: "pending", activeForm: "Checking worktree setup" },
  { content: "[P4] Load tasks & implementation plan", status: "pending", activeForm: "Loading implementation tasks" },
  { content: "[P4] Check checklists status", status: "pending", activeForm: "Verifying checklists complete" },
  { content: "[P4] Execute implementation tasks", status: "pending", activeForm: "Implementing features" },
  { content: "[P4] Verify tests PASS (GREEN phase)", status: "pending", activeForm: "Running full test suite" },
  { content: "[P4] Run manual testing", status: "pending", activeForm: "Delegating to manual-tester" },
  { content: "[P4] Rebase onto staging", status: "pending", activeForm: "Rebasing onto staging" },
  { content: "[P4] Commit & create PR", status: "pending", activeForm: "Creating pull request" },
  { content: "[P4] Update GitHub issue", status: "pending", activeForm: "Updating GitHub issue status" },
  { content: "[P4] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

**Rules (from official docs):**
- Initialize ALL todos upfront before starting work
- Mark `in_progress` BEFORE beginning each step
- Only ONE todo `in_progress` at a time
- Mark `completed` immediately when done

---

## MANDATORY: Worktree Check (FAIL FAST)

**CRITICAL: Implementation work MUST use a dedicated worktree.**

```bash
# STEP 1: ALWAYS verify worktree usage before ANY implementation work
if [ "$(pwd)" = "/Users/karabil/Documents/scheduler-systems/products/gal" ]; then
  echo "STOP: Implementation work detected in main worktree"
  echo "   REQUIRED: Use dedicated worktree for implementation"
  echo ""
  echo "   Create implementation worktree:"
  echo "   IMPL_BRANCH=\"$(git branch --show-current)-impl-$(date +%s)\""
  echo "   WORKTREE_PATH=\"/tmp/worktrees/\$IMPL_BRANCH\""
  echo "   git worktree add \"\$WORKTREE_PATH\" -b \"\$IMPL_BRANCH\""
  echo "   cd \"\$WORKTREE_PATH\""
  echo ""
  echo "   Then re-run the implementation command."
  exit 1
fi
echo "Worktree verified: $(pwd)"
```

---

## Phase Gate Check (REQUIRED - FAIL FAST)

**CRITICAL: Verify Phase 3 (TEST) completed before proceeding. TDD requires tests FIRST.**

```bash
# Detect feature directory from branch name
BRANCH=$(git branch --show-current)
FEATURE_NUM=$(echo "$BRANCH" | grep -oE '^[0-9]+' || echo "")

if [ -n "$FEATURE_NUM" ]; then
  SPECS_DIR=$(find specs -maxdepth 1 -type d -name "${FEATURE_NUM}-*" 2>/dev/null | head -1)
fi

# Check for spec.md (from Phase 1)
SPEC_FILE=""
if [ -n "$SPECS_DIR" ]; then
  SPEC_FILE=$(find "$SPECS_DIR" -name "spec.md" -o -name "*-spec.md" 2>/dev/null | head -1)
fi

if [ -z "$SPEC_FILE" ] || [ ! -f "$SPEC_FILE" ]; then
  echo "❌ PHASE GATE FAILED: spec.md not found"
  echo ""
  echo "┌─────────────────────────────────────────────────────────────┐"
  echo "│ SDLC Phase Order Violation                                  │"
  echo "├─────────────────────────────────────────────────────────────┤"
  echo "│ Current Phase:  4-IMPLEMENT                                 │"
  echo "│ Required Phase: 1-SPECIFY (not completed)                   │"
  echo "│ Missing:        specs/{feature}/spec.md                     │"
  echo "└─────────────────────────────────────────────────────────────┘"
  echo ""
  echo "SDLC flow: SPECIFY → DESIGN → TEST → IMPLEMENT"
  echo "Run: /sdlc:1-specify:run <feature description>"
  exit 1
fi

# Check for plan.md (from Phase 2)
PLAN_FILE=""
if [ -n "$SPECS_DIR" ]; then
  PLAN_FILE=$(find "$SPECS_DIR" -name "plan.md" -o -name "*-plan.md" 2>/dev/null | head -1)
fi

if [ -z "$PLAN_FILE" ] || [ ! -f "$PLAN_FILE" ]; then
  echo "❌ PHASE GATE FAILED: plan.md not found"
  echo ""
  echo "┌─────────────────────────────────────────────────────────────┐"
  echo "│ SDLC Phase Order Violation                                  │"
  echo "├─────────────────────────────────────────────────────────────┤"
  echo "│ Current Phase:  4-IMPLEMENT                                 │"
  echo "│ Required Phase: 2-DESIGN (not completed)                    │"
  echo "│ Missing:        specs/{feature}/plan.md                     │"
  echo "└─────────────────────────────────────────────────────────────┘"
  echo ""
  echo "SDLC flow: SPECIFY → DESIGN → TEST → IMPLEMENT"
  echo "Run: /sdlc:2-design:run"
  exit 1
fi

# Check for test files (from Phase 3) - THE CRITICAL TDD GATE
# Extract feature name for test file matching
FEATURE_NAME=$(basename "$SPECS_DIR" | sed 's/^[0-9]*-//')

# Look for test files that match this feature
TEST_FILES=$(find . -type f \( \
  -path "*/e2e/**/*.spec.ts" -o \
  -path "*/tests/**/*.test.ts" -o \
  -path "*/__tests__/*.test.ts" \
\) 2>/dev/null | grep -i "${FEATURE_NAME//-/}" | head -5 || echo "")

# Also check tasks.md for test tasks (Phase 3 should have created this)
TASKS_FILE=""
if [ -n "$SPECS_DIR" ]; then
  TASKS_FILE=$(find "$SPECS_DIR" -name "tasks.md" 2>/dev/null | head -1)
fi

# Gate logic: Either test files exist OR tasks.md has test tasks marked done
HAS_TESTS=false

if [ -n "$TEST_FILES" ]; then
  HAS_TESTS=true
  echo "✅ Phase Gate: Test files found:"
  echo "$TEST_FILES" | while read f; do echo "   - $f"; done
fi

if [ -z "$TEST_FILES" ]; then
  echo "❌ PHASE GATE FAILED: No test files found for this feature"
  echo ""
  echo "┌─────────────────────────────────────────────────────────────┐"
  echo "│ SDLC Phase Order Violation - TDD VIOLATION                  │"
  echo "├─────────────────────────────────────────────────────────────┤"
  echo "│ Current Phase:  4-IMPLEMENT                                 │"
  echo "│ Required Phase: 3-TEST (not completed)                      │"
  echo "│ Missing:        Test files for feature '$FEATURE_NAME'      │"
  echo "└─────────────────────────────────────────────────────────────┘"
  echo ""
  echo "TDD Principle: Tests must be written BEFORE implementation."
  echo ""
  echo "Expected test locations:"
  echo "  - apps/*/e2e/**/${FEATURE_NAME}*.spec.ts"
  echo "  - tests/**/${FEATURE_NAME}*.test.ts"
  echo ""
  echo "Run this command first:"
  echo "  /sdlc:3-test:run"
  echo ""
  echo "Why: TDD (Test-Driven Development) requires:"
  echo "  1. Write failing tests (RED)"
  echo "  2. Implement code to pass (GREEN) ← You are trying to skip to here"
  echo "  3. Refactor (REFACTOR)"
  exit 1
fi

echo "✅ Phase Gate: spec.md found"
echo "✅ Phase Gate: plan.md found"
echo "✅ Phase Gate: Test files found - TDD verified"
echo ""
echo "Ready to implement. Tests should currently FAIL (TDD Red phase)."
```

**If gate fails → STOP. Do NOT proceed. TDD requires tests to exist before implementation.**

---

## Phase 0: Prime Context (REQUIRED - AUTO-EXECUTE)

**IMPORTANT: Execute the prime command FIRST before proceeding.**

```
Skill(skill: "sdlc:prime:4-implement")
```

This loads implementation context including:
- Documentation: `docs/sdlc/4-implement/` structure
- API structure: routes, DI container, security
- Core services: `packages/core/src/services/`
- Dashboard pages: `apps/dashboard/src/pages/`
- Dev server status: API and Dashboard health

**DO NOT PROCEED until prime command completes.**

---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. Run `openspec/scripts/check-prerequisites.sh --json --require-tasks --include-tasks` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Check checklists status** (if FEATURE_DIR/checklists/ exists):
   - Scan all checklist files in the checklists/ directory
   - For each checklist, count:
     - Total items: All lines matching `- [ ]` or `- [X]` or `- [x]`
     - Completed items: Lines matching `- [X]` or `- [x]`
     - Incomplete items: Lines matching `- [ ]`
   - Create a status table:

     ```text
     | Checklist | Total | Completed | Incomplete | Status |
     |-----------|-------|-----------|------------|--------|
     | ux.md     | 12    | 12        | 0          | ✓ PASS |
     | test.md   | 8     | 5         | 3          | ✗ FAIL |
     | security.md | 6   | 6         | 0          | ✓ PASS |
     ```

   - Calculate overall status:
     - **PASS**: All checklists have 0 incomplete items
     - **FAIL**: One or more checklists have incomplete items

   - **If any checklist is incomplete**:
     - Display the table with incomplete item counts
     - **STOP** and ask: "Some checklists are incomplete. Do you want to proceed with implementation anyway? (yes/no)"
     - Wait for user response before continuing
     - If user says "no" or "wait" or "stop", halt execution
     - If user says "yes" or "proceed" or "continue", proceed to step 3

   - **If all checklists are complete**:
     - Display the table showing all checklists passed
     - Automatically proceed to step 3

3. Load and analyze the implementation context:
   - **REQUIRED**: Read tasks.md for the complete task list and execution plan
   - **REQUIRED**: Read plan.md for tech stack, architecture, and file structure
   - **IF EXISTS**: Read data-model.md for entities and relationships
   - **IF EXISTS**: Read contracts/ for API specifications and test requirements
   - **IF EXISTS**: Read research.md for technical decisions and constraints
   - **IF EXISTS**: Read quickstart.md for integration scenarios

4. **Project Setup Verification**:
   - **REQUIRED**: Create/verify ignore files based on actual project setup:

   **Detection & Creation Logic**:
   - Check if the following command succeeds to determine if the repository is a git repo (create/verify .gitignore if so):

     ```sh
     git rev-parse --git-dir 2>/dev/null
     ```

   - Check if Dockerfile* exists or Docker in plan.md → create/verify .dockerignore
   - Check if .eslintrc* exists → create/verify .eslintignore
   - Check if eslint.config.* exists → ensure the config's `ignores` entries cover required patterns
   - Check if .prettierrc* exists → create/verify .prettierignore
   - Check if .npmrc or package.json exists → create/verify .npmignore (if publishing)
   - Check if terraform files (*.tf) exist → create/verify .terraformignore
   - Check if .helmignore needed (helm charts present) → create/verify .helmignore

   **If ignore file already exists**: Verify it contains essential patterns, append missing critical patterns only
   **If ignore file missing**: Create with full pattern set for detected technology

   **Common Patterns by Technology** (from plan.md tech stack):
   - **Node.js/JavaScript/TypeScript**: `node_modules/`, `dist/`, `build/`, `*.log`, `.env*`
   - **Python**: `__pycache__/`, `*.pyc`, `.venv/`, `venv/`, `dist/`, `*.egg-info/`
   - **Java**: `target/`, `*.class`, `*.jar`, `.gradle/`, `build/`
   - **C#/.NET**: `bin/`, `obj/`, `*.user`, `*.suo`, `packages/`
   - **Go**: `*.exe`, `*.test`, `vendor/`, `*.out`
   - **Ruby**: `.bundle/`, `log/`, `tmp/`, `*.gem`, `vendor/bundle/`
   - **PHP**: `vendor/`, `*.log`, `*.cache`, `*.env`
   - **Rust**: `target/`, `debug/`, `release/`, `*.rs.bk`, `*.rlib`, `*.prof*`, `.idea/`, `*.log`, `.env*`
   - **Kotlin**: `build/`, `out/`, `.gradle/`, `.idea/`, `*.class`, `*.jar`, `*.iml`, `*.log`, `.env*`
   - **C++**: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.so`, `*.a`, `*.exe`, `*.dll`, `.idea/`, `*.log`, `.env*`
   - **C**: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.a`, `*.so`, `*.exe`, `Makefile`, `config.log`, `.idea/`, `*.log`, `.env*`
   - **Swift**: `.build/`, `DerivedData/`, `*.swiftpm/`, `Packages/`
   - **R**: `.Rproj.user/`, `.Rhistory`, `.RData`, `.Ruserdata`, `*.Rproj`, `packrat/`, `renv/`
   - **Universal**: `.DS_Store`, `Thumbs.db`, `*.tmp`, `*.swp`, `.vscode/`, `.idea/`

   **Tool-Specific Patterns**:
   - **Docker**: `node_modules/`, `.git/`, `Dockerfile*`, `.dockerignore`, `*.log*`, `.env*`, `coverage/`
   - **ESLint**: `node_modules/`, `dist/`, `build/`, `coverage/`, `*.min.js`
   - **Prettier**: `node_modules/`, `dist/`, `build/`, `coverage/`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
   - **Terraform**: `.terraform/`, `*.tfstate*`, `*.tfvars`, `.terraform.lock.hcl`
   - **Kubernetes/k8s**: `*.secret.yaml`, `secrets/`, `.kube/`, `kubeconfig*`, `*.key`, `*.crt`

5. Parse tasks.md structure and extract:
   - **Task phases**: Setup, Tests, Core, Integration, Polish
   - **Task dependencies**: Sequential vs parallel execution rules
   - **Task details**: ID, description, file paths, parallel markers [P]
   - **Execution flow**: Order and dependency requirements

6. Execute implementation following the task plan:
   - **Phase-by-phase execution**: Complete each phase before moving to the next
   - **Respect dependencies**: Run sequential tasks in order, parallel tasks [P] can run together  
   - **Follow TDD approach**: Execute test tasks before their corresponding implementation tasks
   - **File-based coordination**: Tasks affecting the same files must run sequentially
   - **Validation checkpoints**: Verify each phase completion before proceeding

7. Implementation execution rules:
   - **Setup first**: Initialize project structure, dependencies, configuration
   - **Tests before code**: If you need to write tests for contracts, entities, and integration scenarios
   - **Core development**: Implement models, services, CLI commands, endpoints
   - **Integration work**: Database connections, middleware, logging, external services
   - **Polish and validation**: Unit tests, performance optimization, documentation

8. Progress tracking and error handling:
   - Report progress after each completed task
   - Halt execution if any non-parallel task fails
   - For parallel tasks [P], continue with successful tasks, report failed ones
   - Provide clear error messages with context for debugging
   - Suggest next steps if implementation cannot proceed
   - **IMPORTANT** For completed tasks, make sure to mark the task off as [X] in the tasks file.

9. Completion validation:
   - Verify all required tasks are completed
   - Check that implemented features match the original specification
   - Validate that tests pass and coverage meets requirements
   - Confirm the implementation follows the technical plan
   - Report final status with summary of completed work

10. **Final verification and PR creation** (REQUIRES USER CONSENT):

    **Step 10a: Run unit/integration tests**
    ```bash
    # Run tests based on project type
    pnpm test  # or npm test, flutter test, etc.
    ```

    **Step 10b: Start Docker dev environment for verification**

    **CRITICAL: Use Docker for consistent dev-local testing.**

    ```bash
    # Stop any existing containers
    docker compose down 2>/dev/null || true

    # Start dev environment in Docker
    make docker-dev-detached

    # Wait for services to be healthy
    for i in {1..30}; do
      if curl -s http://localhost:3000/health | grep -q "ok"; then
        echo "✅ API healthy"
        break
      fi
      sleep 2
    done

    # Verify Dashboard is accessible
    curl -s -o /dev/null -w "%{http_code}" http://localhost:5173 | grep -q "200" && echo "✅ Dashboard healthy"
    ```

    **If Docker fails to start**: Check `make docker-logs` for errors and fix before proceeding.

    ---

    **Step 10c: Manual Testing Loop (REQUIRED - BLOCKS PR)**

    **⛔ CRITICAL: Manual testing MUST pass before PR creation. Loop until success.**

    ```
    MANUAL_TEST_PASSED=false
    MAX_ATTEMPTS=3
    ATTEMPT=0

    while [ "$MANUAL_TEST_PASSED" = false ] && [ $ATTEMPT -lt $MAX_ATTEMPTS ]; do
      ATTEMPT=$((ATTEMPT + 1))
      echo "📋 Manual Testing Attempt $ATTEMPT of $MAX_ATTEMPTS"

      Task(
        subagent_type: "manual-tester",
        prompt: "Verify implementation for $BRANCH_NAME (Attempt $ATTEMPT):

          **Docker Environment**: Services running at:
          - API: http://localhost:3000
          - Dashboard: http://localhost:5173
          - Website: http://localhost:5174

          **Test Checklist**:
          1. Test any new UI pages/components in browser (Playwright MCP)
          2. Test any new CLI commands in terminal (tmux)
          3. Test any new API endpoints via curl
          4. Check for anomalies - compare actual vs expected behavior
          5. Take screenshots of key UI states
          6. Report PASS or FAIL with specific issues found

          **Expected behavior from spec**: [summarize spec requirements]
          **Files changed**: [list key files]

          **IMPORTANT**: Return structured result:
          - PASS: All tests passed
          - FAIL: [List specific failures]"
      )

      if [ RESULT = "PASS" ]; then
        MANUAL_TEST_PASSED=true
        echo "✅ Manual testing passed"
      else
        echo "❌ Manual testing failed on attempt $ATTEMPT"
        echo "Issues found: [list issues]"

        if [ $ATTEMPT -lt $MAX_ATTEMPTS ]; then
          AskUserQuestion(
            questions: [{
              question: "Manual testing failed. How do you want to proceed?",
              header: "Test Failed",
              options: [
                { label: "Fix and retry", description: "Fix the issues and run manual tests again" },
                { label: "Skip and proceed", description: "Proceed to PR anyway (not recommended)" },
                { label: "Abort", description: "Stop implementation, address issues separately" }
              ],
              multiSelect: false
            }]
          )

          if [ ANSWER = "Fix and retry" ]; then
            # Fix issues, then continue loop
            echo "🔧 Fixing issues..."
            # [Apply fixes based on failure report]
          elif [ ANSWER = "Skip and proceed" ]; then
            echo "⚠️ Skipping manual tests - proceeding to PR"
            MANUAL_TEST_PASSED=true  # Force proceed
          else
            echo "🛑 Aborting implementation"
            exit 1
          fi
        fi
      fi
    done

    if [ "$MANUAL_TEST_PASSED" = false ]; then
      echo "❌ Manual testing failed after $MAX_ATTEMPTS attempts"
      echo "Cannot proceed to PR creation. Fix issues and re-run."
      exit 1
    fi
    ```

    ---

    **Step 10d: Stop Docker and cleanup**
    ```bash
    docker compose down
    echo "✅ Docker environment cleaned up"
    ```

    ---

    **Step 10e: Ask user consent for commit and PR**
    ```
    AskUserQuestion(
      questions: [{
        question: "Implementation complete and manual tests passed. Ready to commit, push, and create PR?",
        header: "Create PR",
        options: [
          { label: "Yes, create PR", description: "Commit changes and create PR linked to issue" },
          { label: "No, need more changes", description: "Continue working on implementation" }
        ],
        multiSelect: false
      }]
    )
    ```

    **Step 10f: If user approves, create PR with issue linked**

    Extract issue number from branch name or spec folder:
    ```bash
    # Get issue number from branch name (e.g., 247-update-logo -> 247)
    ISSUE_NUMBER=$(git branch --show-current | grep -oE '^[0-9]+' | head -1)

    # Or from FEATURE_DIR if available
    # ISSUE_NUMBER=$(basename "$FEATURE_DIR" | grep -oE '^[0-9]+')
    ```

    ---

    **Step 10h: Rebase onto staging (handle conflicts)**

    Before creating PR, ensure branch is up-to-date with staging:

    ```bash
    git fetch origin staging
    git rebase origin/staging
    ```

    **If conflicts occur**, use the resolve-conflicts command:
    ```
    Skill(skill: "_internal:git:resolve-conflicts")
    ```

    This systematically resolves conflicts by:
    - Analyzing both sides of each conflict
    - Priming context for informed decisions
    - Applying appropriate resolution strategy

    ---

    **Step 10i: Commit and push**
    ```bash
    BRANCH_NAME=$(git branch --show-current)

    # Stage all changes
    git add -A

    # Commit with branch name prefix
    git commit -m "[$BRANCH_NAME] Implementation complete

    Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"

    # Push to remote
    git push origin $BRANCH_NAME
    ```

    Create PR with issue linked:
    ```bash
    gh pr create \
      --base staging \
      --title "[$BRANCH_NAME] <Brief description from spec>" \
      --body "$(cat <<'EOF'
    ## Summary
    <1-3 bullet points summarizing changes>

    ## Test Plan
    - [ ] Tests pass locally
    - [ ] Visual verification complete
    - [ ] E2E tests pass (if applicable)

    ## Related Issue
    Relates to #$ISSUE_NUMBER

    ---
    🤖 Generated with [Claude Code](https://claude.com/claude-code)
    EOF
    )"
    ```

    **Important**: Use `Relates to #XXX` to link PR to issue WITHOUT auto-closing. The issue will be closed later in DEPLOY/MAINTAIN phase after full verification.

    **Step 10e: Update GitHub Issue Status (if applicable)**

    If implementation is for a GitHub issue, update its status to "In Progress":
    ```
    Skill(skill: "sdlc:7-maintain:helpers:update-gh-issue", args: "$ISSUE_NUMBER")
    ```
    This sets status to "In Progress" automatically when a linked PR is detected.

Note: This command assumes a complete task breakdown exists in tasks.md. If tasks are incomplete or missing, suggest running `/sdlc:2-design:helpers:tasks` first to regenerate the task list.

---

## MANDATORY: Capture Learnings (AUTO-EXECUTE)

**DO NOT end without invoking:**

```
Skill(skill: "capture-learnings")
```

This step is REQUIRED when ANY of these occurred:
- User had to manually correct agent behavior
- Workflow improvements were discovered
- New patterns/anti-patterns were identified
- `.claude/` files were modified during the session

This captures:
- Process deviations that occurred
- Manual interventions from user
- Improvements to agentic layer

**Update TodoWrite when complete:**
```javascript
TodoWrite([
  // ... previous steps as completed ...
  { content: "[P4] Capture learnings", status: "completed", activeForm: "Learnings captured" }
])
```

**When to invoke:**
1. After committing feature changes
2. Before ending SDLC phase
3. When user provides workflow corrections
