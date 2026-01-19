# Prime DESIGN - Load Architecture & Design Context

Prime context window with everything needed for architecture and design work.

## Step 1: Read Architecture Documentation

Read these files to understand system architecture:

1. Read `docs/sdlc/2-design/README.md`
2. Read `docs/sdlc/2-design/architecture/README.md`
3. Read `docs/sdlc/2-design/architecture/DEPLOYMENT_FLOW.md`
4. Read `docs/sdlc/2-design/architecture/CLEAN_ARCHITECTURE_MIGRATION_GUIDE.md`

## Step 2: Read Core Package Structure

Read the core domain package to understand business logic:

1. Read `packages/core/src/index.ts`
2. Read `packages/core/package.json`
3. Read `packages/types/src/index.ts`

## Step 3: Scan Implementation Patterns

```bash
# Core domain structure
ls -la packages/core/src/

# API layer structure
ls -la apps/api/src/
ls -la apps/api/src/routes/

# Dashboard structure
ls -la apps/dashboard/src/
ls -la apps/dashboard/src/pages/

# List all service files
find packages/core/src -name "*.service.ts" 2>/dev/null
find packages/core/src -name "*.repository.ts" 2>/dev/null
```

## Step 4: Read Key Implementation Files

Read examples of each architectural layer:

1. Read `apps/api/src/di/container.ts` (dependency injection)
2. Read `apps/api/src/index.ts` (API entry point)
3. Read one service file from `packages/core/src/services/`
4. Read one repository interface from `packages/core/src/repositories/`

## Step 5: Check Current Design Work

```bash
# Current branch
git branch --show-current

# Any uncommitted architecture changes
git status --short | grep -E "packages/core|apps/api/src|docs/architecture"

# Recent architecture commits
git log --oneline -5 -- "packages/core/" "docs/architecture/"
```

## Architecture Patterns (Memorize These)

### Clean Architecture Layers

```
Presentation → Application → Domain → Infrastructure
     │              │           │            │
  React/CLI    Contexts    @gal/core    Firestore
```

### Dependency Rule
**Dependencies point INWARD. Domain never imports from Infrastructure.**

### Key Patterns

| Pattern | Location | Purpose |
|---------|----------|---------|
| Repository | `packages/core/src/repositories/` | Data access abstraction |
| Service | `packages/core/src/services/` | Business logic |
| Entity | `packages/core/src/domain/` | Domain models |
| Adapter | `apps/*/src/adapters/` | Infrastructure implementations |
| DI Container | `apps/api/src/di/` | Dependency injection |

### API Design Rules
- RESTful endpoints
- Never remove fields (backward compat)
- Optional additions only
- `/v2/` for breaking changes

## CRITICAL RULES (MUST FOLLOW)

### CI Check Protocol
**NEVER approve or merge a PR until ALL CI checks pass.** Don't run tests locally while CI runs - wait for CI. Don't use `--admin` to bypass failures - fix them first.

### Fix Failures On The Spot
When discovering a bug or failure during SDLC workflow: fix immediately while having context. Don't skip past failures. Never use `--admin` bypass.

### GitHub Issue Linkage
**ALWAYS use "Closes #X" (not "Relates to") in PR descriptions for auto-close.**

## After Priming

You are now ready to run:
- `/sdlc/design/run` - Create implementation plan
- `/sdlc/design/helpers/analyze` - Analyze existing architecture
- `/sdlc/design/helpers/tasks` - Break down into tasks

## Report After Priming

Summarize:
1. Current branch and architecture-related changes
2. Core package structure overview
3. Key services and repositories available
4. Ready to proceed with design work
