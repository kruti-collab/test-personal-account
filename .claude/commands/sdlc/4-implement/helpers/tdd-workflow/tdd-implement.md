---
description: "TDD implementation phase - DISABLED FOR SECURITY REVIEW"
argument-hint: "<feature-name> [--minimal]"
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

## 🧪 TDD Implementation Phase - TEMPORARILY DISABLED

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
- TDD Green phase implementation guidance
- Minimal code generation to make tests pass
- Feature implementation templates
- Code quality validation
- Integration with test runner

**All operational code has been disabled for security reasons.**

### Manual TDD Implementation Alternative
Until the command is refactored, use manual implementation:

1. **Run Tests to See Failures**:
   ```bash
   flutter test test/features/[feature_name]/
   # Review test failures to understand requirements
   ```

2. **Implement Minimal Code**:
   ```bash
   # Edit implementation files manually
   # lib/features/[feature_name]/[feature_name].dart
   # Add minimal code to make tests pass
   ```

3. **Verify Tests Pass**:
   ```bash
   flutter test test/features/[feature_name]/
   # Confirm all tests now pass (Green phase achieved)
   ```

4. **Run Full Test Suite**:
   ```bash
   flutter test
   # Ensure no regressions in existing functionality
   ```

### Related Commands (Also Under Review)
- `/tdd-start` - Also disabled for security review
- `/tdd-refactor` - Also disabled for security review
- `/tdd-cycle` - Already disabled for security review