# Agentic HPC Workflow Architecture

## Purpose

This document is the canonical source of truth for the Agentic HPC workflow.

It defines:

- the layered architecture;
- the role and authority of each component;
- how information moves between layers;
- how strategic tasks are represented;
- how Codex orchestrates execution workers;
- where raw evidence, operational reports, and strategic analysis belong;
- which decisions require human authorization;
- how the workflow integrates with the existing HPC optimization repository structure.

The design goal is to increase automation without giving up human control of scientific and engineering decisions.

The central principle is:

> **The human leads strategy. The Strategic Analyst reasons about evidence and proposes bounded next actions. Codex orchestrates execution. Execution workers collect evidence and perform approved work. Strategic interpretation returns to the Strategic Analyst before the human makes the next decision.**

---

# 1. High-Level Architecture

The workflow is divided into three major layers.

```text
                    STRATEGY LAYER

                USER — HUMAN LEADER
                       ↕
              ChatGPT Web / Sol
              Strategic Analyst
                       │
                       │ draft Strategic Specification
                       ▼
                Human Leader reviews
                and explicitly approves
                       │
                       │ Strategic Analyst writes directly
                       │ or approved fallback materializes
                       │ commit + push
                       ▼
                 tasks/TASK-XXX.md

                       │ approved task
                       ▼

                 EXECUTION LAYER

                Codex Orchestrator
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      OpenCode      OpenCode      OpenCode
       Worker        Worker        Worker
          │            │            │
          └────────────┼────────────┘
                       ▼
                Codex Validation
                       │
                       │ Codex Execution Report
                       ▼

                  REPORT / STRATEGY

              ChatGPT Web / Sol
              Strategic Analyst
                       │
                       │ Strategic Analysis
                       ▼
                USER — FINAL DECISION
```

The system therefore contains two loops.

## 1.1 Inner Loop — Operational Autonomy

```text
Codex
  ↓
worker
  ↓
Codex
  ↓
follow-up worker task if needed
  ↓
Codex
```

Codex may iterate autonomously inside the boundaries of an approved task.

The user does not need to supervise every command, file read, probe, or worker exchange.

## 1.2 Outer Loop — Strategic Human Control

```text
User
 ↓
Strategic Analyst
 ↓
Codex + workers
 ↓
Strategic Analyst
 ↓
User
 ↓
next decision
```

The outer loop controls:

- what problem is worth solving;
- what hypothesis is being investigated;
- what experiment is allowed;
- how much compute or risk is acceptable;
- whether evidence is strong enough to change direction;
- what the next strategic action should be.

The outer loop remains human-led.

---

# 2. Roles and Authority

## 2.1 Human Leader

The user is the highest-authority decision maker.

The Human Leader owns:

- research and optimization direction;
- prioritization;
- hypothesis selection;
- approval of strategic tasks;
- approval of experiments and high-risk actions;
- authorization of the Strategy → Execution handoff and any repository write
  needed to materialize approved task content;
- acceptable compute/resource expenditure;
- final interpretation of recommendations;
- promotion of new baselines;
- decisions to continue, stop, reopen, or change optimization direction.

The Human Leader should gradually avoid low-value mechanical work such as:

- manually opening many files;
- manually grepping configuration values;
- manually comparing large numbers of logs;
- manually writing repetitive experiment variants;
- manually coordinating independent evidence-gathering tasks.

The system exists to automate those activities while preserving human ownership of strategy.

---

## 2.2 Strategic Analyst

The Strategic Analyst is normally ChatGPT Web using a strong reasoning model.

Its role is strategic and analytical, not operational.

The Strategic Analyst owns:

- interpretation of raw evidence;
- quantitative comparison;
- trend analysis;
- hypothesis generation;
- hypothesis evaluation;
- causal reasoning;
- consideration of alternative explanations;
- experimental design;
- identification of information gaps;
- design of bounded next actions;
- writing the Strategic Specification for an approved task;
- strategic analysis under `planning/analysis/`;
- updates to campaign-level plans when authorized.

