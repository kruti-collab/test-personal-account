---
description: "Start TDD workflow - DISABLED FOR SECURITY REVIEW"
argument-hint: "<feature-name> [test-type]"
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

## 🧪 TDD Workflow Initiation - TEMPORARILY DISABLED

⚠️ **COMMAND DISABLED FOR SECURITY REVIEW** ⚠️

This command has been disabled due to security vulnerabilities identified by Claude bot security review.

**Critical Issues Found:**
1. **Command Injection Vector** - Shell-script-in-markdown architecture has fundamental injection risks
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
- TDD workflow initialization with test-first approach
- Feature name validation and sanitization
- Test directory structure creation
- Test type selection (unit/widget/integration)
- Template test file generation
- Integration with TDD cycle commands

**All operational code has been disabled for security reasons.**

### Manual TDD Alternative
Until the command is refactored, use manual TDD approach:

1. **Create Test Structure Manually**:
   ```bash
   # Create test directories manually
   mkdir -p test/features/[feature_name]
   mkdir -p test/widgets/[feature_name]
   mkdir -p test/unit/[feature_name]
   ```

2. **Write Tests First**:
   ```bash
   # Create test files manually
   touch test/features/[feature_name]/[feature_name]_test.dart
   # Write failing tests in the test file
   ```

3. **Run Tests to Verify Failure**:
   ```bash
   flutter test test/features/[feature_name]/
   # Verify tests fail as expected (Red phase)
   ```

4. **Implement Feature**:
   ```bash
   # Create implementation files
   touch lib/features/[feature_name]/[feature_name].dart
   # Implement minimal code to make tests pass
   ```

5. **Verify Tests Pass**:
   ```bash
   flutter test test/features/[feature_name]/
   # Verify tests now pass (Green phase)
   ```

### Related Commands (Also Under Review)
- `/tdd-implement` - Under review for similar security issues
- `/tdd-refactor` - Under review for similar security issues
- `/tdd-cycle` - Already disabled for security review
- `/headless-test` - Already disabled for security review