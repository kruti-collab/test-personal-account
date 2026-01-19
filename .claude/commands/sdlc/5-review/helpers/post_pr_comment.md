---
allowed-tools: Task, TodoWrite, Bash, mcp__github__create_or_update_issue_comment, mcp__github__get_pull_request
argument-hint: <pr_number> <comment_type> [verification_results_path]
description: Posts structured comments to GitHub PRs with MVVM migration verification results, test coverage reports, and workflow status updates
---

# Post PR Comment

This command posts structured, informative comments to GitHub Pull Requests as part of the MVVM migration workflow. It integrates with GitHub's API to provide automated status updates, verification results, and actionable feedback directly in PR conversations.

## Variables

- **PR_NUMBER**: First argument from $ARGUMENTS
  - The GitHub Pull Request number to post the comment to
  - Must be a valid, existing PR number in the repository
  - Used to target the specific PR for automated workflow updates
  
- **COMMENT_TYPE**: Second argument from $ARGUMENTS
  - The type of comment to post: "verification", "testing", "workflow_complete", "error", or "custom"
  - Determines the comment template and content structure
  - Each type has specific formatting and actionable elements
  
- **VERIFICATION_RESULTS**: Third argument from $ARGUMENTS (optional)
  - Path to verification results file or direct results text
  - Used to include detailed MVVM migration analysis in comments
  - Can reference test coverage reports, architecture compliance data

## Usage

```bash
/post_pr_comment <pr_number> <comment_type> [verification_results_path]
```

Examples:
- `/post_pr_comment 123 verification verification_report.md`
- `/post_pr_comment 456 testing test_results.json`
- `/post_pr_comment 789 workflow_complete`
- `/post_pr_comment 321 error "Migration verification failed: missing business logic"`

## Comment Types & Templates

### **Verification Comments** (`verification`)
Posts MVVM migration verification results with:
- ✅ Logic preservation status
- 🏗️ Architecture compliance score
- 📋 Detailed checklist of verified components
- ⚠️ Critical issues requiring attention
- 📄 Links to full verification reports

### **Testing Comments** (`testing`)
Posts comprehensive test coverage results with:
- 🧪 Test suite generation summary
- 📊 Coverage percentages by component type
- 🌐 Web testing (Playwright) results
- 🎯 Test execution status and CI integration
- 📝 Setup instructions for running tests

### **Workflow Complete** (`workflow_complete`)
Posts final workflow status with:
- ✅ Complete migration verification summary
- 🧪 Test coverage confirmation
- 🚀 Deployment readiness checklist
- 📋 Final approval status for merge
- 🔄 Next steps and recommendations

### **Error Comments** (`error`)
Posts structured error information with:
- ❌ Clear error description and context
- 🔧 Suggested remediation steps
- 📖 Links to relevant documentation
- 🆘 Escalation path for complex issues
- 🔄 Retry instructions where applicable

### **Custom Comments** (`custom`)
Posts user-defined content with:
- 📝 Custom message formatting
- 🏷️ Consistent branding and structure
- 🔗 Integration with workflow context
- 📊 Optional data visualization elements

## GitHub Integration Process

This command implements a **Authenticate → Format → Post → Verify** cycle:

```
PR Number → Comment Type → Content Generation → GitHub API → Verification Post
```

## Execution Workflow

### Phase 1: GitHub MCP Authentication & Validation  
- Use GitHub MCP server (mcp__github__*) for authenticated API access
- Use mcp__github__get_pull_request to validate PR exists and is accessible
- Check permissions for commenting on the target PR via MCP server
- Verify repository context and branch information

### Phase 2: Content Generation & Formatting
- Generate appropriate comment content based on comment type
- Format verification results, test data, or custom content
- Apply consistent markdown styling and repository branding
- Include actionable elements like checklists and links

### Phase 3: Comment Posting & Verification
- Use mcp__github__create_or_update_issue_comment to post structured comment to the specified PR
- Verify comment was successfully created via MCP server response
- Handle MCP errors and retry logic if needed
- Log posting activity for audit and debugging

## Direct Execution Instructions

**Note**: This command uses GitHub MCP server for authenticated API access and repository operations.

1. **Parse Arguments**: Extract PR_NUMBER, COMMENT_TYPE, and VERIFICATION_RESULTS from $ARGUMENTS
2. **Validate Arguments**: Ensure PR_NUMBER is numeric and COMMENT_TYPE is valid
3. **GitHub MCP Validation**: Use mcp__github__get_pull_request to validate PR exists and access permissions
4. **Generate Comment Content**: 
   - Use the Task tool to delegate content generation if needed
   - Format verification results, test data, or custom content appropriately
   - Apply consistent markdown styling and branding
