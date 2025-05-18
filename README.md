# Hybrid_AI_OS vX.2 - Ruleset & Operating Principles

## 1. Introduction

Hybrid_AI_OS vX.2 is an autonomous AI operating system designed to manage and execute projects through a series of defined operational phases. It operates on a structured data ecosystem, primarily using `.txt` files that contain JSON objects, and strictly adheres to defined schemas for all data generation and manipulation.

The core philosophy is "Minimalist Mandate & Data Contracts," emphasizing clear, verbose, comprehensive, and schema-compliant operations. A key feature is its ability to leverage external capabilities through the Model Context Protocol (MCP) by interacting with configured MCP servers.

## 2. Overall Mandate & Core Principles

*   **Objective:** Manage projects through phases: **ANALYZE, BLUEPRINT, CONSTRUCT, VALIDATE, IDLE.**
*   **Data Adherence:** Strict compliance with defined SCHEMAS for all content and structure of its managed data files (even if they have a `.txt` extension, their content is often JSON).
*   **Output Quality:** All actions and generated artifacts (plans, issues, embedded annotations) MUST BE FULLY VERBOSE, COMPREHENSIVE, AND SCHEMA-COMPLIANT.
*   **Event Tracking:** Uses a global event counter `g` (from `state.txt`) for significant events.
*   **Versioning:** Manages file versions (`v` field) using optimistic locking for designated `.txt` files.
*   **MCP Integration:** When planning or executing tasks involving external tool interaction or data retrieval, Hybrid_AI_OS SHOULD consider Model Context Protocol (MCP) concepts and leverage configured MCP servers where appropriate for standardized access to external capabilities.

## 3. Data Ecosystem

Hybrid_AI_OS manages a defined set of core files and expects project-specific artifacts. All data interactions are governed by non-negotiable data contracts (schemas).

### 3.1. Core Files (Managed by Hybrid_AI_OS)

All files typically use a `.txt` extension, but their content structure (e.g., JSON) dictates internal formatting and schema adherence.

*   **s: `state.txt`**: Tracks the OS's current state, including phase, status, active plan/task, error info, version, and global event counter. (Content: JSON)
*   **p: `plans/plan_<id>.txt`**: Detailed project plans, decomposed into tasks and sub-tasks. Includes a top-level plan status. (Content: JSON)
*   **i: `issues/issue_<id>.txt`**: Tracks issues (bugs, enhancements, tasks, errors) linked to plans, tasks, or artifacts. (Content: JSON)
*   **is: `issues_summary.txt`**: A summary map of all issues and their key details. (Content: JSON)
*   **r: `registry_map.txt`**: A hierarchically organized index mapping unique `artifact_id`s to their primary filepaths. (Content: JSON)
*   **mcpc: `.hybrid_ai_os/mcp_config.txt`** (Optional): Project-specific (or global) configuration defining available MCP servers, how to invoke them (stdio/sse), and default policies (e.g., auto-approval for read-only tools). (Content: JSON)
*   **mcpc_caps: `.hybrid_ai_os/capabilities/caps_<server_id>.txt`** (Optional): Cached/curated capabilities (tools, prompts, resources with their schemas) for a specific configured MCP server. This helps the OS plan MCP interactions. (Content: JSON)
*   **Project-Specific Artifacts:** Source code files (e.g., `.js.txt`), documents (`.md.txt`), etc. These often contain an `EmbeddedAnnotationBlock`.

### 3.2. Key Data Schemas

(Refer to the full ruleset document for detailed schema definitions.)

1.  **State (`state.txt` Schema):** `v, g, ph, st, cp_id, ct_id, rt, active_issue_ids, last_error`.
2.  **Plan (`plan_*.txt` Schema):** Top-level `status`, `goal`, `tasks` (with `id, title, description, intent, execution_type, mcp_tool_call_details`, inputs, outputs, dependencies, status, etc.).
    *   `execution_type`: Can be `NATIVE_OS_ACTION`, `MCP_TOOL_CALL`, or `HUMAN_TASK`.
    *   `mcp_tool_call_details`: Specifies `server_key_ref` (from `mcp_config.txt`), `tool_name`, `arguments`, and `task_specific_approval` policy.
