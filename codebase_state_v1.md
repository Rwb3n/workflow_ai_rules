# Codebase State
*This file tracks current workflow phases for multiple projects/features, ties in workflow rules, and logs recent events.*
---
## Global Constants
BreakpointSizes: [320, 768, 1024]  # Canonical breakpoint definitions used across all rules
AutoFixLineThreshold: 5            # Maximum lines that can be auto-fixed
ChunkSizeLimit: 100                # Maximum lines per code chunk
PerformanceBudgets:                # Performance thresholds that must be met
  FCP: 1500                        # First Contentful Paint in ms
  TTI: 3500                        # Time to Interactive in ms
  LCP: 2500                        # Largest Contentful Paint in ms
ChunkExemptions:                   # Modules exempt from strict chunk limits
  - "types.ts"                     # Type definitions
  - "constants.ts"                 # Constants
  - "utils/*.ts"                   # Utility functions
---
## Projects
ActiveProjectID: project-001    # Currently active project
Projects:
  - id: project-001
    name: "Initial Project"
    state:
      Phase: ANALYZE                   # Current workflow phase
      Status: READY                    # Current status
      CurrentTaskID: null              # No active task yet
      PlanRef: project-001/plan.md     # Project plan reference
      RetryCount: 0                    # Project retry counter
      Blockers: []                     # Active blockers
      LastTransitionLogID: 2           # Log entry ID of last phase transition
      ModelPerspective: Default        # Current perspective
      PerspectiveHistory: []           # History of perspective changes
      LastSyncTime: null               # Last state sync timestamp
    metrics:
      PhaseCounts:
        ANALYZE: 0
        BLUEPRINT: 0
        CONSTRUCT: 0
        VALIDATE: 0
      ProgressSteps:
        completed: 0
        total: 0
        percentage: 0
      LastProgressCheckLogID: null
      ComponentsComplete: 0
      ComponentsTotal: 0
      ErrorCounts:
        lint: 0
        test: 0
        responsive: 0
        mobileFirst: 0
        chunkBoundary: 0
        vendorImport: 0
        seo: 0
        accessibility: 0
        performance: 0
      AutoFixAttempts: 0
      AutoFixSuccesses: 0
      ResponsiveFixAttempts: 0
      ResponsiveFixSuccesses: 0
      AutoFixSuccessRate: null
      ResponsiveChecks:
        "320": { passed: null, severity1Count: 0, severity2Count: 0, severity3Count: 0 }
        "768": { passed: null, severity1Count: 0, severity2Count: 0, severity3Count: 0 }
        "1024": { passed: null, severity1Count: 0, severity2Count: 0, severity3Count: 0 }
      MobileFirstChecks:
        breakpointCommentExists: false
        correctBreakpointOrder: false
        mediaQueriesUseMinWidth: false
      ChunkBoundaryChecks:
        oversizedModulesWithoutChunks: []
        invalidChunkFormats: []
      PerformanceMetrics:
        FCP: null
        TTI: null
        LCP: null
      SEOChecks:
        metaTags: { required: ["title", "description"], found: [] }
        structuredData: { required: true, found: false }
      AccessibilityChecks:
        missingAltText: []
        missingAriaLabels: []
        contrastIssues: []
        landmarkRoles: { required: ["banner", "main", "navigation"], found: [] }
    blueprintValidation:
      status: incomplete
      checklist:
        - [ ] Scope statement
        - [ ] Complete component/module list
        - [ ] Structure layout or file map
        - [ ] Interface definitions
        - [ ] Acceptance criteria
      chunkBoundariesDefined: false    # Flag for CQ3 validation at blueprint phase
      breakpointsDefined: false        # Flag for CQ5 validation at blueprint phase
      # Status computed automatically from checklist state
    componentRegistry:
      path: project-001/component-registry.md
      lastUpdateLogID: null
      dependencyGraphStatus: null
      namespacePrefix: "p001_"  # Project namespace for components
      components: {}
      componentDocumentation: {}
      changeLog: []
      vendorAbstractionStatus: {}  # Track which vendor components have proper wrappers
