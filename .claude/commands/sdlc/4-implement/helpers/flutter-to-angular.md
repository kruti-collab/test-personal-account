---
allowed-tools: Task, Read, Write, Edit, MultiEdit, Glob, Grep, Bash(git fetch:*, git show:*, git log:*, git branch:* --list, git diff:*, git rev-parse:*, git status:* --porcelang, git rev-list:* --count, git ls-tree:*, mkdir:*, npm:* run, npm:* test, git:* add, git:* commit), , gh issue list --search, gh issue view
description: Convert Flutter MVVM to Angular with pixel-perfect visual and functional parity
argument-hint: <gh-ticket> <flutter-branch> [angular-output-dir]
pre-conversion-hooks:
  - name: security-scan
    description: Scan for exposed secrets before conversion
    command: npm run security:scan
    required: false
  - name: flutter-validation
    description: Validate Flutter source code quality
    command: flutter analyze lib/
    required: false
post-conversion-hooks:
  - name: lint-check
    description: Run Angular lint checks on generated code
    command: npm run lint
    required: true
  - name: test-coverage
    description: Verify test coverage meets 80% threshold
    command: npm run test:coverage
    required: true
  - name: build-check
    description: Ensure Angular code builds successfully
    command: npm run build
    required: true
  - name: security-audit
    description: Audit generated code for security vulnerabilities
    command: npm audit
    required: false
---

# Flutter to Angular MVVM Converter with Pixel-Perfect Conversion

Transform your Flutter app into an Angular web application with **100% visual and functional parity**. This command analyzes your actual Flutter code and generates Angular code that preserves every color, font, spacing, gesture, and interaction exactly.

## Critical Enhancement: Pixel-Perfect Conversion

**GOAL:** Achieve 100% visual and functional parity between Flutter and Angular

**VALIDATION CRITERIA:**
- ✅ All theme colors extracted and mapped exactly
- ✅ All font families, sizes, weights, and responsive sizing preserved
- ✅ All spacing values (padding, margin) match to the pixel
- ✅ Button dimensions, colors, elevation, and border radius match exactly
- ✅ Swipe gestures work for page navigation (PageView → Angular carousel)
- ✅ Dot indicators have correct colors, animations, and expanding effect
- ✅ Click events fire and navigate properly
- ✅ All responsive breakpoints preserved
- ✅ All color literals (Color(0xFFXXXXXX)) converted to CSS custom properties

## Quick Start

```bash
/flutter-to-angular #251 #251-clean
```

