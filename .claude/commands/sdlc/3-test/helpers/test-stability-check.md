---
allowed-tools: Task, Read, Bash, Glob, Grep, Write, mcp__github__create_or_update_issue_comment
argument-hint: [test-pattern] [iterations] [stability-threshold]
description: Statistical test reliability analysis with flaky test detection and stability metrics
---

# Test Stability Check

This command performs statistical analysis of test reliability by running tests multiple times to detect flaky behavior, environmental dependencies, and stability patterns. Provides comprehensive stability metrics and actionable recommendations for improving test reliability.

## Variables

- **TEST_PATTERN**: First argument from $ARGUMENTS (optional, default: "test/")
  - Test pattern or specific file to analyze for stability
  - Supports glob patterns and directory paths
  - Examples: "test/unit/", "test/widget/auth_*_test.dart", "specific_test.dart"

- **ITERATIONS**: Second argument from $ARGUMENTS (optional, default: 10)
  - Number of test execution iterations for statistical analysis
  - Range: 5-50 for meaningful statistical data
  - Higher iterations provide more accurate stability metrics

- **STABILITY_THRESHOLD**: Third argument from $ARGUMENTS (optional, default: 95)
  - Minimum pass rate percentage to consider test stable
  - Tests below threshold are flagged as flaky
  - Typical enterprise threshold: 95-99%

## Usage

```bash
/test-stability-check [test-pattern] [iterations] [stability-threshold]
```

Examples:
- `/test-stability-check test/unit/ 20 98`
- `/test-stability-check test/widget/billing_drawer_test.dart 15`
- `/test-stability-check "test/integration/*auth*" 25 99`
- `/test-stability-check` (default: all tests, 10 iterations, 95% threshold)

## Statistical Analysis Protocol

### Phase 1: Test Discovery & Preparation
```markdown
1. **Pattern Matching**: Use Glob to identify all tests matching the pattern
2. **Environment Baseline**: Establish consistent testing environment
3. **Dependency Verification**: Ensure all test dependencies are available
4. **Isolation Setup**: Configure for reliable test execution
```

### Phase 2: Iterative Execution Framework
```markdown
**Execution Strategy**:
- Execute each test file independently across all iterations
- Capture timing, exit codes, and failure details for each run
- Monitor system resources during execution
- Record environmental conditions for each iteration

**Data Collection Points**:
- Execution time per iteration
- Memory usage patterns
- Exit codes and failure types
- Environmental conditions (emulator status, system load)
- Error messages and stack traces
```

### Phase 3: Statistical Analysis Engine

#### Stability Metrics Calculation
```markdown
**Pass Rate Analysis**:
- Overall pass rate: (Successful runs / Total runs) × 100
- Trend analysis: Pass rate over time within iterations
- Variance analysis: Consistency of execution times

**Failure Pattern Detection**:
- Intermittent failures: Random pass/fail patterns
- Progressive failures: Degrading performance over iterations
- Environmental failures: Correlated with system conditions
- Timing failures: Duration-dependent failure patterns
```

#### Flaky Test Classification
```markdown
**Flaky Categories**:

1. **Highly Unstable** (< 70% pass rate)
   - Frequent random failures
   - Unpredictable behavior
   - Requires immediate attention

2. **Moderately Flaky** (70-90% pass rate)
   - Occasional failures
   - Potentially timing-related
   - Needs investigation and stabilization

3. **Marginally Unstable** (90-95% pass rate)
   - Rare failures under stress
   - Edge case issues
   - Monitor and improve when possible

4. **Stable** (> 95% pass rate)
   - Consistent performance
   - Reliable execution
   - Meets production standards
```

## Execution Implementation

