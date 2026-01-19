---
description: Validate all coding principles across the repository
tags: [principles, validation, comprehensive]
---

# Comprehensive Principles Validator

You are a comprehensive coding principles validator. Run ALL principle validations and provide a consolidated report.

## Available Principle Validators

1. **Script Hierarchy** - `/validate-script-hierarchy`
   - Validates script organization and delegation
   - Ensures no logic duplication
   - Reference: `docs/coding-principles/script-hierarchy.md`

2. **Git Workflow** - (Coming Soon)
   - Validates commit messages
   - Checks branch naming
   - Verifies SMR ticket references

3. **Code Quality** - (Coming Soon)
   - Runs Flutter analyze
   - Checks for deprecated APIs
   - Validates code style

4. **Security Standards** - (Coming Soon)
   - Scans for hardcoded secrets
   - Validates credential handling
   - Checks for security vulnerabilities

## Validation Process

### 1. Determine Scope
- If user provides file path: Validate that file against all relevant principles
- If user provides directory: Validate all applicable files in directory
- If no path provided: Validate recent changes (git diff)
- If `--full` flag: Validate entire repository

### 2. Run All Applicable Validators
For each changed/specified file:
- Detect file type (script, Dart, config, etc.)
- Run relevant principle validators
- Collect all violations

### 3. Generate Consolidated Report

```markdown
# 🔍 Comprehensive Principles Validation Report

**Scope:** [files/directory analyzed]
**Date:** [timestamp]
**Status:** [✅ PASS / 🟡 WARNING / 🔴 FAIL]

---

## 📊 Summary

| Principle | Status | Blocking | Warnings |
|-----------|--------|----------|----------|
| Script Hierarchy | [status] | [count] | [count] |
| Git Workflow | [status] | [count] | [count] |
| Code Quality | [status] | [count] | [count] |
| Security | [status] | [count] | [count] |

**Total Blocking Issues:** [count]
**Total Warnings:** [count]
**Action Required:** [Yes/No]

---

## 🔴 BLOCKING ISSUES (Must Fix Before Merge)

### Script Hierarchy Violations
[List blocking issues from script hierarchy validator]

### Code Quality Violations
[List blocking issues from code quality validator]

### Security Violations
[List blocking issues from security validator]

---

## 🟡 WARNINGS (Should Address)

[List all warnings across all validators]

---

## ✅ PASSED VALIDATIONS

[List all principles that passed successfully]

---

## 🔧 RECOMMENDED ACTIONS

### Immediate (Blocking):
1. [Action 1 with specific file/line references]
2. [Action 2 with specific file/line references]

### Soon (Warnings):
1. [Action 1]
2. [Action 2]

### Future Improvements:
1. [Suggestion 1]
2. [Suggestion 2]

---

## 📝 Next Steps

```bash
# Fix blocking issues first:
[specific commands to run]

# Then address warnings:
[specific commands to run]

# Validate again:
/validate-all-principles
```
```

## Usage Examples

### Validate Recent Changes
```
/validate-all-principles
```
(Analyzes `git diff` against all principles)

### Validate Specific File
```
/validate-all-principles scripts/ci/new_script.sh
```

### Validate Directory
```
/validate-all-principles scripts/
```

### Full Repository Scan
```
/validate-all-principles --full
```

### Pre-Commit Validation
```
/validate-all-principles --staged
```
(Validates only git staged files)

## Integration Points

### Pre-Commit Hook
```bash
# .git/hooks/pre-commit
claude-code /validate-all-principles --staged --blocking-only
```

### CI Pipeline
```yaml
# .github/workflows/principles-validation.yml
- name: Validate Coding Principles
  run: claude-code /validate-all-principles --full
```

### Code Review
Before approving PR:
```
/validate-all-principles --pr [PR-number]
```

## Special Modes

### `--blocking-only`
Only report blocking issues (exit 1 if found)

### `--fix-auto`
Automatically fix issues where possible (e.g., formatting, imports)

### `--explain`
Provide detailed explanations for each violation

### `--since [commit-hash]`
Validate changes since specific commit

## Current Implementation Status

✅ **Active Validators:**
- Script Hierarchy

🚧 **Coming Soon:**
- Git Workflow Validator
- Code Quality Validator
- Security Validator
- Firebase Config Validator

---

**Note:** This command will evolve as more principle validators are added. Each new principle automatically integrates into this comprehensive validation.
