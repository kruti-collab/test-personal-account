# Prime IMPLEMENT - Load Implementation Context

Prime context window with everything needed for feature implementation work.

## Step 1: Read Implementation Documentation

Read these files to understand implementation patterns:

1. Read `docs/sdlc/4-implement/README.md`
2. Read `docs/sdlc/4-implement/development/SETUP.md` (if exists)
3. Read `.claude/rules/api-development.md`
4. Read `.claude/rules/dashboard-development.md`

## Step 2: Read Core Implementation Files

Read key files that define implementation patterns:

1. Read `apps/api/src/index.ts`
2. Read `apps/api/src/di/container.ts`
3. Read `packages/core/src/index.ts`
4. Read `apps/dashboard/src/main.tsx`

## Step 3: Read Security & API Guidelines

1. Read `apps/api/SECURITY.md`
2. Read `docs/development/API_SECURITY.md` (if exists)

## Step 4: Scan Implementation Structure

```bash
# API routes (what endpoints exist)
ls -la apps/api/src/routes/

# Core services (business logic)
ls -la packages/core/src/services/

# Dashboard pages
ls -la apps/dashboard/src/pages/

# Dashboard components
ls -la apps/dashboard/src/components/

# Shared types
ls -la packages/types/src/
```

## Step 5: Read Example Implementations

Read examples of each layer:

1. Read one route file from `apps/api/src/routes/`
2. Read one service from `packages/core/src/services/`
3. Read one page from `apps/dashboard/src/pages/`
4. Read one context from `apps/dashboard/src/contexts/`

## Step 6: Check Current Work

```bash
# Current branch
git branch --show-current

# Uncommitted implementation changes
git status --short | grep -E "apps/|packages/"

# Recent implementation commits
git log --oneline -5 -- "apps/" "packages/"

# Check if dev servers running
curl -s http://localhost:3000/health 2>/dev/null && echo "API: ✓" || echo "API: ✗"
curl -s http://localhost:5173 2>/dev/null && echo "Dashboard: ✓" || echo "Dashboard: ✗"
```

## Implementation Patterns (Memorize These)

### API Route Pattern

```typescript
// apps/api/src/routes/organizations.ts
import { container } from '../di/container'

router.get('/organizations', async (req, res) => {
  const service = container.get<OrganizationService>('OrganizationService')
  const orgs = await service.listOrganizations(req.user.id)
  res.json(orgs)
})
```

### Service Pattern

```typescript
// packages/core/src/services/organization.service.ts
export class OrganizationService {
  constructor(private repo: OrganizationRepository) {}

  async listOrganizations(userId: string): Promise<Organization[]> {
    return this.repo.findByUserId(userId)
  }
}
```

### Dashboard Context Pattern

```typescript
// apps/dashboard/src/contexts/AuthContext.tsx
export const AuthProvider: React.FC = ({ children }) => {
  const [user, setUser] = useState<User | null>(null)
  
  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  )
}
```

### Security Rules

| Rule | Implementation |
|------|----------------|
| Tokens | httpOnly cookies only |
| Secrets | GCP Secret Manager |
| Auth | Validate on every request |
| Errors | Sanitize messages |
| Rate limit | Auth endpoints: 10/min |

### Development Commands

```bash
# Start local dev
./scripts/run.sh

# Build component
./scripts/build.sh dashboard dev
./scripts/build.sh api dev

# Type check
pnpm tsc --noEmit

# Lint
pnpm lint
```

## CRITICAL RULES (MUST FOLLOW)

### CI Check Protocol
**NEVER approve or merge a PR until ALL CI checks pass.** Don't run tests locally while CI runs - wait for CI. Don't use `--admin` to bypass failures - fix them first.

### Fix Failures On The Spot
When discovering a bug or failure during SDLC workflow: fix immediately while having context. Don't skip past failures. Never use `--admin` bypass.

### GitHub Issue Linkage
**ALWAYS use "Closes #X" (not "Relates to") in PR descriptions for auto-close.**

## After Priming

You are now ready to run:
- `/sdlc/implement/run` - Execute implementation plan
- `/sdlc/implement/helpers/implement-feature` - Implement feature
- `/sdlc/implement/helpers/tdd-workflow/tdd-start` - Start TDD cycle

## Report After Priming

Summarize:
1. Current branch and implementation changes
2. Dev environment status
3. Key services and routes available
4. Ready to proceed with implementation work