### Iterative Test Runner
```bash
#!/bin/bash
# Statistical test execution framework

declare -A test_results
declare -A execution_times
declare -A failure_details

# Initialize statistics tracking
total_tests=0
total_executions=0
total_failures=0

for iteration in $(seq 1 $ITERATIONS); do
  echo "🔄 Stability Analysis - Iteration $iteration/$ITERATIONS"
  echo "=================================================="

  # Execute tests with detailed timing using project scripts
  start_time=$(date +%s.%N)

  # Determine test type and use appropriate project script
  if [[ "$TEST_PATTERN" == integration_test* ]] || [[ "$TEST_PATTERN" == *integration* ]]; then
    # Use integration test script
    if ./scripts/testing/run_integration_tests.sh > "iteration_${iteration}_results.json" 2>&1; then
      execution_status="PASS"
    else
      execution_status="FAIL"
      total_failures=$((total_failures + 1))
      failure_details["iteration_$iteration"]=$(cat "iteration_${iteration}_results.json")
    fi
  elif [[ "$TEST_PATTERN" == *web* ]] || [[ "$TEST_PATTERN" =~ web.*test ]]; then
    # Use web test script with dev flavor
    if ./scripts/test/run_test_web.sh dev "$TEST_PATTERN" > "iteration_${iteration}_results.json" 2>&1; then
      execution_status="PASS"
    else
      execution_status="FAIL"
      total_failures=$((total_failures + 1))
      failure_details["iteration_$iteration"]=$(cat "iteration_${iteration}_results.json")
    fi
  else
    # Use mobile test script with dev flavor (default)
    if ./scripts/test/run_test_mobile.sh dev "$TEST_PATTERN" > "iteration_${iteration}_results.json" 2>&1; then
      execution_status="PASS"
    else
      execution_status="FAIL"
      total_failures=$((total_failures + 1))
      failure_details["iteration_$iteration"]=$(cat "iteration_${iteration}_results.json")
    fi
  fi

  end_time=$(date +%s.%N)
  execution_time=$(echo "$end_time - $start_time" | bc)

  # Record results
  test_results["iteration_$iteration"]=$execution_status
  execution_times["iteration_$iteration"]=$execution_time

  echo "   Status: $execution_status (${execution_time}s)"

  # Brief pause between iterations to avoid resource conflicts
  sleep 1
done
```

### Environmental Correlation Analysis
```bash
# Capture environmental factors that might affect stability
monitor_environment() {
  local iteration=$1

  # System resources
  echo "$(date): Memory: $(free -h | grep Mem: | awk '{print $3}')" >> "env_${iteration}.log"
  echo "$(date): CPU Load: $(uptime | awk -F'load average:' '{print $2}')" >> "env_${iteration}.log"

  # Firebase emulator status (if applicable)
  if pgrep -f "firebase.*emulators" > /dev/null; then
    echo "$(date): Firebase emulators: RUNNING" >> "env_${iteration}.log"
  else
    echo "$(date): Firebase emulators: NOT_RUNNING" >> "env_${iteration}.log"
  fi

  # Network connectivity
  if ping -c 1 google.com &> /dev/null; then
    echo "$(date): Network: CONNECTED" >> "env_${iteration}.log"
  else
    echo "$(date): Network: DISCONNECTED" >> "env_${iteration}.log"
  fi
}
```

## Statistical Analysis & Reporting

### Stability Metrics Generation
```markdown
**Key Metrics Calculated**:

1. **Overall Pass Rate**: (Successful iterations / Total iterations) × 100
2. **Mean Execution Time**: Average time across all iterations
3. **Execution Variance**: Consistency of timing (lower is better)
4. **Failure Clustering**: Whether failures occur in patterns
5. **Environmental Correlation**: Relationship between failures and environment
```

