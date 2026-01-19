---
allowed-tools: Task, TodoWrite
argument-hint: <original_file_path> <model_file_path> [cleanup_directory]
description: READ-ONLY verification of MVVM migration. Agents review migration completeness, add TODO comments for issues, but do NOT fix code. Developer must implement fixes based on agent findings.
---

# Verify MVVM Migration

This command provides comprehensive READ-ONLY verification for Flutter page migrations from Provider pattern to MVVM architecture. Agents will analyze migration completeness and inject TODO comments for any issues found, but will NOT fix code automatically.

**IMPORTANT**: This is a verification and documentation tool. Agents will:
- ✅ Analyze migration completeness 
- ✅ Add TODO comments for missing logic
- ✅ Post detailed verification reports to PR
- ❌ NOT fix actual code issues
- ❌ NOT add missing business logic
- ❌ NOT make functional changes

**Developer responsibility**: Implement fixes based on agent TODO comments and recommendations.

## Variables

- **ORIGINAL_FILE**: First argument from $ARGUMENTS
  - The path to the original file that was migrated to MVVM
  - Used to locate the MVVM counterpart and backup file
  
- **MODEL_FILE**: Second argument from $ARGUMENTS
  - The path to the model file associated with the MVVM implementation
  - Used to verify data structures and business logic alignment
  
- **CLEANUP_DIR**: Third argument from $ARGUMENTS (optional, defaults to "cleanup/")
  - Directory where the original file backup is stored
  - Used to verify backup integrity and compare implementations

## Usage

```bash
/verify-mvvm-migration <original_file_path> <model_file_path> [cleanup_directory]
```

Examples:
- `/verify-mvvm-migration lib/pages/login_page.dart lib/models/login_model.dart` (assumes cleanup in cleanup/)
- `/verify-mvvm-migration lib/pages/dashboard_page.dart lib/models/dashboard_model.dart cleanup/old_pages/` (custom cleanup dir)
- `/verify-mvvm-migration lib/widgets/user_profile.dart lib/models/user_model.dart backup/widgets/`

## Migration Verification Process

This command implements a **Compare → Analyze → Validate → Report** cycle:

```
Original File → Find MVVM Version → Model File Analysis → Line-by-Line Compare → Logic Validation → Backup Check → Report
```

## Execution Workflow

### Phase 1: File Discovery & Mapping
- Locate original file in cleanup/backup directory
- Identify corresponding MVVM implementation
- Analyze model file structure and data definitions
- Map file structure and component relationships
- Validate Provider to MVVM architectural patterns

### Phase 2: Logic Preservation Analysis (PRIMARY FOCUS)
- **Business Logic**: Every method, function, calculation preserved - CRITICAL
- **Code Order Preservation**: Every line of code should maintain the same order as in the original file after migration - CRITICAL
- **State Management**: Provider integration remains functional - CRITICAL  
- **UI Logic**: Widget behavior, user interactions identical - CRITICAL
- **Data Flow**: API calls, data transformations unchanged - CRITICAL
- **Error Handling**: Exception handling and validation preserved - CRITICAL

### Phase 3: Architecture Compliance Check (SECONDARY)
- **MVVM Pattern**: ViewModel separation from View (after logic verified)
- **Provider Integration**: Existing Provider state management intact
- **Dependency Injection**: Proper service layer implementation  
- **Separation of Concerns**: Clear Model-View-ViewModel boundaries

### Phase 4: Comprehensive Validation
- Line-by-line code comparison using AST analysis
- Function signature matching and behavior verification
- State variable mapping and usage patterns
- Import and dependency consistency check

## Verification Checklist

### ✅ **Logic Preservation (PRIMARY - MUST PASS)**
- [ ] All business logic methods migrated completely
- [ ] Code order maintained exactly as in original file - CRITICAL
- [ ] State management functionality identical
- [ ] User interaction handlers preserved
- [ ] Data validation and transformation intact
- [ ] API integration and error handling unchanged
- [ ] ALL user-facing functionality produces identical results
- [ ] NO missing features or behaviors from original

