*This file tracks the current workflow phase, ties in workflow rules, and logs recent events. The detailed plan is maintained separately in `plan.md`.*

---

## State

Phase: ANALYZE                   # Current workflow phase
Status: READY                    # Current status
CurrentTaskID: null              # No active task yet
PlanRef: plan.md                 # External plan reference
RetryCounts: {}                  # Per-task retry map
Blockers: []                     # Active blockers
LastTransitionLogID: 1           # Log entry ID of last phase transition
ModelPerspective: Default        # Current perspective
PerspectiveHistory: []           # History of perspective changes

---

## Metrics

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
AutoFixSuccessRate: null         # No fixes attempted yet
ComponentRegistry:
  path: component-registry.md
  lastUpdateLogID: null
  dependencyGraphStatus: null    # Not created yet

---

## Rules

*Structured constraints enforced by the agent.*

- id: R1
  description: "All HTML and CSS code must include detailed explanatory comments."
  severity: error
- id: R2
  description: "Task complexity ratings determine minimum number of subtasks."
  severity: warning
- id: R3
  description: "Limit code generation to chunks of 100 lines or fewer."
  severity: error
- id: R4
  description: "Max retry limit of 3 per task before requesting human assistance."
  severity: error
- id: R5
  description: "Enforce mobile-first designs and responsive breakpoints."
  severity: error 

---

## Log

*Rolling window of recent events (up to last 50 entries). Entries use sequential numeric IDs.*

[0001] [INIT] New workflow session started.
[0002] [PHASE] Phase set to ANALYZE.
[0003] [STATUS] Status set to READY.
