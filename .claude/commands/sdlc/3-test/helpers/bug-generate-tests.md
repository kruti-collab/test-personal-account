---
allowed-tools: Task, Read, Glob, Grep, Bash, mcp__github__create_or_update_issue_comment, gh issue list --search
argument-hint: <bug_identifier> [test_type] [regression_scope]
description: Creates targeted regression tests for bug fixes with validation of fix effectiveness
---

# Bug Fix Test Generation

This command creates comprehensive regression test suites for bug fixes, validates fix effectiveness, and ensures no new issues are introduced. It leverages parallel specialized testing agents to provide thorough bug validation and regression prevention.

## Variables

- **BUG_IDENTIFIER**: First argument from $ARGUMENTS
  - Bug ticket number (e.g., "#278") or descriptive bug name
  - Used to identify the specific bug context and affected components
  - Links test generation to issue tracking and resolution workflow

- **TEST_TYPE**: Second argument from $ARGUMENTS (optional, default: "comprehensive")
  - Type of testing focus: "unit", "integration", "regression", "comprehensive"
  - Determines which testing agents are deployed and testing scope
  - Allows targeted testing for specific bug categories

- **REGRESSION_SCOPE**: Third argument from $ARGUMENTS (optional, default: "affected")
  - Scope of regression testing: "affected", "module", "system", "full"
  - Controls how broadly regression tests are applied
  - Balances thoroughness with execution efficiency

## Usage

```bash
/bug-generate-tests <bug_identifier> [test_type] [regression_scope]
```

Examples:
- `/bug-generate-tests #278 comprehensive system`
- `/bug-generate-tests login-crash-bug unit affected`
- `/bug-generate-tests #299 regression module`
- `/bug-generate-tests calendar-sync-issue integration affected`

## Parallel Agent Orchestration

This command implements **targeted multi-agent deployment** based on bug characteristics:

### **Phase 1: Bug Analysis & Impact Assessment**
- Fetch bug details from GitHub if GitHub issue provided
- Analyze affected components and potential impact areas
- Identify root cause category and testing strategy
- Plan regression testing scope and coverage requirements

### **Phase 2: Adaptive Agent Deployment**
Deploy specialized agents based on bug type and scope:

#### **Comprehensive Testing** (Default)
```markdown
Task 1: qa-engineer (Bug Reproduction Specialist)
- Focus: Reproduce bug conditions and validate fix
- Target: Create reliable reproduction test case
- Scope: Exact bug scenario recreation and fix validation

Task 2: qa-engineer (Regression Testing Specialist)
- Focus: Prevent regression in affected and related components
- Target: Comprehensive regression test suite
- Scope: Related functionality validation and edge case testing

Task 3: qa-engineer (Integration Impact Specialist)
- Focus: System-wide impact assessment and testing
- Target: End-to-end workflow validation
- Scope: Integration points and dependent component testing

Task 4: security-researcher (Security Impact Specialist)
- Focus: Security implications of bug and fix
- Target: Vulnerability assessment and security regression
- Scope: Authentication, authorization, and data protection validation
```

#### **Targeted Testing** (Unit/Integration)
```markdown
Task 1: qa-engineer (Focused Testing Specialist)
- Focus: Targeted testing based on TEST_TYPE parameter
- Target: Specific test domain coverage
- Scope: Component-level or integration-level testing

Task 2: qa-engineer (Regression Prevention Specialist)
- Focus: Minimal regression testing for focused fixes
- Target: Critical path validation
- Scope: Core functionality preservation testing
```

### **Phase 3: Bug Fix Validation Protocol**
- Validate bug reproduction before fix testing
- Confirm fix resolves original issue
- Verify no new issues introduced by fix
- Generate regression prevention recommendations

## Bug Categories & Testing Strategies

### **Critical System Bugs** (#278 type)
- **Focus**: Security, stability, data integrity
- **Agents**: All 4 specialists for comprehensive coverage
- **Scope**: System-wide regression testing
- **Validation**: Security audit and stability testing

### **UI/UX Bugs** (Visual, interaction issues)
- **Focus**: Widget testing and user experience validation
- **Agents**: Widget specialist + Integration specialist
- **Scope**: UI component regression and user flow testing
- **Validation**: Cross-platform visual and interaction testing

### **Business Logic Bugs** (Calculation, workflow errors)
- **Focus**: Unit testing and business rule validation
- **Agents**: Unit specialist + Regression specialist
- **Scope**: Business logic comprehensive testing
- **Validation**: Edge case and boundary condition testing

### **Performance Bugs** (Memory leaks, slow responses)
- **Focus**: Performance testing and optimization
- **Agents**: Performance specialist + Integration specialist
- **Scope**: Performance regression and optimization testing
- **Validation**: Benchmark comparison and resource usage monitoring

### **Integration Bugs** (API, service connectivity)
- **Focus**: Integration testing and error handling
- **Agents**: Integration specialist + Security specialist
- **Scope**: API integration and error condition testing
- **Validation**: Service interaction and failure scenario testing

## Execution Workflow

### Bug Context Analysis
1. **GitHub Integration**: Fetch bug details using GitHub CLI if GitHub issue provided
2. **Component Discovery**: Identify affected files and dependencies
3. **Impact Assessment**: Analyze potential regression areas
4. **Testing Strategy**: Determine optimal agent deployment pattern

### Bug Reproduction & Validation
```markdown
**Bug Reproduction Agent Instructions:**
"Create comprehensive bug reproduction tests for $BUG_IDENTIFIER. Focus on:
- Exact reproduction of reported bug conditions
- Multiple reproduction scenarios and edge cases
- Before-fix test validation (should fail)
- After-fix test validation (should pass)
- Clear reproduction steps and expected outcomes
- Environment-specific reproduction if applicable"
```

