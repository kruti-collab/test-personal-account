---
description: "TDD refactor phase - DISABLED FOR SECURITY REVIEW"
argument-hint: "<feature-name> [focus-area]"
allowed-tools: ["*"]
---

<!-- TODO: CRITICAL SECURITY REFACTOR REQUIRED
This entire command needs to be rewritten as a proper CLI tool due to:
1. Command injection vulnerabilities in shell-script-in-markdown architecture
2. No proper input sanitization beyond basic regex
3. Cannot be unit tested or debugged effectively
4. Security audit trails missing
5. Shell-script complexity makes maintenance difficult

RECOMMENDED: Rewrite as Python/JavaScript CLI with proper:
- Input validation and sanitization
- Unit testing capabilities
- Professional error handling
- Security audit trails
- Modular TDD workflow management

DISABLED UNTIL SECURITY REVIEW COMPLETE
-->

## 🧪 TDD Refactor Phase - TEMPORARILY DISABLED

⚠️ **COMMAND DISABLED FOR SECURITY REVIEW** ⚠️

This command has been disabled due to security vulnerabilities identified by Claude bot security review.

**Critical Issues Found:**
1. **Command Injection Vector** - Shell-script-in-markdown architecture has fundamental injection risks
2. **Limited Input Sanitization** - Regex validation insufficient for comprehensive security
3. **Cannot be Unit Tested** - Shell-in-markdown cannot be properly tested or debugged
4. **No Security Sandboxing** - Commands execute with full privileges
5. **No Audit Trails** - No monitoring of command execution

**Architecture Problems:**
- Shell-script-in-markdown creates fundamental security weaknesses
- Complex TDD logic embedded in shell scripts is unmaintainable
- Cannot be adequately tested or debugged
- No proper error handling or recovery mechanisms

**Recommended Solution:**
Rewrite as proper Python/JavaScript CLI tool with:
- Professional input validation and sanitization
- Modular TDD workflow components that can be unit tested
- Proper error handling and rollback mechanisms
- Security audit trails and monitoring
- Integration with proper testing frameworks

**Status:** Under security review - not operational until refactored

### Previous Functionality (Disabled)
This command previously provided:
- TDD Refactor phase (Blue phase) guidance
- Code quality improvement suggestions
- Test maintenance and cleanup
- Performance optimization recommendations
- Architecture improvement suggestions

**All operational code has been disabled for security reasons.**

### Manual TDD Refactor Alternative
Until the command is refactored, use manual refactoring:

1. **Ensure Tests Are Green**:
   ```bash
   flutter test test/features/[feature_name]/
   # Verify all tests pass before refactoring
   ```

2. **Identify Refactoring Opportunities**:
   ```bash
   # Manual code review for:
   # - Code duplication
   # - Long methods/classes
   # - Poor naming
   # - Missing abstractions
   ```

3. **Refactor Incrementally**:
   ```bash
   # Make small changes and test frequently
   # Keep tests green throughout the process
   ```

4. **Run Tests After Each Change**:
   ```bash
   flutter test test/features/[feature_name]/
   # Ensure no regressions with each refactoring step
   ```

5. **Final Validation**:
   ```bash
   flutter test
   # Run full test suite to ensure no system-wide regressions
   ```

### Related Commands (Also Under Review)
- `/tdd-start` - Also disabled for security review
- `/tdd-implement` - Also disabled for security review
- `/tdd-cycle` - Already disabled for security review