The Strategic Analyst drafts the complete `TASK-XXX.md` content for Human
Leader review. After explicit human approval, it may materialize the approved
task directly in GitHub when authorized access exists. Direct write access does
not let it approve its own proposal. Without that access, the Human Leader or
an authorized mechanical repository agent materializes the exact approved
content. A draft in conversation remains only a proposal.

The Strategic Analyst must distinguish clearly between:

- directly observed facts;
- derived values;
- plausible explanations;
- supported conclusions;
- unresolved uncertainty.

The Strategic Analyst must not simplify away uncertainty.

Example:

Preferred:

> The slower run used a different CPU-affinity layout. This is consistent with a host-placement contribution, but the current evidence does not distinguish NUMA locality from MPI progress effects.

Avoid:

> CPU affinity caused the slowdown.

unless the evidence actually establishes that conclusion.

### Strategic Analyst authority boundary

The Strategic Analyst proposes actions.

It does not authorize its own proposals.

The workflow is:

```text
Strategic Analyst proposes
          ↓
Human Leader reviews
          ↓
Human Leader approves/modifies/rejects
          ↓
Codex executes only the approved scope
```

---

## 2.3 Codex Orchestrator

Codex is the operational orchestrator.

Codex receives a clear, approved Strategic Specification and determines how to execute it efficiently.

Codex owns:

- reading the active approved task;
- decomposing the task into bounded execution work;
- identifying dependencies between subtasks;
- identifying work that can run in parallel;
- assigning work to execution workers;
- issuing follow-up worker instructions;
- deciding when a worker result is incomplete;
- validating that requested evidence exists;
- resolving operational inconsistencies;
- checking execution scope;
- collecting and indexing evidence;
- completing the Codex Execution Report in the task file.

Codex should use subagents when useful, not ritualistically.

A trivial task may be performed directly by Codex.

A complex task may be split among several workers.

### Codex is not the Strategic Analyst

Codex must not:

- change the strategic objective;
- perform campaign-level causal interpretation;
- choose a new optimization direction;
- declare a root cause from correlation alone;
- promote a new configuration as the campaign baseline;
- rewrite the Strategic Specification;
- create strategic conclusions simply because an execution result looks promising.

Codex may perform mechanical processing such as:

- percentage differences;
- averages;
- hashes;
- pass/fail counts;
- extracting measurements;
- checking that multiple files agree.

Codex may state directly observable facts.

Examples:

Allowed:

> Run B is 21.3% slower than Run A.

> `mlx5_bond_0` appears only in the failed run.

> Worker A and Worker C report different launcher values; the conflict was rechecked and the launcher was confirmed as X.

Not allowed:

> RoCE caused the performance regression.

> NUMA placement is the bottleneck.

> The next experiment should tune NCCL.

Those belong to the Strategic Analyst.

---

## 2.4 Execution Workers

Execution workers are normally OpenCode sessions using a high-volume model such as GLM.

Workers are bounded evidence and execution agents.

They may:

- inspect files;
- inspect source;
- inspect logs;
- run approved commands;
- execute approved probes;
- implement explicitly scoped changes;
- run approved tests;
- extract measurements;
- validate local outputs;
- preserve raw evidence;
- report missing information.

Workers may use local reasoning required to complete their assigned task.

For example, a worker may determine:

- whether a command succeeded;
- which file contains a requested value;
- whether an output contains the expected correctness marker;
- whether a requested measurement is unavailable.

Workers must not perform campaign-level strategic analysis.

They must not:

- infer the overall root cause;
- change the hypothesis;
- expand the task objective;
- recommend a new optimization direction;
- promote a new baseline;
- exceed Codex's authority;
- exceed the approved Strategic Specification.

### Worker authority inheritance

Authority always narrows downward.

```text
Human-approved task scope
          ↓
Codex authority
          ↓
Worker authority
```

A worker can never receive broader authority than Codex has.

---

# 3. Communication Model

Persistent files are used only at architectural boundaries.

