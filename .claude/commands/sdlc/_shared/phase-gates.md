# SDLC Phase Gates

**Purpose**: Enforce strict phase order in the SDLC workflow. Each phase MUST verify prerequisites before executing.

## Phase Dependency Graph

```
1-SPECIFY → 2-DESIGN → 3-TEST → 4-IMPLEMENT → 5-REVIEW → 6-DEPLOY → 7-MAINTAIN
     │           │          │           │            │           │
     ▼           ▼          ▼           ▼            ▼           ▼
  spec.md    plan.md    *.spec.ts    code       PR merged   deployed
            research.md  (failing)   changes
            data-model.md
```

## Phase Gate Requirements

### Phase 2: DESIGN
**Requires from Phase 1 (SPECIFY):**
- `specs/{feature}/spec.md` OR `specs/{feature}/*-spec.md`

### Phase 3: TEST
**Requires from Phase 2 (DESIGN):**
- `specs/{feature}/spec.md` OR `specs/{feature}/*-spec.md`
- `specs/{feature}/plan.md` OR `specs/{feature}/*-plan.md`

### Phase 4: IMPLEMENT
**Requires from Phase 3 (TEST):**
- `specs/{feature}/spec.md` OR `specs/{feature}/*-spec.md`
- `specs/{feature}/plan.md` OR `specs/{feature}/*-plan.md`
- Test file exists (e.g., `apps/*/e2e/**/*.spec.ts` or `tests/**/*.test.ts`)

### Phase 5: REVIEW
**Requires from Phase 4 (IMPLEMENT):**
- Code changes committed (git status shows changes or commits ahead)
- OR PR already exists

### Phase 6: DEPLOY
**Requires from Phase 5 (REVIEW):**
- PR merged OR on target branch

### Phase 7: MAINTAIN
**Requires from Phase 6 (DEPLOY):**
- Deployment verified

## Gate Check Script Template

```bash
#!/bin/bash
# Phase Gate Check - Include in each phase command

check_phase_gate() {
  local PHASE=$1
  local SPECS_DIR=$2

  case $PHASE in
    2) # DESIGN requires SPECIFY
      if ! find "$SPECS_DIR" -name "spec.md" -o -name "*-spec.md" 2>/dev/null | grep -q .; then
        echo "❌ PHASE GATE FAILED: spec.md not found"
        echo ""
        echo "Required: Run /sdlc:1-specify:run first"
        echo "Location: $SPECS_DIR should contain spec.md"
        return 1
      fi
      ;;

    3) # TEST requires DESIGN
      if ! find "$SPECS_DIR" -name "plan.md" -o -name "*-plan.md" 2>/dev/null | grep -q .; then
        echo "❌ PHASE GATE FAILED: plan.md not found"
        echo ""
        echo "Required: Run /sdlc:2-design:run first"
        echo "Location: $SPECS_DIR should contain plan.md"
        return 1
      fi
      ;;

    4) # IMPLEMENT requires TEST
      # Check for test files related to the feature
      if ! find . -path "*/e2e/**/*.spec.ts" -newer "$SPECS_DIR/plan.md" 2>/dev/null | grep -q . && \
         ! find . -path "*/tests/**/*.test.ts" -newer "$SPECS_DIR/plan.md" 2>/dev/null | grep -q .; then
        echo "❌ PHASE GATE FAILED: No tests found"
        echo ""
        echo "Required: Run /sdlc:3-test:run first"
        echo "TDD Principle: Tests must be written BEFORE implementation"
        return 1
      fi
      ;;

    5) # REVIEW requires IMPLEMENT
      if [ -z "$(git status --porcelain)" ] && [ -z "$(git log @{u}..HEAD 2>/dev/null)" ]; then
        # Check if PR exists
        if ! gh pr view --json state >/dev/null 2>&1; then
          echo "❌ PHASE GATE FAILED: No code changes to review"
          echo ""
          echo "Required: Run /sdlc:4-implement:run first"
          return 1
        fi
      fi
      ;;

    6) # DEPLOY requires REVIEW
      PR_STATE=$(gh pr view --json state --jq '.state' 2>/dev/null || echo "NONE")
      if [ "$PR_STATE" != "MERGED" ]; then
        echo "❌ PHASE GATE FAILED: PR not merged"
        echo ""
        echo "Required: Complete /sdlc:5-review:run and merge PR first"
        echo "Current PR state: $PR_STATE"
        return 1
      fi
      ;;
  esac

  return 0
}
```

## Error Messages

Each gate failure should provide:
1. **What's missing**: Clear identification of the missing artifact
2. **Which phase to run**: Exact command to execute
3. **Why this order matters**: Brief explanation (optional)

Example:
```
❌ PHASE GATE FAILED: plan.md not found

SDLC Phase Order Violation:
├─ Current Phase: 3-TEST
├─ Required Phase: 2-DESIGN (not completed)
└─ Missing Artifact: specs/{feature}/plan.md

Run this command first:
  /sdlc:2-design:run

Why: Tests are generated from the design plan. Without a plan,
     we cannot determine what acceptance criteria to test.
```

## Anti-Skip Protection

**Critical Rule**: A phase command MUST NOT proceed if its prerequisites are missing.

The gate check runs BEFORE:
- Priming context
- Loading files
- Executing any phase logic

This ensures fast failure with clear guidance.
