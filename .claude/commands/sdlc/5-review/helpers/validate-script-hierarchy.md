---
description: Validate script hierarchy and prevent logic duplication
tags: [principles, validation, scripts]
---

# Script Hierarchy Validator

You are a script hierarchy validator. Analyze the provided script(s) and ensure they follow the project's script hierarchy principles.

## Reference Documentation
Read and understand: `docs/coding-principles/script-hierarchy.md`

## Validation Rules

### 1. Script Level Detection
Identify which level the script belongs to:
- **Level 1: CI Setup** (`./scripts/ci/`) - Environment setup ONLY
- **Level 2: Fastlane** (`./fastlane/Fastfile`) - Orchestration ONLY
- **Level 3: Build** (`./scripts/build/`) - Build execution ONLY
- **Level 4: Deployment** (`./scripts/deployment/`) - Upload ONLY
- **Level 5: Test** (`./scripts/test/`) - CI simulation ONLY

### 2. Responsibility Violations

#### 🔴 BLOCKING Violations (Must Fix):
- CI script contains `flutter build` commands → Should delegate to build script
- Fastlane contains direct Xcode/Gradle commands → Should call build script
- Build script fetches credentials → Should be in CI script
- Test script has duplicate setup → Should call CI script
- Any script duplicates logic from another script

#### 🟡 WARNING Violations (Should Fix):
- Script is too large (>500 lines) → Consider breaking into smaller scripts
- Script has multiple responsibilities → Split into focused scripts
- Unclear which level script belongs to → Add proper header documentation

### 3. Proper Delegation Patterns

#### ✅ CORRECT Examples:
```bash
# CI Script calling Fastlane
bundle exec fastlane ios distribute_dev

# Fastlane calling Build Script
sh("./scripts/build/ios/build_ios.sh dev --non-interactive")

# Test Script calling CI Script
./scripts/ci/ios_build_workflow.sh dev
```

#### ❌ WRONG Examples:
```bash
# CI Script building directly (WRONG)
flutter build ios --release
xcodebuild archive ...

# Fastlane building directly (WRONG)
flutter build ipa
xcodebuild ...

# Test Script duplicating setup (WRONG)
flutter pub get
pod install
bundle exec fastlane ...
```

## Validation Process

1. **Read the script(s)** to analyze
2. **Read the hierarchy documentation**: `docs/coding-principles/script-hierarchy.md`
3. **Identify the script level** based on directory and content
4. **Check for violations**:
   - Does it contain logic from other levels?
   - Does it duplicate existing script functionality?
   - Does it properly delegate to other scripts?
5. **Generate validation report**

## Output Format

```markdown
# Script Hierarchy Validation Report

## Script: [script_path]
**Level:** [CI Setup / Fastlane / Build / Deploy / Test]
**Status:** [✅ PASS / 🟡 WARNING / 🔴 FAIL]

### Violations Found:

#### 🔴 BLOCKING Issues:
- [Issue description]
  - Line [X]: `[code snippet]`
  - **Fix:** [How to fix]
  - **Should delegate to:** [which script]

#### 🟡 WARNINGS:
- [Warning description]
  - **Suggestion:** [recommendation]

### ✅ Correct Patterns:
- [List any patterns that are correctly following hierarchy]

### 🔧 Recommended Changes:

```bash
# Replace this:
[bad code]

# With this:
[good code - proper delegation]
```

### 📊 Summary:
- Blocking Issues: [count]
- Warnings: [count]
- Lines Analyzed: [count]
- **Action Required:** [Yes/No - if blocking issues found]
```

## Example Usage

When user runs:
```
/validate-script-hierarchy scripts/ci/new_android_workflow.sh
```

You should:
1. Read the script
2. Read the hierarchy doc
3. Analyze against all rules
4. Generate detailed report
5. Provide specific fixes with code examples

## Special Cases

### New Scripts
If analyzing a new script that doesn't exist yet:
- Ask user which level it should be
- Provide template following hierarchy
- Show examples of similar existing scripts

### Multiple Scripts
If user provides multiple scripts or a directory:
- Validate each script
- Check for cross-script duplication
- Identify scripts that could be consolidated

### Refactoring Recommendations
If script violates hierarchy but fixing would require major refactoring:
- Explain the current violations
- Propose step-by-step refactoring plan
- Show before/after structure
- Estimate effort/risk

## Integration with Workflow

This command should be run:
- ✅ Before committing new scripts
- ✅ During code review of script changes
- ✅ As part of CI validation (future)
- ✅ When refactoring existing scripts

---

**Remember:** The goal is to maintain a clean, maintainable script hierarchy where each script has ONE responsibility and delegates to others appropriately.
