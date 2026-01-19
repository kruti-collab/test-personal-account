---
allowed-tools: Task, Read, Bash, Glob, Grep, Write, mcp__github__create_or_update_issue_comment
argument-hint: <test-results> [analysis-depth] [auto-fix]
description: Automated failure categorization with test bug vs app bug classification and fix recommendations
---

# Test Failure Analysis

This command provides automated analysis of test failure patterns and outputs, categorizing failures as test bugs vs app bugs using research-based criteria. Offers intelligent failure classification, root cause analysis, and actionable fix recommendations.

## Variables

- **TEST_RESULTS**: First argument from $ARGUMENTS
  - Path to test results file or directory containing test outputs
  - Supports JSON, log files, or directory with multiple result files
  - Can also be a test file path to analyze its latest failures

- **ANALYSIS_DEPTH**: Second argument from $ARGUMENTS (optional, default: "comprehensive")
  - Analysis thoroughness: "quick", "standard", "comprehensive", "forensic"
  - Controls depth of pattern analysis and recommendation detail
  - Higher levels provide more detailed root cause analysis

- **AUTO_FIX**: Third argument from $ARGUMENTS (optional, default: "suggest")
  - Auto-fix behavior: "suggest", "apply", "none"
  - "suggest": Provide fix recommendations only
  - "apply": Automatically apply safe test fixes via qa-engineer
  - "none": Analysis only, no fix recommendations

## Usage

```bash
/test-failure-analyze <test-results> [analysis-depth] [auto-fix]
```

Examples:
- `/test-failure-analyze test_results.json comprehensive apply`
- `/test-failure-analyze logs/flutter_test_output.log standard suggest`
- `/test-failure-analyze test/unit/auth_test.dart forensic none`
- `/test-failure-analyze test-results/ quick suggest`

## Failure Classification Framework

### Research-Based Classification Criteria

#### Test Bug Indicators (Internal Test Issues)
```markdown
**Pattern Recognition Signatures**:

1. **Timing/Async Issues**
   - "flutter test timeout"
   - "pumpAndSettle.*timeout"
   - "Future.*not completed"
   - "Timer.*still active"
   - Random pass/fail behavior

2. **Mock/Setup Problems**
   - "NoSuchMethodError.*mock"
   - "Mock.*not configured"
   - "setUp.*failed"
   - "tearDown.*error"
   - Inconsistent mock behavior

3. **Test Environment Issues**
   - "emulator.*not.*running"
   - "connection.*refused"
   - "Firebase.*emulator.*timeout"
   - Environment-dependent failures

4. **Test Code Logic Errors**
   - "expect.*called.*wrong"
   - "finder.*returned.*multiple"
   - "widget.*not.*found.*in.*tree"
   - Test assertion logic problems

5. **Flaky Test Patterns**
   - Intermittent failures
   - Platform-specific test issues
   - Resource contention errors
   - Timing-dependent assertions
```

#### App Bug Indicators (Application Code Issues)
```markdown
**Application Failure Signatures**:

1. **Business Logic Errors**
   - "assertion.*failed.*business.*logic"
   - "calculation.*incorrect"
   - "validation.*failed"
   - Consistent logical assertion failures

2. **State Management Issues**
   - "Provider.*not.*found"
   - "ChangeNotifier.*not.*updated"
   - "state.*inconsistent"
   - "listener.*not.*called"

3. **API/Integration Problems**
   - "http.*error.*5xx"
   - "Firebase.*auth.*failed"
   - "network.*request.*failed"
   - Actual API functionality issues

4. **Data/Database Issues**
   - "Firestore.*document.*not.*found"
   - "data.*integrity.*violation"
   - "serialization.*failed"
   - Real data problems

5. **Cross-Platform Bugs**
   - Platform-specific functionality errors
   - UI rendering inconsistencies
   - Native code integration issues
```

## Intelligent Analysis Engine