Transient agent-to-agent communication is used inside the execution layer.

This avoids unnecessary documentation while preserving a strong audit trail.

## 3.1 Persistent communication

The main persistent communication object is:

```text
tasks/TASK-XXX.md
```

The repository copy contains:

1. the Strategic Specification;
2. the Codex Execution Report.

Strategic analysis is stored separately under:

```text
planning/analysis/<analysis-id>.md
```

Raw experiment evidence remains in its canonical location, such as:

```text
experiments/.../outputs/
results/metrics.csv
scripts/outputs/
profiling/
```

Task files reference raw evidence rather than duplicating it.

Only an approved task materialized in the repository is an executable
Strategy → Execution handoff. Draft task content in a conversation is not
executable repository state.

---

## 3.2 Strategy → Execution

The Strategic Analyst drafts the Strategic Specification from the canonical
`workflow/TASK-TEMPLATE.md`. The Human Leader reviews, modifies, and
explicitly approves it. With authorized direct GitHub access, the Strategic
Analyst materializes the approved `TASK-XXX.md` itself. Otherwise the Human
Leader writes the task manually or instructs a repository agent to copy the
exact approved content mechanically. The task is committed and pushed
according to the project Git policy and human authorization.

Codex pulls the repository and reads the approved task. Codex executes
repository state, not unpublished conversation content, and only within the
approved scope.

A fallback repository agent must not change the Strategic Specification,
reinterpret the request, expand scope, add strategic decisions, or begin
execution unless separately instructed and authorized.

Codex must not infer the active task merely from file modification time.

The task must be explicitly identified by the user, automation wrapper, project state, or another unambiguous mechanism.

---

## 3.3 Codex → Workers

Codex communicates directly with workers using transient prompts or sessions.

Per-worker Markdown task files are not required.

Per-worker report files are not required.

A typical worker instruction may contain:

```text
TASK
Compare CPU and NUMA placement between Run A and Run B.

INPUTS
- path/to/run-a/
- path/to/run-b/

RETURN
- rank → CPU mapping
- rank → NUMA mapping
- exact evidence paths
- missing evidence
- errors

DO NOT
- modify configuration
- submit jobs
- infer root cause
- recommend the next experiment
```

Codex may issue follow-up instructions if required evidence is incomplete.

---

## 3.4 Worker → Codex

Workers return structured evidence directly to Codex.

The response should normally contain:

- actions performed;
- observed values;
- raw evidence paths;
- files changed;
- missing evidence;
- operational errors;
- scope compliance.

Workers should not generate a strategic report.

---

## 3.5 Codex → Strategic Analyst

After execution and operational validation, Codex completes the Execution
Report in the same task file, sets front matter to `status: EXECUTED` and
`current_owner: strategic-analyst`, and makes the latest repository revision
available according to Git policy.

The report records:

- whether execution completed;
- what was executed;
- how work was orchestrated;
- operational validation;
- evidence locations;
- files changed;
- missing evidence;
- failures or exceptions;
- scope compliance.

Codex does not perform strategic interpretation. The Human Leader explicitly
authorizes `ANALYSE_RESULTS` before strategic analysis begins. The Strategic
Analyst reads the task, raw evidence, and relevant prior context directly
whenever practical, then persists analysis under
`planning/analysis/<analysis-id>.md` and updates
`planning/PLANS.md` when applicable. With authorized GitHub access it writes
these files directly; otherwise it supplies the exact analysis and task
metadata update for the Human Leader or an authorized mechanical agent to
materialize. After analysis is complete and persisted, the Strategic Analyst
is responsible for moving each task whose analysis is complete to
`status: ANALYZED` and `current_owner: user`. The Human Leader then makes the
strategic decision.

---

# 4. Canonical Task File

Each bounded strategic action uses one task file.

Example:

```text
tasks/TASK-003.md
```

A task file is both:

- an execution contract;
- an operational history for that bounded task.

It is not the main home of strategic analysis.

## 4.1 Task lifecycle