### Flaky Test Identification Algorithm
```bash
# Analyze results for flaky behavior patterns
analyze_stability() {
  local pass_count=0
  local total_time=0
  local times=()

  # Calculate basic statistics
  for iteration in $(seq 1 $ITERATIONS); do
    if [ "${test_results[iteration_$iteration]}" = "PASS" ]; then
      pass_count=$((pass_count + 1))
    fi

    time=${execution_times[iteration_$iteration]}
    total_time=$(echo "$total_time + $time" | bc)
    times+=($time)
  done

  # Calculate pass rate
  pass_rate=$(echo "scale=2; $pass_count * 100 / $ITERATIONS" | bc)

  # Calculate timing statistics
  mean_time=$(echo "scale=2; $total_time / $ITERATIONS" | bc)

  # Determine stability classification
  if (( $(echo "$pass_rate >= $STABILITY_THRESHOLD" | bc -l) )); then
    stability_status="STABLE"
    stability_color="🟢"
  elif (( $(echo "$pass_rate >= 90" | bc -l) )); then
    stability_status="MARGINALLY_UNSTABLE"
    stability_color="🟡"
  elif (( $(echo "$pass_rate >= 70" | bc -l) )); then
    stability_status="MODERATELY_FLAKY"
    stability_color="🟠"
  else
    stability_status="HIGHLY_UNSTABLE"
    stability_color="🔴"
  fi
}
```

## Environmental Dependency Detection

### Emulator Dependency Analysis
```bash
# Test stability with/without Firebase emulators
test_emulator_dependency() {
  echo "🔥 Testing Firebase emulator dependency..."

  # Check emulator status using project testing tools
  ./scripts/testing/test_emulators.sh

  # Note: Project scripts automatically manage emulator lifecycle
  echo "📋 Project test scripts automatically manage Firebase emulators"
  echo "📋 Testing stability with project-managed emulator state"

  # Run stability subset with current emulator configuration
  run_stability_subset "PROJECT_MANAGED_EMULATORS"
}
```

### Network Dependency Analysis
```bash
# Test behavior under different network conditions
test_network_dependency() {
  echo "🌐 Testing network dependency patterns..."

  # Monitor network-related failures
  grep -i "network\|connection\|timeout" iteration_*_results.json > network_failures.log

  if [ -s network_failures.log ]; then
    echo "⚠️  Network-dependent test behavior detected"
  fi
}
```

## Comprehensive Stability Report

### Report Generation
```markdown
# Test Stability Analysis Report

## 📊 Statistical Summary
- **Test Pattern**: $TEST_PATTERN
- **Iterations Analyzed**: $ITERATIONS
- **Stability Threshold**: $STABILITY_THRESHOLD%
- **Analysis Date**: $(date)

## 🎯 Overall Stability Metrics
- **Pass Rate**: X.X% ($stability_color $stability_status)
- **Mean Execution Time**: X.Xs ± Y.Ys
- **Consistency Score**: X/10 (based on variance)
- **Environmental Stability**: [STABLE/DEPENDENT]

## 📈 Detailed Results

### Per-Test Stability Analysis
[For each test file analyzed]

**test/unit/example_test.dart**
- Pass Rate: XX% (X/Y successful)
- Avg Time: X.Xs
- Failure Pattern: [Random/Clustered/Progressive]
- Classification: [STABLE/FLAKY/UNSTABLE]

### 🔍 Flaky Tests Identified
[Tests below stability threshold]

| Test File | Pass Rate | Classification | Primary Issue |
|-----------|-----------|----------------|---------------|
| test1.dart | 85% | MODERATELY_FLAKY | Timing issues |
| test2.dart | 60% | HIGHLY_UNSTABLE | Mock problems |

## 🌍 Environmental Dependencies
- **Firebase Emulator Dependency**: [YES/NO]
- **Network Dependency**: [YES/NO]
- **System Resource Sensitivity**: [HIGH/MEDIUM/LOW]
- **Time-of-Day Variations**: [DETECTED/NONE]

## ⚠️  Failure Pattern Analysis
### Common Failure Types
1. **Timing Issues** (X% of failures)
   - Async operations not properly awaited
   - Race conditions in state updates
   - Insufficient test timeouts

2. **Environmental Issues** (X% of failures)
   - Emulator connectivity problems
   - Resource contention
   - Network dependencies

3. **Mock/Setup Issues** (X% of failures)
   - Inconsistent mock behavior
   - Setup/teardown problems
   - State pollution between runs

## 🛠️  Recommendations

### Immediate Actions Required
- [ ] **Fix highly unstable tests** (< 70% pass rate)
- [ ] **Investigate moderate flakiness** (70-90% pass rate)
- [ ] **Add proper async handling** where timing issues detected
- [ ] **Improve test isolation** for environmental dependencies

### Test Improvement Strategies
1. **For Timing Issues**:
   ```dart
   // Add proper waiting
   await tester.pumpAndSettle(Duration(seconds: 5));
   await expectLater(future, completion(isA<Result>()));
   ```

2. **For Environmental Issues**:
   ```dart
   // Use project test environment setup
   setUpAll(() async {
     // Project scripts handle emulator lifecycle
     await setupProjectTestEnvironment();
   });
   ```

3. **For Mock Issues**:
   ```dart
   // Reset mocks between tests
   tearDown(() {
     reset(mockService);
     clearInteractions(mockService);
   });
   ```

### Monitoring Strategy
- **Rerun stability check** after improvements
- **Set up automated stability monitoring** in CI/CD
- **Track stability trends** over time
- **Alert on stability degradation** below threshold

## 📋 Action Items
1. Address all tests below $STABILITY_THRESHOLD% pass rate
2. Implement recommended fixes for common failure patterns
3. Set up continuous stability monitoring
4. Review and update test isolation practices
5. Consider infrastructure improvements for environmental dependencies

---
*Generated by Claude Code Test Stability Analysis*
```

