*to be used with ruleset v1.2*

# Codebase State
*This file tracks current workflow phases for multiple projects/features, ties in workflow rules, and logs recent events.*
---
## Global Constants
BreakpointSizes: [320, 768, 1024]  # Canonical breakpoint definitions used across all rules
AutoFixLineThreshold: 5            # Maximum lines that can be auto-fixed
ChunkSizeLimit: 100                # Maximum lines per code chunk
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
    blueprintValidation:
      status: incomplete
      checklist:
        - [ ] Scope statement
        - [ ] Complete component/module list
        - [ ] Structure layout or file map
        - [ ] Interface definitions
        - [ ] Acceptance criteria
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
- id: WF2
  description: "TRANSITION_ON_BLOCKER - Revert to blueprint/analyze on blocking conditions"
  severity: error
  prerequisite: "Must attempt auto-fix if issue affects ≤${AutoFixLineThreshold} lines before triggering"
- id: WF3
  description: "RULE_MEM_SYNC_AT_BOUNDARY - Sync state before phase transitions"
  severity: error
- id: WF4
  description: "RULE_MEM_UPDATE_AFTER_ACTION - Update state after significant actions"
  severity: error
- id: WF5
  description: "MULTI_PROJECT_SYNC - Ensure active project state is loaded before transitions"
  severity: error
- id: WF6
  description: "PRE_CONSTRUCT_GUARD - Block BLUEPRINT→CONSTRUCT transition until blueprint validation complete"
  severity: error
  computation: "canTransition = (blueprintValidation.status === 'complete' && all checklist items checked)"

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
  actions: "Adds missing media queries, corrects breakpoint order, fixes min-width usage"

### Error Handling Rules
- id: EH1
  description: "AUTO_FIX_THRESHOLD - Only auto-fix issues affecting ≤${AutoFixLineThreshold} lines"
  severity: warning
  integration: "Must be attempted before WF2 (TRANSITION_ON_BLOCKER) can be triggered"
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
---
## Phase Transition Guards
*Rules that must be satisfied before phase transitions are allowed*
- ANALYZE→BLUEPRINT: No specific guards
- BLUEPRINT→CONSTRUCT: 
  - blueprintValidation.status must be "complete"
  - All blueprint checklist items must be checked
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
