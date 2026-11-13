---
name: planner-worker
description: "Planner-Worker collaborative workflow for complex engineering tasks. The main model (Claude Opus/Sonnet, Gemini Pro, or any high-reasoning model) serves as Lead Architect to analyze requirements and decompose atomic tasks, dispatches ultra-fast Gemini Flash subagents (Model: 'flash') via invoke_subagent for implementation and testing, and triggers an independent, clean-context Pro subagent (Model: 'pro') for a strict read-only final audit before delivery. Use for complex development, multi-file refactoring, batch task delegation, 'planner worker', 'delegate to flash', or '指挥 Flash / 分工执行'."
---

# Planner-Worker Collaborative Workflow

Activate this workflow when the user wants a "Lead Architect Planning + Fast Model Execution + Independent Final Audit" development mode, or uses trigger phrases such as "planner worker", "delegate to flash", "batch dispatch", "指挥 Flash", or "分工执行".

> **Language Consistency**: Always communicate with the user, report progress, and record newly learned rules in the user's primary conversation language.

## 1. Three-Pillar Role Architecture

1. **Lead Architect / Planner (Current Main Model — Claude Opus/Sonnet, Gemini Pro, or any high-reasoning model)**:
   - **Responsibilities**: Global oversight, requirement analysis, codebase exploration, architectural design, atomic task decomposition, prompt engineering (precise dispatching), intermediate verification, and fallback takeover.
   - **Delegation Boundaries**:
     - **Delegate Heavy/Batch Tasks**: Heavy coding, large-scale refactoring, batch unit test generation, multi-chart plotting, or lengthy document creation should be offloaded to Flash subagents in parallel.
     - **Handle Trivial Tasks Directly**: For minor tweaks, organizing folders, small fill-in-the-blank edits, or quick shell commands taking seconds, the **main model should execute them directly** to avoid unnecessary subagent communication overhead.

2. **Execution Worker (Gemini Flash Subagent)**:
   - **Responsibilities**: Focused single-responsibility coding, file creation/editing, running build commands, executing unit tests, and capturing error logs.
   - **Characteristics**: Extremely fast, high throughput, minimal token cost.

3. **Independent Final Auditor (Clean-Context Pro Subagent)**:
   - **Responsibilities**: Invoked before final delivery on significant tasks (explicitly specifying `Model: "pro"`). Operates with a clean, isolated context free from the Lead Architect's confirmation bias to inspect changes for logic gaps, edge-case bugs, unhandled exceptions, and contract mismatches.
   - **Iron Rule**: **Strictly Read-Only!** Never invoke any file-editing or file-writing tools. Output only a structured inspection report based on code reading and test results.

---

## 2. Standard Five-Phase Workflow

### Phase 1: Analysis & Task Breakdown
1. Quickly explore relevant files and context using read-only tools.
2. Present a concise architectural plan and an **atomic task checklist** to the user.
3. **Decomposition Rules**:
   - **Single Responsibility**: Each task must have a clear, isolated focus (e.g., "Implement X data model", "Write route handler for Y", "Add unit tests for Z").
   - **Concurrent Write Isolation (Single-File Ownership)**: Multiple workers running in parallel **must never modify overlapping target files**. Tasks touching shared files (e.g., route registries, lockfiles, global configs) must be serialized into sequential steps or have workers output isolated snippets for the Lead Architect to integrate.
   - Independent tasks without write conflicts may be dispatched in parallel; dependent tasks must execute sequentially.

### Phase 2: Dispatch to Gemini Flash
When calling `invoke_subagent`, strictly adhere to the following parameters:

- **`Model`**: Must be set strictly to `"flash"` (never switch to expensive models during standard coding execution, except for the Phase 4 independent audit). If Flash quota is exhausted or system-level errors occur repeatedly, pause dispatching immediately, inform the user, and ask whether to switch models or take over directly—never silently switch models or take over everything without confirmation.
- **`TypeName`**: `"self"`.
- **`Role`**: Descriptive task-specific role name (e.g., `"Flash Coder - Auth Module"`, `"Flash Tester - Unit Tests"`).
- **On-Demand Rule Injection**: Only extract and inject the 1–2 most relevant rules from Section 4 into the Worker prompt; never dump the entire knowledge base as irrelevant noise.
- **Structured Prompt Template**:

```markdown
【Task Objective】
<One clear sentence stating the exact goal>

【Target Files & Context】
- Files to create/modify: <Full or explicit relative paths>
- Reference interfaces/models: <Paths and symbol/function names>
- Relevant Pitfall Rules: <1–2 rules selected from Section 4>

【Implementation Requirements】
1. <Requirement 1: Exact function signatures, schemas, or business logic>
2. <Requirement 2: Error handling and coding conventions>
3. Preserve existing comments and code style; do not introduce unnecessary external dependencies.

【Verification & Acceptance Criteria】
- After editing, run the verification command: `<exact command, e.g. npm test / pytest>`
- Check exit code and output.
- **Local Retry Circuit Breaker**: If errors occur, debug and fix locally for **a maximum of 2 attempts**. If tests still fail after 2 consecutive attempts, stop editing immediately, preserve the workspace state, mark the status as `[BLOCKED]`, and report the root cause plus what you tried.

【Required Output Format】
Report back concisely using only this structure:
1. Status: [PASSED / BLOCKED]
2. Modified Files: [List of file paths]
3. Key Changes: [1–3 concise bullet points]
4. Test Output: [Exit code and key output lines]
5. Blocker Details: [Only if BLOCKED: root cause and attempted fixes]
```

### Phase 3: Review & Verification
1. Review the Flash Worker's structured summary, modified files, and test results.
2. **Verification Checks**:
   - Confirm the atomic objective is fully satisfied.
   - Check for edge-case omissions or regression risks.
3. **Closed-Loop Repair & Global Takeover**:
   - If the Worker hits a blocker or leaves minor flaws, diagnose the root cause and either guide the same subagent via `send_message` or dispatch a targeted fix.
   - **Global Circuit Breaker & Takeover**: When the same atomic task reports `[BLOCKED]` for the **2nd time** (i.e., failing again after 1 round of Lead Architect guidance), stop delegating immediately—**the Lead Architect must take over and complete the fix directly**.

### Phase 4: Final Audit & Delivery

Executed once all coding tasks are complete and before presenting final delivery to the user:

#### 4.1 Independent Read-Only Audit
- **Trigger Condition**: Triggered **only when changes span 4+ files, introduce major core/security-sensitive modules, or when explicitly requested by the user**. For routine 1–3 file changes, the Lead Architect inspects `git diff` directly to avoid unnecessary Pro token overhead.
- **Dispatch the Auditor**:
  - Call `invoke_subagent` with `Model="pro"` and `Role="Pro Code Auditor"` to spin up a clean-context high-reasoning Pro instance.
  - **Read-Only Sandbox Default**: **Default to `TypeName="research"`** (a physical read-only sandbox with no file-modification tools; include test/build results in the prompt). Only use `TypeName="self"` if the auditor strictly needs to execute shell tests directly (with explicit read-only instructions in the prompt).
  - **Auditor Prompt Template**:
    ```markdown
    【Role】Independent Code Quality Auditor (Pro Code Auditor)
    【Hard Constraint】STRICTLY READ-ONLY! Never invoke any file-editing or file-writing tools. Only inspect and report!

    【Audit Scope】
    - Modified/Added Files: <List of file paths>
    - Core Objectives: <Brief summary>
    - Stage Test Results: <Summary of test runs provided by Lead Architect>

    【Inspection Focus】
    1. Robustness & Edge Cases: Unhandled exceptions, null dereferences, resource leaks, or logic gaps?
    2. Contract Alignment: Do API signatures, schemas, and types match across modules?
    3. Security & Cleanliness: Hardcoded secrets, leaked PII, or leftover debug logs?

    【Output Format】
    Output a structured audit report:
    - [P0 Blockers]: Critical issues that must be fixed before delivery (or "None")
    - [P1 Suggestions]: Code smells, minor performance, or readability improvements
    - [Verdict]: [APPROVED] or [FIX RECOMMENDED]
    ```