## Direct Execution Instructions

1. **Parse Arguments**: Extract TEST_PATTERN, ITERATIONS, and STABILITY_THRESHOLD from $ARGUMENTS
2. **Test Discovery**: Use Glob to identify all tests matching the pattern
3. **Environment Setup**: Ensure consistent testing environment
4. **Iterative Execution**: Run tests multiple times with detailed data collection
5. **Statistical Analysis**: Calculate stability metrics and identify patterns
6. **Environmental Testing**: Check dependencies on emulators and network
7. **Report Generation**: Create comprehensive stability report
8. **Actionable Recommendations**: Provide specific improvement suggestions

## GitHub Integration

After analysis completion, **automatically post stability report to PR**:

```markdown
## 📊 Test Stability Analysis Complete

**Pattern Analyzed**: $TEST_PATTERN
**Iterations**: $ITERATIONS
**Threshold**: $STABILITY_THRESHOLD%

### 🎯 Stability Results
- **Overall Pass Rate**: X.X%
- **Stable Tests**: X (> $STABILITY_THRESHOLD%)
- **Flaky Tests**: X (< $STABILITY_THRESHOLD%)
- **Highly Unstable**: X (< 70%)

### ⚠️  Action Required
- [ ] Fix X highly unstable tests immediately
- [ ] Investigate X moderately flaky tests
- [ ] Address environmental dependencies detected

**Detailed Report**: [Link to full stability report]

🤖 Generated with Claude Code - Statistical Test Reliability Analysis
```

## QA Engineer Integration

For fixing identified flaky tests:

```markdown
Task: qa-engineer

"Fix flaky tests identified by stability analysis:

**Flaky Tests**: [List from analysis]
**Failure Patterns**: [Specific issues detected]
**Stability Metrics**: [Pass rates and timing data]
**Environmental Dependencies**: [Emulator/network issues]

Focus on improving test reliability through proper async handling, better isolation,
and environmental dependency management. Target: > $STABILITY_THRESHOLD% pass rate."
```

This command provides comprehensive statistical analysis of test reliability, enabling teams to identify and fix flaky tests before they impact development productivity.