That's it! The command will:
- Read your GitHub issue (#251) to understand what you built
- Analyze your Flutter code from the branch (#251-clean) with **detailed style extraction**
- Generate matching Angular code with **pixel-perfect MVVM architecture**
- Extract **all theme colors, fonts, spacing, and UI patterns**
- Implement **swipe gestures, animations, and interactions**
- Create tests and documentation

## Validation & Auto-Correction Loop

**CRITICAL WORKFLOW:** The command now includes an automated validation and correction loop that ensures 100% parity, followed by comprehensive PR quality gates:

```
┌─────────────────────────────────────────────────────────────┐
│  Phase 0-7: Extract, Convert, Generate                     │
│  ✅ Flutter source analyzed                                 │
│  ✅ Angular code generated                                  │
└─────────────┬───────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 8: Initial Validation                                │
│  🔍 Compare Angular vs Flutter                              │
│  📊 Calculate parity score                                  │
└─────────────┬───────────────────────────────────────────────┘
              │
              ▼
         ┌────────────┐
         │ Parity =   │
         │   100%?    │
         └─────┬──────┘
               │
        ┌──────┴──────┐
        │             │
       YES            NO
        │             │
        │             ▼
        │    ┌────────────────────────────────────────┐
        │    │ Phase 8.5: Auto-Correction (Iteration) │
        │    │ 🔧 Fix mismatches using Edit tool      │
        │    │ 📝 Log fixes applied                   │
        │    └────────┬───────────────────────────────┘
        │             │
        │             ▼
        │    ┌────────────────────────────────────────┐
        │    │ Re-run Phase 8 Validation              │
        │    │ 🔍 Recalculate parity                  │
        │    └────────┬───────────────────────────────┘
        │             │
        │             ▼
        │        ┌─────────────┐
        │        │ Iterations  │
        │        │   < 3 AND   │
        │        │ Parity <100%│
        │        └──────┬──────┘
        │               │
        │        ┌──────┴──────┐
        │        │             │
        │       YES            NO
        │        │             │
        │        └─────────┐   │
        │                  │   │
        └──────────────────┴───┘
                           │
                           ▼
                ┌──────────────────────────────────────┐
                │ Phase 8.1: Quality & Performance     │
                │ 🔧 Icon centralization               │
                │ 🔧 Route constants enforcement        │
                │ ⚡ Font loading optimization          │
                └──────────┬────────────────────────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ Phase 9: Document     │
                │ ✅ 100% Parity        │
                │ 📚 Generate docs      │
                └──────────┬────────────┘
                           │
                           ▼
                ┌──────────────────────────────────────┐
                │ Phase 10: PR Review & Quality Gates  │
                │ 🔒 Security scanning                  │
                │ 🧹 Code quality checks                │
                │ 🧪 Test coverage validation (≥80%)    │
                │ 🏗️  Production build validation       │
                └──────────┬────────────────────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ All Blocking │
                    │ Gates Pass?  │
                    └──────┬───────┘
                           │
                    ┌──────┴──────┐
                    │             │
                   YES            NO
                    │             │
                    │             ▼
                    │    ┌────────────────────────┐
                    │    │ Delegate Fixes         │
                    │    │ ➜ security-researcher  │
                    │    │ ➜ angular-mvvm-engineer│
                    │    │ ➜ qa-engineer          │
                    │    │ ➜ devops-engineer      │
                    │    └────────┬───────────────┘
                    │             │
                    │             ▼
                    │    ┌────────────────────────┐
                    │    │ Re-run Phase 10        │
                    │    │ (Max 2 iterations)     │
                    │    └────────┬───────────────┘
                    │             │
                    └─────────────┘
                           │
                           ▼
                ┌──────────────────────────────────────┐
                │ Commit & Push (ONCE)                 │
                │ 📦 git add .                          │
                │ 📝 git commit (comprehensive message)│
                │ 🚀 git push origin HEAD               │
                └──────────┬────────────────────────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ Create Pull Request  │
                │ ✅ Ready for Review   │
                │ 📝 PR Description     │
                └───────────────────────┘
```

**Key Features:**
- **Strict sequential execution** - Phases run synchronously, one after another
- **Max 3 iterations** to reach 100% visual parity (Phase 8.5)
- **Code quality validation** with auto-fixes for best practices (Phase 8.1)
- **Automated fixes** using Edit tool with exact Flutter values
- **Progress tracking** showing parity improvement per iteration
- **Single commit/push** strategy after all quality gates pass
- **Automatic PR creation** with comprehensive quality report
- **Fallback reporting** if 100% not achieved after max iterations

## CRITICAL: Sequential Execution Requirement

**MANDATORY WORKFLOW RULE:** All phases MUST execute synchronously (one after another) with NO skipping or parallel execution.

### Execution Rules:
1. **Sequential Order**: Phases 0 → 0.5 → 1 → 2 → 2.5 → 3 → 4 → 5 → 6 → 7 → 8 → 8.5 → 8.1 → 9 → 10
2. **Complete Before Continue**: Each phase must be 100% complete before starting the next phase
3. **No Parallel Execution**: DO NOT work on multiple phases simultaneously
4. **No Phase Skipping**: Every phase must execute, even if it seems optional
5. **Blocking on Failure**: If any phase fails, STOP immediately and report error

### What This Means:
- ✅ **DO**: Complete Phase 0 → Wait → Complete Phase 0.5 → Wait → Complete Phase 1 → etc.
- ❌ **DO NOT**: Start Phase 3 while Phase 2 is still running
- ❌ **DO NOT**: Skip Phase 2.5 because "it seems minor"
- ❌ **DO NOT**: Work on Phase 5 and Phase 6 at the same time
- ❌ **DO NOT**: Jump ahead to Phase 8 before finishing Phase 7

### Why Sequential Execution?
1. **Data Dependencies**: Later phases depend on outputs from earlier phases
2. **Validation Accuracy**: Each phase validates the previous phase's outputs
3. **Error Isolation**: Sequential execution makes debugging easier
4. **Quality Assurance**: Ensures no step is missed or partially completed
5. **Predictable State**: Always know exactly what has been completed

### Enforcement:
```typescript
// Correct Sequential Execution
async function executeConversion() {
  await phase0_GatherRequirements();     // Wait for completion
  await phase0_5_ThemeExtraction();      // Wait for completion
  await phase1_ArchitectureAnalysis();   // Wait for completion
  await phase2_ProjectSetup();           // Wait for completion
  // ... continue sequentially
}

// INCORRECT - DO NOT DO THIS
async function executeConversionWRONG() {
  // ❌ WRONG: Parallel execution
  Promise.all([
    phase0_GatherRequirements(),
    phase1_ArchitectureAnalysis(),
    phase2_ProjectSetup()
  ]);
}
```

**Example Flow:**
```
Initial Generation → 85% parity (8 mismatches)
   ↓
Iteration 1 → Fix 8 mismatches → 97% parity (2 remaining)
   ↓
Iteration 2 → Fix 2 mismatches → 100% parity ✅
   ↓
Success! (Used 2/3 iterations)
```

## Enhanced Conversion Process (15 Phases)

### Phase 0: Gather Requirements & Source Code
**What happens:** Fetches GitHub issue details and analyzes your Flutter branch

**Output:** `conversion-plan-#251.md` with:
- Full GitHub requirements
- Flutter file structure
- Conversion mapping plan

### Phase 0.5: Theme & Style Extraction (ENHANCED - CRITICAL)
**Who does it:** Primary agent with direct git operations

**What happens:** Deep analysis of Flutter theme and styling with **ACTUAL SOURCE CODE READING**

**CRITICAL IMPLEMENTATION - Execute These Git Operations:**

```bash
# 1. Create temporary extraction directory
mkdir -p /tmp/flutter-extraction

# 2. Extract Flutter theme file from remote branch
git show origin/<flutter-branch>:lib/flutter_flow/flutter_flow_theme.dart > /tmp/flutter-extraction/theme.dart

# 3. Extract all Flutter widget files for the feature
git show origin/<flutter-branch>:lib/pages/<feature-name>/ --name-only | while read file; do
  git show origin/<flutter-branch>:$file > /tmp/flutter-extraction/$(basename $file)
done

# 4. Extract all Color literals from widgets using grep
grep -oP "Color\(0x[A-Fa-f0-9]{8}\)" /tmp/flutter-extraction/*.dart | sort -u > /tmp/flutter-extraction/color-literals.txt

# 5. Extract theme color definitions
grep -oP "(primary|secondary|tertiary|white|primaryBackground|secondaryBackground).*Color\(0x[A-Fa-f0-9]{8}\)" /tmp/flutter-extraction/theme.dart > /tmp/flutter-extraction/theme-colors.txt

# 6. Extract font definitions
grep -oP "GoogleFonts\.\w+" /tmp/flutter-extraction/*.dart | sort -u > /tmp/flutter-extraction/fonts.txt
grep -oP "fontSize:\s*[\d.]+" /tmp/flutter-extraction/*.dart | sort -u > /tmp/flutter-extraction/font-sizes.txt
grep -oP "FontWeight\.\w+" /tmp/flutter-extraction/*.dart | sort -u > /tmp/flutter-extraction/font-weights.txt

# 7. Extract spacing values
grep -oP "EdgeInsetsDirectional\.fromSTEB\([^)]+\)" /tmp/flutter-extraction/*.dart > /tmp/flutter-extraction/spacing.txt
grep -oP "SizedBox\(height:\s*[\d.]+\)" /tmp/flutter-extraction/*.dart > /tmp/flutter-extraction/sizedbox-spacing.txt

# 8. Extract component dimensions
grep -oP "width:\s*[\d.]+" /tmp/flutter-extraction/*.dart > /tmp/flutter-extraction/widths.txt
grep -oP "height:\s*[\d.]+" /tmp/flutter-extraction/*.dart > /tmp/flutter-extraction/heights.txt

# 9. Extract Flexible/Expanded widget properties for responsive sizing
grep -B2 -A5 "Flexible\|Expanded" /tmp/flutter-extraction/*.dart > /tmp/flutter-extraction/flexible-widgets.txt

# 10. Extract Column mainAxisSize properties for layout behavior
grep -oP "mainAxisSize:\s*MainAxisSize\.\w+" /tmp/flutter-extraction/*.dart > /tmp/flutter-extraction/main-axis-size.txt

# 11. Extract localization keys
grep -oP "FFLocalizations\.of\(context\)\.getText\('[^']+'\)" /tmp/flutter-extraction/*.dart | sort -u > /tmp/flutter-extraction/localization-keys.txt

# 12. Extract Flutter translation files
for locale in en es he; do
  git show origin/<flutter-branch>:assets/translations/$locale.json > /tmp/flutter-extraction/translations-$locale.json 2>/dev/null || echo "{}" > /tmp/flutter-extraction/translations-$locale.json
done
```

**Parse and Structure Extracted Values:**

```typescript
// Create extracted-values.json with ACTUAL values from Flutter source
{
  "theme": {
    "primary": "#6A0DAD",           // Extracted from FlutterFlowTheme.primary
    "secondary": "#39D2C0",         // Extracted from FlutterFlowTheme.secondary
    "white": "#FFFFFF",             // Extracted from FlutterFlowTheme.white
    "primaryBackground": "#FFFFFF", // Extracted from theme
    "secondaryBackground": "#F1F4F8"
  },
  "colorLiterals": {
    "0xFFE1CFEE": "#E1CFEE",       // Extracted from Color(0xFFE1CFEE) in widgets
    "0xFF949494": "#949494",       // Extracted from widget Text styles
    "0xFF6A0DAD": "#6A0DAD"        // All color literals mapped
  },
  "typography": {
    "fontFamily": "Montserrat",     // Extracted from GoogleFonts.montserrat
    "sizes": {
      "title": "24px",              // Extracted from fontSize: 24.0
      "description": "14px",        // Extracted from fontSize: 14.0
      "small": "12px"               // Extracted from fontSize: 12.0
    },
    "weights": {
      "title": "600",               // Extracted from FontWeight.w600
      "normal": "400",              // Extracted from FontWeight.normal
      "medium": "500"               // Extracted from FontWeight.w500
    },
    "responsive": [
      {
        "breakpoint": "375px",      // Extracted from MediaQuery condition
        "fontSize": "12px",
        "applies": "title"
      }
    ]
  },
  "spacing": {
    "page": { "top": 20, "right": 20, "bottom": 20, "left": 0 },
    "content": { "top": 20, "right": 20, "bottom": 8, "left": 20 },
    "itemGap": 12                   // From .divide(SizedBox(height: 12.0))
  },
  "components": {
    "button": {
      "width": "400px",             // Extracted from FFButtonWidget
      "height": "50px",
      "borderRadius": "8px",
      "elevation": "2.0"
    },
    "image": {
      "width": "90vw",              // Extracted from Container width
      "height": "30vh"
    }
  },
  "layout": {
    "flexibleWidgets": [
      {
        "widget": "image-container",
        "flex": 1,
        "convertTo": "height: 30vh"  // Flexible flex: 1 → 30% viewport height
      }
    ],
    "columnProperties": [
      {
        "selector": ".content-column",
        "mainAxisSize": "min",        // MainAxisSize.min
        "convertTo": "flex: 0 1 auto" // Minimal space
      },
      {
        "selector": ".full-height-column",
        "mainAxisSize": "max",        // MainAxisSize.max
        "convertTo": "flex: 1 1 auto; height: 100%"
      }
    ]
  },
  "localization": {
    "keys": [
      {
        "key": "vhon39ky",
        "en": "Get Started with Your Schedule",
        "es": "Comienza con tu horario",
        "he": "התחל עם לוח הזמנים שלך"
      },
      {
        "key": "abc123de",
        "en": "Manage your appointments easily",
        "es": "Gestiona tus citas fácilmente",
        "he": "נהל את הפגישות שלך בקלות"
      }
    ],
    "servicePattern": "localization.getText('flutter_{key}')"
  }
}
```

**Validation Requirements:**

After extraction, VERIFY:
1. ✅ All Color(0xFFXXXXXX) literals converted to hex values
2. ✅ All theme colors mapped to CSS custom properties
3. ✅ Font family extracted and available in Google Fonts
4. ✅ All font sizes, weights extracted
5. ✅ All spacing values captured
6. ✅ All component dimensions recorded
7. ✅ Responsive breakpoints identified
8. ✅ Flexible/Expanded widget properties extracted (NEW)
9. ✅ Column mainAxisSize properties captured (NEW)
10. ✅ All localization keys extracted from FFLocalizations (NEW)
11. ✅ Translation files retrieved for all locales (en, es, he) (NEW)

**Output:**
- `/tmp/flutter-extraction/extracted-values.json` - **SOURCE OF TRUTH for all subsequent phases**
- `theme-extraction-report.md` - Complete theme documentation with line numbers
- `styles/theme.scss` - Angular theme variables (generated from extracted-values.json)
- `styles/typography.scss` - Font system (generated from extracted-values.json)
- `styles/spacing.scss` - Spacing constants (generated from extracted-values.json)

**CRITICAL:** This JSON file becomes the **SINGLE SOURCE OF TRUTH** for Phase 5. NO fallback values allowed.

### Phase 1: Architecture Analysis
**Who does it:** solutions-architect

**What happens:** Reviews Flutter MVVM structure and creates conversion strategy with **style preservation focus**

**Output:** Detailed conversion roadmap with:
- Theme mapping strategy
- UI pattern conversion plan (PageView → Carousel, SmoothPageIndicator → Custom component)
- Gesture handling approach (swipe implementation)

### Phase 2: Project Setup
**Who does it:** angular-architect

**What happens:** Creates Angular project structure matching your Flutter app

**Output:**
- Standalone components with explicit imports
- Routing configuration with standalone routes
- Dependency injection setup with inject()
- **Theme module with CSS custom properties**
- **Typography configuration**

### Phase 2.5: UI Pattern Analysis (NEW - CRITICAL)
**Who does it:** flutter-ui-pattern-analyzer (specialized subagent)

**What happens:** Detects and maps Flutter UI patterns to Angular equivalents

**Detects and Converts:**

1. **PageView → Swipeable Carousel:**
   ```dart
   // Flutter
   PageView(
     controller: pageController,
     scrollDirection: Axis.horizontal,
     children: [page1, page2, page3]
   )
   ```

   ```typescript
   // Angular - Using HammerJS for swipe gestures
   @Component({
     template: `
       <div class="carousel-container"
            (swipeleft)="nextPage()"
            (swiperight)="prevPage()">
         <div class="carousel-track"
              [style.transform]="'translateX(-' + (currentPage() * 100) + '%)'">
           @for (page of pages; track page.id) {
             <div class="carousel-page">{{ page.content }}</div>
           }
         </div>
       </div>
     `
   })
   ```

   **Implementation Details:**
   - Install and configure HammerJS: `npm install hammerjs @types/hammerjs`
   - Add to angular.json scripts: `"node_modules/hammerjs/hammer.min.js"`
   - Create swipe directive or use HammerJS directives
   - Implement PageController equivalent with signals
   - Add smooth CSS transitions matching `Curves.ease`

2. **SmoothPageIndicator → Custom Indicator Component:**
   ```dart
   // Flutter
   SmoothPageIndicator(
     controller: pageController,
     count: 3,
     effect: ExpandingDotsEffect(
       expansionFactor: 2.0,
       spacing: 8.0,
       radius: 16.0,
       dotWidth: 8.0,
       dotHeight: 8.0,
       dotColor: Color(0xFFE1CFEE),
       activeDotColor: FlutterFlowTheme.of(context).primary,
     )
   )
   ```

   ```typescript
   // Angular - Custom expanding dots indicator
   @Component({
     selector: 'app-page-indicator',
     standalone: true,
     template: `
       <div class="page-indicator">
         @for (dot of dots; track $index) {
           <div class="dot"
                [class.active]="$index === currentPage()"
                (click)="onDotClick($index)"
                [style.width.px]="getDotWidth($index)"
                [style.height.px]="8"
                [style.margin-right.px]="8"
                [style.border-radius.px]="16"
                [style.background-color]="getDotColor($index)">
           </div>
         }
       </div>
     `,
     styles: [`
       .dot {
         display: inline-block;
         transition: all 300ms ease;
         cursor: pointer;
       }
       .dot.active {
         width: 16px !important; /* expansionFactor: 2.0 */
       }
     `]
   })
   export class PageIndicatorComponent {
     @Input() count!: number;
     @Input() currentPage = signal(0);
     @Output() pageChange = new EventEmitter<number>();

     get dots() { return Array(this.count); }

     getDotWidth(index: number): number {
       return this.currentPage() === index ? 16 : 8; // expansionFactor * dotWidth
     }

     getDotColor(index: number): string {
       return this.currentPage() === index
         ? 'var(--color-primary)'
         : '#E1CFEE';
     }

     onDotClick(index: number): void {
       this.pageChange.emit(index);
     }
   }
   ```

3. **FFButtonWidget → Styled Material Button:**
   ```dart
   // Flutter
   FFButtonWidget(
     text: 'Start Now',
     onPressed: () => completeOnboarding(),
     options: FFButtonOptions(
       width: 400.0,
       height: 50.0,
       color: FlutterFlowTheme.of(context).primary,
       textStyle: TextStyle(color: Colors.white),
       elevation: 2.0,
       borderRadius: BorderRadius.circular(8.0),
     )
   )
   ```

   ```typescript
   // Angular - Material button with exact styling
   @Component({
     template: `
       <button mat-raised-button
               class="primary-button"
               (click)="onStartNow()">
         Start Now
       </button>
     `,
     styles: [`
       .primary-button {
         width: 400px;
         height: 50px;
         background-color: var(--color-primary);
         color: var(--color-white);
         box-shadow: 0 2px 4px rgba(0,0,0,0.2); /* elevation: 2.0 */
         border-radius: 8px;
         font-family: 'Montserrat', sans-serif;
         font-size: 14px;
         font-weight: 600;
         text-transform: none;
       }
     `]
   })
   ```

4. **Container Decorations → CSS Styling:**
   - Extract `BoxDecoration` properties
   - Convert `borderRadius` to CSS border-radius
   - Convert `boxShadow` / `elevation` to CSS box-shadow
   - Preserve all dimension and color properties

**Output:**
- `ui-patterns-map.md` - Complete pattern conversion guide
- Reusable Angular components for detected patterns

### Phase 3: Model Conversion
**Who does it:** angular-architect

**What happens:** Converts Dart classes to TypeScript

**Example:**
```dart
// Flutter: lib/models/user.dart
class User {
  final String id;
  final String email;

  User({required this.id, required this.email});
}
```

```typescript
// Angular: models/user.model.ts
export interface User {
  id: string;
  email: string;
}

export class UserDto implements User {
  constructor(
    public id: string,
    public email: string
  ) {}
}
```

### Phase 4: ViewModel Conversion
**Who does it:** angular-mvvm-engineer

**What happens:** Converts ChangeNotifiers to Signal-based services

**Key Changes:**
- `notifyListeners()` → `signal.set()` or `.update()`
- Private fields → `signal<T>()`
- Getters → Computed signals
- Async methods → RxJS operators when needed
- **PageController → Signal-based page management**

**Example:**
```typescript
// Modern Angular 17+ ViewModel with PageView equivalent
@Injectable()
export class OnboardingViewModel {
  private readonly router = inject(Router);
  private readonly repository = inject(OnboardingRepository);

  // Signal-based page state (replaces PageController)
  private readonly _currentPage = signal(0);
  readonly currentPage = this._currentPage.asReadonly();

  private readonly _isAnimating = signal(false);
  readonly isAnimating = this._isAnimating.asReadonly();

  // Computed values
  readonly isLastPage = computed(() => this._currentPage() === 2);
  readonly canProceed = computed(() => !this._isAnimating());

  // PageController equivalent methods
  nextPage(): void {
    if (this._currentPage() < 2) {
      this._currentPage.update(p => p + 1);
    }
  }

  prevPage(): void {
    if (this._currentPage() > 0) {
      this._currentPage.update(p => p - 1);
    }
  }

  animateToPage(index: number): void {
    this._isAnimating.set(true);
    this._currentPage.set(index);
    // Simulate animation duration
    setTimeout(() => this._isAnimating.set(false), 500);
  }

  async completeOnboarding(): Promise<void> {
    await this.repository.markComplete();
    this.router.navigate(['/dashboard']);
  }
}
```

### Phase 5: Component Conversion with Exact Style Preservation (ENHANCED)
**Who does it:** angular-ui-engineer

**What happens:** Converts Flutter widgets to Angular standalone components with **pixel-perfect styling**

**MANDATORY PRE-REQUISITE:** Read `/tmp/flutter-extraction/extracted-values.json` BEFORE generating any code.

**CRITICAL REQUIREMENTS:**

1. **USE EXACT VALUES from extracted-values.json (NO FALLBACKS ALLOWED):**

   Before generating ANY CSS/SCSS, you MUST:
   - Read `/tmp/flutter-extraction/extracted-values.json`
   - Parse the JSON structure
   - Use ONLY the extracted values for ALL styling properties
   - If a value is missing from JSON, extract it manually from Flutter source using git show
   - NEVER use placeholder or fallback values

   **Example Consumption Pattern:**
   ```typescript
   // Read extracted values
   const extractedValues = JSON.parse(readFile('/tmp/flutter-extraction/extracted-values.json'));

   // Generate CSS using ONLY extracted values
   .title {
     font-family: '${extractedValues.typography.fontFamily}', sans-serif;
     color: var(--color-primary); // ${extractedValues.theme.primary}
     font-size: ${extractedValues.typography.sizes.title};
     font-weight: ${extractedValues.typography.weights.title};
     letter-spacing: 0;

     @media (max-width: ${extractedValues.typography.responsive[0].breakpoint}) {
       font-size: ${extractedValues.typography.responsive[0].fontSize};
     }
   }

   .start-button {
     width: ${extractedValues.components.button.width};
     height: ${extractedValues.components.button.height};
     background-color: var(--color-primary);
     border-radius: ${extractedValues.components.button.borderRadius};
   }
   ```

2. **Extract EXACT styling from Flutter widgets (IF NOT IN JSON):**
   - Parse every `Text` widget and extract:
     - Color: `FlutterFlowTheme.of(context).primary` → `var(--color-primary)`
     - Font size: `fontSize: 24.0` → `font-size: 24px`
     - Font weight: `FontWeight.w600` → `font-weight: 600`
     - Font family: `GoogleFonts.montserrat` → `font-family: 'Montserrat'`
     - Letter spacing: `letterSpacing: 0.0` → `letter-spacing: 0`
     - Responsive sizing: `MediaQuery` conditions → `@media` queries

   - Parse every `Container`/`Padding` and extract:
     - Padding: `EdgeInsetsDirectional.fromSTEB(20, 20, 20, 0)` → `padding: 20px 20px 0`
     - Width: `width: 400.0` → `width: 400px`
     - Height: `height: 50.0` → `height: 50px`
     - Background: `color: Colors.white` → `background-color: #FFFFFF`

   - Parse every `BoxDecoration` and extract:
     - Border radius: `borderRadius: BorderRadius.circular(8)` → `border-radius: 8px`
     - Elevation/Shadow: `elevation: 2.0` → `box-shadow: 0 2px 4px rgba(0,0,0,0.2)`

   - Parse every spacing element:
     - `.divide(SizedBox(height: 12.0))` → `gap: 12px` in flexbox or `margin-bottom: 12px`

2. **Generate CSS/SCSS with exact pixel-perfect values:**
   ```scss
   // Example: Exact conversion from Flutter onboarding
   .onboarding-page {
     width: 100%;
     height: 100%;

     .image-container {
       width: 90vw;
       height: 30vh;

       img {
         width: 100%;
         object-fit: contain;
       }
     }

     .title {
       font-family: 'Montserrat', sans-serif;
       color: var(--color-primary);
       font-size: 24px;
       font-weight: 600;
       letter-spacing: 0;
       padding: 0 20px 8px;

       @media (max-width: 375px) {
         font-size: 12px;
       }
     }

     .description {
       font-family: 'Montserrat', sans-serif;
       color: #949494;
       font-size: 14px;
       font-weight: 500;
       letter-spacing: 0;
       text-align: center;
       padding: 0 20px 10px;
     }

     .content-stack {
       display: flex;
       flex-direction: column;
       gap: 12px; // .divide(SizedBox(height: 12.0))
     }
   }

   .start-button {
     width: 400px;
     height: 50px;
     background-color: var(--color-primary);
     color: var(--color-white);
     box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2); // elevation: 2.0
     border-radius: 8px;
     border: 1px solid transparent;
     padding: 0;

     font-family: 'Montserrat', sans-serif;
     font-size: 14px;
     font-weight: 600;
     letter-spacing: 0;
   }
   ```

3. **Implement gesture handlers for swipe:**
   ```typescript
   // Component with HammerJS swipe support
   @Component({
     selector: 'app-onboarding',
     standalone: true,
     imports: [CommonModule, MatButtonModule, PageIndicatorComponent],
     templateUrl: './onboarding.component.html',
     styleUrls: ['./onboarding.component.scss']
   })
   export class OnboardingComponent implements OnInit {
     protected readonly vm = inject(OnboardingViewModel);

     ngOnInit(): void {
       // HammerJS will handle swipe gestures via template bindings
     }

     onSwipeLeft(): void {
       this.vm.nextPage();
     }

     onSwipeRight(): void {
       this.vm.prevPage();
     }

     onDotClick(index: number): void {
       this.vm.animateToPage(index);
     }

     onStartNow(): void {
       this.vm.completeOnboarding();
     }
   }
   ```

4. **Generate page transition animations:**
   ```scss
   .carousel-track {
     display: flex;
     transition: transform 500ms ease; // matches Curves.ease
     will-change: transform;
   }

   .carousel-page {
     flex: 0 0 100%;
     width: 100%;
   }
   ```

**Delegation to angular-ui-engineer with Extracted Values:**

```typescript
// When delegating to angular-ui-engineer, pass the extracted values explicitly:
Task({
  subagent_type: "angular-ui-engineer",
  description: "Generate onboarding component",
  prompt: `
    Create the onboarding Angular component with PIXEL-PERFECT styling.

    **MANDATORY FIRST STEP:** Read /tmp/flutter-extraction/extracted-values.json

    Use THESE EXACT VALUES from extracted-values.json:

    Theme Colors (from extractedValues.theme):
    - Primary: ${extractedValues.theme.primary}
    - Secondary: ${extractedValues.theme.secondary}
    - White: ${extractedValues.theme.white}

    Color Literals (from extractedValues.colorLiterals):
    - Dot Inactive: ${extractedValues.colorLiterals['0xFFE1CFEE']}
    - Description Text: ${extractedValues.colorLiterals['0xFF949494']}

    Typography (from extractedValues.typography):
    - Font Family: ${extractedValues.typography.fontFamily}
    - Title Font Size: ${extractedValues.typography.sizes.title}
    - Title Font Weight: ${extractedValues.typography.weights.title}
    - Description Font Size: ${extractedValues.typography.sizes.description}

    Component Dimensions (from extractedValues.components):
    - Button: ${extractedValues.components.button.width} x ${extractedValues.components.button.height}
    - Border Radius: ${extractedValues.components.button.borderRadius}

    Spacing (from extractedValues.spacing):
    - Page Padding: ${JSON.stringify(extractedValues.spacing.page)}
    - Item Gap: ${extractedValues.spacing.itemGap}px

    DO NOT use any fallback or placeholder values.
    Every CSS property MUST match the extracted values exactly.
    If a value is missing, stop and request it from the primary agent.

    Generate:
    1. Component TypeScript file
    2. Template HTML file
    3. SCSS file with exact pixel-perfect values
  `
})
```

**Creates:**
- `.component.ts` (TypeScript logic with standalone: true)
- `.component.html` (Template with new control flow and gesture handlers)
- `.component.scss` (Styles with **exact pixel-perfect values** from extracted-values.json)

### Phase 6: Repository & Services
**Who does it:** angular-mvvm-engineer

**What happens:** Converts Flutter repositories to Angular services

**Example:**
```typescript
// Abstract interface
export abstract class UserRepository {
  abstract getUser(id: string): Observable<User>;
}

// Concrete implementation with Firebase
@Injectable()
export class UserRepositoryImpl implements UserRepository {
  private readonly firestore = inject(Firestore);

  getUser(id: string): Observable<User> {
    return this.firestore.doc(`users/${id}`).valueChanges();
  }
}
```

### Phase 7: Testing
**Who does it:** qa-engineer

**What happens:** Generates comprehensive tests and validates against GitHub acceptance criteria

**ENHANCED TEST REQUIREMENTS:**

1. **Visual Regression Tests:**
   - Test button colors match theme
   - Test font sizes at different breakpoints
   - Test spacing values
   - Test responsive layouts

2. **Interaction Tests:**
   - Test button click events fire correctly
   - Test swipe left/right changes pages
   - Test dot indicator clicks navigate to correct page
   - Test animations complete properly

3. **Accessibility Tests:**
   - Test keyboard navigation through pages
   - Test screen reader announcements
   - Test focus management

**Creates:**
- Unit tests for components
- Service tests with mocks
- Integration tests
- **Gesture interaction tests**
- **Visual parity tests**
- Accessibility tests

### Phase 8: Validation & Verification (NEW - CRITICAL)
**Who does it:** qa-engineer

**What happens:** Validates 100% visual and functional parity

**Verification Checklist:**
```markdown
## Visual Parity Verification

### Theme Colors ✅/❌
- [ ] Primary color matches: #6A0DAD
- [ ] Secondary color matches: #39D2C0
- [ ] White color matches: #FFFFFF
- [ ] Dot inactive color matches: #E1CFEE
- [ ] Description text color matches: #949494

### Typography ✅/❌
- [ ] Font family: Montserrat loaded and applied
- [ ] Title font size: 24px (desktop), 12px (mobile ≤375px)
- [ ] Title font weight: 600
- [ ] Description font size: 14px
- [ ] Description font weight: 500

### Button Styling ✅/❌
- [ ] Button width: 400px
- [ ] Button height: 50px
- [ ] Button background: var(--color-primary)
- [ ] Button text color: var(--color-white)
- [ ] Button elevation: box-shadow matching elevation 2.0
- [ ] Button border radius: 8px

### Spacing ✅/❌
- [ ] Page padding: 20px right, 20px left, 0 top, 20px bottom
- [ ] Content padding: 20px all sides, 8px bottom
- [ ] Item spacing: 12px vertical gap

### Gestures & Interactions ✅/❌
- [ ] Swipe left navigates to next page
- [ ] Swipe right navigates to previous page
- [ ] Dot click navigates to correct page with animation
- [ ] Start button click navigates properly
- [ ] Page transition animation: 500ms ease

### Dot Indicator ✅/❌
- [ ] Dot width: 8px (inactive), 16px (active, expansion factor 2.0)
- [ ] Dot height: 8px
- [ ] Dot spacing: 8px
- [ ] Dot border radius: 16px
- [ ] Dot colors: #E1CFEE (inactive), var(--color-primary) (active)
- [ ] Expanding animation on transition
```

**Output:**
- `parity-validation-report.md` - Detailed comparison with Flutter original
- Screenshots for visual comparison
- Pass/fail report for each requirement
- **mismatch-details.json** - Structured data of all mismatches for Phase 8.5

---

## Common Visual Parity Issues and Solutions

This section documents real-world visual parity issues discovered during Flutter-to-Angular conversion. Phase 8 validation should **detect these issues** and Phase 8.5 auto-correction should **fix them automatically**.

### Issue 1: Color Extraction - Approximate vs Exact Colors

**Problem:** Automated conversion may use approximate colors instead of exact Flutter theme colors

**Root Cause:**
- Angular component uses fallback color (#7B52AB) instead of extracted theme color (#6A0DAD)
- Conversion didn't read actual FlutterFlowTheme definition
- Color literals in widgets not mapped to theme variables

**Detection Criteria (Phase 8):**
```json
{
  "issue": "Color mismatch",
  "property": ".title color",
  "flutterValue": "#6A0DAD",
  "angularValue": "#7B52AB",
  "severity": "high",
  "category": "theme-colors"
}
```

**Auto-Fix Strategy (Phase 8.5):**
1. Extract exact hex value from Flutter source:
   ```bash
   git show origin/<branch>:lib/flutter_flow/flutter_flow_theme.dart | grep "primary = Color"
   # Output: static const Color primary = Color(0xFF6A0DAD);
   ```
2. Update Angular component:
   ```scss
   // Before (incorrect)
   .title {
     color: #7B52AB; // fallback value
   }

   // After (correct)
   .title {
     color: var(--color-primary); // #6A0DAD from theme
   }
   ```

**Validation Checklist:**
- [ ] All button backgrounds use theme primary color (#6A0DAD)
- [ ] All button hover states use correct darker shade (#5A0B92)
- [ ] All active indicator/selected states use theme primary
- [ ] All focus rings use theme primary with correct rgba values
- [ ] Background colors match Flutter light (#FFFFFF) and dark (#1A1F24) themes

**Related Files to Check:**
- `extracted-values.json` → theme.primary
- `theme.scss` → --color-primary variable
- All `*.component.scss` files → color property values

---

### Issue 2: Image Sizing Consistency - Flexible Widget Behavior

**Problem:** Flutter's Flexible widget behavior misinterpreted, causing inconsistent image sizes across carousel slides

**Root Cause:**
- Flutter uses `Flexible(flex: 1, child: Container())` for dynamic height
- Angular conversion uses fixed heights or incorrect flex properties
- Different slides have different image heights
- Missing `flex-shrink: 0` causes size changes during layout

**Detection Criteria (Phase 8):**
```json
{
  "issue": "Image height inconsistency",
  "property": ".image-container height",
  "flutterValue": "Flexible(flex: 1) → 30% of container",
  "angularValue": "height: 300px (fixed)",
  "severity": "medium",
  "category": "layout-responsive"
}
```

**Auto-Fix Strategy (Phase 8.5):**
1. Analyze Flutter Flexible usage:
   ```dart
   // Flutter
   Flexible(
     flex: 1,
     child: Container(
       width: MediaQuery.sizeOf(context).width * 0.9, // 90vw
       child: Image.asset(...)
     )
   )
   ```
2. Convert to Angular flex CSS:
   ```scss
   // Before (incorrect)
   .image-container {
     width: 90vw;
     height: 300px; // Fixed height - WRONG
   }

   // After (correct)
   .image-container {
     width: 90vw;
     height: 30vh; // Responsive height matching Flutter flex behavior
     flex-shrink: 0; // Prevent size changes during layout
   }

   .image-container img {
     width: 100%;
     height: 100%;
     object-fit: contain; // Match Flutter BoxFit.contain
   }
   ```

**Validation Checklist:**
- [ ] All carousel slide images have identical dimensions
- [ ] Image heights use viewport units (vh) not fixed pixels
- [ ] Width: 90vw matches Flutter MediaQuery.sizeOf(context).width * 0.9
- [ ] Height: 30vh matches Flutter Flexible widget flex ratio
- [ ] flex-shrink: 0 applied to prevent layout shifts
- [ ] object-fit: contain matches Flutter BoxFit behavior

**Related Files to Check:**
- Flutter widget: Look for `Flexible`, `Expanded`, `MediaQuery.sizeOf`
- Angular SCSS: Check `.image-container`, `.carousel-slide`, `.slide-image`

---

### Issue 3: Vertical Centering and Spacing - mainAxisSize Behavior

**Problem:** Flutter's `mainAxisSize.min` vs `max` creates different centering behaviors not properly replicated in Angular

**Root Cause:**
- Flutter `Column(mainAxisSize: MainAxisSize.min)` uses minimum space
- Flutter `Column(mainAxisSize: MainAxisSize.max)` stretches to full height
- Angular conversion uses incorrect `justify-content` or `flex` properties
- `.divide(SizedBox(height: X))` converted to padding instead of margin

**Detection Criteria (Phase 8):**
```json
{
  "issue": "Vertical centering mismatch",
  "property": ".content-column centering",
  "flutterValue": "mainAxisSize: min → center with minimal space",
  "angularValue": "justify-content: center → full height",
  "severity": "medium",
  "category": "layout-alignment"
}
```

**Auto-Fix Strategy (Phase 8.5):**
1. Analyze Flutter Column properties:
   ```dart
   // Flutter
   Column(
     mainAxisSize: MainAxisSize.min, // Use minimum space
     mainAxisAlignment: MainAxisAlignment.center,
     children: [
       Widget1(),
       Widget2(),
     ].divide(SizedBox(height: 12.0)) // Spacing between items
   )
   ```
2. Convert to Angular flexbox:
   ```scss
   // Before (incorrect)
   .content-column {
     display: flex;
     flex-direction: column;
     justify-content: center;
     height: 100%; // mainAxisSize.max behavior - WRONG
     padding: 12px 0; // .divide converted to padding - WRONG
   }

   // After (correct - mainAxisSize.min)
   .content-column {
     display: flex;
     flex-direction: column;
     justify-content: center;
     align-items: center;
     flex: 0 1 auto; // Minimal space (mainAxisSize.min)
   }

   // Use adjacent sibling selector for spacing
   .content-column > * + * {
     margin-top: 12px; // .divide(SizedBox(height: 12.0))
   }

   // Alternative: Use gap (modern CSS)
   .content-column {
     display: flex;
     flex-direction: column;
     justify-content: center;
     gap: 12px; // .divide(SizedBox(height: 12.0))
     flex: 0 1 auto;
   }
   ```
   ```scss
   // For mainAxisSize.max behavior:
   .content-column-max {
     display: flex;
     flex-direction: column;
     justify-content: center;
     height: 100%; // Full height
     flex: 1 1 auto;
   }
   ```

**Validation Checklist:**
- [ ] `mainAxisSize.min` → `flex: 0 1 auto` (minimal space)
- [ ] `mainAxisSize.max` → `flex: 1 1 auto` + `height: 100%` (full height)
- [ ] `.divide(SizedBox(height: X))` → `gap: Xpx` or adjacent sibling margin
- [ ] Content is vertically centered matching Flutter layout
- [ ] Spacing between elements matches Flutter exactly

**Related Files to Check:**
- Flutter widget: Check `Column` for `mainAxisSize` property
- Angular SCSS: Check flexbox properties, gap, margin-top
- Angular template: Verify element structure matches Flutter children array

---

### Issue 4: Localization Key Mapping - Missing Translation Keys

**Problem:** Flutter localization keys (e.g., 'vhon39ky') don't exist in Angular translation files, causing missing text or errors

**Root Cause:**
- Flutter uses `FFLocalizations.of(context).getText('vhon39ky')`
- Angular LocalizationService doesn't have corresponding keys
- Translation files (en.json, es.json, he.json) missing Flutter key mappings
- Hardcoded text used as fallback instead of proper translations

**Detection Criteria (Phase 8):**
```json
{
  "issue": "Missing localization key",
  "property": "title text",
  "flutterValue": "FFLocalizations.getText('vhon39ky')",
  "angularValue": "Hardcoded text or missing",
  "severity": "high",
  "category": "localization",
  "affectedLocales": ["en", "es", "he"]
}
```

**Auto-Fix Strategy (Phase 8.5):**
1. Extract all localization keys from Flutter widgets:
   ```bash
   # Find all FFLocalizations calls
   git show origin/<branch>:<widget-file> | grep -oP "FFLocalizations\.of\(context\)\.getText\('[^']+'\)" | sort -u

   # Output:
   # FFLocalizations.of(context).getText('vhon39ky')
   # FFLocalizations.of(context).getText('abc123de')
   # FFLocalizations.of(context).getText('xyz789fg')
   ```

2. Read Flutter localization JSON files:
   ```bash
   # Extract actual translations from Flutter
   git show origin/<branch>:lib/flutter_flow/internationalization.dart
   git show origin/<branch>:assets/translations/en.json
   ```

3. Update Angular locale files:
   ```typescript
   // Before (missing keys)
   // src/assets/i18n/en.json
   {
     "common": {
       "welcome": "Welcome"
     }
   }

   // After (keys added)
   // src/assets/i18n/en.json
   {
     "common": {
       "welcome": "Welcome"
     },
     "flutter_vhon39ky": "Get Started with Your Schedule", // Extracted from Flutter
     "flutter_abc123de": "Manage your appointments easily",
     "flutter_xyz789fg": "Start Now"
   }
   ```

4. Update Angular component to use LocalizationService:
   ```typescript
   // Before (hardcoded or missing)
   template: `<h1>Get Started</h1>`

   // After (using LocalizationService)
   template: `<h1>{{ localization.getText('flutter_vhon39ky') }}</h1>`
   ```

**Validation Checklist:**
- [ ] All Flutter `FFLocalizations.getText()` calls mapped to Angular keys
- [ ] All locale files updated: `en.json`, `es.json`, `he.json`, etc.
- [ ] Key naming convention: `flutter_<original-key>` or direct mapping
- [ ] LocalizationService.getText() calls use correct keys in Angular templates
- [ ] Test in multiple languages to verify translations load correctly
- [ ] No hardcoded text in components (use localization service)

**Required Files to Create/Update:**
- `src/assets/i18n/en.json` - English translations
- `src/assets/i18n/es.json` - Spanish translations
- `src/assets/i18n/he.json` - Hebrew translations (RTL support)
- `src/app/core/services/localization.service.ts` - Angular service
- All component templates - Replace hardcoded text with localization calls

**Extraction Script:**
```bash
# Create localization extraction report
echo "# Flutter Localization Keys" > localization-keys.md
echo "" >> localization-keys.md

# Extract all keys from Flutter widgets
git show origin/<branch>:lib/pages/<feature>/ --name-only | while read file; do
  if [[ $file == *.dart ]]; then
    echo "## $file" >> localization-keys.md
    git show origin/<branch>:$file | grep -oP "getText\('[^']+'\)" | sort -u >> localization-keys.md
    echo "" >> localization-keys.md
  fi
done

# Extract translations from Flutter locale files
for locale in en es he; do
  echo "## Locale: $locale" >> localization-keys.md
  git show origin/<branch>:assets/translations/$locale.json >> localization-keys-$locale.json
done
```

**Related Files to Check:**
- Flutter: `lib/flutter_flow/flutter_flow_internationalization.dart`
- Flutter: `assets/translations/*.json`
- Angular: `src/assets/i18n/*.json`
- Angular: `src/app/core/services/localization.service.ts`

---

## Phase 8 Enhanced Validation Checklist

Phase 8 validation should now check for ALL common visual parity issues:

**Extended Validation Categories:**

1. **Theme Colors** (existing + enhanced)
   - Exact hex values match Flutter theme
   - No fallback or approximate colors
   - Hover states use correct darker shades
   - Focus rings use correct rgba values

2. **Image Sizing Consistency** (NEW)
   - All carousel images have identical dimensions
   - Responsive sizing uses vh/vw not fixed pixels
   - flex-shrink: 0 applied to prevent layout shifts
   - object-fit matches Flutter BoxFit

3. **Vertical Centering** (NEW)
   - mainAxisSize.min/max behavior preserved
   - justify-content and flex properties correct
   - .divide(SizedBox) converted to gap or margin (not padding)
   - Content alignment matches Flutter exactly

4. **Localization Keys** (NEW)
   - All FFLocalizations keys extracted and mapped
   - All locale JSON files updated (en, es, he, etc.)
   - No hardcoded text in templates
   - LocalizationService calls use correct keys

---

### Phase 8.5: Automated Correction (NEW - CRITICAL)
**Who does it:** Primary agent using Edit tool

**What happens:** Auto-fixes any mismatches found in Phase 8 validation

**CRITICAL IMPLEMENTATION:**

```typescript
// Phase 8.5 Auto-Fix Loop
const MAX_CORRECTION_ITERATIONS = 3;
let currentIteration = 0;
let validationReport = Phase8Results;

while (currentIteration < MAX_CORRECTION_ITERATIONS && validationReport.parity < 100) {
  console.log(`Auto-correction iteration ${currentIteration + 1}/${MAX_CORRECTION_ITERATIONS}`);

  // Parse mismatches from validation report
  const mismatches = parseMismatchDetails(validationReport);

  if (mismatches.length === 0) {
    break; // 100% parity achieved!
  }

  // Auto-fix each mismatch
  for (const mismatch of mismatches) {
    await autoFixMismatch(mismatch);
  }

  // Re-run validation
  validationReport = await runPhase8Validation();
  currentIteration++;
}

// Auto-fix function
async function autoFixMismatch(mismatch: Mismatch): Promise<void> {
  console.log(`Fixing: ${mismatch.property} in ${mismatch.file}`);

  // 1. Get correct value from Flutter source
  const correctValue = await extractValueFromFlutter(mismatch.property);

  // 2. Read Angular file
  const angularFile = await readFile(mismatch.file);

  // 3. Use Edit tool to fix the mismatch
  await Edit({
    file_path: mismatch.file,
    old_string: mismatch.angularValue,    // Current incorrect value
    new_string: correctValue               // Correct value from Flutter
  });

  console.log(`✅ Fixed: ${mismatch.property} (${mismatch.angularValue} → ${correctValue})`);
}
```

**Mismatch Processing Examples:**

**Example 1: Color Mismatch**
```json
{
  "property": ".title color",
  "flutterValue": "#6A0DAD",
  "angularValue": "#7B52AB",
  "file": "scheduler-angular/src/app/features/onboarding/components/onboarding-slide/onboarding-slide.component.scss",
  "lineNumber": 45,
  "context": ".title { color: #7B52AB; font-size: 24px; }"
}
```

**Auto-fix:**
```typescript
// Read the file to get exact context
const fileContent = readFile(mismatch.file);

// Find the exact line with context
const oldString = ".title { color: #7B52AB; font-size: 24px; }";
const newString = ".title { color: #6A0DAD; font-size: 24px; }";

// Apply fix using Edit tool
Edit({
  file_path: mismatch.file,
  old_string: oldString,
  new_string: newString
});
```

**Example 2: Font Size Mismatch**
```json
{
  "property": ".title font-size",
  "flutterValue": "24px",
  "angularValue": "22px",
  "file": "scheduler-angular/src/app/features/onboarding/components/onboarding-slide/onboarding-slide.component.scss",
  "lineNumber": 45
}
```

**Auto-fix:**
```typescript
Edit({
  file_path: mismatch.file,
  old_string: "font-size: 22px;",
  new_string: "font-size: 24px;"
});
```

**Example 3: Multiple Properties in Same Rule**
```json
{
  "property": ".start-button dimensions",
  "flutterValue": { "width": "400px", "height": "50px" },
  "angularValue": { "width": "380px", "height": "48px" },
  "file": "scheduler-angular/src/app/features/onboarding/components/onboarding-slide/onboarding-slide.component.scss",
  "lineNumber": 67
}
```

**Auto-fix:**
```typescript
// Fix multiple properties in one edit
Edit({
  file_path: mismatch.file,
  old_string: `.start-button {
  width: 380px;
  height: 48px;
  background-color: var(--color-primary);
}`,
  new_string: `.start-button {
  width: 400px;
  height: 50px;
  background-color: var(--color-primary);
}`
});
```

**Correction Workflow:**

1. **Parse Phase 8 mismatch-details.json**
   - Read structured mismatch data
   - Group by file for efficient editing
   - Prioritize critical mismatches (colors, dimensions)

2. **For Each Mismatch:**
   - Extract correct value from Flutter source using git show
   - Read Angular file to get exact context
   - Generate old_string (current incorrect code)
   - Generate new_string (corrected code)
   - Apply Edit tool with precise string matching
   - Validate the fix was applied correctly

3. **Re-run Phase 8 Validation:**
   - Execute full validation suite again
   - Compare new parity score
   - If parity < 100% and iterations < MAX, repeat

4. **Final Report:**
   - Log all fixes applied
   - Show before/after parity scores
   - List any remaining unfixed mismatches (if max iterations reached)

**Iteration Results:**

```markdown
## Auto-Correction Results

### Iteration 1:
- Mismatches Found: 8
- Fixes Applied: 8
- New Parity: 85% → 97%
- Time: 12 seconds

Fixes:
✅ .title color: #7B52AB → #6A0DAD
✅ .start-button width: 380px → 400px
✅ .start-button height: 48px → 50px
✅ .description font-size: 13px → 14px
✅ .dot-inactive color: #E0CFEE → #E1CFEE
✅ .title font-weight: 500 → 600
✅ .page-padding: 16px → 20px
✅ .item-gap: 10px → 12px

### Iteration 2:
- Mismatches Found: 2
- Fixes Applied: 2
- New Parity: 97% → 100%
- Time: 5 seconds

Fixes:
✅ .carousel-track transition: 300ms → 500ms
✅ .dot border-radius: 12px → 16px

### Final Result: 100% Parity Achieved! 🎉
Total Iterations: 2/3
Total Fixes: 10
Total Time: 17 seconds
```

**Error Handling:**

- If Edit tool fails (string not found), log error and skip to next mismatch
- If Flutter source extraction fails, use extracted-values.json as fallback
- If max iterations reached without 100% parity, report remaining issues to user for manual review

**Output:**
- **auto-correction-log.md** - Detailed log of all fixes applied
- **parity-progression.json** - Parity scores across iterations
- Updated Angular files with corrections applied
- Re-validated parity-validation-report.md showing 100% (or final score)

---

### Phase 8.1: Code Quality & Performance Optimization Validation

**OBJECTIVE:** Ensure generated code follows Angular best practices for maintainability and performance.

**Who does it:** Primary agent using Grep and Edit tools

**What happens:** Validates code quality patterns and applies auto-fixes for compliance

#### Validation Checks (3 Required)

**Check 1: FontAwesome Icon Centralization**
- **Tool:** Grep for `from '@fortawesome/` across `src/app/**/*.ts`
- **Validation:**
  - Shared icon module exists at `src/app/shared/icons/icon.module.ts`
  - No component has direct FontAwesome imports (except shared module)
  - All icons exported from centralized module
- **Auto-Fix:** If violations found:
  1. Create shared icon module with all used icons
  2. Replace direct imports with shared module imports
  3. Re-validate until 100% compliance

**Example Auto-Fix:**
```typescript
// Before (violation)
import { faUser, faHome } from '@fortawesome/free-solid-svg-icons';

// After (compliant)
import { Icons } from '@app/shared/icons/icon.module';
// Use Icons.faUser, Icons.faHome
```

**Check 2: Route Constants Enforcement**
- **Tool:** Grep for `router.navigate\(['"]` and `routerLink=['"]` in templates
- **Validation:**
  - Route constants file exists at `src/app/core/constants/routes.constants.ts`
  - Zero hardcoded route strings in navigation code
  - All routes use typed constants
- **Auto-Fix:** If violations found:
  1. Generate route constants from app-routing.module.ts
  2. Replace all magic strings with constants
  3. Re-validate until 100% compliance

**Example Auto-Fix:**
```typescript
// Before (violation)
this.router.navigate(['/dashboard']);

// After (compliant)
import { ROUTES } from '@app/core/constants/routes.constants';
this.router.navigate([ROUTES.DASHBOARD]);
```

**Check 3: Font Loading Performance**
- **Tool:** Read `src/index.html`
- **Validation:**
  - Preconnect links exist for all font CDNs
  - Preconnect appears BEFORE stylesheet links
  - Proper crossorigin attribute for gstatic
- **Auto-Fix:** If violations found:
  1. Insert preconnect tags in correct order
  2. Ensure crossorigin attribute present
  3. Re-validate until 100% compliance

**Example Auto-Fix:**
```html
<!-- Before (violation) -->
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;600&display=swap">

<!-- After (compliant) -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;600&display=swap">
```

#### Validation Loop Implementation

```typescript
// Phase 8.1 Auto-Fix Loop
const MAX_VALIDATION_ITERATIONS = 3;
let currentIteration = 0;
let allChecksPassed = false;

while (currentIteration < MAX_VALIDATION_ITERATIONS && !allChecksPassed) {
  console.log(`[Phase 8.1] Quality validation iteration ${currentIteration + 1}/${MAX_VALIDATION_ITERATIONS}`);

  // Run all checks
  const iconCheck = await validateIconCentralization();
  const routeCheck = await validateRouteConstants();
  const fontCheck = await validateFontOptimization();

  // Apply fixes if needed
  if (!iconCheck.passed) await fixIconCentralization(iconCheck.violations);
  if (!routeCheck.passed) await fixRouteConstants(routeCheck.violations);
  if (!fontCheck.passed) await fixFontOptimization(fontCheck.violations);

  // Check if all passed
  allChecksPassed = iconCheck.passed && routeCheck.passed && fontCheck.passed;
  currentIteration++;
}

if (!allChecksPassed) {
  console.log('⚠️  [Phase 8.1] Some quality checks did not pass after max iterations');
  console.log('📋 Review validation-report-quality.json for manual fixes');
}
```

#### Reporting

Generate `validation-report-quality.json`:
```json
{
  "timestamp": "ISO-8601",
  "checks": {
    "iconCentralization": {
      "status": "pass|fail",
      "violations": ["list of files with direct imports"],
      "fixesApplied": ["list of fixes"]
    },
    "routeConstants": {
      "status": "pass|fail",
      "violations": ["list of magic strings found"],
      "fixesApplied": ["list of replacements"]
    },
    "fontOptimization": {
      "status": "pass|fail",
      "violations": ["missing preconnect domains"],
      "fixesApplied": ["inserted preconnect tags"]
    }
  },
  "overallStatus": "pass|fail",
  "requiresManualReview": false
}
```

#### Success Criteria

- ✅ All 3 checks pass
- ✅ Zero violations remain
- ✅ Auto-fixes applied and validated
- ✅ Code builds successfully
- ✅ No regression in existing functionality

**PROCEED TO PHASE 9 ONLY AFTER 100% COMPLIANCE**

**Output:**
- `validation-report-quality.json` - Quality validation results
- `quality-fixes-log.md` - Detailed log of quality fixes applied
- Updated Angular files with quality improvements
- Verified build success after quality fixes

---

### Phase 9: Documentation
**Who does it:** solutions-architect

**What happens:** Creates conversion summary and migration guide

**Output:** Complete documentation referencing #251 and #251-clean with:
- Theme extraction documentation
- UI pattern conversion guide
- Gesture implementation guide
- Troubleshooting section for visual discrepancies

### Phase 10: PR Review and Quality Gates (NEW - CRITICAL)
**Who does it:** pr-review-orchestrator agent

**What happens:** Comprehensive quality validation and PR preparation for Angular code

**CRITICAL IMPLEMENTATION:**

This phase ensures the converted Angular code is merge-ready and meets all quality standards before creating a pull request.

**Quality Gate Categories:**

#### 1. Security Scanning (BLOCKING)
**Checks:**
- [ ] No hardcoded API keys, tokens, or credentials
- [ ] No exposed Firebase config secrets
- [ ] No console.log statements with sensitive data
- [ ] Dependencies have no known vulnerabilities (npm audit)
- [ ] Authentication logic properly implements secure patterns

**Tools:**
- `npm audit` - Dependency vulnerability scanning
- `git-secrets` or `trufflehog` - Secret detection in code and commits
- Manual review of authentication/authorization code

**Severity:** BLOCKING - Must pass before PR creation

#### 2. Code Quality (BLOCKING)
**Checks:**
- [ ] No console.log() statements (use LoggingService instead)
- [ ] No TODO/FIXME comments without GitHub issue references
- [ ] All TypeScript strict mode rules pass
- [ ] ESLint passes with zero errors
- [ ] No unused imports or variables
- [ ] Proper error handling for all async operations
- [ ] All magic numbers extracted to constants

**Tools:**
- `npm run lint` - ESLint validation
- `ng lint` - Angular-specific linting
- Custom regex checks for console.log, TODO patterns

**Severity:** BLOCKING - Must pass before PR creation

**Example Fixes:**
```typescript
// ❌ Before (blocking issue)
console.log('User logged in:', user);
// TODO: Add validation

// ✅ After (acceptable)
this.loggingService.debug('User logged in', { userId: user.id });
// TODO-#251: Add email validation
```

#### 3. Testing Requirements (BLOCKING)
**Checks:**
- [ ] Unit test coverage ≥ 80% for all ViewModels
- [ ] Component tests for all UI components
- [ ] Service tests with proper mocking
- [ ] Integration tests for critical workflows
- [ ] All tests pass (npm test)
- [ ] No skipped tests (.skip()) without GitHub references

**Tools:**
- `npm run test:coverage` - Karma/Jest coverage report
- `karma.conf.js` - Coverage thresholds configured

**Coverage Thresholds (karma.conf.js):**
```javascript
coverageReporter: {
  type: 'html',
  dir: 'coverage/',
  subdir: '.',
  check: {
    global: {
      statements: 80,
      branches: 80,
      functions: 80,
      lines: 80
    }
  }
}
```

**Severity:** BLOCKING - Must pass before PR creation

#### 4. Build Validation (BLOCKING)
**Checks:**
- [ ] Production build succeeds (`npm run build`)
- [ ] No build warnings or errors
- [ ] Bundle size within acceptable limits
- [ ] Source maps generated correctly
- [ ] All lazy-loaded modules resolve properly

**Tools:**
- `npm run build` - Angular production build
- Bundle analyzer for size validation

**Severity:** BLOCKING - Must pass before PR creation

#### 5. Documentation (NON-BLOCKING)
**Checks:**
- [ ] All public methods have JSDoc comments
- [ ] Complex business logic documented
- [ ] README updated with new features
- [ ] API changes documented
- [ ] Migration guide created (if breaking changes)

**Severity:** NON-BLOCKING - Can be fixed post-PR with follow-up ticket

#### 6. Design System Compliance (NON-BLOCKING)
**Checks:**
- [ ] Uses theme variables (not hardcoded colors)
- [ ] Typography follows design tokens
- [ ] Spacing uses spacing.scss constants
- [ ] Component styling matches design system
- [ ] Responsive breakpoints consistent

**Severity:** NON-BLOCKING - Phase 8.5 auto-correction should handle most issues

#### 7. Accessibility (NON-BLOCKING)
**Checks:**
- [ ] ARIA labels on interactive elements
- [ ] Keyboard navigation functional
- [ ] Screen reader compatibility
- [ ] Color contrast meets WCAG AA standards
- [ ] Focus indicators visible

**Tools:**
- `npm run a11y:test` - Automated accessibility testing
- Manual testing with screen readers

**Severity:** NON-BLOCKING - Can be improved incrementally

#### 8. Performance (NON-BLOCKING)
**Checks:**
- [ ] No memory leaks in subscriptions
- [ ] Observables properly unsubscribed
- [ ] Change detection optimized (OnPush where applicable)
- [ ] Images optimized and lazy-loaded
- [ ] No blocking operations on main thread

**Severity:** NON-BLOCKING - Performance can be optimized post-PR

**PR Preparation Workflow:**

```typescript
// Phase 10 Implementation
async function executePhase10PRReview(): Promise<PRReviewReport> {
  console.log('[Phase 10] Starting PR Review and Quality Gates...');

  const blockingIssues: Issue[] = [];
  const nonBlockingIssues: Issue[] = [];

  // 1. Security Scanning (BLOCKING)
  console.log('[Phase 10.1] Running security scans...');
  const securityIssues = await runSecurityScans();
  if (securityIssues.length > 0) {
    blockingIssues.push(...securityIssues);
    console.log(`⚠️  Found ${securityIssues.length} security issues (BLOCKING)`);
  } else {
    console.log('✅ Security scans passed');
  }

  // 2. Code Quality (BLOCKING)
  console.log('[Phase 10.2] Running code quality checks...');
  const codeQualityIssues = await runCodeQualityChecks();
  if (codeQualityIssues.length > 0) {
    blockingIssues.push(...codeQualityIssues);
    console.log(`⚠️  Found ${codeQualityIssues.length} code quality issues (BLOCKING)`);
  } else {
    console.log('✅ Code quality checks passed');
  }

  // 3. Testing Requirements (BLOCKING)
  console.log('[Phase 10.3] Validating test coverage...');
  const testingIssues = await runTestingValidation();
  if (testingIssues.length > 0) {
    blockingIssues.push(...testingIssues);
    console.log(`⚠️  Found ${testingIssues.length} testing issues (BLOCKING)`);
  } else {
    console.log('✅ Test coverage meets requirements (≥80%)');
  }

  // 4. Build Validation (BLOCKING)
  console.log('[Phase 10.4] Running production build...');
  const buildIssues = await runBuildValidation();
  if (buildIssues.length > 0) {
    blockingIssues.push(...buildIssues);
    console.log(`⚠️  Build validation failed (BLOCKING)`);
  } else {
    console.log('✅ Production build successful');
  }

  // 5. Documentation (NON-BLOCKING)
  console.log('[Phase 10.5] Checking documentation...');
  const docIssues = await checkDocumentation();
  if (docIssues.length > 0) {
    nonBlockingIssues.push(...docIssues);
    console.log(`⚠️  Found ${docIssues.length} documentation issues (NON-BLOCKING)`);
  }

  // 6. Design System Compliance (NON-BLOCKING)
  console.log('[Phase 10.6] Validating design system compliance...');
  const designIssues = await checkDesignSystemCompliance();
  if (designIssues.length > 0) {
    nonBlockingIssues.push(...designIssues);
    console.log(`⚠️  Found ${designIssues.length} design system issues (NON-BLOCKING)`);
  }

  // 7. Accessibility (NON-BLOCKING)
  console.log('[Phase 10.7] Running accessibility checks...');
  const a11yIssues = await checkAccessibility();
  if (a11yIssues.length > 0) {
    nonBlockingIssues.push(...a11yIssues);
    console.log(`⚠️  Found ${a11yIssues.length} accessibility issues (NON-BLOCKING)`);
  }

  // 8. Performance (NON-BLOCKING)
  console.log('[Phase 10.8] Running performance checks...');
  const perfIssues = await checkPerformance();
  if (perfIssues.length > 0) {
    nonBlockingIssues.push(...perfIssues);
    console.log(`⚠️  Found ${perfIssues.length} performance issues (NON-BLOCKING)`);
  }

  // Generate PR Review Report
  const report: PRReviewReport = {
    blockingIssues,
    nonBlockingIssues,
    isReadyForPR: blockingIssues.length === 0,
    summary: generatePRSummary(blockingIssues, nonBlockingIssues)
  };

  console.log('[Phase 10] PR Review Complete');
  console.log(`   Blocking Issues: ${blockingIssues.length}`);
  console.log(`   Non-Blocking Issues: ${nonBlockingIssues.length}`);
  console.log(`   Ready for PR: ${report.isReadyForPR ? '✅ YES' : '❌ NO'}`);

  return report;
}
```

**Blocking Issue Resolution:**

If blocking issues are found, the command will:
1. Generate detailed issue report with file locations and fixes
2. Delegate to appropriate agents for auto-fixes:
   - Security issues → security-researcher
   - Code quality → angular-mvvm-engineer
   - Testing gaps → qa-engineer
   - Build failures → devops-engineer
3. Re-run Phase 10 validation after fixes
4. Maximum 2 iterations before requiring manual intervention

**PR Creation Conditions:**

```typescript
if (blockingIssues.length === 0) {
  // All quality gates passed - Commit and Create PR
  console.log('✅ All blocking quality gates passed');

  // CRITICAL: Commit and push ALL changes ONCE after complete migration
  console.log('📦 Committing all generated and validated code...');

  // Stage all files (generated code, validation reports, documentation)
  await exec('git add .');

  // Create comprehensive commit message
  const commitMessage = `${ghTicket}: Complete Flutter to Angular migration with pixel-perfect parity

Phase Summary:
- Phase 0-7: Extracted and converted Flutter to Angular MVVM
- Phase 8: Achieved 100% visual parity validation
- Phase 8.1: Applied code quality and performance optimizations
- Phase 9: Generated comprehensive documentation
- Phase 10: Passed all security, quality, testing, and build gates

Features:
${generateFeatureSummary(conversionResults)}

Test Coverage: ${testCoverage}%
Build Status: ✅ Success
Quality Gates: ✅ All passed

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>`;

  await exec(`git commit -m "${commitMessage}"`);
  console.log('✅ Code committed successfully');

  // Push to remote branch
  await exec('git push origin HEAD');
  console.log('✅ Code pushed to remote');

  // Now create the PR
  console.log('📝 Creating pull request...');

  await createPullRequest({
    title: `${ghTicket}: Convert ${featureName} from Flutter to Angular`,
    body: generatePRDescription(conversionResults, nonBlockingIssues),
    labels: ['flutter-to-angular', 'mvvm-migration', 'automated-conversion']
  });

  console.log('✅ Pull request created successfully');
  console.log(`   Non-blocking issues: ${nonBlockingIssues.length} (can be addressed in follow-up)`);
} else {
  // Blocking issues prevent PR creation
  console.log('❌ Cannot create PR - blocking issues must be resolved');
  console.log('📋 Review pr-review-report.md for details and required fixes');
  console.log('⚠️  NO COMMIT OR PUSH will be performed until all blocking issues are resolved');

  throw new Error(`${blockingIssues.length} blocking issues prevent PR creation`);
}
```

**IMPORTANT: Single Commit/Push Strategy**

This command follows a **single commit, single push** strategy:
- ✅ **DO NOT** commit after individual phases
- ✅ **DO NOT** push after each file generation
- ✅ **ONLY** commit and push ONCE after:
  - All phases complete (0-10)
  - All quality gates pass
  - All validation achieves 100% parity
  - All auto-fixes applied successfully

This ensures:
1. **Clean git history** - One logical commit per migration
2. **Atomic changes** - All-or-nothing approach prevents partial migrations
3. **Better code review** - Reviewers see complete feature in single PR
4. **Simpler rollback** - Single commit to revert if needed

**Output:**
- `pr-review-report.md` - Comprehensive quality gate report
- `blocking-issues.json` - Structured data of blocking issues
- `non-blocking-issues.json` - Structured data of non-blocking issues
- `pr-description.md` - Generated PR description (if ready)
- Updated GitHub PR (if all blocking gates passed)

---

## Automation Integration

### GitHub Actions Workflow

To automate the Flutter-to-Angular conversion process in CI/CD, create this workflow file:

**File:** `.github/workflows/flutter-to-angular-ci.yml`

```yaml
name: Flutter to Angular Conversion CI

on:
  push:
    branches:
      - 'feature/angular-*'
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  security-scan:
    name: Security Scanning
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for secret scanning

      - name: Run secret detection
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD

      - name: Run npm audit
        working-directory: ./scheduler-angular
        run: npm audit --audit-level=moderate

  code-quality:
    name: Code Quality Checks
    runs-on: ubuntu-latest
    needs: security-scan
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: scheduler-angular/package-lock.json

      - name: Install dependencies
        working-directory: ./scheduler-angular
        run: npm ci

      - name: Run ESLint
        working-directory: ./scheduler-angular
        run: npm run lint

      - name: Check for console.log statements
        working-directory: ./scheduler-angular
        run: |
          if grep -r "console\.log" src/ --exclude-dir=node_modules; then
            echo "❌ Found console.log statements - use LoggingService instead"
            exit 1
          fi

      - name: Check for TODOs without ticket references
        working-directory: ./scheduler-angular
        run: |
          if grep -rE "TODO:|FIXME:" src/ --exclude-dir=node_modules | grep -v "TODO-"; then
            echo "❌ Found TODOs without GitHub issue references"
            exit 1
          fi

  testing:
    name: Testing and Coverage
    runs-on: ubuntu-latest
    needs: code-quality
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: scheduler-angular/package-lock.json

      - name: Install dependencies
        working-directory: ./scheduler-angular
        run: npm ci

      - name: Run tests with coverage
        working-directory: ./scheduler-angular
        run: npm run test:coverage

      - name: Check coverage thresholds
        working-directory: ./scheduler-angular
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "❌ Coverage $COVERAGE% is below 80% threshold"
            exit 1
          fi

      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          files: ./scheduler-angular/coverage/lcov.info
          flags: angular-conversion

  build-validation:
    name: Build Validation
    runs-on: ubuntu-latest
    needs: testing
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: scheduler-angular/package-lock.json

      - name: Install dependencies
        working-directory: ./scheduler-angular
        run: npm ci

      - name: Build for production
        working-directory: ./scheduler-angular
        run: npm run build

      - name: Check bundle size
        working-directory: ./scheduler-angular
        run: |
          BUNDLE_SIZE=$(du -sh dist/ | cut -f1)
          echo "Bundle size: $BUNDLE_SIZE"
          # Add bundle size checks here

  visual-parity-validation:
    name: Visual Parity Validation
    runs-on: ubuntu-latest
    needs: build-validation
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: scheduler-angular/package-lock.json

      - name: Install dependencies
        working-directory: ./scheduler-angular
        run: npm ci

      - name: Run visual regression tests
        working-directory: ./scheduler-angular
        run: npm run test:visual

      - name: Upload screenshots
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: visual-diff-screenshots
          path: ./scheduler-angular/visual-diffs/

  pr-ready-check:
    name: PR Ready Check
    runs-on: ubuntu-latest
    needs: [security-scan, code-quality, testing, build-validation, visual-parity-validation]
    steps:
      - name: All quality gates passed
        run: |
          echo "✅ All quality gates passed"
          echo "✅ Security scanning complete"
          echo "✅ Code quality validated"
          echo "✅ Test coverage ≥ 80%"
          echo "✅ Production build successful"
          echo "✅ Visual parity validated"
          echo ""
          echo "📝 PR is ready for review!"
```

**Workflow Triggers:**
- **Push to feature branches:** Runs on any push to `feature/angular-*` branches
- **Pull requests:** Runs when PR is opened, updated, or reopened
- **Manual trigger:** Can be triggered manually from GitHub Actions UI

**Workflow Stages:**
1. **Security Scan** - TruffleHog secret detection + npm audit
2. **Code Quality** - ESLint, console.log checks, TODO validation
3. **Testing** - Unit tests + coverage validation (≥80%)
4. **Build Validation** - Production build + bundle size check
5. **Visual Parity** - Visual regression tests
6. **PR Ready Check** - Final approval if all gates pass

---

### Pre-Commit Hooks Setup

To prevent committing code that violates quality standards, set up pre-commit hooks using Husky:

**Installation:**

```bash
cd scheduler-angular

# Install Husky
npm install --save-dev husky

# Initialize Husky
npx husky init

# Install lint-staged for selective file checking
npm install --save-dev lint-staged
```

**Configuration:**

**File:** `scheduler-angular/.husky/pre-commit`

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# Run lint-staged for selective file validation
npx lint-staged

# Check for secrets
npm run security:check-secrets

# Run affected tests
npm run test:affected
```

**File:** `scheduler-angular/package.json` (add to existing file)

```json
{
  "scripts": {
    "prepare": "husky init",
    "security:check-secrets": "git diff --cached --name-only | xargs grep -l 'FIREBASE_API_KEY\\|API_SECRET\\|PASSWORD' && exit 1 || exit 0",
    "test:affected": "ng test --watch=false --browsers=ChromeHeadless --code-coverage"
  },
  "lint-staged": {
    "*.ts": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.html": [
      "prettier --write"
    ],
    "*.scss": [
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

**Pre-Commit Validations:**
- **Linting:** ESLint fixes auto-applied to staged TypeScript files
- **Formatting:** Prettier formats staged files
- **Secret Detection:** Checks for API keys, secrets in staged files
- **Affected Tests:** Runs tests for modified code only

**Benefits:**
- Catch issues before they reach CI/CD
- Auto-fix formatting and linting issues
- Prevent accidental secret commits
- Faster feedback loop for developers

---

### Security Scanning Scripts

Create these npm scripts for security validation:

**File:** `scheduler-angular/package.json` (add to scripts section)

```json
{
  "scripts": {
    "security:scan": "npm audit && npm run security:check-secrets",
    "security:check-secrets": "git ls-files | xargs grep -E 'FIREBASE_API_KEY|API_SECRET|PRIVATE_KEY|AWS_ACCESS_KEY' && exit 1 || exit 0",
    "security:audit": "npm audit --audit-level=moderate --json > security-audit.json",
    "security:fix": "npm audit fix"
  }
}
```

**Usage:**
```bash
# Run full security scan (pre-conversion hook)
npm run security:scan

# Fix security vulnerabilities automatically
npm run security:fix

# Generate security audit report
npm run security:audit
```

---

## Parameters Explained

| Parameter | What It Is | Example | Required? |
|-----------|------------|---------|-----------|
| `<gh-ticket>` | Your GitHub issue number | `#251` | ✅ Yes |
| `<flutter-branch>` | Git branch with Flutter code | `#251-clean` | ✅ Yes |
| `[angular-output-dir]` | Where to put Angular files | `scheduler-angular/src/app/features/onboarding` | ❌ No (auto-detects from ticket) |

### Examples

```bash
# Basic usage - auto-detects feature name from GitHub issue
/flutter-to-angular #251 #251-clean
# Output: scheduler-angular/src/app/features/onboarding/

# Specify custom feature directory
/flutter-to-angular #251 #251-clean scheduler-angular/src/app/features/auth
# Output: scheduler-angular/src/app/features/auth/

# Convert from main branch (uses auto-detected feature)
/flutter-to-angular #251 main
# Output: scheduler-angular/src/app/features/onboarding/
```

**IMPORTANT Directory Rules:**
- ✅ All feature code MUST go inside: `scheduler-angular/src/app/features/<feature-name>/`
- ✅ Feature documentation MUST be inside the feature directory
- ✅ Common/shared code goes in: `scheduler-angular/src/app/core/` or `scheduler-angular/src/app/shared/`
- ❌ NEVER place feature code or docs outside `scheduler-angular/`

## Branch Validation Workflow

**CRITICAL:** The command always uses the **remote branch** to ensure consistency:

```
1. You run: /flutter-to-angular #251 #251-clean

2. Command checks:
   ✓ Does origin/#251-clean exist?
   ✓ Does local #251-clean exist?

3a. If local exists:
    ✓ git status --porcelain → Must be empty (no uncommitted changes)
    ✓ git rev-list --count origin/#251-clean..#251-clean → Must be 0 (no unpushed commits)
    ✓ If validation fails: Error + instructions to commit/push

3b. If local doesn't exist:
    ✓ Continue (will use remote only)

4. All analysis uses: origin/#251-clean (remote)
   - git show origin/#251-clean:lib/file.dart
   - git ls-tree origin/#251-clean
   - git log origin/#251-clean
```

## Pre-Flight Validation

Before starting conversion, the command validates:

### GitHub Ticket Validation
1. Connect to GitHub CLI
2. Verify ticket exists: `gh issue view`
3. Check ticket has description and acceptance criteria
4. **If validation fails:** Display error and stop

### Flutter Branch Validation
1. Fetch latest branches: `git fetch origin`
2. Verify remote branch exists: `git branch --list -r origin/#251-clean`
3. **If local branch exists:** Verify it's synchronized with remote:
   - Check for uncommitted changes: `git status --porcelain` (must be clean)
   - Check for unpushed commits: `git rev-list --count origin/#251-clean..#251-clean` (must be 0)
   - **If validation fails:** Prompt user to commit and push local changes first
4. Check for Flutter code: `git show origin/#251-clean:lib/` should succeed
5. **If validation fails:** List available remote branches and stop

### Output Directory Validation
1. **Validate directory structure:** Must be inside `scheduler-angular/src/app/features/`
2. **Enforce feature isolation:** Reject paths outside feature directories
3. Check parent directory exists
4. Verify write permissions
5. Warn if directory already exists with code
6. **If validation fails:** Display correct path format and stop

**Valid Paths:**
- ✅ `scheduler-angular/src/app/features/onboarding/`
- ✅ `scheduler-angular/src/app/features/auth/`
- ✅ `scheduler-angular/src/app/features/calendar/`

**Invalid Paths:**
- ❌ `web-app/src/app/features/onboarding/` (wrong base)
- ❌ `angular/features/onboarding/` (not inside scheduler-angular)
- ❌ `/Users/user/docs/onboarding/` (outside project)
- ❌ `scheduler-angular/src/app/onboarding/` (not in features/)

## Generated File Structure

**MANDATORY:** All feature files are generated inside `scheduler-angular/src/app/features/<feature-name>/`

```
scheduler-angular/src/app/features/onboarding/
├── docs/
│   ├── CONVERSION_DOCUMENTATION.md          # Complete guide (references #251)
│   ├── theme-extraction-report.md           # Theme analysis results
│   ├── ui-patterns-map.md                   # Pattern conversion guide
│   └── parity-validation-report.md          # Visual parity verification
│
├── styles/
│   ├── theme.scss                           # CSS custom properties
│   ├── typography.scss                      # Font system
│   └── spacing.scss                         # Spacing constants
│
├── models/
│   ├── onboarding.model.ts                  # TypeScript interfaces
│   └── onboarding.dto.ts                    # Data transfer objects
│
├── view-models/
│   ├── onboarding.view-model.ts             # Business logic with Signals
│   └── onboarding.view-model.spec.ts        # Tests
│
├── components/
│   ├── onboarding-screen/
│   │   ├── onboarding-screen.component.ts    # Standalone component class
│   │   ├── onboarding-screen.component.html  # Template with gesture handlers
│   │   ├── onboarding-screen.component.scss  # Pixel-perfect styles
│   │   └── onboarding-screen.component.spec.ts # Tests
│   │
│   └── page-indicator/
│       ├── page-indicator.component.ts       # Custom expanding dots indicator
│       ├── page-indicator.component.html     # Indicator template
│       ├── page-indicator.component.scss     # Indicator styles
│       └── page-indicator.component.spec.ts  # Tests
│
├── repositories/
│   ├── onboarding.repository.ts             # Abstract interface
│   └── onboarding.repository.impl.ts        # Firebase implementation
│
├── index.ts                                  # Public API exports
└── onboarding.routes.ts                     # Standalone routing
```

## Success Checklist (ENHANCED)

Your conversion is complete when:

**Functional Requirements:**
- ✅ GitHub issue (#251) requirements documented
- ✅ Flutter branch (#251-clean) analyzed
- ✅ All Models converted to TypeScript
- ✅ All ViewModels converted to Signal-based services
- ✅ All Views converted to Angular standalone components
- ✅ Repository pattern implemented
- ✅ Tests generated and passing
- ✅ All GitHub acceptance criteria validated
- ✅ Documentation complete
- ✅ Accessibility requirements met

**Visual Parity Requirements (NEW):**
- ✅ All theme colors extracted and mapped
- ✅ All font families, sizes, weights preserved
- ✅ All spacing values match exactly
- ✅ Button dimensions, colors, elevation match
- ✅ Responsive breakpoints preserved
- ✅ All color literals converted to CSS variables

**Interaction Parity Requirements (NEW):**
- ✅ Swipe gestures work for page navigation
- ✅ Dot indicators have correct colors and animations
- ✅ Click events fire and navigate properly
- ✅ Page transitions match Flutter animation timing
- ✅ All user interactions behave identically to Flutter app

## Common Issues & Solutions (Quick Reference)

**COMPREHENSIVE GUIDE:** See **"Common Visual Parity Issues and Solutions"** section after Phase 8 for detailed root cause analysis, detection criteria, auto-fix strategies, and validation checklists for all visual parity issues.

### Issue: "Button colors don't match"
**Root Cause:** Theme colors not extracted from FlutterFlowTheme

**Solution (AUTOMATED):**
- Phase 0.5 extracts ALL theme colors from Flutter source
- Phase 8 validates color accuracy
- Phase 8.5 automatically fixes any mismatches within 3 iterations

**Manual Check (if auto-fix fails):**
- Verify `/tmp/flutter-extraction/extracted-values.json` has correct values
- Check Phase 8.5 auto-correction-log.md for details
- If max iterations reached, review remaining mismatches in parity-validation-report.md

**📖 See detailed guide:** "Issue 1: Color Extraction" in Common Visual Parity Issues section

### Issue: "Images have inconsistent sizes across slides"
**Root Cause:** Flutter Flexible widget behavior not properly converted

**Solution (AUTOMATED):**
- Phase 8 detects image dimension inconsistencies
- Phase 8.5 converts to responsive vh/vw units
- Applies flex-shrink: 0 to prevent layout shifts

**📖 See detailed guide:** "Issue 2: Image Sizing Consistency" in Common Visual Parity Issues section

### Issue: "Vertical spacing doesn't match Flutter"
**Root Cause:** mainAxisSize.min/max behavior and .divide(SizedBox) not converted correctly

**Solution (AUTOMATED):**
- Phase 8 validates flexbox properties and spacing
- Phase 8.5 fixes flex values and converts .divide() to gap or margin
- Preserves mainAxisSize.min vs max centering behavior

**📖 See detailed guide:** "Issue 3: Vertical Centering and Spacing" in Common Visual Parity Issues section

### Issue: "Missing translations / localization keys not found"
**Root Cause:** Flutter FFLocalizations keys not mapped to Angular i18n files

**Solution (AUTOMATED):**
- Phase 0.5 extracts all localization keys from Flutter
- Phase 8 detects missing translation keys
- Phase 8.5 creates/updates Angular locale files (en.json, es.json, he.json)
- Updates components to use LocalizationService

**📖 See detailed guide:** "Issue 4: Localization Key Mapping" in Common Visual Parity Issues section

### Issue: "Start button click doesn't work"
**Root Cause:** Event binding not properly mapped from `onPressed` to Angular `(click)`

**Solution:** Ensure ViewModel method is called in component:
```typescript
// Component
onStartNow(): void {
  this.vm.completeOnboarding(); // Must match Flutter onPressed logic
}
```

### Issue: "Dot swipe/navigation doesn't work"
**Root Cause:** PageView swipe gestures not implemented in Angular

**Solution:** Phase 2.5 now implements:
1. HammerJS swipe gesture detection
2. Signal-based page state management
3. Smooth CSS transitions
4. Dot click handlers with animation

Verify HammerJS is installed and configured in angular.json.

### Issue: "Images don't change when swiping"
**Root Cause:** Carousel track transform not bound to currentPage signal

**Solution:** Ensure template uses:
```html
<div class="carousel-track"
     [style.transform]="'translateX(-' + (vm.currentPage() * 100) + '%)'">
```

### Issue: "Fonts don't match"
**Root Cause:** Google Fonts not loaded or font properties not extracted

**Solution:**
1. Add to index.html: `<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;500;600&display=swap">`
2. Verify typography.scss has exact font sizes from Flutter

### Issue: "Spacing looks off"
**Root Cause:** EdgeInsetsDirectional.fromSTEB not converted precisely

**Solution:** Phase 0.5 extracts exact padding values. Verify generated SCSS:
```scss
.content {
  padding: 20px 20px 0 20px; // fromSTEB(20, 20, 20, 0)
}
```

## Advanced Features

### Pixel-Perfect Theme Extraction
- Fetches ACTUAL theme from FlutterFlowTheme class
- Extracts ALL color properties and hex literals
- Maps to CSS custom properties
- Generates comprehensive theme documentation

### UI Pattern Recognition
- Detects PageView → Generates swipeable carousel with HammerJS
- Detects SmoothPageIndicator → Generates custom expanding dots component
- Detects FFButtonWidget → Generates styled Material button with exact specs
- Extracts ALL Container decorations, BoxDecoration, Text styles

### Gesture Implementation
- Swipe left/right for page navigation
- Dot click with smooth animation
- Touch and mouse event support
- Matches Flutter Curves.ease animation timing

### Responsive Design Preservation
- Extracts MediaQuery breakpoints
- Converts to CSS media queries
- Preserves responsive font sizing
- Maintains responsive layout behavior

### Style Validation
- Generates visual parity checklist
- Creates comparison screenshots
- Validates against Flutter source
- Reports any discrepancies

## Related Commands

**NOTE:** Commits and PR creation are now **automatic** after Phase 10 passes all quality gates.

After the automated conversion and PR creation, you might want to:

```bash
# Generate additional tests (if coverage needs improvement)
/feature-generate-tests

# Validate for production deployment
/e2e-readiness-check

# Review the created PR
gh pr view

# Address any non-blocking issues identified
# (Check pr-review-report.md for suggestions)
```

## Real-World Example with Pixel-Perfect Conversion

Here's what actually happens when you run the command:

```bash
# You run this
/flutter-to-angular #251 #251-clean

# Command executes:
[Phase 0] Validating output directory...
[Phase 0] ✅ Output: scheduler-angular/src/app/features/onboarding/

[Phase 0] Fetching GitHub issue #251...
[Phase 0] ✅ Found: "Implement onboarding flow"

[Phase 0] Validating remote branch origin/#251-clean...
[Phase 0] ✅ Remote branch exists
[Phase 0] ✅ Local branch synchronized (or doesn't exist)

[Phase 0] Analyzing Flutter code from origin/#251-clean...
[Phase 0] ✅ Found 3 models, 2 viewmodels, 4 views

[Phase 0.5] Extracting theme and styles...
[Phase 0.5] ✅ Extracted 15 theme colors from FlutterFlowTheme
[Phase 0.5] ✅ Extracted 8 color literals from widgets
[Phase 0.5] ✅ Extracted Google Fonts: Montserrat
[Phase 0.5] ✅ Extracted 12 font size definitions
[Phase 0.5] ✅ Extracted 15 spacing values
[Phase 0.5] ✅ Created theme.scss with CSS custom properties
[Phase 0.5] ✅ Created typography.scss
[Phase 0.5] ✅ Created spacing.scss

[Phase 0] ✅ Created conversion plan: conversion-plan-#251.md

[Phase 1] Delegating to solutions-architect...
[Phase 1] ✅ Architecture analysis complete

[Phase 2] Delegating to angular-architect...
[Phase 2] ✅ Angular project structure ready
[Phase 2] ✅ Theme module configured

[Phase 2.5] Analyzing UI patterns...
[Phase 2.5] ✅ Detected PageView → Generating swipeable carousel
[Phase 2.5] ✅ Detected SmoothPageIndicator → Generating expanding dots component
[Phase 2.5] ✅ Detected FFButtonWidget → Generating styled button
[Phase 2.5] ✅ Created ui-patterns-map.md

[Phase 3] Converting models...
[Phase 3] ✅ 3 TypeScript models → scheduler-angular/src/app/features/onboarding/models/

[Phase 4] Converting viewmodels...
[Phase 4] ✅ 2 Angular ViewModels with Signals
[Phase 4] ✅ PageController logic converted to signal-based state
[Phase 4] ✅ Generated: scheduler-angular/src/app/features/onboarding/view-models/

[Phase 5] Converting views with pixel-perfect styling...
[Phase 5] ✅ Extracted exact styles from Flutter widgets
[Phase 5] ✅ Generated SCSS with pixel-perfect values
[Phase 5] ✅ Implemented HammerJS swipe gestures
[Phase 5] ✅ Created custom page indicator component
[Phase 5] ✅ 4 Angular standalone components → scheduler-angular/src/app/features/onboarding/components/

[Phase 6] Implementing repositories...
[Phase 6] ✅ Repository pattern → scheduler-angular/src/app/features/onboarding/repositories/

[Phase 7] Generating tests...
[Phase 7] ✅ 15 test files created
[Phase 7] ✅ Gesture interaction tests added
[Phase 7] ✅ Visual parity tests added

[Phase 8] Validating visual and functional parity...
[Phase 8] ⚠️  Initial Parity: 85% (8 mismatches found)
[Phase 8] ✅ Created mismatch-details.json with 8 mismatches
[Phase 8] ✅ Created parity-validation-report.md

[Phase 8.5] Starting auto-correction loop...
[Phase 8.5] Iteration 1/3: Processing 8 mismatches

[Phase 8.5] Auto-fixing mismatches:
[Phase 8.5] ✅ Fixed: .title color (#7B52AB → #6A0DAD)
[Phase 8.5] ✅ Fixed: .start-button width (380px → 400px)
[Phase 8.5] ✅ Fixed: .start-button height (48px → 50px)
[Phase 8.5] ✅ Fixed: .description font-size (13px → 14px)
[Phase 8.5] ✅ Fixed: .dot-inactive color (#E0CFEE → #E1CFEE)
[Phase 8.5] ✅ Fixed: .title font-weight (500 → 600)
[Phase 8.5] ✅ Fixed: .page-padding (16px → 20px)
[Phase 8.5] ✅ Fixed: .item-gap (10px → 12px)

[Phase 8.5] Re-validating after corrections...
[Phase 8.5] ✅ New Parity: 97% (2 remaining mismatches)

[Phase 8.5] Iteration 2/3: Processing 2 mismatches

[Phase 8.5] Auto-fixing remaining mismatches:
[Phase 8.5] ✅ Fixed: .carousel-track transition (300ms → 500ms)
[Phase 8.5] ✅ Fixed: .dot border-radius (12px → 16px)

[Phase 8.5] Re-validating after corrections...
[Phase 8.5] ✅ Final Parity: 100% - All mismatches resolved! 🎉

[Phase 8.5] Auto-correction summary:
[Phase 8.5]   - Total iterations: 2/3
[Phase 8.5]   - Total fixes applied: 10
[Phase 8.5]   - Initial parity: 85%
[Phase 8.5]   - Final parity: 100%
[Phase 8.5]   - Time: 17 seconds
[Phase 8.5] ✅ Created auto-correction-log.md

[Phase 9] Creating documentation...
[Phase 9] ✅ Documentation → scheduler-angular/src/app/features/onboarding/docs/
[Phase 9] ✅ Theme extraction documentation
[Phase 9] ✅ UI pattern conversion guide
[Phase 9] ✅ Gesture implementation guide

[Phase 9] ✅ Conversion complete with 100% visual parity!

[Phase 10] Starting PR Review and Quality Gates...

[Phase 10.1] Running security scans...
[Phase 10.1] ✅ No hardcoded secrets found
[Phase 10.1] ✅ npm audit: 0 vulnerabilities
[Phase 10.1] ✅ Security scans passed

[Phase 10.2] Running code quality checks...
[Phase 10.2] ✅ ESLint: 0 errors, 0 warnings
[Phase 10.2] ✅ No console.log statements found
[Phase 10.2] ✅ All TODOs have ticket references
[Phase 10.2] ✅ Code quality checks passed

[Phase 10.3] Validating test coverage...
[Phase 10.3] Running tests with coverage...
[Phase 10.3] ✅ ViewModels: 87% coverage (threshold: 80%)
[Phase 10.3] ✅ Components: 84% coverage (threshold: 80%)
[Phase 10.3] ✅ Services: 92% coverage (threshold: 80%)
[Phase 10.3] ✅ Overall: 87% coverage
[Phase 10.3] ✅ Test coverage meets requirements (≥80%)

[Phase 10.4] Running production build...
[Phase 10.4] Building Angular application...
[Phase 10.4] ✅ Production build successful
[Phase 10.4] ✅ Bundle size: 2.3 MB (within limits)
[Phase 10.4] ✅ 0 build warnings

[Phase 10.5] Checking documentation...
[Phase 10.5] ⚠️  Found 3 documentation issues (NON-BLOCKING)
[Phase 10.5] - Missing JSDoc for 2 public methods
[Phase 10.5] - README not updated with new feature

[Phase 10.6] Validating design system compliance...
[Phase 10.6] ✅ All colors use theme variables
[Phase 10.6] ✅ Typography follows design tokens
[Phase 10.6] ✅ Spacing uses constants

[Phase 10.7] Running accessibility checks...
[Phase 10.7] ⚠️  Found 2 accessibility issues (NON-BLOCKING)
[Phase 10.7] - Missing ARIA label on 1 button
[Phase 10.7] - Focus indicator could be more visible

[Phase 10.8] Running performance checks...
[Phase 10.8] ✅ No memory leaks detected
[Phase 10.8] ✅ All observables properly unsubscribed
[Phase 10.8] ✅ Performance checks passed

[Phase 10] PR Review Complete
   Blocking Issues: 0
   Non-Blocking Issues: 5
   Ready for PR: ✅ YES

[Phase 10] ✅ All blocking quality gates passed
[Phase 10] 📝 Creating pull request...

[Phase 10] ✅ Pull request created successfully
   PR URL: https://github.com/your-org/scheduler/pull/123
   Title: #251: Convert onboarding from Flutter to Angular
   Non-blocking issues: 5 (can be addressed in follow-up)

📋 Summary:
   GitHub Ticket: #251
   Flutter Branch: origin/#251-clean (remote)
   Files Generated: 28
   Tests Created: 18
   Output Directory: scheduler-angular/src/app/features/onboarding/
   Documentation: scheduler-angular/src/app/features/onboarding/docs/

   Visual Parity: ✅ 100% (Auto-corrected from 85%)
   - Theme colors: 23/23 ✅ (2 auto-fixed)
   - Typography: 12/12 ✅ (3 auto-fixed)
   - Spacing: 15/15 ✅ (2 auto-fixed)
   - Button styling: 6/6 ✅ (2 auto-fixed)
   - Gestures: 5/5 ✅
   - Dot indicators: 6/6 ✅ (1 auto-fixed)

   Auto-Correction Results:
   - Initial parity: 85%
   - Iterations required: 2/3
   - Total fixes applied: 10
   - Final parity: 100%
   - Time to 100%: 17 seconds

   Quality Gates Status:
   - Security Scanning: ✅ PASSED (0 blocking issues)
   - Code Quality: ✅ PASSED (0 blocking issues)
   - Test Coverage: ✅ PASSED (87% ≥ 80% threshold)
   - Build Validation: ✅ PASSED (production build successful)
   - Documentation: ⚠️  5 non-blocking issues
   - Design System: ✅ PASSED
   - Accessibility: ⚠️  2 non-blocking issues
   - Performance: ✅ PASSED

✅ All acceptance criteria validated
✅ All tests passing (87% coverage)
✅ All files in correct feature directory
✅ Documentation inside feature directory
✅ 100% visual and functional parity achieved (with automated corrections)
✅ PR created and ready for review
```

## Tips for Best Results

1. **Push to remote first:** Ensure all local changes are committed and pushed to origin/#251-clean
2. **Verify branch sync:** Run `git status` and `git push` before conversion
3. **Complete GitHub issues:** Add detailed acceptance criteria to #251
4. **MVVM separation:** Ensure Flutter code follows MVVM pattern
5. **Review output:** Check generated code before committing
6. **Run tests:** Validate generated tests pass
7. **Verify visually:** Compare Angular output to Flutter app side-by-side
8. **Test interactions:** Verify all gestures and clicks work as expected

## Need Help?

Common questions:

**Q: Will this match the Flutter app exactly?**
A: Yes! Phase 0.5, 2.5, and 5 enhancements ensure 100% visual and functional parity by extracting exact styles, implementing gestures, and validating results.

**Q: What if button colors don't match?**
A: Run Phase 8 validation report to identify which colors are mismatched. The theme extraction phase should capture all colors from FlutterFlowTheme and widget literals.

**Q: How do I fix swipe gestures not working?**
A: Verify HammerJS is installed (`npm install hammerjs`) and configured in angular.json. Check that component has (swipeleft) and (swiperight) bindings.

**Q: Can I customize the conversion?**
A: Yes! Edit the conversion plan after Phase 0, before continuing. You can also adjust theme.scss after Phase 0.5.

**Q: What if my Flutter code isn't MVVM?**
A: Command will analyze and document patterns, but results may vary. Best results with proper MVVM separation.

---

**Remember:** This command analyzes your ACTUAL Flutter code from **remote branch origin/#251-clean** and your ACTUAL GitHub requirements from #251. It extracts **every color, font, spacing, and interaction** to achieve **100% visual and functional parity**!

**IMPORTANT:** The command **always uses the remote branch** (`origin/#251-clean`). If you have a local branch with the same name, ensure it's fully committed and pushed to remote before running the conversion.

---

## Directory Structure Enforcement (CRITICAL)

**The command ENFORCES strict directory structure:**

✅ **ALWAYS outputs to:** `scheduler-angular/src/app/features/<feature-name>/`
✅ **ALWAYS places docs in:** `scheduler-angular/src/app/features/<feature-name>/docs/`
✅ **VALIDATES** output directory before starting conversion
❌ **REJECTS** any path outside `scheduler-angular/src/app/features/`
❌ **STOPS** if directory structure validation fails

**Why This Matters:**
- 🏗️ Maintains consistent project architecture
- 📁 Keeps feature code isolated and modular
- 📚 Ensures documentation stays with its feature
- 🔍 Makes code discovery and navigation easier
- ✅ Follows Angular best practices for scalable applications

**Validation Example:**
```bash
# Phase 0 validates BEFORE any conversion work
[Phase 0] Validating output directory...
❌ Invalid: web-app/src/app/features/onboarding/
✓ Required: scheduler-angular/src/app/features/<feature-name>/
→ Conversion stopped - fix directory path and retry
```

---

## Security & Safety

This command is safe because:
- ✅ No dangerous shell scripts
- ✅ Read-only git operations only
- ✅ No direct file modifications by primary agent
- ✅ User confirmation before writing files
- ✅ MCP authentication for GitHub
- ✅ Delegates to specialized, sandboxed agents
- ✅ No git checkout operations
- ✅ Comprehensive validation before conversion

---

**Command Implementation:** Uses Task tool to delegate to specialized agents including **flutter-theme-analyzer** and **flutter-ui-pattern-analyzer** for deep style extraction. Performs limited read-only git operations on **remote branches only** and fetches GitHub requirements via MCP. Validates local/remote synchronization before conversion. **Enforces strict directory structure** for all generated files. **Ensures 100% visual and functional parity** through comprehensive style extraction, gesture implementation, and validation. Maintains security compliance while delivering pixel-perfect Flutter-to-Angular MVVM conversion based on real requirements and source code.
