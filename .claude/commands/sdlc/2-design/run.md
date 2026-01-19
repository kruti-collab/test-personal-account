---
description: Execute the implementation planning workflow using the plan template to generate design artifacts.
handoffs:
  - label: "Next: TEST (Step 3)"
    agent: sdlc:3-test:run
    prompt: Generate test cases for this feature (TDD - tests first, they should FAIL)
  - label: Create Tasks
    agent: sdlc:2-design:helpers:tasks
    prompt: Break the plan into tasks
    send: true
  - label: Create Checklist
    agent: sdlc:1-specify:helpers:checklist
    prompt: Create a checklist for the following domain...
---

## MANDATORY: Initialize Progress Tracking

**Before ANY work, create TodoWrite:**

```javascript
TodoWrite([
  { content: "[P2] Verify Phase 1 (SPECIFY) complete", status: "pending", activeForm: "Verifying spec.md exists" },
  { content: "[P2] Prime context", status: "pending", activeForm: "Loading design context" },
  { content: "[P2] Load spec & project context", status: "pending", activeForm: "Loading feature context" },
  { content: "[P2] Generate research tasks", status: "pending", activeForm: "Dispatching research agents" },
  { content: "[P2] Create data model", status: "pending", activeForm: "Creating data-model.md" },
  { content: "[P2] Generate API contracts", status: "pending", activeForm: "Generating API contracts" },
  { content: "[P2] Update agent context", status: "pending", activeForm: "Updating agent context" },
  { content: "[P2] Auto-generate tasks", status: "pending", activeForm: "Generating tasks.md" },
  { content: "[P2] Commit & report", status: "pending", activeForm: "Committing design artifacts" },
  { content: "[P2] Capture learnings", status: "pending", activeForm: "Capturing learnings" }
])
```

**Rules (from official docs):**
- Initialize ALL todos upfront before starting work
- Mark `in_progress` BEFORE beginning each step
- Only ONE todo `in_progress` at a time
- Mark `completed` immediately when done

---

## Phase Gate Check (REQUIRED - FAIL FAST)

**CRITICAL: Verify Phase 1 (SPECIFY) completed before proceeding.**

```bash
# Detect feature directory from branch name or user input
BRANCH=$(git branch --show-current)
FEATURE_NUM=$(echo "$BRANCH" | grep -oE '^[0-9]+' || echo "")

if [ -n "$FEATURE_NUM" ]; then
  SPECS_DIR=$(find specs -maxdepth 1 -type d -name "${FEATURE_NUM}-*" 2>/dev/null | head -1)
fi

# Check for spec.md
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
  echo "│ Current Phase:  2-DESIGN                                    │"
  echo "│ Required Phase: 1-SPECIFY (not completed)                   │"
  echo "│ Missing:        specs/{feature}/spec.md                     │"
  echo "└─────────────────────────────────────────────────────────────┘"
  echo ""
  echo "Run this command first:"
  echo "  /sdlc:1-specify:run <feature description>"
  echo ""
  exit 1
fi

echo "✅ Phase Gate: spec.md found at $SPEC_FILE"
```

**If gate fails → STOP. Do NOT proceed. User must run Phase 1 first.**

---

## Phase 0: Prime Context (REQUIRED - AUTO-EXECUTE)

**IMPORTANT: Execute the prime command FIRST before proceeding.**

```
Skill(skill: "sdlc:prime:2-design")
```

This loads architecture context including:
- Documentation: `docs/sdlc/2-design/` and `docs/architecture/`
- Core package: `packages/core/src/` structure
- DI container: `apps/api/src/di/container.ts`
- Key patterns: Clean Architecture, Repository pattern, dependencies point inward

**DO NOT PROCEED until prime command completes.**

---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. **Setup**: Run `openspec/scripts/setup-plan.sh --json` from repo root and parse JSON for FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Load context**: Read FEATURE_SPEC and `openspec/project.md`. Load IMPL_PLAN template (already copied).

3. **Execute plan workflow**: Follow the structure in IMPL_PLAN template to:
   - Fill Technical Context (mark unknowns as "NEEDS CLARIFICATION")
   - Fill Constitution Check section from constitution
   - Evaluate gates (ERROR if violations unjustified)
   - Phase 0: Generate research.md (resolve all NEEDS CLARIFICATION)
   - Phase 1: Generate data-model.md, contracts/, quickstart.md
   - Phase 1: Update agent context by running the agent script
   - Re-evaluate Constitution Check post-design

4. **Auto-generate tasks.md**: After Phase 2 completes, automatically invoke the tasks helper:
   ```
   Skill(skill: "sdlc:2-design:helpers:tasks")
   ```
   **DO NOT mark design phase complete until tasks.md is generated.**

5. **Stop and report**: Command ends after tasks.md generation. Report branch, IMPL_PLAN path, tasks.md, and generated artifacts.

## Phases

### Phase 0: Outline & Research

1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

2. **Generate and dispatch research agents**:

   ```text
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Output**: research.md with all NEEDS CLARIFICATION resolved

### Phase 1: Design & Contracts

**Prerequisites:** `research.md` complete

1. **Extract entities from feature spec** → `data-model.md`:
   - Entity name, fields, relationships
   - Validation rules from requirements
   - State transitions if applicable

2. **Generate API contracts** from functional requirements:
   - For each user action → endpoint
   - Use standard REST/GraphQL patterns
   - Output OpenAPI/GraphQL schema to `/contracts/`

3. **Agent context update**:
   - Run `openspec/scripts/update-agent-context.sh claude`
   - These scripts detect which AI agent is in use
   - Update the appropriate agent-specific context file
   - Add only new technology from current plan
   - Preserve manual additions between markers

**Output**: data-model.md, /contracts/*, quickstart.md, agent-specific file

### Phase 2: Task Generation (AUTO-EXECUTE)

**Prerequisites:** Phase 1 complete

**IMPORTANT**: This phase runs automatically. Do NOT skip.

1. **Invoke tasks helper**:
   ```
   Skill(skill: "sdlc:2-design:helpers:tasks")
   ```

2. **Verify tasks.md created** in FEATURE_DIR

3. **Validate completeness**:
   - All user stories have corresponding tasks
   - Dependencies are marked
   - Parallel tasks [P] identified

**Output**: tasks.md with complete implementation roadmap

**Exit Condition**: Design phase is ONLY complete when tasks.md exists and is validated.

## Key rules

- Use absolute paths
- ERROR on gate failures or unresolved clarifications
- NEVER skip Phase 2 task generation

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
  { content: "[P2] Capture learnings", status: "completed", activeForm: "Learnings captured" }
])
```
