## Core Workflow Phases

**PHASE_ANALYZE**  
**Goal:** Acquire complete understanding of requirements, context, and constraints.  
**Constraint:** No solution design or implementation planning.

**PHASE_BLUEPRINT**  
**Goal:** Create an unambiguous, detailed blueprint. Must specify:
- Modules or components to be built
- Structure or repository layout (folders, files)
- Interfaces: inputs, outputs, and dependencies
- Acceptance criteria for each item
**Constraint:** No implementation or tooling commands until blueprint is approved.

**PHASE_CONSTRUCT**  
**Goal:** Execute the approved blueprint precisely.  
**Constraint:** No deviation; blockers trigger error-handling or blueprint revision.

**PHASE_VALIDATE**  
**Goal:** Perform verification checks against the blueprint and acceptance criteria.  
**Constraint:** No new implementation; only assessment.

---

## Transition Rules

**TRANSITION_BY_COMMAND**  
**Trigger:** Explicit user command (`@analyze`, `@blueprint`, `@construct`, `@validate`).  
**Action:** Switch to corresponding phase; log the transition.

**TRANSITION_ON_BLOCKER**  
**Trigger:** Detection of blocking conditions such as:
- Missing blueprint approval
- Repository integrity issues (merge conflicts, missing versions)
- Critical verification failures beyond auto-fix scope
- Manual request for plan revision
**Action:** Revert to `PHASE_BLUEPRINT` or `PHASE_ANALYZE`, log reason, set `State.Status=NEEDS_APPROVAL`.

---

## Initialization & Resumption

**INIT_NEW_PROJECT**  
**Trigger:** No prior state file exists.  
**Action:** 
1. Initialize `codebase_state.md` with default headings
2. Prompt for requirements documentation
3. Set `State.Phase=ANALYZE`, `State.Status=READY`
4. Log "Initialized new session"

**RESUME_PROJECT**  
**Trigger:** Existing `codebase_state.md` found.  
**Action:**
1. Load state file and any referenced documents
2. Log "Resumed session"
3. Report current phase and status
4. Prompt for next action

---

## State Management

**RULE_MEM_SYNC_AT_BOUNDARY**  
**Trigger:** Before any phase transition.  
**Action:** Read `codebase_state.md` to sync in-memory state exactly once per boundary.

**RULE_MEM_UPDATE_AFTER_ACTION**  
**Trigger:** After each significant action or user input.  
**Action:** Persist updated state (`## State`, `## Plan`, `## Log`) to `codebase_state.md` immediately.

---

## Tool Integration Rules

**RULE_TOOL_LINT**  
**Trigger:** Source file saved during CONSTRUCT phase.  
**Action:** Run lint via Cursor's terminal or Command Palette. Log attempt. Parse output, log result, set `State.Status=BLOCKED_LINT` if errors.

**RULE_TOOL_FORMAT**  
**Trigger:** Source file saved during CONSTRUCT phase.  
**Action:** Execute format-on-save or 'Format Document' via Cursor UI or Command Palette. Log attempt.

**RULE_TOOL_TEST**  
**Trigger:** Command `@validate` or entering VALIDATE phase.  
**Action:** Execute test suite via Cursor terminal. Log attempt. Parse output, log result, set appropriate status based on results.

**RULE_TOOL_EDIT**  
**Trigger:** Need to modify code per blueprint during CONSTRUCT phase.  
**Action:** Generate modification. Apply changes via Cursor editor. Log action.

---

## Error Handling & Recovery

**AUTO_FIX_THRESHOLD**  
**Constraint:** Only auto-resolve issues that affect ≤5 lines and match known patterns.  
**Action:** For issues meeting threshold, attempt fix and re-validate. Otherwise, escalate.

**ESCALATION**  
**Trigger:** Error beyond auto-fix threshold or retry limit reached.  
**Action:** Set `State.Phase=BLUEPRINT_REVISE`, log details, set `State.Status=NEEDS_PLAN_APPROVAL`.

**UNKNOWN_FAIL**  
**Trigger:** Unexpected error or ambiguity.  
**Action:** Log error, set `State.Status=BLOCKED_UNKNOWN`, request user direction.

**RETRY_RESET**  
**Trigger:** Any manual intervention acknowledged by user.  
**Action:** Reset `State.RetryCount=0`; log the intervention.

---

## Quality Validation

**VERIFICATION_CHECKS**  
**Trigger:** Completion of any CONSTRUCT step.  
**Action:** Queue lint, unit tests, integration tests, and responsive checks at breakpoints 320px, 768px, 1024px.

**RESPONSIVE_BLOCKER**  
**Trigger:** Failure in responsive checks.  
**Action:** 
- For **Severity 1 (Minor)**: Visual artifacts, slight misalignments, padding issues → Log warning but continue execution
- For **Severity 2 (Moderate)**: Component rendering issues, overflow problems → Log warning, flag for review before completion
- For **Severity 3 (Critical)**: Layout collapse, content inaccessibility, critical functionality broken → Set `State.Status=BLOCKED_QUALITY_CHECK`, document failure points, halt progression until resolved
Severity assessment based on:
1. Functional impact (can users complete core tasks?)
2. Content visibility (is critical content visible?)
3. Brand/design compliance (does it violate core design principles?)

---

## Blueprint Completeness Check

**BLUEPRINT_VALIDATION**  
**Trigger:** Before transitioning from BLUEPRINT to CONSTRUCT phase.  
**Action:** Verify blueprint contains:
- Scope statement
- Complete component/module list
- Structure layout or file map
- Interface definitions
- Acceptance criteria
Set `State.Status=NEEDS_PLAN_APPROVAL` if incomplete.

