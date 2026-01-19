---
allowed-tools: Task, Read, Glob, Grep, Bash, mcp__github__create_or_update_issue_comment, gh issue list --search
argument-hint: <deployment_target> [environment] [readiness_criteria]
description: Validates production deployment readiness with definitive YES/NO assessment
---

# End-to-End Readiness Check

This command provides comprehensive deployment readiness assessment for production environments like Netlify. It validates OAuth configurations, API integrations, performance benchmarks, and security compliance to deliver a definitive **YES/NO** deployment decision.

## Variables

- **DEPLOYMENT_TARGET**: First argument from $ARGUMENTS
  - Target deployment environment: "netlify", "production", "staging", "preview"
  - Determines specific validation requirements and readiness criteria
  - Used to configure environment-specific testing protocols

- **ENVIRONMENT**: Second argument from $ARGUMENTS (optional, default: "production")
  - Specific environment configuration to validate: "production", "staging", "dev"
  - Controls which configuration files and settings are validated
  - Affects OAuth endpoints, API URLs, and environment variables

- **READINESS_CRITERIA**: Third argument from $ARGUMENTS (optional, default: "standard")
  - Readiness assessment level: "minimal", "standard", "comprehensive", "critical"
  - Determines depth and scope of validation checks
  - Controls the strictness of pass/fail criteria

## Usage

```bash
/e2e-readiness-check <deployment_target> [environment] [readiness_criteria]
```

Examples:
- `/e2e-readiness-check netlify production comprehensive`
- `/e2e-readiness-check production staging standard`
- `/e2e-readiness-check preview dev minimal`
- `/e2e-readiness-check netlify production critical`

## Parallel Agent Orchestration

This command implements **comprehensive validation deployment** with specialized assessment agents:

### **Phase 1: Environment Assessment & Configuration Validation**
- Analyze target deployment configuration and requirements
- Validate environment variables and configuration files
- Check OAuth provider configurations and endpoints
- Assess API integration readiness and connectivity

### **Phase 2: Multi-Domain Readiness Validation**
Deploy 6 specialized validation agents **simultaneously**:

```markdown
Task 1: devops-engineer (Infrastructure Readiness Specialist)
- Focus: Deployment configuration, build processes, environment setup
- Target: 100% deployment pipeline validation
- Scope: CI/CD, environment variables, build optimization, deploy scripts

Task 2: security-researcher (Security Compliance Specialist)
- Focus: Security configurations, OAuth setup, API security
- Target: Zero security vulnerabilities and compliance validation
- Scope: HTTPS, authentication flows, API keys, CORS, CSP headers

Task 3: qa-engineer (Integration Testing Specialist)
- Focus: API integrations, third-party service connectivity
- Target: All external integrations functional
- Scope: Firebase, OAuth providers, calendar APIs, notification services

Task 4: qa-engineer (Performance Validation Specialist)
- Focus: Production performance benchmarks and optimization
- Target: <3s load time, >90 lighthouse score, <100MB memory
- Scope: Bundle size, load performance, runtime optimization

Task 5: qa-engineer (Cross-Platform Compatibility Specialist)
- Focus: Browser compatibility, mobile responsiveness, platform testing
- Target: 100% compatibility across target browsers/devices
- Scope: Chrome, Firefox, Safari, Edge, iOS Safari, Android Chrome

Task 6: flutter-engineer (Application Readiness Specialist)
- Focus: Application functionality, feature completeness, error handling
- Target: All critical user flows functional
- Scope: End-to-end user journeys, error scenarios, edge cases
```

### **Phase 3: Comprehensive Readiness Decision**
- Aggregate validation results from all 6 specialists
- Apply readiness criteria thresholds
- Generate definitive **YES/NO** deployment decision
- Provide detailed remediation plan if readiness fails

## Deployment Readiness Domains

### **Infrastructure Readiness** (DevOps Validation)
```markdown
**Infrastructure Agent Instructions:**
"Validate $DEPLOYMENT_TARGET infrastructure readiness for $ENVIRONMENT. Focus on:
- Build process validation and optimization
- Environment variable configuration and security
- CI/CD pipeline functionality and performance
- Deploy script validation and error handling
- Resource allocation and scaling configuration
- Monitoring and logging setup verification
- Backup and rollback procedures validation"
```

### **Security Compliance** (Security Validation)
```markdown
**Security Agent Instructions:**
"Assess security readiness for $DEPLOYMENT_TARGET deployment. Focus on:
- HTTPS configuration and SSL certificate validation
- OAuth provider setup and authentication flow testing
- API security implementation and token validation
- CORS policy configuration and testing
- Content Security Policy (CSP) header validation
- Input sanitization and XSS prevention
- Sensitive data handling and privacy compliance
- OWASP security checklist validation"
```

