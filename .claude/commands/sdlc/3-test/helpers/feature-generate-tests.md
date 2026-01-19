---
allowed-tools: Task, Read, Glob, Grep, Bash, mcp__github__create_or_update_issue_comment
argument-hint: <feature_name> [smr_ticket] [coverage_target]
description: Generates comprehensive test suites for new features using parallel specialized testing agents
---

# Feature Test Generation

This command generates comprehensive test coverage for new features like #202 calendar integration by launching multiple specialized testing agents in parallel. It provides complete production readiness assessment through coordinated multi-agent testing workflows.

## Variables

- **FEATURE_NAME**: First argument from $ARGUMENTS
  - The name of the feature being tested (e.g., "calendar-integration", "oauth-flow")
  - Used to identify components and scope testing efforts
  - Should match the feature branch or component name for consistency

- **SMR_TICKET**: Second argument from $ARGUMENTS (optional)
  - GitHub issue number (e.g., "#202") for context and traceability
  - Used to fetch requirements and acceptance criteria
  - Links test results to project management workflow

- **COVERAGE_TARGET**: Third argument from $ARGUMENTS (optional, default: 90)
  - Target test coverage percentage for the feature
  - Drives test generation depth and scope
  - Used for quality gate validation

## Usage

```bash
/feature-generate-tests <feature_name> [smr_ticket] [coverage_target]
```

Examples:
- `/feature-generate-tests calendar-integration #202 95`
- `/feature-generate-tests oauth-authentication #245`
- `/feature-generate-tests user-onboarding #198 85`

## Parallel Agent Orchestration

This command implements **simultaneous multi-agent deployment** for maximum efficiency:

### **Phase 1: Feature Analysis & Planning**
- Analyze feature scope and component dependencies
- Identify testing domains and coverage requirements
- Plan coordinated testing strategy across all agents

### **Phase 2: Parallel Agent Deployment**
Deploy 5 specialized testing agents **simultaneously**:

```markdown
Task 1: qa-engineer (Unit Testing Specialist)
- Focus: Business logic, ViewModels, services, utilities
- Target: >95% line coverage for critical paths
- Scope: Individual component testing with comprehensive mocking

Task 2: qa-engineer (Widget Testing Specialist)
- Focus: UI components, state binding, user interactions
- Target: All user-facing widgets and navigation flows
- Scope: Flutter widget testing with Provider integration

Task 3: qa-engineer (Integration Testing Specialist)
- Focus: End-to-end feature workflows and system integration
- Target: Complete user journeys across platforms
- Scope: Flutter integration_test package for cross-platform validation

Task 4: qa-engineer (Performance Testing Specialist)
- Focus: Load testing, benchmarking, memory optimization
- Target: Frame rate >60fps, memory <100MB, load time <2s
- Scope: Performance profiling and optimization recommendations

Task 5: security-researcher (Security Testing Specialist)
- Focus: Authentication, authorization, data protection
- Target: OWASP compliance and vulnerability assessment
- Scope: Security audit with penetration testing scenarios
```

### **Phase 3: Coordinated Result Aggregation**
- Collect results from all 5 agents in parallel
- Validate zero compilation errors across all tests
- Generate unified coverage report and production readiness assessment
- Post comprehensive results to PR via GitHub MCP integration

## Execution Workflow

### Feature Scope Analysis
1. **Component Discovery**: Use Glob and Grep to identify feature-related files
2. **Dependency Mapping**: Analyze imports and relationships between components
3. **Existing Test Review**: Check current test coverage and identify gaps
4. **Requirements Gathering**: Fetch GitHub issue details if provided

