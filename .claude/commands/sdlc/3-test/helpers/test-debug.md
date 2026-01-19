---
allowed-tools: Task, Read, Bash, Glob, Grep, Write
argument-hint: <test-file> [debug-level] [retry-count]
description: Enhanced test debugging with verbose analysis and systematic failure categorization
---

# Enhanced Test Debugging

This command provides systematic test debugging with enhanced failure analysis, categorizing issues as test bugs vs app bugs, and implementing intelligent retry logic for comprehensive debugging.

## Variables

- **TEST_FILE**: First argument from $ARGUMENTS
  - Path to specific test file or test directory to debug
  - Can be absolute path or relative to test/ directory
  - Supports glob patterns (e.g., "test/unit/auth_*_test.dart")

- **DEBUG_LEVEL**: Second argument from $ARGUMENTS (optional, default: "comprehensive")
  - Debug intensity: "basic", "detailed", "comprehensive", "deep"
  - Controls verbosity and analysis depth
  - Determines which debugging techniques are applied

- **RETRY_COUNT**: Third argument from $ARGUMENTS (optional, default: 3)
  - Number of test execution retries for flaky test detection
  - Used to identify intermittent vs consistent failures
  - Range: 1-10 for reasonable execution time

## Usage

```bash
/test-debug <test-file> [debug-level] [retry-count]
```

Examples:
- `/test-debug test/unit/auth_migration_test.dart comprehensive 5`
- `/test-debug test/widget/billing_drawer_test.dart detailed`
- `/test-debug test/integration/ basic 2`
- `/test-debug "test/unit/auth_*_test.dart" deep 1`

## Enhanced Debugging Protocol

### Phase 1: Pre-Debug Analysis
```markdown
1. **Test File Discovery**: Use Glob to identify all matching test files
2. **Environment Validation**: Check Flutter environment and dependencies
3. **Firebase Emulator Check**: Verify emulator status if Firebase tests detected
4. **Existing Coverage Analysis**: Review current test implementation patterns
```

### Phase 2: Systematic Test Execution
```markdown
Multiple execution rounds with increasing verbosity:

**Round 1: Standard Execution**
- Basic project script execution with proper flavor
- Capture exit codes and basic output
- Identify immediate compilation or runtime errors

**Round 2: Verbose Analysis**
- Execute with --verbose flag for detailed output
- Capture stack traces and detailed error messages
- Analyze timing and async behavior patterns

**Round 3: Isolated Execution** (if Round 2 fails)
- Run individual tests in isolation
- Use specific test file targeting
- Identify interaction conflicts between tests

**Round 4: Debug Mode** (for comprehensive/deep levels)
- Execute with debug flags passed to project scripts
- Enable extended logging and state inspection
- Capture memory and performance metrics
```

### Phase 3: Failure Pattern Analysis

#### Test Bug Indicators
```markdown
- **Timing Issues**: Inconsistent failures across retry attempts
- **Environment Dependencies**: Failures related to emulator connectivity
- **Mock Inaccuracy**: Assertions failing due to mock setup issues
- **Async Race Conditions**: Timing-dependent test failures
- **Setup/Teardown Issues**: State pollution between tests
```

#### App Bug Indicators
```markdown
- **Consistent Logic Failures**: Same assertion fails across all retries
- **State Management Issues**: Provider or business logic errors
- **API Integration Problems**: Real functionality breaking
- **Cross-Platform Inconsistencies**: Platform-specific behavior issues
- **Data Integrity Problems**: Database or storage layer issues
```

## Intelligent Retry Logic

### Retry Strategy Implementation
```bash
# Execute test multiple times to detect patterns
for i in $(seq 1 $RETRY_COUNT); do
  echo "🔄 Execution Round $i/$RETRY_COUNT"

  # Determine test type and use appropriate script
  if [[ "$TEST_FILE" == integration_test* ]]; then
    # Use integration test script
    ./scripts/testing/run_integration_tests.sh > "test_run_${i}.log" 2>&1
    EXIT_CODE=$?
  elif [[ "$TEST_FILE" == *web* ]] || [[ "$TEST_FILE" =~ web.*test ]]; then
    # Use web test script with dev flavor
    ./scripts/test/run_test_web.sh dev "$TEST_FILE" > "test_run_${i}.log" 2>&1
    EXIT_CODE=$?
  else
    # Use mobile test script with dev flavor (default)
    ./scripts/test/run_test_mobile.sh dev "$TEST_FILE" --verbose > "test_run_${i}.log" 2>&1
    EXIT_CODE=$?
  fi

  # Record timing and results
  echo "Round $i: Exit Code $EXIT_CODE" >> execution_summary.txt

  # Early termination for consistent failures (app bugs)
  if [ $EXIT_CODE -ne 0 ] && [ $i -gt 1 ]; then
    # Check if failure pattern is consistent (likely app bug)
    if grep -q "same assertion failure" execution_summary.txt; then
      echo "🎯 Consistent failure detected - likely app bug"
      break
    fi
  fi
done
```