### ✅ **MVVM Architecture (SECONDARY - AFTER LOGIC VERIFIED)**
- [ ] ViewModel properly separated from UI
- [ ] Model classes correctly structured
- [ ] View contains only presentation logic
- [ ] Provider integration maintained
- [ ] Dependency injection implemented correctly

### ✅ **Code Quality**
- [ ] No code duplication between old and new
- [ ] Proper naming conventions followed
- [ ] Import statements optimized
- [ ] Memory management considerations addressed
- [ ] Performance implications evaluated

### ✅ **Backup & Cleanup**
- [ ] Original file properly backed up
- [ ] Cleanup directory structure maintained
- [ ] File permissions and metadata preserved
- [ ] Git history references available
- [ ] Rollback capability confirmed

## Direct Execution Instructions

**Note**: This command delegates verification analysis to the specialized mvvm-migration-architect agent.

1. **Parse Arguments**: Extract ORIGINAL_FILE, MODEL_FILE, and CLEANUP_DIR from $ARGUMENTS
2. **Validate Arguments**: Ensure ORIGINAL_FILE and MODEL_FILE are provided and exist
3. **Set Defaults**: Use "cleanup/" as default if CLEANUP_DIR not specified
4. **Launch Parallel Agent Workflows**: 
   - Use multiple Task tool calls in a SINGLE message to launch agents in parallel
   - Each agent MUST provide separate, independent reports
   - Do NOT combine or summarize agent results - let each agent report directly
     
   **Agent 1: mvvm-migration-architect (SEQUENTIAL COORDINATOR)**
   - Execute 3 specialized analyses in SEQUENCE (not parallel):
     1. **Logic Preservation** → commit → push → PR comment #1
     2. **UI Preservation** → commit → push → PR comment #2  
     3. **Architecture Compliance** → commit → push → PR comment #3
   - Each analysis includes TODO comments for issues found (NO CODE FIXES)
   - **Result**: 3 separate PR comments with detailed findings from each analysis
   
   **Agent 2: qa-engineer (TEST GENERATOR & VALIDATOR)**
   - Generate comprehensive test suite for the NEW MVVM components ONLY
   - Focus on migrated View, ViewModel, and Model files (not original Provider files)
   - Create unit tests, widget tests, and integration tests for the MVVM architecture
   - **MANDATORY**: Execute all generated tests to ensure they work
   - **MANDATORY SEQUENCE**: After test generation and execution complete:
     1. Commit test files using `git add -A && git commit -m "#XXX: Generate comprehensive MVVM test suite for [feature]"`
     2. Push changes using `git push` 
     3. ONLY THEN post test results to PR using GitHub MCP tools
   - Report ACTUAL test counts and execution results (not estimated counts)
   - List specific test files created with exact paths and commit reference
   - **CRITICAL**: Reset test metrics - report only tests created for THIS specific migration, not accumulated totals

## Verification Report Structure

### **Migration Summary**
- **File**: Original file path and MVVM counterpart
- **Completeness**: Percentage of logic successfully migrated
- **Architecture**: MVVM compliance score
- **Provider**: State management integration status

### **Detailed Analysis**
- **Preserved Components**: List of successfully migrated elements
- **Missing Elements**: Any logic not found in MVVM version
- **Architectural Issues**: MVVM pattern violations or concerns
- **Provider Compatibility**: Integration assessment

### **Action Items**
- **Critical Issues**: Must-fix items before merge
- **Recommendations**: Architecture improvements
- **Validation Results**: Pass/fail status for each check
- **Next Steps**: Required actions for completion

## Safety Mechanisms

### **Pre-Merge Protection**
- Fails verification if ANY critical logic is missing
- Blocks merge if Provider integration is broken
- Requires 100% business logic preservation
- Validates backup integrity before proceeding

### **Rollback Capability**
- Original file backup verification
- Restoration procedure validation
- Git history preservation check
- Emergency rollback instructions

## Integration with PR Workflow

### **GitHub Actions Integration**
This command has a dedicated workflow (`claude-mvvm-verification.yml`) that triggers specifically on `@claude /verify-mvvm-migration` usage:

```bash
# In PR comments - triggers dedicated MVVM verification workflow
@claude /verify-mvvm-migration lib/production_pages/onboarding/onboarding_widget.dart lib/models/onboarding_model.dart
```

**Workflow Features:**
- Runs independently from general Claude workflow
- Optimized for MVVM architecture analysis
- Focused permissions and environment for verification tasks

### **Complete MVVM Migration Workflow**
```bash
# Step 1: Verify MVVM migration integrity
@claude /verify-mvvm-migration lib/pages/login_page.dart lib/models/login_model.dart

# Step 2: Post verification results to PR
@claude /post_pr_comment_agent_prompt 123 verification verification_results.md

# Step 3: Generate comprehensive test coverage (after verification passes)
@claude /generate_tests_agent_prompt lib/viewmodels/login_viewmodel.dart all lib/models/login_model.dart
@claude /generate_tests_agent_prompt lib/views/login_view.dart widget lib/models/login_model.dart

# Step 4: Post testing results to PR
@claude /post_pr_comment_agent_prompt 123 testing test_coverage_report.json

# Step 5: Complete workflow and post final status
@claude /post_pr_comment_agent_prompt 123 workflow_complete

# Step 6: Run tests to validate migration quality
# (Manual step - run generated tests in your development environment)
```

### **Manual PR Verification**
```bash
# In PR comments - analyze migration from original file
@claude /verify-mvvm-migration lib/pages/modified_page.dart lib/models/page_model.dart
@claude /post_pr_comment_agent_prompt 456 verification

# Generate tests for verified components
@claude /generate_tests_agent_prompt lib/viewmodels/modified_viewmodel.dart all lib/models/page_model.dart
@claude /post_pr_comment_agent_prompt 456 testing

# With custom cleanup directory reference
@claude /verify-mvvm-migration lib/widgets/user_widget.dart lib/models/user_model.dart backup/pre-mvvm/
@claude /post_pr_comment_agent_prompt 456 verification
@claude /generate_tests_agent_prompt lib/widgets/user_widget.dart widget lib/models/user_model.dart
@claude /post_pr_comment_agent_prompt 456 workflow_complete
```

**Note**: The verification command provides read-only analysis and recommendations. The test generation command creates test files. The PR comment command posts workflow updates to GitHub. Any file moves or cleanup actions must be performed separately after verification.

## Success Criteria

A migration is **verified and ready for developer review** when:
1. **Comprehensive Analysis Complete** - Agents have analyzed all files and logic (VERIFICATION)
2. **TODO Comments Injected** - All missing logic documented with specific TODO comments (DOCUMENTATION)
3. **Test Suite Generated** - Complete test coverage for new MVVM components (TEST CREATION)
4. **PR Reports Posted** - Detailed verification and test results posted to GitHub PR (COMMUNICATION)
5. **Migration Score Provided** - Percentage completion with breakdown by category (ASSESSMENT)

**MERGE-READY STATUS** depends on agent findings:
- **✅ APPROVED**: 100% logic preservation, no blocking issues found
- **⚠️ CONDITIONAL**: Minor issues with TODO comments - developer review required  
- **❌ BLOCKED**: Critical missing logic - developer fixes required before merge

**Developer Action Required**: Review agent TODO comments and implement any necessary fixes before merge.

## Complete Migration Workflow

This verification command works in conjunction with `/generate_tests_agent_prompt` and `/post_pr_comment_agent_prompt` to ensure that MVVM migrations maintain complete functionality while properly implementing the new architectural pattern. The three-phase approach (verify → test → communicate) provides comprehensive confidence for safe PR merges:

1. **Verification Phase**: `/verify-mvvm-migration` ensures logic preservation and architecture compliance
2. **Testing Phase**: `/generate_tests_agent_prompt` creates comprehensive test coverage for ongoing maintainability
3. **Communication Phase**: `/post_pr_comment_agent_prompt` provides real-time workflow status and results to GitHub PR
4. **Validation Phase**: Manual test execution confirms migration quality and prevents regressions