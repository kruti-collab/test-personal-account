---
description: "Automated testing pipeline - DISABLED FOR SECURITY REVIEW"
argument-hint: "[test-type] [--coverage]"
allowed-tools: ["Bash"]
---

<!-- TODO: CRITICAL SECURITY REFACTOR REQUIRED
This entire command needs to be rewritten as a proper CLI tool due to:
1. Test result manipulation in subshells - exit codes not properly captured
2. Command injection vulnerabilities in shell-script-in-markdown architecture
3. No proper input sanitization or validation
4. Cannot be unit tested or debugged effectively
5. Security audit trails missing

RECOMMENDED: Rewrite as Python/JavaScript CLI with proper:
- Proper exit code capture and handling
- Input validation and sanitization
- Unit testing capabilities
- Professional error handling
- Security audit trails

DISABLED UNTIL SECURITY REVIEW COMPLETE
-->

## 🧪 Automated Testing Pipeline - TEMPORARILY DISABLED

⚠️ **COMMAND DISABLED FOR SECURITY REVIEW** ⚠️

This command has been disabled due to critical security vulnerabilities identified by Claude bot security review.

**Critical Issues Found:**
1. **Test Result Manipulation** - Exit codes not properly captured in subshells, always reports success
2. **Command Injection Vector** - Validation can be easily bypassed
3. **No Security Sandboxing** - Commands execute with full privileges
4. **Cannot be Unit Tested** - Shell-in-markdown cannot be properly tested
5. **No Audit Trails** - No monitoring of command execution

**Specific Technical Issue:**
```bash
# This pattern fails to capture exit codes:
TEST_EXIT_CODE=0
$( [[ "$2" == "--coverage" ]] && echo "flutter test --coverage; TEST_EXIT_CODE=\$?" || echo "flutter test; TEST_EXIT_CODE=\$?" )
# TEST_EXIT_CODE remains 0 regardless of test results
```

**Architecture Problems:**
- Shell-script-in-markdown creates fundamental security weaknesses
- Cannot be adequately addressed with current design
- No input sanitization or proper validation
- Cannot be unit tested or debugged effectively

**Recommended Solution:**
Rewrite as proper Python/JavaScript CLI tool with:
- Direct command execution instead of subshells
- Professional input validation and sanitization
- Comprehensive unit testing capabilities
- Proper error handling and logging
- Security audit trails and monitoring

**Status:** Under security review - not operational until refactored

### Previous Functionality (Disabled)
This command previously provided:
- Unit test execution with coverage
- Widget test execution
- Integration test execution
- Test result analysis and reporting
- Performance metrics collection
- Test artifact management

**All operational code has been disabled for security reasons.**

### Manual Testing Alternative
Until the command is refactored, use manual testing:
```bash
# Manual test commands (run directly)
flutter test                    # Unit tests
flutter test --coverage        # Unit tests with coverage
flutter test test/widget/       # Widget tests
flutter test integration_test/  # Integration tests
```

### Related Commands (Also Under Review)
- `/ci-pipeline` - Also disabled for security review
- `/headless-build` - Also disabled for security review
- `/tdd-cycle` - Partially disabled - TDD logic under review