5. **Post Comment**: Use mcp__github__create_or_update_issue_comment to post the formatted comment to the PR
6. **Verify Success**: Confirm comment was posted via MCP response and handle any errors

## Comment Content Standards

### **Formatting Requirements**
- Consistent markdown styling with emojis and sections
- Clear, actionable language with specific next steps
- Links to relevant documentation and resources
- Professional tone with repository branding

### **Content Standards**
- Factual, data-driven information from verification results
- Clear success/failure indicators with specific metrics
- Actionable recommendations for addressing issues
- Context-aware messaging based on workflow stage

### **Accessibility & Usability**
- Screen reader compatible formatting
- Clear headings and section organization
- Concise summaries with expandable details
- Mobile-friendly layout and structure

## Integration with MVVM Migration Workflow

### **Phase 1: Verification Reporting**
```bash
# After MVVM migration verification
@claude /verify-mvvm-migration lib/pages/login_page.dart lib/models/login_model.dart
@claude /post_pr_comment 123 verification verification_results.md
```

### **Phase 2: Testing Reporting**
```bash
# After test generation and execution
@claude /generate_tests lib/viewmodels/login_viewmodel.dart all lib/models/login_model.dart
@claude /post_pr_comment 123 testing test_coverage_report.json
```

### **Phase 3: Workflow Completion**
```bash
# After complete workflow validation
@claude /post_pr_comment 123 workflow_complete
```

### **Error Handling**
```bash
# When workflow encounters issues
@claude /post_pr_comment 123 error "Specific error description and remediation steps"
```

## Comment Examples

### **Verification Result Comment**
```markdown
## 🔍 MVVM Migration Verification Results

### ✅ Migration Status: **PASSED**
- **Logic Preservation**: 100% ✅
- **Code Order**: Maintained ✅  
- **Provider Integration**: Functional ✅
- **Architecture Compliance**: 95% ✅

### 📋 Verification Summary
- **Files Analyzed**: `lib/pages/login_page.dart` → MVVM components
- **Business Logic**: All methods migrated successfully
- **State Management**: Provider patterns preserved
- **Test Coverage**: Ready for comprehensive testing

### 🎯 Next Steps
1. ✅ Migration verification complete
2. 🧪 Generate comprehensive test suite
3. 🌐 Run Playwright web tests  
4. 🚀 Validate deployment readiness

*Generated by MVVM Migration Workflow • [View Full Report](link)*
```

### **Testing Complete Comment**
```markdown
## 🧪 Test Coverage Report

### 📊 Coverage Summary
- **Unit Tests**: 95% coverage ✅
- **Widget Tests**: 88% coverage ✅
- **Integration Tests**: 92% coverage ✅
- **Web Tests (Playwright)**: 85% coverage ✅

### 🎯 Test Results
- **Generated Test Files**: 12 files
- **Test Cases**: 156 tests
- **Execution Time**: 2.3 seconds
- **All Tests Passing**: ✅

### 🚀 Deployment Ready
Migration verification and testing complete. Ready for merge!

*Generated by MVVM Migration Workflow • [View Test Reports](link)*
```

## Success Criteria

PR comment posting is **successful** when:
1. **GitHub CLI Authentication** - GitHub CLI access confirmed with proper credentials (REQUIRED)
2. **Successful Posting** - Comment appears in target PR via CLI response (REQUIRED)
3. **Proper Formatting** - Markdown renders correctly with all elements (REQUIRED)
4. **Actionable Content** - Clear next steps and status information (REQUIRED)
5. **Workflow Integration** - Comment aligns with current workflow phase (REQUIRED)

## GitHub MCP Integration

This command leverages the GitHub MCP server (https://github.com/github/github-mcp-server) to provide:
- **Authenticated Access**: Uses GitHub MCP server with proper authentication
- **Direct API Access**: Direct GitHub API calls via MCP server
- **Real-time Updates**: Immediate PR comment posting with MCP response verification
- **Error Handling**: Comprehensive error handling and retry logic via MCP responses

**MCP Server Configuration**: GitHub MCP server configured in `.mcp.json`:
- **Server**: github-mcp-server with GitHub API access
- **Authentication**: Uses GitHub token from environment variables
- **Permissions**: Configured for repository access and PR comment operations

**Usage Example**:
```markdown
Use mcp__github__create_or_update_issue_comment with:
{
  "issue_number": 123,
  "body": "## 🔍 MVVM Migration Verification Results\n### ✅ Migration Status: **PASSED**\n- Logic Preservation: 100% ✅\n- Provider Integration: Functional ✅"
}
```

This command ensures that MVVM migration workflows provide transparent, real-time communication through GitHub PR comments, enabling efficient collaboration and clear status tracking throughout the migration process using the reliable GitHub MCP server integration.