### **Integration Functionality** (Integration Testing)
```markdown
**Integration Agent Instructions:**
"Validate all external integrations for $DEPLOYMENT_TARGET. Focus on:
- Firebase configuration and connectivity testing
- OAuth provider integration (Google, Microsoft, etc.)
- Calendar API integration and synchronization
- Notification service configuration and delivery
- Third-party service authentication and error handling
- API rate limiting and throttling validation
- Network resilience and retry logic testing
- Service dependency health checks"
```

### **Performance Benchmarks** (Performance Validation)
```markdown
**Performance Agent Instructions:**
"Assess production performance readiness for $DEPLOYMENT_TARGET. Focus on:
- Bundle size optimization and lazy loading validation
- Initial load time measurement (<3s requirement)
- Lighthouse performance score (>90 target)
- Memory usage optimization (<100MB target)
- Frame rate consistency (>60fps target)
- Network request optimization and caching
- Image optimization and delivery
- JavaScript execution performance profiling"
```

### **Cross-Platform Compatibility** (Compatibility Testing)
```markdown
**Compatibility Agent Instructions:**
"Validate cross-platform readiness for $DEPLOYMENT_TARGET. Focus on:
- Browser compatibility testing (Chrome, Firefox, Safari, Edge)
- Mobile browser validation (iOS Safari, Android Chrome)
- Responsive design and mobile optimization
- Touch interaction and gesture support
- Device-specific feature availability
- Progressive Web App (PWA) functionality
- Offline capability and service worker validation
- Accessibility compliance (WCAG 2.1 AA)"
```

### **Application Functionality** (End-to-End Validation)
```markdown
**Application Agent Instructions:**
"Validate complete application readiness for $DEPLOYMENT_TARGET. Focus on:
- Critical user journey testing (login, scheduling, calendar sync)
- Error handling and graceful degradation
- Data persistence and state management
- Navigation and routing functionality
- Feature flag and configuration validation
- User experience and workflow completion
- Edge case handling and boundary conditions
- Production environment specific testing"
```

## Readiness Criteria Levels

### **Minimal Readiness** (minimal)
- Basic functionality working
- No critical security vulnerabilities
- Deployment pipeline functional
- Core user flows operational

### **Standard Readiness** (standard)
- All critical features functional
- Performance meets basic thresholds
- Security compliance validated
- Cross-browser compatibility confirmed

### **Comprehensive Readiness** (comprehensive)
- All features fully functional
- Performance optimization complete
- Complete security audit passed
- Full cross-platform validation

### **Critical Readiness** (critical)
- Zero tolerance for any issues
- Performance exceeds all benchmarks
- Complete security and compliance audit
- Comprehensive testing across all domains

## Success Criteria & Decision Matrix

### **YES - Ready for Deployment** ✅
All criteria must pass for deployment approval:
1. **Infrastructure**: Build and deploy pipeline 100% functional ✅
2. **Security**: Zero critical/high vulnerabilities, all configs validated ✅
3. **Integration**: All external services functional and tested ✅
4. **Performance**: All benchmarks met or exceeded ✅
5. **Compatibility**: 100% compatibility across target platforms ✅
6. **Functionality**: All critical user flows operational ✅

### **NO - Not Ready for Deployment** ❌
Any failure in critical domains blocks deployment:
- Critical security vulnerabilities identified
- Performance benchmarks not met
- Essential integrations non-functional
- Critical user flows broken
- Infrastructure issues preventing deployment

## Direct Execution Instructions

1. **Parse Arguments**: Extract DEPLOYMENT_TARGET, ENVIRONMENT, and READINESS_CRITERIA from $ARGUMENTS
2. **Environment Analysis**: Analyze target deployment configuration and requirements
3. **Deploy Validation Agents**: Launch all 6 validation specialists simultaneously
4. **Infrastructure Validation**: Verify build processes, environment setup, CI/CD pipeline
5. **Security Assessment**: Complete security audit and vulnerability scan
6. **Integration Testing**: Validate all external service integrations
7. **Performance Benchmarking**: Execute performance tests and optimization validation
8. **Compatibility Testing**: Cross-platform and cross-browser validation
9. **Application Testing**: End-to-end functionality and user journey validation
10. **Decision Aggregation**: Apply readiness criteria and generate YES/NO decision
11. **Documentation**: Post comprehensive readiness report to PR