### Phase 1: Result Data Extraction
```bash
# Extract and parse test results from various formats
extract_test_results() {
  local input_path="$1"

  if [ -f "$input_path" ]; then
    # Single file analysis
    if [[ "$input_path" == *.json ]]; then
      parse_json_results "$input_path"
    elif [[ "$input_path" == *.log ]]; then
      parse_log_results "$input_path"
    elif [[ "$input_path" == *_test.dart ]]; then
      # Run test to get fresh results using project scripts
      if [[ "$input_path" == integration_test* ]]; then
        ./scripts/testing/run_integration_tests.sh > "temp_results.json" 2>&1
      elif [[ "$input_path" == *web* ]] || [[ "$input_path" =~ web.*test ]]; then
        ./scripts/test/run_test_web.sh dev "$input_path" > "temp_results.json" 2>&1
      else
        ./scripts/test/run_test_mobile.sh dev "$input_path" --reporter=json > "temp_results.json" 2>&1
      fi
      parse_json_results "temp_results.json"
    fi
  elif [ -d "$input_path" ]; then
    # Directory analysis - aggregate multiple result files
    find "$input_path" -name "*.json" -o -name "*.log" | while read result_file; do
      echo "📄 Analyzing: $result_file"
      extract_test_results "$result_file"
    done
  fi
}
```

### Phase 2: Pattern Recognition & Classification
```bash
# Analyze failure patterns using research criteria
classify_failures() {
  local results_data="$1"

  # Initialize classification counters
  test_bug_score=0
  app_bug_score=0

  # Test Bug Pattern Detection
  echo "🔍 Analyzing test bug patterns..."

  # Timing/Async issues
  timing_issues=$(grep -c -E "(timeout|pumpAndSettle.*timeout|Future.*not completed)" "$results_data")
  if [ $timing_issues -gt 0 ]; then
    test_bug_score=$((test_bug_score + timing_issues * 3))
    echo "   ⏱️  Timing issues detected: $timing_issues"
  fi

  # Mock/Setup problems
  mock_issues=$(grep -c -E "(NoSuchMethodError.*mock|Mock.*not configured|setUp.*failed)" "$results_data")
  if [ $mock_issues -gt 0 ]; then
    test_bug_score=$((test_bug_score + mock_issues * 3))
    echo "   🎭 Mock issues detected: $mock_issues"
  fi

  # Environment dependencies
  env_issues=$(grep -c -E "(emulator.*not.*running|connection.*refused|Firebase.*emulator)" "$results_data")
  if [ $env_issues -gt 0 ]; then
    test_bug_score=$((test_bug_score + env_issues * 2))
    echo "   🌍 Environment issues detected: $env_issues"
  fi

  # App Bug Pattern Detection
  echo "🔍 Analyzing app bug patterns..."

  # Business logic errors
  logic_errors=$(grep -c -E "(assertion.*failed.*business|calculation.*incorrect|validation.*failed)" "$results_data")
  if [ $logic_errors -gt 0 ]; then
    app_bug_score=$((app_bug_score + logic_errors * 4))
    echo "   💼 Business logic errors detected: $logic_errors"
  fi

  # State management issues
  state_issues=$(grep -c -E "(Provider.*not.*found|ChangeNotifier.*not.*updated|state.*inconsistent)" "$results_data")
  if [ $state_issues -gt 0 ]; then
    app_bug_score=$((app_bug_score + state_issues * 3))
    echo "   📊 State management issues detected: $state_issues"
  fi

  # API/Integration problems
  api_issues=$(grep -c -E "(http.*error.*5xx|Firebase.*auth.*failed|network.*request.*failed)" "$results_data")
  if [ $api_issues -gt 0 ]; then
    app_bug_score=$((app_bug_score + api_issues * 3))
    echo "   🔌 API integration issues detected: $api_issues"
  fi
}
```

### Phase 3: Root Cause Analysis
```markdown
**Forensic Analysis Process**:

1. **Stack Trace Analysis**: Parse call stacks to identify failure origin
2. **Timing Pattern Analysis**: Check for time-dependent failure patterns
3. **Environmental Correlation**: Match failures with environmental conditions
4. **Code Path Analysis**: Trace execution paths leading to failures
5. **Historical Pattern Matching**: Compare with known failure patterns
```

## Analysis Depth Levels

### Quick Analysis
```markdown
- Basic pattern recognition using key failure signatures
- Simple test bug vs app bug classification
- Essential fix recommendations only
- Turnaround time: < 30 seconds
```