---
## Global Metrics
TotalProjects: 1
ActiveProjects: 1
CompletedProjects: 0
ProjectPhases:
  ANALYZE: 1
  BLUEPRINT: 0
  CONSTRUCT: 0
  VALIDATE: 0
  COMPLETED: 0
---
## Rules
*Structured constraints enforced by the agent.*

### Workflow Phase Rules
- id: WF1
  description: "TRANSITION_BY_COMMAND - Phase transitions triggered by explicit commands"
  severity: error
  action: "Reset ModelPerspective to Default on transitions to ANALYZE or BLUEPRINT"
  prerequisite: "ActiveProjectID must be loaded and in sync (WF5)"
- id: WF2
  description: "TRANSITION_ON_BLOCKER - Revert to blueprint/analyze on blocking conditions"
  severity: error
  prerequisite: "Invoke AUTO_FIX_THRESHOLD (EH1) and RULE_TOOL_RESPONSIVE_FIX (TI5) before blocking"
- id: WF3
  description: "RULE_MEM_SYNC_AT_BOUNDARY - Sync state before phase transitions"
  severity: error
- id: WF4
  description: "RULE_MEM_UPDATE_AFTER_ACTION - Update state after significant actions"
  severity: error
- id: WF5
  description: "MULTI_PROJECT_SYNC - Ensure active project state is loaded before transitions"
  severity: error
  applies: "Required before any phase transition or rule application"
- id: WF6
  description: "PRE_CONSTRUCT_GUARD - Block BLUEPRINT→CONSTRUCT transition until blueprint validation complete"
  severity: error
  computation: "canTransition = (blueprintValidation.status === 'complete' && all checklist items checked)"
  checks: [
    "All blueprint checklist items must be checked",
    "Chunk boundaries must be defined (blueprintValidation.chunkBoundariesDefined)",
    "Breakpoints must be defined (blueprintValidation.breakpointsDefined)"
  ]

### Tool Integration Rules
- id: TI1
  description: "RULE_TOOL_LINT - Run linting after file saves"
  severity: error
- id: TI2
  description: "RULE_TOOL_FORMAT - Format documents after file saves"
  severity: error
- id: TI3
  description: "RULE_TOOL_TEST - Execute test suite during validation"
  severity: error
- id: TI4
  description: "RULE_TOOL_EDIT - Apply code modifications per blueprint"
  severity: error
- id: TI5
  description: "RULE_TOOL_RESPONSIVE_FIX - Attempt auto-fixes for responsive layout issues before blocking"
  severity: error
  threshold: "${AutoFixLineThreshold} lines"
  tracking: "Increments ResponsiveFixAttempts and ResponsiveFixSuccesses metrics"
  fixes: [
    "Missing media query: Add '@media (min-width: ${BreakpointSizes[n]}px) { ... }'",
    "Incorrect order: Reorder media queries from smallest to largest breakpoint",
    "Max-width usage: Replace '@media (max-width: Npx)' with '@media (min-width: (N+1)px)'",
    "Missing breakpoint comment: Add '// BREAKPOINTS: ${BreakpointSizes.join(', ')}'",
    "Nested media queries: Flatten and deduplicate"
  ]
- id: TI6
  description: "RULE_TOOL_LIGHTHOUSE - Run performance, SEO, and accessibility checks"
  severity: error
  integration: "Uses Lighthouse or similar tool against staging deployment"
  metrics: "Updates PerformanceMetrics, SEOChecks, and AccessibilityChecks"

### Error Handling Rules
- id: EH1
  description: "AUTO_FIX_THRESHOLD - Only auto-fix issues affecting ≤${AutoFixLineThreshold} lines"
  severity: warning
  integration: "Must be invoked before WF2 (TRANSITION_ON_BLOCKER) can be triggered"
- id: EH2
  description: "ESCALATION - Escalate errors beyond auto-fix threshold"
  severity: error
- id: EH3
  description: "RETRY_LIMIT - Max 3 retries before requiring assistance"
  severity: error
