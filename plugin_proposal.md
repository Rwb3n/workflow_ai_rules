# Blueprint: Cursor Plugin Scaffold for Phase-Driven Workflow

This blueprint defines a Cursor plugin scaffold that enforces the PHASE_ANALYZE → PHASE_BLUEPRINT → PHASE_CONSTRUCT → PHASE_VALIDATE protocol automatically. It outlines modules, folder layout, interfaces, dependencies, and acceptance criteria before any implementation.

---

## 1. Modules & Components

1. **State Manager**  
   - **Responsibilities**: Load, persist, and sync `codebase_state.md`; track `State.Phase`, `State.Status`, `State.RetryCount`, and `State.ModelPerspective`.  
   - **Interfaces**:
     - `getState(): State`
     - `updateState(patch: Partial<State>): void`
     - `onBoundary(): void` (sync before transitions)

2. **Phase Controller**  
   - **Responsibilities**: Handle explicit commands and blockers to transition phases.  
   - **Interfaces**:
     - `onCommand(command: string): void`  // Parses `@analyze`, `@blueprint`, etc.
     - `detectBlockers(context: Context): Blocker[]`
     - `transitionTo(phase: Phase, reason?: string): void`

3. **Registry Manager**  
   - **Responsibilities**: Maintain `component-registry.md` and dependency graph.  
   - **Interfaces**:
     - `registerComponent(component: ComponentMeta): void`
     - `updateGraph(): void`

4. **Metrics & Logging**  
   - **Responsibilities**: Track timestamps, durations, retry counts, perspective shifts, and persistent logs.  
   - **Interfaces**:
     - `logEvent(event: LogEvent): void`
     - `recordMetric(metric: Metric): void`

5. **Tool Integration Handlers**  
   - **Lint Runner**  
     - Trigger: file save in CONSTRUCT  
     - Interface: `runLint(files: string[]): LintResult[]`
   - **Formatter**  
     - Trigger: file save in CONSTRUCT  
     - Interface: `runFormat(files: string[]): FormatResult`
   - **Test Runner**  
     - Trigger: VALIDATE phase or `@validate`  
     - Interface: `runTests(): TestResult`

6. **Retry & Perspective Engine**  
   - **Responsibilities**: Manage retry counts and rotate model perspectives on failure.  
   - **Interfaces**:
     - `onError(error: Error): void`
     - `rotatePerspective(): void`

7. **Responsive Checker**  
   - **Responsibilities**: Render target components/pages at breakpoints (320px, 768px, 1024px) and report visual checks.  
   - **Interfaces**:
     - `runResponsiveCheck(component: ComponentRef): ResponsiveReport`

8. **Event Hooks & Plugin Registration**  
   - **Responsibilities**: Tie into Cursor lifecycle events.  
   - **Interfaces**:
     - `onPluginInit(): void`
     - `onCommandReceived(command: string): void`
     - `onFileSaved(filePath: string): void`
     - `onPhaseEnter(phase: Phase): void`
     - `onPhaseExit(phase: Phase): void`

---

## 2. Repository Structure

```plaintext
cursor-plugin/
├─ src/
│   ├─ index.ts              # Plugin entrypoint, registers hooks
│   ├─ stateManager.ts       # State Manager module
│   ├─ phaseController.ts    # Phase Controller module
│   ├─ registryManager.ts    # Component Registry module
│   ├─ metrics.ts            # Metrics & Logging
│   ├─ toolHandlers/
│   │   ├─ lintRunner.ts     # Lint integration
│   │   ├─ formatter.ts      # Format integration
│   │   └─ testRunner.ts     # Test integration
│   ├─ retryEngine.ts        # Retry & Perspective Engine
│   ├─ responsiveChecker.ts  # Responsive Checker
│   └─ types.ts              # Shared type definitions (State, Phase, etc.)
├─ codebase_state.md         # Live state file managed by plugin
├─ component-registry.md     # Dependency registry
├─ README.md                 # Usage & commands
└─ package.json              # Plugin dependencies & scripts
```

---

## 3. Interfaces & Dependencies

| Module                | Dependencies                 | Inputs                            | Outputs                      |
|-----------------------|------------------------------|-----------------------------------|------------------------------|
| State Manager         | `fs`, types                  | (none)                            | `State`                      |
| Phase Controller      | State Manager, metrics       | command string, Blocker[]         | phase transition events      |
| Registry Manager      | State Manager                | `ComponentMeta`                   | updated `component-registry.md` |
| Metrics & Logging     | State Manager                | `LogEvent`, `Metric`              | appended logs, metrics data  |
| Lint Runner           | external linter library      | file paths                        | `LintResult[]`               |
| Formatter             | external formatter library   | file paths                        | `FormatResult`               |
| Test Runner           | test framework (jest, etc.)  | (none)                            | `TestResult`                 |
| Retry Engine          | State Manager, metrics       | `Error`                           | rotated perspective, updated state |
| Responsive Checker    | headless browser (playwright)| `ComponentRef`                    | `ResponsiveReport`           |
| Plugin Hooks          | Cursor plugin API            | plugin lifecycle events           | invokes module methods       |

---

## 4. Acceptance Criteria

1. **State Synchronization:**  
   - `codebase_state.md` reflects all phase transitions and state updates in real time.  
   - On plugin init, state loads correctly and resumes prior phase.  

2. **Phase Enforcement:**  
   - Commands `@analyze`, `@blueprint`, `@construct`, `@validate` reliably transition phases.  
   - Blockers detected mid-phase revert to `PHASE_BLUEPRINT` or `PHASE_ANALYZE` as specified.  

3. **Tool Automation:**  
   - Saving a source file in CONSTRUCT triggers lint and format handlers without errors.  
   - Entering VALIDATE phase or `@validate` triggers test suite and reports results.  

4. **Retry & Perspective:**  
   - On three consecutive failures of any module, perspective cycles through Technical → Creative → Pedagogical, documented in state.  

5. **Registry Maintenance:**  
   - Each component registration updates `component-registry.md` with accurate dependencies.  
   - Dependency graph contains no circular references.  

6. **Responsive Checks:**  
   - Running responsive checks generates reports for breakpoints 320px, 768px, 1024px with severity levels.  

7. **Metrics Logging:**  
   - Phase entry/exit timestamps and durations recorded.  
   - Warnings logged if phases exceed thresholds.  

8. **Plugin Stability:**  
   - Plugin initializes, loads state, and registers hooks without runtime errors.  