3.  **Issue (`issue_*.txt` Schema):** Standard issue tracking fields, linking to plans, tasks, and artifacts.
4.  **IssuesSummary (`issues_summary.txt` Schema):** Map of `issue_id` to `{status, title, severity}`.
5.  **RegistryMap (`registry_map.txt` Schema):** Hierarchical `artifact_registry_tree` mapping `artifact_id`s to filepaths.
6.  **EmbeddedAnnotationBlock (Schema for within artifact files):** Rich metadata block including `artifact_id`, `version_tag`, `g_last_modified`, description, dependencies, linked issues, quality notes, etc. Stored as a comment block in non-JSON files or as an `_annotationBlock` key in JSON-content files.
7.  **MCPConfig (`mcp_config.txt` Schema):** Defines `mcp_servers` with invocation details (command for `stdio`, endpoint for `sse`), approval policies, and optional references to capabilities summaries.
8.  **MCPCapabilitiesSummary (`caps_<server_id>.txt` Schema):** Describes known `tools` (with name, description, input/output schemas, annotations), `prompts`, and `resources` for a specific MCP server.

## 4. Operational Phases & Core AI Actions

Hybrid_AI_OS guides projects through five distinct phases:

1.  **ANALYZE Phase:**
    *   **Goal:** Understand requirements, determine plan suitability. Consider if configured MCP servers can fulfill needs.
    *   **Action:** `AI_Analyze_Requirements(input)`.
    *   **Outcome:** Updated `state.txt`, report.

2.  **BLUEPRINT Phase:**
    *   **Goal:** Create a new, verbose, comprehensive, decomposed project plan. Specify MCP tool calls (`execution_type`, `mcp_tool_call_details`) in tasks where appropriate.
    *   **Action:** `AI_Create_Plan(...)`.
    *   **Outcome:** New `plan_<id>.txt` with initial status, updated `state.txt`.

3.  **CONSTRUCT Phase:**
    *   **Goal:** Execute tasks/sub-tasks from the active plan.
    *   **Action:** `AI_Execute_Task_From_Plan()`.
        *   **For `NATIVE_OS_ACTION`:** Performs work, creates/modifies OS-managed artifacts.
        *   **For `MCP_TOOL_CALL`:**
            1.  Consults `mcp_config.txt` for server details.
            2.  Determines approval policy (task-specific or server default, considering tool's `readOnlyHint`).
            3.  If approval is required: OS status -> `BLOCK_INPUT`, awaits user via `AI_Manage_Idle_Or_AwaitingImpl`.
            4.  If approved/auto-approved: Invokes MCP server (e.g., stdio subprocess), performs MCP `initialize` and `tools/call`.
            5.  Processes response, creates/updates output artifacts.
        *   Maintains consistency with project design decisions.
    *   **Outcome:** Artifacts created/modified, `registry_map.txt` updated, plan/task statuses updated, `state.txt` updated.

4.  **VALIDATE Phase:**
    *   **Goal:** Verify outputs of the last CONSTRUCTED task against plan, schemas, and quality standards.
    *   **Action:** `AI_Validate_Executed_Task()`.
        *   If task was an `MCP_TOOL_CALL` with a known `outputSchema`, validates the tool's output against it.
    *   **Outcome:** Task status updated, issues created/updated, `state.txt` updated.

5.  **IDLE / AWAIT_IMPL Phase:**
    *   **Goal:** Await new directives, manage human implementation, or await user approval for MCP tool calls.
    *   **Action:** `AI_Manage_Idle_Or_AwaitingImpl(user_input?)`.
        *   Processes user inputs like `APPROVE_TOOL_CALL <task_id>` or `DENY_TOOL_CALL <task_id>`.
    *   **Outcome:** Interactions logged, issues/plan status updated, state managed.

## 5. AI Agent Responsibilities (Underlying)

The AI Agent is responsible for all underlying file operations, optimistic locking, managing `state.txt` (including `g` and `v` counters), sub-task iteration, parent task status roll-ups, and the full lifecycle of issues and MCP interactions as defined.

## 6. Final Mandate

Hybrid_AI_OS vX.2 interprets user requests and executes the appropriate Core AI Action. All operations and generated data MUST be exceptionally verbose, comprehensive, and strictly adhere to the provided SCHEMAS, leveraging configured MCP servers for external capabilities where appropriate.

---

This README provides a user/developer-friendly summary. The full, detailed ruleset document remains the ultimate source of truth for implementation.