- **Decision**:
  - If **P0 Blockers** are identified, the Lead Architect evaluates and fixes them directly rather than delegating blind fixes back to Workers.
  - If approved or only P1 suggestions exist, proceed to final delivery.

#### 4.2 Integration Summary & Final Delivery
Summarize clearly for the user:
1. **Implementation Summary**: Architectural changes and completed features.
2. **Test & Audit Status**: Unit test results and the Independent Auditor's verdict.
3. **Next Steps**: Any follow-up actions or manual setup needed from the user.

### Phase 5: Retrospective & Skill Self-Evolution
After task completion (or when the user invokes `/learn`):
1. **Bottleneck Reflection & Trigger Condition**: Did Workers misinterpret prompts, lack context, fail tests repeatedly, or did the Final Audit uncover blind spots? **Only proceed to update the knowledge base and notify the user if new, reusable, generalizable lessons emerged**; otherwise, end silently without editing this file.
2. **Capacity Cap & Deduplication**:
   - Keep the Lessons Learned section concise: **maximum 10 rules total, and maximum 4 rules per sub-section**. If adding a new rule would exceed either cap, **do not append directly**.
   - First consolidate, deduplicate, and abstract existing rules in that category into higher-level principles, pruning outdated or overly narrow edge cases, before integrating the new insight.
3. **Persist**: Update **`## 4. Lessons Learned`** cleanly using file-editing tools while preserving structure.
4. **Notify User**: Briefly inform the user that newly discovered lessons have been saved to `planner-worker` for future precision.

---

## 3. Collaboration Tips

- **Parallel Acceleration**: When decomposed subtasks are independent and modify disjoint files (e.g., frontend component A, backend route B, and unit tests for C), invoke multiple `invoke_subagent(Model="flash")` calls in a single turn.
- **Lean Context**: Pass only directly relevant file paths and core signatures in prompts—never paste thousands of lines of irrelevant logs.

---

## 4. Lessons Learned
*Note: Comes pre-seeded with 6 stack-agnostic core rules. Remaining capacity is reserved for rules automatically distilled by your projects during Phase 5. Hard cap: 10 rules total, 4 per sub-section.*

### 4.1 Delegation & Collaboration Contracts
- [Explicit Paths & Interface Contracts] Always provide full paths when dispatching tasks. For multi-file tasks, specify function signatures, data schemas, and return formats upfront in the prompt so Workers never guess.
- [Verify Commands Before Dispatch] Confirm that test or build commands included in the prompt actually work in the current environment (dependencies installed, scripts present) before dispatching; otherwise, Workers mistake environment issues for code bugs and trigger circuit breakers unnecessarily.
- [Lockfiles & Global Config Belong to Lead] Workers must never install dependencies, edit lockfiles (`package-lock.json`, `poetry.lock`), or alter global configs on their own. Have them report missing dependencies so the Lead Architect handles them centrally without parallel write conflicts.

### 4.2 Verification & Delivery Quality
- [Never Trust Summaries Alone] After a Worker reports `[PASSED]`, the Lead Architect must personally re-run key verification commands or inspect core diffs before marking the step done.
- [Visual Verification for Visual Outputs] For UI pages, charts, PPTs, or Word/PDF documents, code execution alone does not prove correctness. Render outputs to screenshots/images and inspect them with `read_file` to catch text overlap, clipping, occlusion, or misalignment.
- [Pre-Delivery Cleanup] Remove temporary scripts, debug artifacts, and leftover files generated by Workers before final delivery, and ensure no hardcoded keys, tokens, or sensitive PII remain.

### 4.3 Domain-Specific Rules
*(Initially empty. Populate automatically during Phase 5 retrospectives tailored to your stack—e.g., Frontend UI, Database Migrations, Data Science.)*