Use a small lifecycle:

```text
DRAFT
  ↓
APPROVED
  ↓
EXECUTING
  ↓
EXECUTED
  ↓
ANALYZED
  ↓
CLOSED
```

A blocked task may temporarily use:

```text
BLOCKED
```

A failed operational task may use:

```text
FAILED
```

## 4.2 Current owner

Front matter identifies who acts next:

| Status | `current_owner` |
| --- | --- |
| `DRAFT` | `strategic-analyst` |
| `APPROVED` | `codex` |
| `EXECUTING` | `codex` |
| `EXECUTED` | `strategic-analyst` |
| `ANALYZED` | `user` |
| `CLOSED` | `user` |

The Strategic Analyst drafts the task. Human approval moves `DRAFT` to
`APPROVED`. Codex moves `APPROVED` to `EXECUTING` when work starts, then to
`EXECUTED` only after operational validation and the Execution Report are
complete. Authorized, persisted strategic analysis moves `EXECUTED` to
`ANALYZED`; Codex never makes that transition. The Human Leader may close the
task or approve a new child task. `BLOCKED` and `FAILED` identify the actor who
must act next.

---

# 5. Task File Structure

Use the following structure. The reusable copy is
`workflow_v2/TASK-TEMPLATE.md`; project task files are copied from it. Lifecycle
`status` and `current_owner` live in front matter. Human approval and its exact
scope live in `### 1.11 Authorization`, not in a second front-matter field.
Authorization stays `APPROVED` as the front-matter lifecycle status advances.

```markdown
---
task_id: TASK-XXX
title: <short descriptive title>
status: DRAFT
current_owner: strategic-analyst
parent_task: <TASK-XXX | none>
analysis_id: <stable-analysis-id | TBD>
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
---

# TASK-XXX — <Task Title>

## 1. STRATEGIC SPECIFICATION

### 1.1 Objective
State exactly what this task is trying to determine, accomplish, or validate.

### 1.2 Context
Provide only the context needed to understand why the task exists.

Reference existing repository artifacts instead of duplicating large amounts of information.

### 1.3 Strategic Question / Hypotheses
Define the question being investigated.

Where useful:

- H1:
- H2:
- H3:

This may be omitted for purely mechanical implementation tasks.

### 1.4 Required Evidence / Deliverables
Define what Codex must obtain or produce before execution can be considered complete.

Examples:

- benchmark outputs;
- CPU/GPU/rank mappings;
- profiling data;
- configuration comparisons;
- validated scripts;
- correctness results;
- raw logs;
- extracted metrics.

### 1.5 Relevant Inputs and References
List the artifacts Codex should begin with.

Prefer exact repository paths, experiment IDs, commit IDs, and other precise references.

### 1.6 Execution Scope

#### Allowed
Codex and workers may:

- ...
- ...

#### Prohibited
Codex and workers must not:

- ...
- ...

Anything outside the approved scope requires escalation.

### 1.7 Execution Constraints
Examples:

- read-only investigation;
- no PBS submissions;
- maximum number of jobs;
- permitted queues;
- sequential submissions only;
- preserve failed attempts;
- specific correctness criteria;
- permitted repository write locations.

### 1.8 Success Criteria
Execution is complete when:

- ...
- ...

These criteria define execution completeness, not whether a hypothesis is scientifically proven.

### 1.9 Stop / Escalation Conditions
Codex must stop and return control if:

- required evidence cannot be obtained within scope;
- contradictory operational evidence cannot be resolved;
- an unauthorized job or configuration change becomes necessary;
- correctness becomes uncertain;
- a new experimental direction would be required;
- execution would exceed approved scope.

### 1.10 Strategic Analyst Notes
Optional guidance about priorities or decomposition.

### 1.11 Authorization
status: <DRAFT | APPROVED>

approved_scope: <brief exact description>

approved_by: <user | null>

---

## 2. CODEX EXECUTION REPORT

> This section is owned by Codex.
>
> Codex may append to this section but must not rewrite the Strategic Specification.

### 2.1 Execution Status
status: <COMPLETE | PARTIAL | BLOCKED | FAILED>

### 2.2 Orchestration Summary
Record:

- number of workers used;
- broad responsibility assigned to each;
- parallel versus sequential work;
- follow-up worker work issued by Codex.

Do not include unnecessary full worker transcripts.

### 2.3 Work Executed
Concise factual record of what was actually done.

### 2.4 Operational Validation
Record, where applicable:

- requested evidence collected: YES / NO / PARTIAL
- correct experiment/run IDs verified: YES / NO
- expected files present: YES / NO
- correctness criteria satisfied: YES / NO / N/A
- provenance verified: YES / NO
- worker outputs internally consistent: YES / NO
- unresolved factual conflicts: <none / details>
- scope violations: <none / details>

### 2.5 Evidence and Artifacts
Reference the actual evidence.

Examples:

- `experiments/.../outputs/...`
- `results/metrics.csv`
- `scripts/outputs/...`
- `profiling/...`
- commit `<sha>`

Do not duplicate large raw outputs here.

### 2.6 Files Changed
List repository files created or modified.

If none:

`None.`

### 2.7 Missing / Unavailable Evidence
List requested evidence that could not be obtained.

If none:

`None.`

### 2.8 Execution Errors / Exceptions
Record failures, retries, worker failures, unexpected conditions, or operational exceptions.

If none:

`None.`

### 2.9 Scope Compliance
State whether execution remained inside the approved scope.

If execution stopped because additional authority was required, explain exactly what action would be necessary.

### 2.10 Handoff to Strategic Analyst
State only what the Strategic Analyst needs to know before reading the evidence.

Do not perform strategic interpretation here.
```