### Standard Analysis (Default)
```markdown
- Comprehensive pattern analysis with detailed categorization
- Root cause analysis for each failure type
- Specific fix recommendations with code examples
- Environmental dependency analysis
```

### Comprehensive Analysis
```markdown
- Deep pattern recognition with cross-reference analysis
- Historical failure pattern comparison
- Multi-dimensional failure classification
- Detailed fix implementation plans with priority ranking
```

### Forensic Analysis
```markdown
- Complete failure chain reconstruction
- Advanced pattern correlation analysis
- Predictive failure analysis based on trends
- Expert-level debugging recommendations
- Performance impact assessment
```

## Intelligent Fix Recommendation Engine

### Test Bug Fix Patterns
```markdown
**Timing Issue Fixes**:
```dart
// Problem: Timing-dependent test failures
// Fix: Proper async handling
await tester.pumpAndSettle(Duration(seconds: 5));
await expectLater(
  future,
  completion(equals(expected)),
  reason: 'Clear timeout context'
);
```

**Mock Setup Fixes**:
```dart
// Problem: Inconsistent mock behavior
// Fix: Proper mock lifecycle management
setUp(() async {
  mockService = MockAuthService();
  when(mockService.authenticate(any))
    .thenAnswer((_) async => AuthResult.success());
});

tearDown(() {
  reset(mockService);
  clearInteractions(mockService);
});
```

**Environment Dependency Fixes**:
```dart
// Problem: Firebase emulator dependency
// Fix: Use project test environment setup
setUpAll(() async {
  // Project scripts handle emulator lifecycle automatically
  await setupProjectTestEnvironment();
  await initializeTestFirebase();
});

test('auth test with project environment', () async {
  // Project scripts ensure emulators are available
  // Test implementation - no manual emulator checks needed
});
```
```

### App Bug Fix Patterns
```markdown
**State Management Fixes**:
```dart
// Problem: Provider state not updating
// Fix: Proper state management
class AuthProvider extends ChangeNotifier {
  void login(User user) {
    _user = user;
    notifyListeners(); // This was missing
  }
}
```

**Business Logic Fixes**:
```dart
// Problem: Calculation errors
// Fix: Corrected business logic
double calculateTotal(List<Item> items) {
  return items.fold(0.0, (sum, item) =>
    sum + (item.price * item.quantity)); // Fixed multiplication
}
```
```

## Automated Fix Application

### Safe Fix Application Logic
```bash
# Apply safe test fixes automatically when AUTO_FIX="apply"
apply_safe_fixes() {
  local analysis_results="$1"

  # Only apply fixes for clear test bugs with high confidence
  if [ $test_bug_score -gt $((app_bug_score * 2)) ]; then
    echo "🔧 Applying safe test fixes..."

    # Delegate to qa-engineer for test code fixes
    cat > fix_instructions.md << EOF
# Automated Test Fix Instructions

**Classification**: TEST BUG (Confidence: High)
**Primary Issues**: [List from analysis]

## Required Fixes:
$(generate_specific_fixes_from_analysis)

## Validation:
- Ensure all fixes maintain test logic integrity
- Verify fixes address root causes identified
- Run stability check after fixes
- Update test documentation if needed

Focus on test code improvements only - no application code changes.
EOF

    # Execute fixes via qa-engineer
    task_qa_engineer "Apply test fixes based on analysis: $(cat fix_instructions.md)"
  else
    echo "⚠️  Mixed classification - manual review required"
    echo "   Test bug score: $test_bug_score"
    echo "   App bug score: $app_bug_score"
  fi
}
```

## Comprehensive Analysis Report

