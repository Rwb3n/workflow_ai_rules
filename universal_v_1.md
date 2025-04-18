# A universal ruleset scaffolding for all other rulesets to be generated from

# Universal Development Workflow

This document defines a **technology‑agnostic** framework for building and evolving complex systems—software, hardware, biological protocols, data pipelines, or any engineered artifact—through disciplined, phase‑driven processes.

---

## Core Workflow Phases

**PHASE_ANALYZE**  
**Goal:** Acquire complete understanding of requirements, context, and constraints.  
**Constraint:** No solution design or implementation planning.

**PHASE_BLUEPRINT**  
**Goal:** Create an unambiguous, detailed blueprint. Must specify:
- Modules or components to be built
- Structure or repository layout (folders, registries)
- Interfaces: inputs, outputs, and dependencies
- Acceptance criteria for each item
**Constraint:** No implementation or tooling commands until blueprint is approved.

**PHASE_CONSTRUCT**  
**Goal:** Execute the approved blueprint precisely.  
**Constraint:** No deviation; blockers trigger error‑handling or blueprint revision.

**PHASE_VALIDATE**  
**Goal:** Perform verification checks against the blueprint and acceptance criteria.  
**Constraint:** No new implementation; only assessment.

---

## Transition Rules

**TRANSITION_BY_COMMAND**  
Trigger: Explicit user or steward command (`@analyze`, `@blueprint`, `@construct`, `@validate`).  
Action: Switch to corresponding phase; log the transition.

**TRANSITION_ON_BLOCKER**  
Trigger: Detection of blocking conditions such as:
- Missing blueprint approval
- Repository integrity issues (merge conflicts, missing versions)
- Critical verification failures beyond auto‑fix scope
- Manual request for plan revision
Action: Revert to `PHASE_BLUEPRINT` or `PHASE_ANALYZE`, log reason, set status `NEEDS_APPROVAL`.

---

## Initialization & Resumption

- **INIT_NEW_PROJECT:** No prior state; initialize default state repository, prompt for requirements, set phase to ANALYZE.  
- **RESUME_PROJECT:** Existing state found; load blueprint and state, set phase and status, prompt for next action.

---

## State Management

**RULE_MEM_SYNC_AT_BOUNDARY**  
Trigger: Before any phase transition.  
Action: Read or write the canonical state repository (e.g., `state.md`, database) exactly once per boundary.

**RULE_MEM_UPDATE_AFTER_ACTION**  
Trigger: After each significant action or user input.  
Action: Persist updated state (`## State`, `## Plan`, `## Log`) immediately.

---

## Tool Integration Rules

Abstract tooling into generic adapters:
- **Formatting Tools:** Run static formatting after artifact creation.  
- **Verification Tools:** Invoke linters, compilers, or validators post-construction and during validation.  
- **Testing Tools:** Execute test suites during VALIDATE phase.

Each tool adapter is triggered per environment or phase, not tied to specific CLI.

---

## Error Handling & Recovery

**AUTO_FIX_THRESHOLD:** Only auto‑resolve issues that affect ≤5 lines/artifacts and match known patterns; else escalate.  

**ESCALATION:** For unresolved errors, revert to blueprint, log details, require human review.  

**UNKNOWN_FAIL:** Unexpected errors set status `BLOCKED`, log context, request guidance.

---

## Quality Verification

**VERIFICATION_CHECKS:** At the end of each CONSTRUCT step, run Verification Tools for correctness, performance, security, and adaptability.  

**ADAPTIVITY_CHECKS:** Test artifacts in three contexts:
1. **Minimal Context:** Lowest resource or simplest environment.  
2. **Nominal Context:** Standard operating conditions.  
3. **Peak Context:** High‑load or most demanding environment.

**RESPONSE_BLOCKER:** Failures in critical checks halt progression until resolution.

---

## Blueprint Completeness Check

Before entering CONSTRUCT, automatically verify the blueprint includes:
- Scope statement
- Complete component/module list
- Structure layout or file map
- Interface definitions
- Acceptance criteria

No construction proceeds without passing this checklist.

---

## Environment Adaptivity

**Universal Adaptivity Rule:** Designs must accommodate varied contexts (e.g., device types, resource constraints, user abilities) from inception. Adaptivity tests are integral to VALIDATE.

---

## Task Complexity Management

Assign complexity ratings (1–5). Each task must enumerate at least as many subtasks as its rating. Validation confirms completion of subtasks.

---

## Artifact Generation Rules

Break any generated artifact >100 logical lines into ≤100‑line segments. On generation failure, halve segment size and retry.

---

## Retry & Perspective Shifts

**RETRY_LIMIT:** Up to 3 automated retries. On limit, set status `BLOCKED_MAX_RETRIES`, require human decision.  

**PERSPECTIVE_SHIFTS:** On persistent failure, switch to alternate viewpoints (technical, creative, pedagogical) before retry.

---

## Documentation & Traceability

Maintain a **Documentation Index** stored in a persistent registry (file, database, or wiki). For each component or artifact record:
- Purpose and rationale
- Interface examples and variations
- Dependencies and interactions
- Version history and change rationale

Update immediately after each CONSTRUCT step and verify during validation.

---

*This universal workflow abstracts environment, tooling, and artifact types, ensuring clarity, consistency, and resilience across any engineering domain.*

