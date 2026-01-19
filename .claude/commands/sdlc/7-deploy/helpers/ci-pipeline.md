---
description: "Complete CI/CD pipeline automation - DISABLED FOR SECURITY REVIEW"
argument-hint: "[environment] [--deploy]"
allowed-tools: ["Bash", "Task"]
---

<!-- TODO: CRITICAL SECURITY REFACTOR REQUIRED
This entire command needs to be rewritten as a proper CLI tool due to:
1. Command injection vulnerabilities in shell-script-in-markdown architecture
2. Misleading secret scanning that shows success even when secrets found
3. Hardcoded Firebase project IDs need environment variables
4. No proper input sanitization or validation
5. Cannot be unit tested or debugged effectively
6. Test result manipulation in subshells
7. Incomplete branch validation allowing directory traversal

RECOMMENDED: Rewrite as Python/JavaScript CLI with proper:
- Input validation and sanitization
- Unit testing capabilities
- Professional error handling
- Security audit trails
- Proper secret detection
- Secure command execution

DISABLED UNTIL SECURITY REVIEW COMPLETE
-->

## 🚀 Complete CI/CD Pipeline - TEMPORARILY DISABLED

⚠️ **COMMAND DISABLED FOR SECURITY REVIEW** ⚠️

This command has been disabled due to critical security vulnerabilities identified by Claude bot security review.

**Critical Issues Found:**
1. **Command Injection Vector** - Validation can be easily bypassed
2. **Test Result Manipulation** - Exit codes not properly captured in subshells
3. **Incomplete Branch Validation** - Regex allows directory traversal
4. **Misleading Secret Scanning** - Shows success even when secrets found
5. **Hardcoded Environment References** - Firebase project confusion risk
6. **No Security Sandboxing** - Commands execute with full privileges

**Architecture Problems:**
- Shell-script-in-markdown creates fundamental security weaknesses
- Cannot be adequately addressed with current design
- No input sanitization or proper validation
- Cannot be unit tested or debugged effectively
- Security by obscurity approach

**Recommended Solution:**
Rewrite as proper Python/JavaScript CLI tool with:
- Professional input validation and sanitization
- Comprehensive unit testing capabilities
- Proper error handling and logging
- Security audit trails and monitoring
- Sandboxed execution environment

**Status:** Under security review - not operational until refactored

### Previous Functionality (Disabled)
This command previously provided:
- Environment validation and Flutter doctor checks
- Code quality analysis and static analysis
- Comprehensive testing with coverage
- Security validation and secret scanning
- Multi-environment build process
- Automated deployment capabilities
- Post-deploy validation

**All operational code has been disabled for security reasons.**

### Related Commands (Also Under Review)
- `/headless-build` - Also disabled for security review
- `/headless-test` - Also disabled for security review
- Manual processes: Use Firebase Console and manual scripts until refactor complete