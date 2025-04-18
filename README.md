I asked: **what principles survive abstraction**, beyond web apps, and could underpin **a universal development ruleset** for *any* system?

---

**1. Phase Separation Principle**  
**Work must be separated into clean, irreversible phases:**  
- Analysis (understanding)  
- Blueprint (design)  
- Construction (implementation)  
- Validation (verification)  

**Each phase has hard constraints**: no bleeding implementation into analysis, no bleeding improvisation into construction.  
This principle transcends medium — it governs building a bridge, a database, a neural net, or an AI colony.

**Root:** Discipline in cognitive mode: understanding, imagining, doing, verifying must not mix.

---

**2. Explicit Transition Triggers**  
**Progression must be governed by explicit, objective conditions, not intuition or expedience.**  
- Blueprint is approved → only then may construction begin.  
- Critical test failure → automatically trigger plan revision.  

This principle ensures **systemic coherence** over individual cleverness.  
No field — architecture, mechanical engineering, software, biotech — survives without hard stage gates.

**Root:** Integrity of state change must be verifiable.

---

**3. State Persistence and Sync Principle**  
**At every critical boundary, the system’s internal memory must synchronize to an external, canonical source.**  
- Before phase shifts → sync memory from file/database.
- After major actions → writeback to storage.

This prevents corrupted assumptions. Whether the system is a local filesystem, a blockchain smart contract, or a quantum simulator, **memory state must be auditable and explicit**.

**Root:** Memory cannot rely on transient in-process assumptions.

---

**4. Error Handling Before Progression**  
**Failures (lint, test, performance) must block forward motion until explicitly handled.**  
There is no \"silent failure\" that continues the flow.

- Either fix automatically (if simple), or escalate and revise.  
- Document every error encountered and how it was handled.

Applies to web, hardware, civil engineering, biotech, aerospace.

**Root:** Failure *must be visible* and *change flow control.*

---

**5. Confidence Thresholds for Automation**  
**Automation must only occur when the system can certify high confidence with minimal collateral risk.**  
- Simple lint fix: auto-fix okay.
- Complex integration failure: human intervention mandatory.

Prevents cascading invisible errors in any system.

**Root:** Automation is permissioned by bounded risk.

---

**6. Plan Completeness Before Action**  
**No building starts until the blueprint includes at minimum:**  
- Components/modules  
- Inputs and outputs  
- Success criteria for each item  

Otherwise you are guessing, not engineering.

**Root:** Plans must satisfy structural completeness before execution.

---

**7. Mobility/Accessibility as Native**  
**Outputs must accommodate diversity of context from the start.**  
In web: mobile first.  
In broader systems: adaptivity to different environments (battery limits, different machines, network conditions, user abilities) must be foundational, not bolted on later.

**Root:** Systems must be born flexible, not retrofit flexibility.

---

**8. Retriability and Alternative Perspectives**  
**On repeated failure, system must change approach, not retry mindlessly.**  
Model shifts (technical, creative, pedagogical) are *designed* into the workflow.  
Systems that survive (biological, mechanical, digital) adapt, not hammer.

**Root:** Persistence without adaptability is suicidal.

---

**9. Documentation as First-Class Output**  
**Each output (code, hardware design, biological experiment) must automatically generate:**  
- Purpose, structure, variation notes
- Usage examples
- Change history

Without this, systems become unmaintainable.  
Migration index, changelog, audit log: always essential.

**Root:** System must narrate itself as it grows.

---