### Failure Analysis Report Format
```markdown
# 🔬 Test Failure Analysis Report

## 📋 Analysis Summary
- **Input**: $TEST_RESULTS
- **Analysis Depth**: $ANALYSIS_DEPTH
- **Analysis Date**: $(date)
- **Total Failures Analyzed**: X

## 🎯 Classification Results

### Overall Classification
**PRIMARY ISSUE TYPE**: [TEST BUG / APP BUG / MIXED]
- **Confidence Level**: X% (High/Medium/Low)
- **Test Bug Score**: X points
- **App Bug Score**: X points

### Detailed Classification Breakdown

#### 🐛 Test Bug Issues (X%)
| Issue Type | Count | Confidence | Impact |
|------------|-------|------------|---------|
| Timing/Async | X | High | Critical |
| Mock Setup | X | High | Major |
| Environment | X | Medium | Minor |
| Test Logic | X | High | Major |

#### 🚨 App Bug Issues (X%)
| Issue Type | Count | Confidence | Impact |
|------------|-------|------------|---------|
| Business Logic | X | High | Critical |
| State Management | X | High | Major |
| API Integration | X | Medium | Major |
| Data Issues | X | High | Critical |

## 🔍 Root Cause Analysis

### Primary Root Causes
1. **[Issue Type]** - [Detailed analysis]
   - **Evidence**: [Specific failure patterns]
   - **Impact**: [Effect on test reliability]
   - **Fix Priority**: [High/Medium/Low]

### Failure Chain Reconstruction
[Detailed trace of how failures cascade through the system]

### Environmental Factors
- **Firebase Emulator Dependency**: [Impact analysis]
- **Network Dependencies**: [Analysis of network-related failures]
- **System Resource Impact**: [Memory/CPU correlation]

## 🛠️  Fix Recommendations

### Immediate Actions (High Priority)
- [ ] **Fix critical timing issues** in [specific tests]
- [ ] **Resolve mock setup problems** in [specific components]
- [ ] **Address app logic errors** in [specific modules]

### Test Code Improvements
```dart
// Example fixes for identified test issues
[Specific code examples with before/after]
```

### Application Code Fixes
```dart
// Example fixes for identified app issues
[Specific code examples with before/after]
```

## 📊 Impact Assessment

### Test Reliability Impact
- **Current Reliability**: X% (based on failure patterns)
- **Projected Improvement**: +X% after fixes
- **Risk Level**: [High/Medium/Low]

### Development Velocity Impact
- **Time Lost to Flaky Tests**: X hours/week
- **False Positive Rate**: X%
- **Developer Confidence**: [Affected/Maintained]

## 🎯 Action Plan

### Phase 1: Critical Fixes (Days 1-2)
- [ ] Address all high-confidence test bugs
- [ ] Fix critical app bugs affecting core functionality
- [ ] Improve test environment stability

### Phase 2: Optimization (Days 3-7)
- [ ] Enhance test reliability patterns
- [ ] Implement better error handling
- [ ] Add preventive measures for common patterns

### Phase 3: Monitoring (Ongoing)
- [ ] Set up automated failure analysis
- [ ] Monitor fix effectiveness
- [ ] Establish failure pattern alerts

## 🔄 Validation Plan
1. **Apply Recommended Fixes**
2. **Run Test Stability Check** to verify improvements
3. **Monitor Failure Patterns** for regression
4. **Update Analysis Based on Results**

---
*Generated by Claude Code - Intelligent Test Failure Analysis*
```

## Direct Execution Instructions

1. **Parse Arguments**: Extract TEST_RESULTS, ANALYSIS_DEPTH, and AUTO_FIX from $ARGUMENTS
2. **Data Extraction**: Parse test results from various formats (JSON, logs, files)
3. **Pattern Analysis**: Apply research-based classification criteria
4. **Root Cause Analysis**: Perform deep analysis based on depth level
5. **Fix Generation**: Create specific fix recommendations
6. **Auto-Fix Application**: Apply safe fixes if requested
7. **Report Generation**: Create comprehensive analysis report
8. **Integration**: Post results to GitHub PR if applicable

## QA Engineer Integration

For applying identified test fixes:

```markdown
Task: qa-engineer

"Apply test fixes identified by failure analysis:

**Classification**: [TEST BUG/APP BUG/MIXED from analysis]
**Confidence Level**: [High/Medium/Low]
**Specific Issues**: [Detailed list from analysis]
**Fix Recommendations**: [Specific code fixes provided]
**Priority Level**: [Based on impact assessment]

Focus on implementing the recommended fixes while maintaining test integrity.
Verify all fixes address the root causes identified in the analysis."
```

This command provides intelligent, research-based failure analysis that eliminates guesswork in determining whether test failures are due to test code issues or application bugs, enabling teams to focus their debugging efforts effectively.