# QA Parallel Testing

Launches multiple QA engineers in parallel for comprehensive testing coverage.

## Usage

```bash
/qa-parallel <feature-name> [coverage-level]
```

Examples:
- `/qa-parallel #202 comprehensive`
- `/qa-parallel calendar-integration standard`
- `/qa-parallel oauth-flow minimal`

## Variables

- **FEATURE**: First argument - the feature/component to test
- **COVERAGE**: Second argument (optional, default: "standard") - testing depth level

## Execution

This command immediately launches 4 QA engineers in parallel:

1. **Unit Testing QA** - Core functionality and business logic
2. **Integration Testing QA** - API connections and workflows
3. **UI/Widget Testing QA** - User interface and interactions
4. **Security Testing QA** - Authentication and data security

Each QA engineer runs independently with specialized focus, then results are aggregated for comprehensive coverage assessment.

## Implementation

When invoked, execute these Task calls simultaneously:

```
Task 1: qa-engineer (Unit Testing Focus)
- Focus: Core business logic, utility functions, services
- Target: >90% unit test coverage
- Scope: Mock dependencies, edge cases, error handling

Task 2: qa-engineer (Integration Testing Focus)
- Focus: API integrations, external services, workflows
- Target: Complete integration validation
- Scope: Network calls, authentication flows, data persistence

Task 3: qa-engineer (UI/Widget Testing Focus)
- Focus: User interface components and interactions
- Target: Complete UI test coverage
- Scope: Widget tests, user flows, responsive design

Task 4: qa-engineer (Security Testing Focus)
- Focus: Authentication, authorization, data protection
- Target: Zero security vulnerabilities
- Scope: OAuth flows, input validation, API security
```

## Success Criteria

- All 4 QA engineers complete their specialized testing
- Zero compilation errors across all generated tests
- Comprehensive coverage across all testing domains
- Clear pass/fail assessment for deployment readiness