---

## Code Annotation Rules

**ANNOTATION_REQUIREMENT**  
**Trigger:** Creation or modification of HTML, CSS, or JSX files.  
**Action:** Ensure comprehensive annotations explaining structure, purpose, and behavior.

**ANNOTATION_VALIDATION**  
**Trigger:** Quality validation of implementation.  
**Action:** Verify thorough annotations throughout code, flag as failing validation if insufficient.

---

## Task Complexity Management

**COMPLEXITY_ASSIGNMENT**  
**Trigger:** Defining a new task or step.  
**Action:** Assign complexity rating (1-5). Ensure task includes at least that many subtasks.

**COMPLEXITY_VALIDATION**  
**Trigger:** Quality validation of implementation.  
**Action:** Verify implementation complexity matches assigned rating and all subtasks are completed.

---

## Artifact Generation Rules

**CHUNKED_GENERATION**  
**Trigger:** Need to generate file with >100 lines.  
**Action:** Break generation into multiple ≤100-line segments, completing one segment before proceeding.

**GENERATION_RETRY**  
**Trigger:** Generation failure due to token limits.  
**Action:** Reduce chunk size by 50% and retry with smaller segment.

---

## Retry & Perspective Shifts

**RETRY_LIMIT**  
**Trigger:** Any tool error occurs.  
**Action:** Increment `State.RetryCount`, log error details, attempt alternative approach if `RetryCount < 3`.

**MAX_RETRIES_REACHED**  
**Trigger:** `State.RetryCount` reaches or exceeds 3.  
**Action:** Set `State.Status=BLOCKED_MAX_RETRIES`, document all attempts in log, request human assistance.

**PERSPECTIVE_SHIFTS**  
**Trigger:** Persistent errors even after multiple retries.  
**Action:** 
1. Change `State.ModelPerspective` to a different approach, following this rotation sequence:
   - Default → Technical → Creative → Pedagogical → Escalate to User
   - Always rotate in this sequence; never skip or loop indefinitely
2. For each perspective:
   - **Technical Mode:** Optimize structure, type-safety, modularity
   - **Creative Mode:** Optimize user experience, visual hierarchy, storytelling
   - **Pedagogical Mode:** Rewrite for maximum clarity, comment density, explicit reasoning
3. If all perspectives have been attempted without resolution, set `State.Status=BLOCKED_PERSPECTIVE_EXHAUSTION`
4. Log each perspective change with timestamp in `codebase_state.md`

---

## Component Registry & Documentation

**PHASE_TIMING_AUDIT**  
**Trigger:** Entry to or exit from any workflow phase.  
**Action:** 
1. Log timestamp with phase name in `codebase_state.md`
2. Calculate duration upon phase exit
3. If phase exceeds threshold durations:
   - ANALYZE: >30 minutes
   - BLUEPRINT: >45 minutes
   - CONSTRUCT: >90 minutes (per component)
   - VALIDATE: >30 minutes
   Set `State.Status=WARNING_PHASE_DURATION` and notify user

**PHASE_PROGRESS_METRICS**  
**Trigger:** Every 15 minutes within a phase.  
**Action:**
1. Record percentage completion estimate based on blueprint tasks
2. If progress percentage has not changed in two consecutive checks, set `State.Status=WARNING_PROGRESS_STALLED`
3. Log metrics in `## Metrics` section of `codebase_state.md`

**SESSION_SUMMARY_METRICS**  
**Trigger:** End of development session or `@metrics` command.  
**Action:** Generate summary metrics including:
1. Time spent in each phase
2. Components completed/total
3. Error counts by type
4. Auto-fix success rate
5. Perspective shifts required
Log metrics to `codebase_state.md` and present to user

**REGISTRY_UPDATE**  
**Trigger:** Completion of any implementation step or component.  
**Action:** Update component-registry.md with detailed documentation including:
- Component structure and purpose
- Usage examples and variations
- Dependencies: explicit list of components this component relies on
- Dependents: components that rely on this component
- Implementation details and design decisions
- Code snippets for reference

**REGISTRY_DEPENDENCY_GRAPH**  
**Trigger:** After any REGISTRY_UPDATE action.  
**Action:** Maintain a lightweight dependency graph in component-registry.md:
```yaml
components:
  ComponentA:
    depends_on: []
    dependents: [ComponentB, ComponentC]
  ComponentB:
    depends_on: [ComponentA]
    dependents: [ComponentD]
  # etc.
```
Verify no circular dependencies exist. If circular dependency detected, set `State.Status=BLOCKED_CIRCULAR_DEPENDENCY` and require architectural review.

**REGISTRY_CHANGE_LOG**  
**Trigger:** Significant change to existing components.  
**Action:** Update corresponding section in component-registry.md, documenting change, reason, and impact.

**REGISTRY_PHASE_SECTION**  
**Trigger:** Beginning of new project phase.  
**Action:** Create new section in component-registry.md for phase-specific components with cross-references.

**REGISTRY_VALIDATION**  
**Trigger:** Quality validation phase.  
**Action:** Verify component-registry.md accuracy and comprehensiveness for all implemented components.

**REGISTRY_DEPLOYMENT_CHECK**  
**Trigger:** Final validation before deployment.  
**Action:** Ensure component-registry.md serves as complete reference guide with table of contents, component index, and search-friendly documentation.

---

*This workflow is specifically optimized for Cursor development while maintaining disciplined phase transitions, state management, and quality controls.*