- id: EH4
  description: "PERSPECTIVE_SHIFTS - Rotate approaches on persistent errors"
  severity: warning
  reset: "ModelPerspective resets to Default on any transition to ANALYZE or BLUEPRINT"

### Quality Validation Rules
- id: QV1
  description: "VERIFICATION_CHECKS - Run lint, tests, and responsive checks"
  severity: error
- id: QV2
  description: "RESPONSIVE_BLOCKER - Handle responsive design failures, increment ErrorCounts.responsive, attempt auto-fix when below threshold"
  severity: error
  checkpoints: "${BreakpointSizes}"  # Reference to canonical breakpoint definitions
  blockingCriteria: "Any Severity 3 (Critical) failures block transition to VALIDATE phase"
  autofixRule: "Uses TI5 (RULE_TOOL_RESPONSIVE_FIX) for automated corrections"
- id: QV3
  description: "BLUEPRINT_VALIDATION - Verify blueprint completeness"
  severity: error
  blueprintChecks: [
    "All checklist items must be checked",
    "blueprintValidation.chunkBoundariesDefined must be true",
    "blueprintValidation.breakpointsDefined must be true"
  ]
- id: QV4
  description: "PERFORMANCE_BUDGET - Enforce performance thresholds for production builds"
  severity: error
  thresholds: {
    "FCP": "${PerformanceBudgets.FCP}ms", 
    "TTI": "${PerformanceBudgets.TTI}ms", 
    "LCP": "${PerformanceBudgets.LCP}ms"
  }
  tool: "TI6 (RULE_TOOL_LIGHTHOUSE)"
  blockingCriteria: "Any performance metric exceeding threshold blocks VALIDATE→COMPLETE transition"
- id: QV5
  description: "SEO_ACCESSIBILITY_VALIDATION - Validate SEO and accessibility requirements"
  severity: error
  seoRequirements: [
    "Meta tags (title, description) must exist",
    "Structured data must be present where applicable"
  ]
  a11yRequirements: [
    "All images must have alt text",
    "Interactive elements must have ARIA labels",
    "Color contrast must meet WCAG standards",
    "Landmark roles must be properly used"
  ]
  blockingSeverity: "Severity 2-3 issues block VALIDATE→COMPLETE transition"

### Code Quality Rules
- id: CQ1
  description: "All source files (HTML, CSS, TS/JS, JSX) must include detailed explanatory comments"
  severity: error
- id: CQ2
  description: "Task complexity ratings determine minimum number of subtasks"
  severity: warning
- id: CQ3
  description: "Limit code generation to chunks of ${ChunkSizeLimit} lines or fewer; larger modules must use explicit chunk boundaries"
  severity: error
  format: "// CHUNK: <name> format enforced via regex /^\/\/ CHUNK: [A-Za-z0-9 ]+$/"
  enforcement: "Any module exceeding ${ChunkSizeLimit} lines without proper CHUNK markers triggers a blocker"
  exemptions: "Files matching ${ChunkExemptions.join(', ')} can exceed chunk limit without markers"
  blueprintValidation: "Required in blueprint phase via blueprintValidation.chunkBoundariesDefined"
- id: CQ4
  description: "Enforce mobile-first designs and responsive breakpoints"
  severity: error
  breakpoints: "${BreakpointSizes}"  # Reference to canonical breakpoint definitions
  requirement: "Every component must include or reference responsive breakpoint definitions"
- id: CQ5
  description: "MOBILE_FIRST_COMMENT - Every component file must include breakpoint definition comment"
  severity: error
  format: "// BREAKPOINTS: ${BreakpointSizes.join(', ')}"
  check: "Increments ErrorCounts.mobileFirst if missing"
  blueprintValidation: "Required in blueprint phase via blueprintValidation.breakpointsDefined"
- id: CQ6
  description: "CHUNK_BOUNDARY_REQUIREMENT - Large modules must use proper chunk boundary markers"
  severity: error
  format: "// CHUNK: <name>"
  validation: "regex /^\/\/ CHUNK: [A-Za-z0-9 ]+$/"
  check: "Increments ErrorCounts.chunkBoundary if missing or malformed"
