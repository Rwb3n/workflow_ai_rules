# README — Hybrid\_AI\_OS v1.4

## Purpose

Minimal, token-efficient operational system for an autonomous agent. Provides phased workflow, optimistic-locking persistence, sharded plan and artifact registries, comprehensive logging, retry control, and issue tracking with embedded code annotations.

## Directory Layout

```
/
├── state.json                 # Single authoritative state (optimistic lock)
├── logs/
│   └── log_unit_<n>.jsonl     # Rollover JSON-lines logs
├── registry/
│   └── <domain>/<id>.json     # Artifact records (optimistic lock)
├── registry_summary.json      # id → artifact path map (optimistic lock)
├── plans/
│   └── <domain>/<id>.json     # Plan files (optimistic lock)
├── plan_summary.json          # id → plan path map (optimistic lock)
└── known_issues.json          # Issue ledger (optimistic lock)
```

## Phases

| Code | Name      | Function                                     |
| ---- | --------- | -------------------------------------------- |
| A    | ANALYZE   | Parse goal. Produce internal understanding.  |
| B    | BLUEPRINT | Generate plan file. Persist.                 |
| C    | CONSTRUCT | Execute next pending task. Create artifacts. |
| V    | VALIDATE  | Review task output, enforce annotations.     |
| I    | IDLE      | Wait state.                                  |

Phase transitions follow `A → B → C → V → (loop|I)`.

## Core Files

### `state.json`

| Field | Type | Description                                                  |
| ----- | ---- | ------------------------------------------------------------ |
| `v`   | int  | Version for optimistic lock.                                 |
| `g`   | int  | Global monotonic event counter.                              |
| `ln`  | int  | Entries in current log file.                                 |
| `max` | int  | Log-rollover threshold.                                      |
| `ph`  | enum | Current phase.                                               |
| `st`  | enum | READY, BUSY, BLK\_CON, BLK\_SUM, BLK\_PSUM, BLK\_MAX, ERROR. |
| `cp`  | str? | Current plan id.                                             |
| `ct`  | str? | Current task id.                                             |
| `rt`  | obj  | `{taskId:int}` retry counts.                                 |
| `iss` | obj  | Active issues cache.                                         |
| `err` | str? | Last error snippet.                                          |

### Plan File `plans/<dom>/<id>.json`

Versioned JSON holding goal, task list, per-task state, retry counters.

### Artifact File `registry/<dom>/<id>.json`

Versioned JSON with structural metadata, dependency links, active issue list, annotation requirement.

### Summaries

`plan_summary.json` and `registry_summary.json` map identifiers to file paths; updated after each successful write of a corresponding entity.

### Issue Ledger `known_issues.json`

Versioned store of suppressed lint errors or runtime work-arounds; status ACTIVE / RES.

## Logging

* File format: one JSON object per line.
* Fields: `id` (global event), `L` (I|W|E|D), `ph`, `plan`, `task`, `msg`.
* Rollover triggered when `ln ≥ max`; new file created, counter reset.

## Optimistic Lock Protocol

1. Read file, capture on-disk version.
2. Increment in-memory version.
3. Write; on disk version higher → conflict.
4. Conflict sets `st = BLK_CON` and logs error.

No transactional guarantee across multiple files. Sequence:

* Write primary file.
* Attempt summary write.
* Summary conflict ⇒ `st = BLK_SUM` (artifact) or `st = BLK_PSUM` (plan).

## Task Execution Output Schema

```
{
  "id":  "<artifact id>",
  "dom": "<registry domain>",
  "loc": "<file path or data ref>",
  "type":"<artifact type>",
  "pu":  "<purpose>",          // optional
  "if":  ["<interface>"],      // optional
  "dep": ["<artifact id>"]     // optional
}
```

Returned object triggers artifact registration.

## Annotation Rules

Code, config, and doc artifacts created or modified in phase **C** must embed:

* Purpose statement.
* Logic explanation for complex blocks.
* `// Depends on: [ArtifactID]` list.
* Optional dependents note.
* `// Issue: <issueId>` tags for suppressions.

Validator rejects artifacts lacking these annotations.

## Issue Handling

* **RECORD**: add issue entry, link id into artifact `ki[]`.
* **RESOLVE**: mark `known_issues.json`, unlink from artifact.
* All issue operations use optimistic lock on both ledger and artifact file.

## Retries

Per-task retry counter in `state.rt` and persisted in plan file. Exceeding `MAX` sets `st = BLK_MAX`.

## Consistency Check

Background or on-demand scan:

* Verify summary mappings to physical files.
* Detect orphans, duplicate ids, version drift.
* Log discrepancies; auto-repair allowed.

## Startup Sequence

1. Load `state.json`; if absent, create defaults.
2. Ensure directories and summaries exist.
3. Load active issues into memory.
4. Sync global event counter from last log entry.
5. Log session start; `st = READY`.

## Error States

| Code      | Cause                             |
| --------- | --------------------------------- |
| BLK\_CON  | Any optimistic lock conflict.     |
| BLK\_SUM  | Registry summary write conflict.  |
| BLK\_PSUM | Plan summary load/write conflict. |
| BLK\_MAX  | Task retries exceeded limit.      |
| ERROR     | Logging call with level `E`.      |

Recovery requires manual or programmed conflict resolution followed by state clearance.

## End-of-File
