# SDLC Prime Commands

Context loading commands that are **automatically executed** as Phase 0 of each SDLC command.

## Auto-Prime Behavior

**Each `/sdlc/N-phase/run` command now automatically invokes its corresponding prime command as Phase 0.**

You no longer need to manually run prime commands - they execute automatically when you run any SDLC phase.

## Quick Reference

| # | Phase | Prime Command | SDLC Command | Auto-Prime |
|---|-------|---------------|--------------|------------|
| 1 | **SPECIFY** | `/sdlc/prime/1-specify` | `/sdlc/1-specify/run` | ✅ |
| 2 | **DESIGN** | `/sdlc/prime/2-design` | `/sdlc/2-design/run` | ✅ |
| 3 | **TEST** | `/sdlc/prime/3-test` | `/sdlc/3-test/run` | ✅ |
| 4 | **IMPLEMENT** | `/sdlc/prime/4-implement` | `/sdlc/4-implement/run` | ✅ |
| 5 | **REVIEW** | `/sdlc/prime/5-review` | `/sdlc/5-review/run` | ✅ |
| 6 | **DEPLOY** | `/sdlc/prime/6-deploy` | `/sdlc/6-deploy/run` | ✅ |
| 7 | **MAINTAIN** | `/sdlc/prime/7-maintain` | `/sdlc/7-maintain/run` | ✅ |

## Manual Prime (Optional)

You can still run prime commands manually when you want to:
- Load context without executing the full SDLC phase
- Explore documentation and patterns
- Prepare for a task before running the full command

## Consistent Structure

```
docs/sdlc/                    # Documentation
├── 1-specify/
├── 2-design/
├── 3-test/
├── 4-implement/
├── 5-review/
├── 6-deploy/
└── 7-operate/

.claude/commands/sdlc/        # Slash Commands
├── prime/                    # Context priming
│   ├── 1-specify.md
│   ├── 2-design.md
│   ├── 3-test.md
│   ├── 4-implement.md
│   ├── 5-review.md
│   ├── 6-deploy.md
│   └── 7-operate.md
├── 1-specify/                # Specification commands
├── 2-design/                 # Design commands
├── 3-test/                   # Testing commands
├── 4-implement/              # Implementation commands
├── 5-review/                 # Review commands
├── 6-deploy/                 # Deployment commands
└── 7-maintain/               # Maintenance commands
```

## Workflow Pattern

**Automatic (Recommended):**

```bash
# Just run the SDLC command - prime executes automatically as Phase 0
/sdlc/3-test/run test.spec.ts spec.md
```

**Manual (Optional):**

```bash
# Step 1: Prime the context manually (for exploration)
/sdlc/prime/3-test

# Step 2: Execute the SDLC command (will skip redundant prime if already loaded)
/sdlc/3-test/run test.spec.ts spec.md
```

## What Each Prime Loads

| Prime | Reads | Scans | Checks |
|-------|-------|-------|--------|
| **1-specify** | Spec templates, constitution | Existing specs | Draft specs |
| **2-design** | Architecture docs, core package | Services, repos | Design artifacts |
| **3-test** | Playwright config, test rules | Test structure | Environment status |
| **4-implement** | DI container, routes, services | Pages, components | Dev servers |
| **5-review** | Review checklist, security docs | PR diff | CI status |
| **6-deploy** | CI workflows, deploy scripts | Environment config | Workflow runs |
| **7-maintain** | Maintenance docs, security | Logs | Incidents |

## Why Prime First?

1. **Context** - Load all relevant files before starting
2. **Patterns** - Understand project conventions
3. **Safety** - Know security requirements
4. **Efficiency** - No guessing or rediscovery

**Rule: Prime commands now auto-execute as Phase 0. Just run `/sdlc/N-phase/run` directly.**