## Execution Workflow

### Pre-Deployment Validation
```bash
# Validate environment configuration
flutter doctor
flutter analyze
flutter test --coverage

# Validate build process
flutter build web --release
firebase use $ENVIRONMENT

# Environment variable validation
echo "Checking OAuth configurations..."
echo "Validating API endpoints..."
```

### Performance Benchmarking
```bash
# Bundle size analysis
flutter build web --analyze-size

# Lighthouse CI validation
npm run lighthouse:ci

# Memory usage profiling
flutter run --profile --observatory-port=8080
```

### Security Validation
```bash
# Dependency vulnerability scan
flutter pub deps
dart pub audit

# Security header validation
curl -I https://$DEPLOYMENT_URL

# HTTPS and SSL validation
openssl s_client -connect $DEPLOYMENT_URL:443
```

## GitHub Integration

### Automated Readiness Report
```markdown
## 🚀 Production Readiness Assessment - $DEPLOYMENT_TARGET

### 🎯 **DEPLOYMENT DECISION: [YES ✅ / NO ❌]**

**📊 Readiness Summary:**
- **Infrastructure**: [PASS/FAIL] - Build pipeline and environment setup
- **Security**: [PASS/FAIL] - Security compliance and vulnerability assessment
- **Integration**: [PASS/FAIL] - External service connectivity and functionality
- **Performance**: [PASS/FAIL] - Benchmark validation and optimization
- **Compatibility**: [PASS/FAIL] - Cross-platform and browser support
- **Functionality**: [PASS/FAIL] - End-to-end application validation

**🔍 Detailed Validation Results:**

#### Infrastructure Readiness ✅
- ✅ Build process: Successful web build generation
- ✅ Environment variables: All required configs present
- ✅ CI/CD pipeline: Deployment automation functional
- ✅ Resource allocation: Adequate for expected load

#### Security Compliance ✅
- ✅ HTTPS configuration: SSL certificate valid
- ✅ OAuth integration: All providers configured and tested
- ✅ API security: Proper authentication and authorization
- ✅ Vulnerability scan: Zero critical/high vulnerabilities

#### Integration Functionality ✅
- ✅ Firebase services: Authentication and database operational
- ✅ Calendar APIs: Google Calendar sync functional
- ✅ Notification services: Email and push notifications working
- ✅ Third-party services: All integrations tested and operational

#### Performance Benchmarks ✅
- ✅ Load time: [X.X]s (Target: <3s)
- ✅ Lighthouse score: [XX]/100 (Target: >90)
- ✅ Bundle size: [XX]MB (Optimized)
- ✅ Memory usage: [XX]MB (Target: <100MB)

#### Cross-Platform Compatibility ✅
- ✅ Desktop browsers: Chrome, Firefox, Safari, Edge
- ✅ Mobile browsers: iOS Safari, Android Chrome
- ✅ Responsive design: All breakpoints functional
- ✅ PWA functionality: Service worker and offline support

#### Application Functionality ✅
- ✅ User authentication: Login/logout flows operational
- ✅ Calendar integration: Scheduling and sync functional
- ✅ Core workflows: All critical user journeys tested
- ✅ Error handling: Graceful degradation implemented

**📈 Performance Metrics:**
- Initial Load Time: [X.X]s
- Lighthouse Performance: [XX]/100
- Bundle Size: [XX]MB
- Memory Usage: [XX]MB
- Time to Interactive: [X.X]s

**🛡️ Security Status:**
- SSL/HTTPS: ✅ Valid certificate
- OAuth Providers: ✅ All configured
- API Security: ✅ Proper authentication
- Vulnerabilities: ✅ Zero critical issues

**🌐 Compatibility Matrix:**
| Platform | Browser | Status |
|----------|---------|--------|
| Desktop | Chrome | ✅ |
| Desktop | Firefox | ✅ |
| Desktop | Safari | ✅ |
| Desktop | Edge | ✅ |
| Mobile | iOS Safari | ✅ |
| Mobile | Android Chrome | ✅ |

### **DEPLOYMENT RECOMMENDATION: [PROCEED ✅ / HOLD ❌]**

[If PROCEED]: All readiness criteria satisfied. Application is ready for production deployment.

[If HOLD]: Critical issues identified. Address the following before deployment:
- [List specific issues requiring resolution]
- [Provide remediation steps and timeline]
- [Specify re-validation requirements]

🤖 Generated with Claude Code - Production Readiness Assessment
```

This command provides comprehensive, definitive deployment readiness assessment through coordinated multi-agent validation, ensuring confident production deployments with minimal risk.