- id: CQ7
  description: "MOBILE_FIRST_MEDIA_QUERIES - Media queries must use min-width in ascending order"
  severity: error
  pattern: "@media (min-width: ${BreakpointSizes[0]}px) {...} followed by larger breakpoints"
  check: "Increments ErrorCounts.mobileFirst if using max-width or incorrect order"

### Component Registry Rules
- id: CR1
  description: "REGISTRY_UPDATE - Document components upon completion"
  severity: warning
- id: CR2
  description: "REGISTRY_DEPENDENCY_GRAPH - Maintain component dependencies"
  severity: error
  namespacing: "Components namespaced by project ID to prevent collisions"
- id: CR3
  description: "REGISTRY_VALIDATION - Verify registry accuracy"
  severity: warning

### Vendor Integration Rules
- id: VA1
  description: "WRAP_VENDOR_PRIMITIVES - All third-party UI primitives must be abstracted behind a local component before use"
  severity: error
  targets: ["@radix-ui", "@headlessui", "react-aria"]
  verification: "Direct imports of these packages are only allowed in /components/vendor/ directory"
  check: "Increments ErrorCounts.vendorImport for violations"
  action: "When wrapper is created, update vendorAbstractionStatus[packageName] = 'wrapped'"
---
## Code Generation Guidelines
*Reference for code generation - all requirements formalized as rules in Code Quality Rules section*
- Standard modules: Generate in chunks of ${ChunkSizeLimit} lines or fewer (CQ3, CQ6)
- Complex modules (State Manager, Phase Controller):
  - Use explicit chunk boundaries per CQ6 rule
  - Maintain comprehensive comments between chunks per CQ1
- Mobile-First Implementation:
  - Include breakpoint comment per CQ5
  - Implement mobile-first pattern per CQ4 and CQ7
  - Pass responsive tests at all breakpoints
- Vendor Integration:
  - Follow VA1 rule for all third-party component usage
- SEO & Accessibility:
  - Implement meta tags for SEO (title, description)
  - Ensure all images have alt text
  - Use semantic HTML and proper landmark roles
  - Add ARIA attributes for interactive elements
---
## Phase Transition Guards
*Rules that must be satisfied before phase transitions are allowed*
- ALL TRANSITIONS: 
  - ActiveProjectID must be loaded and in sync (WF5)

- ANALYZE→BLUEPRINT: No additional guards

- BLUEPRINT→CONSTRUCT: 
  - blueprintValidation.status must be "complete"
  - All blueprint checklist items must be checked
  - Chunk boundaries must be defined (blueprintValidation.chunkBoundariesDefined)
  - Breakpoints must be defined (blueprintValidation.breakpointsDefined)

- CONSTRUCT→VALIDATE: 
  - All components must be implemented
  - All responsive checks must be executed at breakpoints ${BreakpointSizes.join(', ')}
  - No Severity 3 (Critical) responsive failures may exist
  - Mobile-first requirements (CQ4, CQ5, CQ7) must be satisfied for all components
  - All components must have valid breakpoint comments (CQ5)
  - Media queries must use min-width in ascending order (CQ7)
  - Auto-fix must be attempted via TI5 for responsive issues before blocking
  - No direct vendor imports outside of /components/vendor/ directory
  - Large modules must have valid chunk boundaries (CQ6)

- VALIDATE→COMPLETE: 
  - All verification checks must pass
  - All vendor components must have proper abstractions registered
  - ErrorCounts for mobileFirst, chunkBoundary, and vendorImport must be zero
  - Performance metrics must be within budget (QV4)
  - No Severity 2-3 SEO or accessibility issues (QV5)
---
## Project Transition History
*Record of project phase transitions*
- Project: project-001, From: INIT to ANALYZE
---
## Log
*Rolling window of recent events (up to last 50 entries). Entries use sequential numeric IDs.*
[0001] [INIT] New workflow session started.
[0002] [PROJECT] Created project-001 "Initial Project".
[0003] [PHASE] project-001 phase set to ANALYZE.
