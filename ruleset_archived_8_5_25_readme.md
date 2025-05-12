# AI Agent Operational Ruleset & Framework

## 1. Overview

This document describes the operational ruleset governing our AI agent. It's designed to ensure disciplined, traceable, and robust operation across all project mediums and complexities. The framework emphasizes structured phase transitions, meticulous state management, proactive quality control, and comprehensive record-keeping through dedicated files.

The core idea is to provide the AI with a clear, systematic way to approach tasks, manage its own operational state, learn from its actions, and maintain a detailed history and inventory of the project it's working on. This not only improves the AI's performance and reliability but also makes its operations transparent and auditable for human oversight.

## 2. Goals of this Ruleset

*   **Structure & Predictability:** Enforce a consistent workflow for all AI-driven projects.
*   **Traceability & Auditability:** Maintain detailed logs of all actions, decisions, and state changes.
*   **Resilience & Recoverability:** Allow the AI to resume work after interruptions and manage long-running tasks effectively, even with context window limitations.
*   **Quality Assurance:** Integrate validation and approval steps throughout the development lifecycle.
*   **Project Maintainability:** Create and maintain a comprehensive registry of all project artifacts and their relationships.
*   **Clarity for Human Oversight:** Utilize human-readable Markdown files for state, logs, plans, and the registry.

## 3. Key Components & Concepts

The ruleset revolves around several key components:

### 3.1. Dedicated Files

All persistent information is stored in four dedicated Markdown files within the project's working directory:

1.  **`codebase_state.md`**:
    *   **Purpose:** A snapshot of the AI's current operational status.
    *   **Content:** Includes the global `LogEntryCounter`, current `Phase` and `Status`, active `TaskID` and `SubTaskID`, retry counts, and other contextual variables.
    *   **Analogy:** The AI's "dashboard" or "short-term memory."

2.  **`codebase_log.md`**:
    *   **Purpose:** An append-only, chronological record of all significant AI actions, events, errors, and state changes.
    *   **Content:** Each entry is prefixed with the `LogEntryCounter` and includes task/sub-task context.
    *   **Analogy:** The AI's "flight recorder" or "detailed diary."

3.  **`codebase_registry.md`**:
    *   **Purpose:** A comprehensive database of all project artifacts (code modules, documents, configuration files, etc.).
    *   **Content:**
        *   `Common Artifact Types`: Definitions of recurring artifact types within the project.
        *   `Codebase Index`: A quick lookup mapping artifact identifiers to their physical locations.
        *   `Artifact Details`: In-depth metadata for each artifact, including its purpose, type, location, interfaces, dependencies, dependents, prerequisites, status, and modification history (via Log Unit).
    *   **Analogy:** The project's "encyclopedia" or "inventory system."

4.  **`codebase_plan.md`**:
    *   **Purpose:** A persistent archive of all approved blueprints. This is crucial for context recovery if the AI's active memory is limited or reset, and for long-term project understanding.
    *   **Content:** Stores complete, approved blueprints, including their complexity assessments and detailed sub-task decompositions.
    *   **Analogy:** The project's "master construction plans archive."

### 3.2. Core Workflow Phases

The AI operates in distinct phases, ensuring a methodical approach:

1.  **ANALYZE:** Understand requirements, define scope, and identify common patterns/artifact types.
2.  **BLUEPRINT:** Create detailed, unambiguous plans for each artifact. This includes defining its purpose, structure, location, interfaces, dependencies, acceptance criteria, complexity, and—crucially for complex items—breaking it down into manageable **sub-tasks**. No construction begins without an approved blueprint.
3.  **CONSTRUCT:** Implement the artifact based on the approved blueprint, working through sub-tasks if defined.
4.  **VALIDATE:** Verify that the constructed artifact meets its acceptance criteria and integrates correctly.

### 3.3. State Management & Logging

