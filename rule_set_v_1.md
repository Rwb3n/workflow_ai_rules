# initial cursor ruleset v.1 based on https://github.com/kleosr/cursorkleosr
 
---

## Core Workflow Rules

**RULE_WF_PHASE_ANALYZE:**  
**Constraint:** Understand the user's request and context thoroughly. 
No solution design or implementation planning in this phase.

**RULE_WF_PHASE_BLUEPRINT:**  
**Constraint:** Produce a detailed, unambiguous step‑by‑step plan. The blueprint must include:
- List of component names
- Target folder/file locations
- Inputs, outputs, and dependencies for each step
- Acceptance criteria for successful completion of each item
No code or implementation during this phase.

**RULE_WF_PHASE_CONSTRUCT:**  
**Constraint:** Execute the approved blueprint exactly. No deviation. 
If blockers arise, invoke error‑handling and/or transition to a revision phase.

**RULE_WF_PHASE_VALIDATE:**  
**Constraint:** Verify the implementation against the blueprint and requirements using automated tools. No new implementation.

**RULE_WF_TRANSITION_01:**  
**Trigger:** Explicit user command (`@analyze`, `@blueprint`, `@construct`, `@validate`).  
**Action:** Update `State.Phase` accordingly. Log the phase change.

**RULE_WF_TRANSITION_02:**  
**Trigger:** Detection of a blocking condition from this checklist:
  - Missing user approval of the blueprint
  - Unresolved merge conflicts or branch discrepancies
  - Critical lint or test failures not auto‑resolvable
  - Manual user request to revise the plan
**Action:** Log reason, update `State.Phase` to `BLUEPRINT_REVISE` (if blueprint change needed) or `ANALYZE` (if scope changed), set `State.Status=NEEDS_PLAN_APPROVAL`, and report to user.

---

## Initialization & Resumption Rules

**RULE_INIT_01:**  
**Trigger:** New session and `codebase_state.md` missing/empty.  
**Action:**
  1. Create `codebase_state.md` with default headings.
  2. Prompt user for missing `readme.md` if absent.
  3. Set `State.Phase=ANALYZE`, `State.Status=READY`.
  4. Log "Initialized new session."
  5. Solicit first task from user.

**RULE_INIT_02:**  
**Trigger:** Session start and `codebase_state.md` exists.  
**Action:**
  1. Load `readme.md` and `codebase_state.md`.
  2. Log "Resumed session."
  3. Inspect `State.Status` and prompt or report accordingly.

---

## Memory Management Rules

**RULE_MEM_READ_PHASE_BOUNDARY:**  
**Trigger:** Before a phase transition.  
**Action:** Read `codebase_state.md` to sync in‑memory state.

**RULE_MEM_READ_STM_01:**  
**Trigger:** When user input changes scope OR on explicit phase transition.  
**Action:** Read `codebase_state.md` once; use in-memory state during phase execution.

**RULE_MEM_UPDATE_STM_01:**  
**Trigger:** After each significant action or information receipt.  
**Action:** Update relevant `## State`, `## Plan`, `## Log` in `codebase_state.md` immediately.

**RULE_MEM_UPDATE_LTM_01:**  
**Trigger:** `@config/update` or successful VALIDATE phase after major changes.  
**Action:** Propose concise updates to `readme.md`, set `State.Status=NEEDS_LTM_APPROVAL`.

**RULE_MEM_VALIDATE_01:**  
**Trigger:** After modifying state or readme.  
**Action:** Perform consistency check; on issues, log and set `State.Status=NEEDS_CLARIFICATION`.

---

## Tool Integration Rules (Cursor Environment)

**RULE_TOOL_LINT_01:**  
**Trigger:** Relevant source file saved during CONSTRUCT phase.  
**Action:** Instruct Cursor terminal to run lint command. Log attempt. 
On completion, parse output, log result, set `State.Status = BLOCKED_LINT` if errors.

**RULE_TOOL_FORMAT_01:**  
**Trigger:** Relevant source file saved during CONSTRUCT phase.  
**Action:** Instruct Cursor to apply formatter or run format command via terminal. Log attempt.

**RULE_TOOL_TEST_RUN_01:**  
**Trigger:** Command `@validate` or entering VALIDATE phase.  
**Action:** Instruct Cursor terminal to run test suite. Log attempt. 
On completion, parse output, log result, set `State.Status = BLOCKED_TEST` if failures, `TESTS_PASSED` if success.

**RULE_TOOL_APPLY_CODE_01:**  
**Trigger:** AI determines code change needed per `## Plan` during CONSTRUCT phase.  
**Action:** Generate modification. Instruct Cursor to apply it. Log action.

---

## Error Handling & Recovery Rules

**RULE_ERR_HANDLE_LINT_01:**  
**Trigger:** `State.Status=BLOCKED_LINT`.  
**Action:**
  1. Parse lint errors.  
  2. If error touches ≤5 lines and matches a known rule, auto‑fix and re-run lint.  
  3. If auto‑fix succeeds, reset `State.Status`.  
  4. If beyond threshold or unknown, set `State.Status=BLOCKED_LINT_UNRESOLVED` and prompt user review.

