---
description: "Complete TDD cycle - DISABLED FOR SECURITY REVIEW"
argument-hint: "<feature-name> [--auto-refactor]"
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

## 🔄 Complete TDD Cycle - TEMPORARILY DISABLED

⚠️ **COMMAND DISABLED FOR SECURITY REVIEW** ⚠️

This command has been disabled due to security vulnerabilities identified by Claude bot security review.

**Critical Issues Found:**
1. **Command Injection Vector** - Despite basic validation, shell-script-in-markdown architecture has fundamental injection risks
2. **Limited Input Sanitization** - Regex validation insufficient for comprehensive security
3. **Cannot be Unit Tested** - Shell-in-markdown cannot be properly tested or debugged
4. **No Security Sandboxing** - Commands execute with full privileges
5. **No Audit Trails** - No monitoring of command execution

**Input Validation Issues:**
```bash
# Current validation is insufficient:
!if [[ ! "$1" =~ ^[a-zA-Z0-9_\ -]+$ ]]; then
  echo "❌ Invalid feature name: $1";
  exit 1;
fi
# This blocks legitimate input but can still be bypassed with advanced techniques
```

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
- Complete Red-Green-Refactor TDD cycle
- Feature name validation and sanitization
- Test directory creation and organization
- TDD phase validation (Red phase verification)
- Minimal implementation guidance
- Refactoring workflow automation

**All operational code has been disabled for security reasons.**

### Manual TDD Alternative
Until the command is refactored, use manual TDD approach:

1. **Red Phase (Manual)**:
   ```bash
   # Create test directories manually
   mkdir -p test/features/[feature_name]
   mkdir -p test/widgets/[feature_name]

   # Write failing tests manually
   # Run tests manually: flutter test
   # Verify they fail
   ```

2. **Green Phase (Manual)**:
   ```bash
   # Implement minimal code to make tests pass
   # Run tests manually: flutter test
   # Verify they pass
   ```

3. **Refactor Phase (Manual)**:
   ```bash
   # Refactor code while keeping tests green
   # Run tests after each change: flutter test
   # Ensure no regressions
   ```

### Related Commands (Also Under Review)
- `/ci-pipeline` - Also disabled for security review
- `/headless-test` - Also disabled for security review
- `/tdd-start` - Under review for similar issues
- `/tdd-implement` - Under review for similar issues
- `/tdd-refactor` - Under review for similar issues