*   The AI's current operational state is meticulously tracked in `codebase_state.md`.
*   Every significant action or state change triggers `RULE_LOG_EVENT`, which increments a global `State.LogEntryCounter` and appends a detailed, context-aware message to `codebase_log.md`.
*   The `State.LogEntryCounter` serves as a universal sequencing and versioning mechanism, linking log entries, registry updates, and plan approvals.
*   `codebase_state.md` is persisted via `RULE_STATE_PERSISTENCE` after critical changes, ensuring recoverability. The log counter is also synchronized from `codebase_log.md` upon session resumption to ensure accuracy.

### 3.4. Task and Sub-Task Management

*   Blueprints for complex artifacts are decomposed into smaller, trackable **sub-tasks**.
*   The AI focuses on one task (artifact blueprint) and one sub-task at a time, identified by `State.CurrentTaskID` and `State.CurrentSubTaskID`.
*   This granular approach helps manage context window limitations, allows for more frequent progress updates, and improves error recovery.

### 3.5. Artifact Registry & Maintenance

*   `RULE_REGISTRY_MAINTENANCE` ensures `codebase_registry.md` is kept up-to-date as artifacts are planned, created, or modified.
*   It includes checks for **circular dependencies** to maintain project integrity. If detected, the problematic registry change is not persisted, and the issue is escalated.
*   The registry also tracks "Dependents," so changes in one artifact can be traced to others that rely on it. Placeholder entries are created for undefined dependencies.

## 4. How the System Operates (High-Level)

1.  **Initialization (`RULE_INIT_NEW_PROJECT`):** If starting a new project, the AI creates the four core Markdown files with initial structures and sets its state to `ANALYZE`.
2.  **Resumption (`RULE_RESUME_PROJECT`):** If `codebase_state.md` exists, the AI loads its previous state, synchronizes its `LogEntryCounter` with `codebase_log.md`, and reports its current status, ready to continue.
3.  **Phased Execution:**
    *   The AI proceeds through the `ANALYZE`, `BLUEPRINT`, `CONSTRUCT`, and `VALIDATE` phases.
    *   Blueprints are critical: they are detailed in `PHASE_BLUEPRINT`, require approval (`RULE_BLUEPRINT_VALIDATION_AND_APPROVAL`), and are archived in `codebase_plan.md`.
    *   Construction follows the blueprint, often one sub-task at a time.
4.  **Continuous Record-Keeping:**
    *   All actions are logged in `codebase_log.md`.
    *   The AI's state is saved in `codebase_state.md` at critical junctures.
    *   The `codebase_registry.md` is updated as artifacts evolve.
5.  **Error Handling & Quality Checks:** Built-in rules handle errors, retries, and escalations. Validation checks and registry accuracy checks ensure project quality.

## 5. Key Benefits

*   **Enhanced AI Reliability:** Structured operations reduce erratic behavior.
*   **Improved Project Outcomes:** Emphasis on planning and validation leads to higher quality results.
*   **Full Traceability:** The detailed logs and registry provide a complete history and understanding of the project's development.
*   **Context Window Resilience:** The `codebase_plan.md` and sub-tasking allow the AI to manage large, complex projects without losing context.
*   **Human-Understandable Operations:** Markdown files make it easy for humans to monitor, understand, and even (in emergencies) intervene in the AI's process.
*   **Scalability:** The system is designed to handle projects of increasing complexity by breaking them down systematically.

## 6. Interacting with the System

While this ruleset primarily governs the AI's internal operations, humans can interact by:

*   **Providing initial requirements.**
*   **Reviewing and approving blueprints** (if configured for manual approval).
*   **Monitoring progress** by reading `codebase_log.md`, `codebase_state.md`, `codebase_plan.md`, and `codebase_registry.md`.
*   **Responding to escalations** when the AI encounters issues it cannot resolve (e.g., circular dependency conflicts, repeated failures).

This framework aims to create a powerful, reliable, and transparent AI development partner.