### Parallel Testing Agent Deployment
```markdown
Launch all 5 specialists simultaneously with detailed instructions:

**Unit Testing Agent Instructions:**
"Generate comprehensive unit tests for $FEATURE_NAME feature components. Focus on:
- Business logic validation with edge cases
- ViewModel testing with state management
- Service layer testing with proper mocking
- Utility function testing with boundary conditions
- Target coverage: $COVERAGE_TARGET%
- Zero compilation errors required"

**Widget Testing Agent Instructions:**
"Create Flutter widget tests for $FEATURE_NAME UI components. Focus on:
- Widget rendering and state changes
- User interaction simulation
- Provider integration testing
- Navigation and routing validation
- Cross-platform widget behavior
- Target coverage: All user-facing components"

**Integration Testing Agent Instructions:**
"Develop end-to-end integration tests for $FEATURE_NAME workflows. Focus on:
- Complete user journey testing
- Cross-platform validation (Web, iOS, Android)
- Data persistence and state management
- API integration and error handling
- Flutter integration_test package exclusively
- Target scenarios: All critical user paths"

**Performance Testing Agent Instructions:**
"Create performance benchmarks for $FEATURE_NAME. Focus on:
- Load testing and stress scenarios
- Memory usage optimization
- Frame rate and rendering performance
- Network request efficiency
- Platform-specific performance characteristics
- Target metrics: >60fps, <100MB memory, <2s load"

**Security Testing Agent Instructions:**
"Conduct security assessment for $FEATURE_NAME. Focus on:
- Authentication and authorization flows
- Data encryption and protection
- Input validation and sanitization
- API security and CORS configuration
- OWASP compliance validation
- Target: Zero high/critical vulnerabilities"
```

### Quality Validation Protocol
1. **Compilation Verification**: Ensure ALL generated tests compile successfully
2. **Execution Validation**: Run all test suites and verify PASS status
3. **Coverage Analysis**: Aggregate coverage metrics across all testing domains
4. **Performance Benchmarking**: Validate performance targets are met
5. **Security Compliance**: Confirm no critical vulnerabilities identified

### Production Readiness Assessment
Generate definitive **YES/NO** production readiness decision based on:
- Test coverage meeting target percentage
- Zero compilation errors across all tests
- All performance benchmarks achieved
- No critical security vulnerabilities
- Complete cross-platform validation

## Success Criteria

Feature testing is **complete** when:
1. **All 5 Testing Agents Complete** - Every specialist reports successful completion ✅
2. **Zero Compilation Errors** - All generated tests compile and run successfully ✅
3. **Coverage Target Met** - Feature coverage exceeds specified target percentage ✅
4. **Performance Benchmarks** - All performance metrics meet or exceed targets ✅
5. **Security Validation** - No critical or high vulnerabilities identified ✅
6. **Cross-Platform Success** - Tests pass on Web, iOS, and Android platforms ✅
7. **PR Documentation** - Comprehensive test report posted to GitHub PR ✅

## Direct Execution Instructions

1. **Parse Arguments**: Extract FEATURE_NAME, SMR_TICKET, and COVERAGE_TARGET from $ARGUMENTS
2. **Feature Analysis**: Use Glob/Grep to discover feature components and dependencies
3. **Deploy Parallel Agents**: Launch all 5 testing specialists simultaneously using Task tool
4. **Monitor Progress**: Track completion status from each agent
5. **Aggregate Results**: Collect test files, coverage data, and performance metrics
6. **Quality Validation**: Verify zero compilation errors and execute all test suites
7. **Production Assessment**: Generate definitive readiness decision
8. **PR Documentation**: Post comprehensive results using GitHub MCP integration

## GitHub Integration

After all agents complete, **automatically post comprehensive PR comment**:

```markdown
## 🧪 Feature Test Generation Complete - $FEATURE_NAME

**✅ Multi-Agent Testing Summary:**
- **Unit Testing Agent**: [X] tests, [Y]% coverage
- **Widget Testing Agent**: [X] tests, all UI components covered
- **Integration Testing Agent**: [X] scenarios, cross-platform validated
- **Performance Testing Agent**: All benchmarks met
- **Security Testing Agent**: Zero critical vulnerabilities

**📊 Comprehensive Coverage Metrics:**
- **Overall Coverage**: [X]% (Target: $COVERAGE_TARGET%)
- **Critical Path Coverage**: [X]%
- **Cross-Platform Validation**: ✅ Web, iOS, Android

**🚀 Production Readiness: [YES/NO]**
- ✅ Zero compilation errors
- ✅ All tests passing
- ✅ Performance targets achieved
- ✅ Security compliance verified

**📁 Generated Test Files:**
[List all test files with absolute paths]

**🎯 Quality Gates Status:** ✅ ALL PASSED
Ready for deployment to production environment.
```

This command ensures comprehensive, production-ready test coverage through intelligent parallel agent coordination, providing definitive quality assurance for new feature development.