---

# 6. Strategic Analysis

Strategic analysis is not stored in the task file.

It belongs in the existing planning hierarchy:

```text
planning/
├── PLANS.md
└── analysis/
    └── <analysis-id>.md
```

This keeps the task file compact and operational.

It also allows one analysis to synthesize evidence from multiple tasks.

Example:

```text
TASK-021 → affinity evidence
TASK-022 → communication evidence
TASK-023 → controlled A/B experiment
                │
                ▼
planning/analysis/host-resource-regression.md
```

---

# 7. Analysis File Structure

Keep the analysis structure deliberately simple.

```markdown
---
task_id: TASK-XXX
title: <relevant title>
analysis_id: <stable analysis id>
status: COMPLETE
parent_task: <TASK-XXX | none>
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
---

# Analysis — <Title>

## 1. Summary

A concise, human-readable summary of:

- what was investigated;
- what evidence was available;
- the main findings;
- what remains uncertain;
- the strategic implication;
- the proposed next action, if any.

This section should make sense without reading the full analysis.

## 2. Analysis

The Strategic Analyst chooses the internal structure based on the question and evidence.

Possible contents include:

- quantitative comparisons;
- trends;
- hypothesis evaluation;
- causal reasoning;
- alternative explanations;
- failed or inconclusive evidence;
- uncertainty and limitations;
- technical implications;
- recommended next action;
- provenance links.
```

The analysis format is intentionally flexible.

The Strategic Analyst should organize the reasoning naturally rather than forcing every task into a rigid template.

---

# 8. Writing Style

All task and analysis files should use technically precise but human-readable engineering prose.

Prefer:

- direct sentences;
- clear causal qualifiers;
- explicit values;
- exact technical terminology where useful;
- compact tables where they improve readability;
- clear distinction between observation and interpretation.

Avoid:

- unnecessarily dense academic prose;
- vague agent-style abstractions;
- inflated wording;
- excessive jargon when simpler language is equally precise.

Example to avoid:

> The observed degradation is temporally correlated with a divergent host-affinity topology that may induce suboptimal locality characteristics across heterogeneous execution resources.

Preferred:

> The slower run used a different CPU-affinity layout. This may reduce NUMA locality or MPI progress, but the current evidence does not show which effect is responsible.