**RULE_ERR_HANDLE_TEST_01:**  
**Trigger:** `State.Status=BLOCKED_TEST`.  
**Action:**
  1. Analyze failed tests.  
  2. If failure isolated (<5 lines, local scope), attempt automated fix and re-run tests.  
  3. On success, reset `State.Status`.  
  4. Otherwise, set `State.Phase=BLUEPRINT_REVISE`, `State.Status=NEEDS_PLAN_APPROVAL`, and propose plan adjustments.

**RULE_ERR_HANDLE_GENERAL_01:**  
**Trigger:** Unexpected error or ambiguity.  
**Action:** Log error, set `State.Status=BLOCKED_UNKNOWN`, and request user direction.

**RULE_ERR_RETRY_RESET_01:**  
**Trigger:** Any manual intervention acknowledged by user.  
**Action:** Reset `State.RetryCount=0`; log the intervention.

---

## Quality Validation Rules

**RULE_QUALITY_CHECK_01:**  
**Trigger:** Completion of any CONSTRUCT step.  
**Action:** Automatically queue tests (lint, unit, integration) and responsive checks at breakpoints 320px, 768px, 1024px.  

**RULE_QUALITY_BLOCKER_01:**  
**Trigger:** Failure in critical quality checks.  
**Action:** Set `State.Status=BLOCKED_QUALITY_CHECK`, document failure points, halt progression until resolved.

---

## Code Annotation Rules

**RULE_CODE_ANNOTATION_01:**  
**Trigger:** Creation or modification of any HTML or CSS file.  
**Action:** Ensure comprehensive annotations are added explaining structure, purpose, and behavior of the code.

**RULE_CODE_ANNOTATION_02:**  
**Trigger:** Quality validation of any implementation step.  
**Action:** Verify thorough annotations exist throughout all HTML and CSS code, flag as failing validation if annotations are insufficient.

---

## Task Complexity Rules

**RULE_TASK_COMPLEXITY_01:**  
**Trigger:** Defining a new task or step.  
**Action:** Assign a complexity rating (1-5) and ensure the task includes at least that many sub-tasks.

**RULE_TASK_COMPLEXITY_02:**  
**Trigger:** Quality validation of a task's implementation.  
**Action:** Verify that implementation complexity matches the assigned rating and all sub-tasks are completed.

---

## Code Generation Rules

**RULE_CODE_GEN_CHUNKING_01:**  
**Trigger:** Need to generate or modify a file with >100 lines.  
**Action:** Break generation into multiple chunks of ≤100 lines each, completing one chunk before proceeding to the next.

**RULE_CODE_GEN_CHUNKING_02:**  
**Trigger:** Tool error during code generation due to token limits.  
**Action:** Reduce chunk size by 50% and retry generation with smaller chunk.

---

## Error Retry Rules

**RULE_ERR_RETRY_LIMIT_01:**  
**Trigger:** Any tool error occurs.  
**Action:** Increment `State.RetryCount`, log error details, attempt alternative approach if `RetryCount < 3`.

**RULE_ERR_RETRY_LIMIT_02:**  
**Trigger:** `State.RetryCount` reaches or exceeds 3.  
**Action:** Set `State.Status = BLOCKED_MAX_RETRIES`, document all attempts in log, request human assistance.

---

## Model Perspective Rules

**RULE_MODEL_PERSPECTIVE_01:**  
**Trigger:** Persistent errors even after multiple retries.  
**Action:** Before clearing context, change `State.ModelPerspective` to a different approach (technical, creative, pedagogical) and retry with new perspective.

**RULE_MODEL_PERSPECTIVE_02:**  
**Trigger:** Complex implementation challenge requiring different viewpoint.  
**Action:** Temporarily change `State.ModelPerspective` to gain insights from alternate approach, then document insights before returning to standard perspective.

---

## Component Registry and Documentation Rules

**RULE_COMP_DOC_01:**  
**Trigger:** Completion of any implementation step or component.  
**Action:** Update component-registry.md with detailed documentation of the implemented component, including:
  1. Component structure and purpose
  2. Usage examples and variations
  3. Relationships with other components
  4. Implementation details and design decisions
  5. Code snippets for reference

**RULE_COMP_DOC_02:**  
**Trigger:** Any significant change to existing components.  
**Action:** Update the corresponding section in component-registry.md, documenting the change, reason, and impact on related components.

**RULE_COMP_DOC_03:**  
**Trigger:** Beginning of a new project phase.  
**Action:** Create a new section in component-registry.md for components specific to that phase, with proper cross-referencing to related components.

**RULE_COMP_DOC_04:**  
**Trigger:** Quality validation phase.  
**Action:** Verify component-registry.md is accurate and comprehensive for all components implemented in the previous phase, with proper documentation and examples.

**RULE_COMP_DOC_05:**  
**Trigger:** Final validation before deployment.  
**Action:** Ensure component-registry.md serves as a complete reference guide for the entire project structure, with table of contents, component index, and comprehensive search-friendly documentation.

---

*Note: These rules replace the previous "Migration Index Rules" section with a more accurate name reflecting their purpose: documenting and cataloging components throughout the development lifecycle.*
---

*All transitions, validations, and error handling now rely on explicit, measurable triggers. State reads are scoped to boundaries to optimize performance. 
Automated fixes respect confidence thresholds. Blueprints must meet template criteria. Responsive design is validated systematically.*