### Regression Test Generation
```markdown
**Regression Testing Agent Instructions:**
"Generate regression test suite for $BUG_IDENTIFIER fix. Focus on:
- Related functionality that could be affected by fix
- Edge cases around the bug area
- Integration points that might be impacted
- Historical bug patterns in similar components
- Cross-platform regression prevention
- Scope: $REGRESSION_SCOPE level testing"
```

### Integration Impact Assessment
```markdown
**Integration Impact Agent Instructions:**
"Assess system-wide impact of $BUG_IDENTIFIER fix. Focus on:
- End-to-end workflow validation
- Dependent service interaction testing
- Data flow and state management validation
- API integration point testing
- Cross-component communication validation
- Performance impact assessment"
```

### Security Impact Validation
```markdown
**Security Impact Agent Instructions:**
"Evaluate security implications of $BUG_IDENTIFIER and its fix. Focus on:
- Security vulnerability introduction prevention
- Authentication and authorization impact
- Data protection and privacy validation
- Input validation and sanitization testing
- Access control and permission verification
- OWASP compliance maintenance"
```

## Quality Validation Protocol

### Pre-Fix Validation
1. **Bug Reproduction**: Confirm tests can reproduce the original bug
2. **Test Environment**: Verify test environment matches bug conditions
3. **Baseline Metrics**: Capture performance and functionality baselines

### Post-Fix Validation
1. **Fix Effectiveness**: Confirm bug reproduction tests now pass
2. **Regression Prevention**: Verify all regression tests pass
3. **Performance Impact**: Ensure fix doesn't degrade performance
4. **Security Assessment**: Confirm no new vulnerabilities introduced

### Comprehensive Validation
1. **Zero Compilation Errors**: All generated tests compile successfully
2. **Test Execution**: All tests pass in target environments
3. **Coverage Analysis**: Adequate coverage of bug area and dependencies
4. **Cross-Platform Testing**: Bug fix works on all target platforms

## Success Criteria

Bug fix testing is **complete** when:
1. **Bug Reproduction Confirmed** - Can reliably reproduce original bug ✅
2. **Fix Validation Passed** - Bug reproduction tests pass after fix ✅
3. **Zero Regression Issues** - All regression tests pass ✅
4. **Zero Compilation Errors** - All generated tests compile successfully ✅
5. **Security Impact Cleared** - No new security vulnerabilities ✅
6. **Performance Maintained** - Fix doesn't degrade system performance ✅
7. **Cross-Platform Validated** - Fix works on all target platforms ✅
8. **Documentation Complete** - Test report posted to bug tracking system ✅

## Direct Execution Instructions

1. **Parse Arguments**: Extract BUG_IDENTIFIER, TEST_TYPE, and REGRESSION_SCOPE from $ARGUMENTS
2. **Bug Analysis**: Fetch bug details from GitHub if GitHub issue provided
3. **Impact Assessment**: Analyze affected components and regression scope
4. **Deploy Adaptive Agents**: Launch appropriate testing specialists based on bug type
5. **Bug Reproduction**: Create and validate bug reproduction tests
6. **Regression Testing**: Generate comprehensive regression test suite
7. **Fix Validation**: Verify fix effectiveness and prevent new issues
8. **Quality Assurance**: Execute all tests and validate results
9. **Documentation**: Post results to bug tracking and PR systems

## GitHub & GitHub Integration

### GitHub Bug Details Fetching
```markdown
Use gh issue list --search to fetch bug context:
- Issue description and reproduction steps
- Affected components and severity
- Acceptance criteria for fix validation
- Related issues and historical context
```

### Automated Bug Test Report
```markdown
## 🐛 Bug Fix Test Generation Complete - $BUG_IDENTIFIER

**🔍 Bug Analysis Summary:**
- **Bug Type**: [Category identified from analysis]
- **Affected Components**: [List of impacted files/modules]
- **Regression Scope**: $REGRESSION_SCOPE
- **Testing Strategy**: $TEST_TYPE

**✅ Test Generation Results:**
- **Bug Reproduction Tests**: [X] tests - Validates fix effectiveness
- **Regression Tests**: [X] tests - Prevents future regressions
- **Integration Impact Tests**: [X] tests - System-wide validation
- **Security Impact Tests**: [X] tests - Vulnerability prevention

**🧪 Validation Results:**
- ✅ Bug Reproduction: CONFIRMED (tests fail before fix, pass after)
- ✅ Regression Prevention: VALIDATED (all related functionality preserved)
- ✅ Integration Impact: ASSESSED (no system-wide issues)
- ✅ Security Impact: CLEARED (no new vulnerabilities)

**📊 Test Coverage:**
- **Bug Area Coverage**: [X]%
- **Regression Coverage**: [X]%
- **Integration Points**: [X] tested

**🚀 Fix Validation Status: ✅ APPROVED**
- Bug reproduction tests confirm fix effectiveness
- Comprehensive regression suite prevents future issues
- No new security vulnerabilities introduced
- System stability and performance maintained

**📁 Generated Test Files:**
[List all test files with absolute paths]

**🎯 Next Steps:**
- Merge fix with confidence - comprehensive test coverage complete
- Monitor production for any unforeseen edge cases
- Consider additional testing for related high-risk areas

🤖 Generated with Claude Code - Bug Fix Validation & Regression Prevention
```

This command ensures thorough bug fix validation through intelligent agent coordination, providing confidence that fixes resolve issues without introducing new problems.