### Pattern Detection Logic
```markdown
**Flaky Test Detection:**
- Success rate < 100% indicates test instability
- Intermittent failures suggest environmental or timing issues
- Pattern analysis across retry attempts

**Consistent Failure Analysis:**
- Same failure across all retries indicates app bug
- Analyze assertion failures and stack traces
- Check for logical errors in application code
```

## Debug Level Specifications

### Basic Debug Level
```markdown
- Standard test execution with basic output
- Single retry attempt for confirmation
- Essential error categorization only
- Quick turnaround for rapid feedback
```

### Detailed Debug Level
```markdown
- Verbose test execution with stack traces
- Multiple retry attempts for pattern detection
- Firebase emulator health check if applicable
- Dependency and environment validation
```

### Comprehensive Debug Level (Default)
```markdown
- Full debugging protocol with all execution rounds
- Complete failure categorization and analysis
- Environment validation and optimization suggestions
- Detailed recommendations for test improvement
```

### Deep Debug Level
```markdown
- Inspector mode debugging with memory profiling
- Extended retry analysis with timing metrics
- Cross-platform compatibility checking
- Performance impact assessment of tests
```

## Environment Checks & Validation

### Firebase Emulator Integration
```bash
# Check if test uses Firebase and validate emulator status
if grep -q "FirebaseAuth\|Firestore" $TEST_FILE; then
  echo "🔥 Firebase test detected - checking emulator status"
  ./scripts/testing/test_emulators.sh

  if [ $? -ne 0 ]; then
    echo "⚠️  Firebase emulators not running - starting automatically"
    # Project scripts handle emulator lifecycle automatically
    echo "📋 Note: Project test scripts will manage emulator lifecycle"
  fi
fi
```

### Dependency Validation
```bash
# Verify Flutter environment
flutter doctor --verbose
flutter pub deps

# Check for missing test dependencies
flutter pub deps | grep -E "(test|mockito|integration_test)"

# Ensure project test environment is set up
if [ -f "./scripts/test/setup-test-environment.sh" ]; then
  echo "🔧 Running project test environment setup..."
  ./scripts/test/setup-test-environment.sh
fi
```

## Failure Categorization Output

### Test Bug Report Format
```markdown
## 🐛 TEST BUG DETECTED

**Failure Pattern**: Intermittent (Success Rate: X/Y)
**Category**: Environmental/Timing/Mock Issue
**Root Cause**: [Specific analysis]

**Immediate Fixes**:
- [ ] Add proper async waiting with pumpAndSettle()
- [ ] Fix mock setup/teardown issues
- [ ] Resolve timing dependencies
- [ ] Update environment configuration

**Test Code Issues**:
[Specific code snippets that need fixing]
```

### App Bug Report Format
```markdown
## 🚨 APPLICATION BUG DETECTED

**Failure Pattern**: Consistent (Failed: Y/Y attempts)
**Category**: Logic/State/Integration Issue
**Root Cause**: [Business logic or app functionality problem]

**Critical Issues**:
- [ ] Fix business logic in [component]
- [ ] Resolve state management issue
- [ ] Correct API integration problem
- [ ] Address cross-platform compatibility

**Application Code Issues**:
[Specific application code that needs fixing]
```

## Direct Execution Instructions

1. **Parse Arguments**: Extract TEST_FILE, DEBUG_LEVEL, and RETRY_COUNT from $ARGUMENTS
2. **Environment Setup**: Validate Flutter environment and start emulators if needed
3. **Test Discovery**: Use Glob to identify all matching test files
4. **Execute Debug Protocol**: Run systematic debugging based on debug level
5. **Pattern Analysis**: Analyze failure patterns across multiple attempts
6. **Categorize Issues**: Determine test bug vs app bug classification
7. **Generate Report**: Create detailed debugging report with recommendations
8. **Delegate Complex Issues**: Use qa-engineer agent for test fixes if needed

## QA Engineer Integration

For complex test fixes identified during debugging:

```markdown
Task: qa-engineer

"Fix identified test issues in $TEST_FILE based on debug analysis:

**Issue Type**: [Test Bug/App Bug from analysis]
**Specific Problems**: [List from debugging output]
**Recommended Fixes**: [From failure categorization]
**Priority Level**: [Based on failure consistency]

Focus on resolving the root cause issues identified through systematic debugging.
Ensure all fixes maintain test reliability and cross-platform compatibility."
```

## Success Criteria

Test debugging is **complete** when:
1. **Root Cause Identified** - Clear categorization as test bug vs app bug ✅
2. **Failure Pattern Analyzed** - Understanding of consistency/intermittency ✅
3. **Environment Validated** - All dependencies and emulators checked ✅
4. **Actionable Recommendations** - Specific fixes provided ✅
5. **Reproducible Results** - Debug results can be replicated ✅
6. **Clear Next Steps** - Whether to fix test code or application code ✅

This command provides systematic, intelligent test debugging that eliminates guesswork and provides clear direction for resolving test failures.