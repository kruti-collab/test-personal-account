---
description: "Headless build pipeline - DISABLED FOR SECURITY REVIEW"
argument-hint: "[target] [--with-tests]"
allowed-tools: ["Bash", "Task"]
---

<!-- TODO: CRITICAL SECURITY REFACTOR REQUIRED
This entire command needs to be rewritten as a proper CLI tool due to:
1. Command injection vulnerabilities in shell-script-in-markdown architecture
2. No proper input sanitization beyond basic regex
3. Cannot be unit tested or debugged effectively
4. Security audit trails missing
5. Build system access without proper sandboxing

RECOMMENDED: Rewrite as Python/JavaScript CLI with proper:
- Input validation and sanitization
- Unit testing capabilities
- Professional error handling
- Security audit trails
- Sandboxed build environment

DISABLED UNTIL SECURITY REVIEW COMPLETE
-->

## 🏗️ Headless Build Pipeline - TEMPORARILY DISABLED

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
- Build system access without proper security controls
- Cannot be adequately tested or debugged
- No proper error handling or recovery mechanisms

**Recommended Solution:**
Rewrite as proper Python/JavaScript CLI tool with:
- Professional input validation and sanitization
- Sandboxed build environment
- Comprehensive unit testing capabilities
- Proper error handling and logging
- Security audit trails and monitoring

**Status:** Under security review - not operational until refactored

### Previous Functionality (Disabled)
This command previously provided:
- Automated Flutter build process
- Multi-target build support (web, iOS, Android)
- Build verification and testing
- Artifact management
- Performance metrics collection

**All operational code has been disabled for security reasons.**

### Manual Build Alternative
Until the command is refactored, use manual Flutter commands:

```bash
# Manual build commands (run directly)
flutter clean                          # Clean previous builds
flutter pub get                        # Get dependencies
flutter build web --release            # Build for web
flutter build apk --release            # Build for Android
flutter build ios --release            # Build for iOS
```

### Related Commands (Also Under Review)
- `/ci-pipeline` - Already disabled for security review
- `/headless-test` - Already disabled for security review