# Codebase State
*This file tracks current workflow phases for multiple projects/features, ties in workflow rules, and logs recent events.*
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
      PlanRef: ./plan.md     # Project plan reference
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
      AutoFixAttempts: 0
      AutoFixSuccesses: 0
      ResponsiveFixAttempts: 0
      ResponsiveFixSuccesses: 0
      AutoFixSuccessRate: null
    blueprintValidation:
      status: incomplete
      checklist:
        - [ ] Scope statement
        - [ ] Complete component/module list
        - [ ] Structure layout or file map
        - [ ] Interface definitions
        - [ ] Acceptance criteria
      canTransition: false       # Explicit flag for transition permission
    componentRegistry:
      path: ./component-registry.md
      lastUpdateLogID: null
      dependencyGraphStatus: null
      components: {}
      componentDocumentation: {}
      changeLog: []
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
- id: WF2
  description: "TRANSITION_ON_BLOCKER - Revert to blueprint/analyze on blocking conditions"
  severity: error
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

### Error Handling Rules
- id: EH1
  description: "AUTO_FIX_THRESHOLD - Only auto-fix issues affecting ≤5 lines"
  severity: warning
- id: EH2
  description: "ESCALATION - Escalate errors beyond auto-fix threshold"
  severity: error
- id: EH3
  description: "RETRY_LIMIT - Max 3 retries before requiring assistance"
  severity: error
- id: EH4
  description: "PERSPECTIVE_SHIFTS - Rotate approaches on persistent errors"
  severity: warning

### Quality Validation Rules
- id: QV1
  description: "VERIFICATION_CHECKS - Run lint, tests, and responsive checks"
  severity: error
- id: QV2
  description: "RESPONSIVE_BLOCKER - Handle responsive design failures, increment ErrorCounts.responsive, attempt auto-fix when below threshold"
  severity: error
- id: QV3
  description: "BLUEPRINT_VALIDATION - Verify blueprint completeness, set canTransition flag"
  severity: error

### Code Quality Rules
- id: CQ1
  description: "All HTML and CSS code must include detailed explanatory comments"
  severity: error
- id: CQ2
  description: "Task complexity ratings determine minimum number of subtasks"
  severity: warning
- id: CQ3
  description: "Limit code generation to chunks of 100 lines or fewer"
  severity: error
- id: CQ4
  description: "Enforce mobile-first designs and responsive breakpoints"
  severity: error

### Component Registry Rules
- id: CR1
  description: "REGISTRY_UPDATE - Document components upon completion"
  severity: warning
- id: CR2
  description: "REGISTRY_DEPENDENCY_GRAPH - Maintain component dependencies"
  severity: error
- id: CR3
  description: "REGISTRY_VALIDATION - Verify registry accuracy"
  severity: warning
---
## Phase Transition Guards
*Rules that must be satisfied before phase transitions are allowed*
- ANALYZE→BLUEPRINT: No specific guards
- BLUEPRINT→CONSTRUCT: 
  - blueprintValidation.status must be "complete"
  - All blueprint checklist items must be checked
  - blueprintValidation.canTransition must be true
- CONSTRUCT→VALIDATE: All components must be implemented
- VALIDATE→COMPLETE: All verification checks must pass
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