Readable language must never erase uncertainty.

---

# 9. Raw Evidence and Canonical Data Locations

Raw evidence should remain in the existing workflow locations.

Examples:

```text
experiments/<run-name>/outputs/
results/metrics.csv
results/RESULTS.md
scripts/outputs/
profiling/
builds/.../outputs/
```

Do not copy large raw outputs into task files.

Task files reference evidence.

Strategic analysis should read the raw evidence directly whenever practical rather than relying only on the Codex Execution Report.

The Codex report is operational context, not the analytical source of truth.

---

# 10. Analysis Authorization

Strategic analysis remains explicitly authorized.

The execution layer does not automatically proceed into strategic interpretation merely because a task completed.

A typical boundary remains:

```text
Codex execution completes
        ↓
raw evidence logged and results updated where applicable
        ↓
Codex Execution Report completed
        ↓
task → EXECUTED; current_owner → strategic-analyst
        ↓
latest repository revision available
        ↓
human authorizes analysis
        ↓
Strategic Analyst reads task and raw evidence, then persists analysis
        ↓
task → ANALYZED; current_owner → user
        ↓
Human Leader decides next action
```

An analysis recommendation is not execution permission.

---

# 11. Next Actions and Task Lineage

A new strategic action should normally create a new task file.

Example:

```text
TASK-003
   ↓
analysis concludes a controlled A/B test is needed
   ↓
Human Leader approves
   ↓
TASK-004
```

Use:

```text
parent_task: TASK-003
```

to preserve lineage.

Do not silently redefine the purpose of an existing task.

A task should remain a record of the bounded objective it originally represented.

---

# 12. Codex Orchestration Rules

Codex may decide:

- how many workers to use;
- which work should run in parallel;
- which work depends on prior results;
- whether a worker needs a follow-up prompt;
- whether Codex should perform a trivial operation directly;
- whether additional operational validation is needed.

Codex should prefer parallelism for independent read-only work.

Example:

```text
                  Codex
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
   comm logs    CPU mapping    GPU telemetry
```

Dependent work should remain sequential.

Example:

```text
discover run IDs
      ↓
inspect identified files
      ↓
validate provenance
```

---

# 13. Worker Independence

Workers should normally not talk directly to one another.

Preferred:

```text
Worker A ──┐
Worker B ──┼──► Codex
Worker C ──┘
```

Avoid:

```text
Worker A ↔ Worker B ↔ Worker C
```

This reduces:

- contamination between evidence streams;
- anchoring;
- accidental scope expansion;
- unclear ownership.

If one worker needs information discovered by another, Codex should pass only the necessary verified information.

---

# 14. Parallel Read vs Parallel Write

For the initial implementation:

- parallel read-only probing is allowed when tasks are independent;
- shared-file writes should be serialized by Codex;
- multiple workers should not edit the same file concurrently.

Later versions may introduce:

- separate Git branches;
- Git worktrees;
- isolated worker sandboxes.

Those are not required for the first working version.

---

# 15. Git Synchronization Across Layers

The Git repository acts as the durable transport between strategic and execution layers.

Before Codex executes a task:

1. ensure the Human Leader-approved task content has been materialized as
   `tasks/TASK-XXX.md`;
2. commit and push the approved task according to the project Git policy;
3. synchronize the local repository according to the project Git policy;
4. verify the intended `TASK-XXX.md` revision is present;
5. verify front matter is `status: APPROVED` with `current_owner: codex`;
6. verify `### 1.11 Authorization` records `status: APPROVED`, the exact
   approved scope, and `approved_by: user`;
7. verify that scope matches the user's instruction.

After Codex completes the Execution Report and marks the task `EXECUTED`:

1. validate the file;
2. commit/push when authorized and required by project policy;
3. ensure the Strategic Analyst reads the latest repository revision.

This prevents stale task specifications or stale execution reports from crossing layers.

---

# 16. Scope and Escalation

Codex and workers must never silently expand task scope.

Example:

Strategic Specification:

```text
Prohibited:
- submit new PBS jobs
```

During execution Codex determines that a diagnostic run is required.

Correct behavior:

```text
Execution blocked.

Required evidence cannot be obtained from existing artifacts.

Additional action required:
- submit one diagnostic PBS job with ...

This action is outside the approved scope.
```

Incorrect behavior:

- silently submit the job;
- modify the task specification;
- reinterpret the prohibition.

The task returns to the Strategic Analyst and Human Leader.

---

# 17. Failure Handling

Operational failures should continue to follow the existing HPC workflow's error classification and evidence-preservation rules.

Codex should:

- preserve failed attempts;
- record execution errors;
- avoid automatic strategic interpretation;
- stop on conditions requiring human or strategic judgment;
- append the blocked/failure state to the active task.

Execution failure is not automatically a scientific conclusion.

---

# 18. Relationship to Existing HPC Workflow

This architecture does not replace the existing HPC workflow safeguards.

It wraps orchestration around them.

Existing concepts remain valid, including:

- repository synchronization;
- build/run directory contracts;
- PBS evidence preservation;
- correctness validation;
- result extraction;
- error-patching boundaries;
- `ANALYSE_RESULTS`;
- dependency checkpoints;
- human approval of next optimization direction;
- progress/session handoff.

The new architecture primarily changes:

- who owns strategic analysis;
- how approved work is represented;
- how Codex delegates execution;
- how execution returns structured evidence to strategy.

---

# 19. Example End-to-End Flow

## Step 1 — Strategic discussion

The Human Leader and Strategic Analyst inspect existing data.

They identify a question:

> Is the observed performance regression associated with host/rank placement rather than communication transport?

## Step 2 — Strategic Specification

The Strategic Analyst drafts the complete task-file content from
`workflow_v2/TASK-TEMPLATE.md` for Human Leader review:

```text
TASK-012.md content
```

with:

- objective;
- hypotheses;
- required evidence;
- allowed actions;
- prohibited actions;
- success criteria;
- escalation conditions.

Status:

```text
DRAFT
```

## Step 3 — Human approval

The Human Leader reviews, modifies if needed, and explicitly approves the
specification. With authorized direct GitHub access, the Strategic Analyst
materializes the approved task. Otherwise the Human Leader writes it or
authorizes a repository agent to copy the exact approved content. Git writes
follow project policy and human authorization.

After approval:

```text
status: APPROVED
current_owner: codex
```

The Authorization section separately records `status: APPROVED`,
`approved_scope`, and `approved_by: user`.

## Step 4 — Codex orchestration

Codex verifies the approved front matter and Authorization section, moves the
task to `EXECUTING`, and decides the work can be split into:

- CPU/NUMA mapping;
- launcher/process placement;
- phase timing and telemetry.

Codex launches three bounded workers.

No per-worker task files are created.

## Step 5 — Worker execution

Workers inspect the requested evidence.

They return:

- factual observations;
- values;
- evidence paths;
- missing data;
- execution errors.

They do not infer the overall root cause.

## Step 6 — Codex validation

Codex checks:

- all requested runs were examined;
- files are present;
- provenance is correct;
- correctness conditions are satisfied;
- worker facts are not contradictory;
- task scope was respected.

If necessary, Codex issues bounded follow-up worker prompts.

## Step 7 — Codex Execution Report

Codex completes the Execution Report section in:

```text
tasks/TASK-012.md
```

The report references raw artifacts.

It does not contain strategic interpretation.

After operational validation and the report are complete, Codex moves the
task to `EXECUTED` with `current_owner: strategic-analyst` and makes the
latest revision available under the Git policy.

## Step 8 — Analysis authorization

The Human Leader authorizes analysis.

## Step 9 — Strategic analysis

The Strategic Analyst reads:

- `TASK-012.md`;
- referenced experiment outputs;
- `results/metrics.csv`;
- relevant prior analysis;
- any required raw logs.

With authorized direct GitHub access, it creates or updates:

```text
planning/analysis/<analysis-id>.md
```

It also updates `planning/PLANS.md` when applicable. Without direct write
access, the Human Leader or an authorized mechanical agent materializes the
exact analysis and task metadata update. Once analysis is complete and
persisted, the Strategic Analyst is responsible for the task's
`status: ANALYZED` and `current_owner: user` transition.

## Step 10 — Human decision

The Strategic Analyst reports:

- findings;
- uncertainty;
- hypothesis status;
- proposed next action.

The Human Leader decides whether to:

- close the issue;
- reopen a previous direction;
- create a new task;
- approve a controlled experiment;
- stop the investigation.

Closing the task sets `status: CLOSED` with `current_owner: user`.

---

# 20. Core Invariants

The following rules should remain true even as the workflow evolves.

## Invariant 1 — Human strategic authority

No agent autonomously changes the optimization campaign's strategic direction.

## Invariant 2 — Strategy and execution are separate

The Strategic Analyst decides what evidence is needed.

Codex decides how to obtain it within approved scope.

## Invariant 3 — Workers are bounded

Workers gather evidence and perform explicitly approved work.

They do not own campaign interpretation.

## Invariant 4 — Raw evidence remains canonical

Strategic analysis reads original data whenever practical.

Summaries are not substitutes for evidence.

## Invariant 5 — Codex validates, not interprets

Codex guarantees that the Strategic Analyst receives trustworthy, well-indexed evidence.

It does not tell the Strategic Analyst what that evidence means strategically.

## Invariant 6 — Persistent files exist only where useful

Persistent files are used at architecture boundaries.

Transient worker interactions remain transient.

## Invariant 7 — Scope never expands silently

If execution requires broader authority, the task stops and escalates.

## Invariant 8 — Readability matters

Technical records must remain accurate, concise, and human-readable.

## Invariant 9 — Evidence and uncertainty are preserved

Invalid runs, failed attempts, missing data, contradictory evidence, and uncertainty must not be hidden.

## Invariant 10 — New strategic actions create new tasks

Task history should remain immutable in purpose.

---

# 21. Initial Implementation Scope

The first implementation should remain deliberately simple.

Implement:

- `tasks/`;
- canonical task-file format;
- Strategic Analyst draft → Human-approved repository task handoff, with
  direct Strategic Analyst writing when authorized;
- Codex orchestration;
- transient OpenCode/GLM workers;
- Codex Execution Report;
- strategic analysis in existing `planning/analysis/`;
- human approval gates;
- Git synchronization between layers.

Do not initially implement:

- automatic model-routing heuristics;
- complex multi-provider scheduling;
- autonomous task queues;
- agent dashboards;
- distributed worker state databases;
- parallel shared-file editing;
- automatic campaign-level goal execution.

Those can be added after the core architecture has been tested successfully.

---

# 22. Summary

The complete architecture is:

```text
USER — Human Leader
        ↕
ChatGPT Web / Sol — Strategic Analyst
        │
        │ draft Strategic Specification content
        ▼
Human Leader reviews / explicitly approves
        │
        │ Strategic Analyst writes directly or approved fallback materializes;
        │ commit + push under Git policy
        ▼
tasks/TASK-XXX.md
        │
        ▼
Codex — Orchestrator
        ↕
OpenCode / GLM — bounded execution workers
        │
        ▼
raw evidence in canonical repository locations
        │
        ▼
Codex — operational validation
        │
        │ completes Codex Execution Report
        ▼
tasks/TASK-XXX.md
        │
        │ EXECUTED; current_owner: strategic-analyst
        ▼
Human authorizes analysis
        │
        ▼
ChatGPT Web / Sol — Strategic Analyst
        │
        ▼
planning/analysis/<analysis-id>.md
        │
        │ ANALYZED; current_owner: user
        ▼
USER — final strategic decision
```

The philosophy is simple:

> **Automate execution aggressively. Centralize interpretation in the strongest reasoning layer. Preserve raw evidence. Keep strategic authority with the human.**
