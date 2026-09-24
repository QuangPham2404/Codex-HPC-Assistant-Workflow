# Workflow v2

This is the complete task-based operational sequence. Follow the current
task, the supporting rules in the other numbered workflow files, and the
project instructions. Do not skip evidence, validation, authorization, or
handoff requirements because a job exits zero.

The workflow wraps the proven HPC build, synchronization, execution, result
logging, correctness, provenance, and failure-handling rules in a layered
architecture:

```text
Strategic discussion
        ↓
Strategic Specification
        ↓
Human approval
        ↓
Codex orchestration
        ↓
existing build / sync / run / result logging mechanisms
        ↓
Codex Execution Report
        ↓
human authorizes analysis
        ↓
Strategic Analyst analyzes raw evidence
        ↓
planning/analysis/
        ↓
Human strategic decision
```

Codex is the operational orchestrator. It may decompose approved work,
coordinate bounded workers, and validate facts. It must not perform campaign-
level interpretation, choose a new optimization direction, rewrite the
Strategic Specification, or promote a new baseline.

## Task files and ownership

Each bounded strategic action uses one project-root task file:

```text
tasks/TASK-XXX.md
```

The task file contains only:

1. `STRATEGIC SPECIFICATION`, created by the Strategic Analyst and approved by
   the Human Leader;
2. `CODEX EXECUTION REPORT`, appended by Codex after execution.

The task file is the execution contract and operational history for that
bounded action. Raw evidence remains in canonical locations such as
`experiments/.../outputs/`, `results/metrics.csv`, `scripts/outputs/`, and
`profiling/`; the task references those paths rather than duplicating them.
Worker communication is transient. Per-worker task or report Markdown files
should not normally be created.

Use the task lifecycle where applicable:

```text
DRAFT → APPROVED → EXECUTING → EXECUTED → ANALYZED → CLOSED
```

Use `BLOCKED` or `FAILED` when the execution state requires it. A task should
also identify its current owner, normally `strategic-analyst`, `codex`, or
`user`.

## Step 1: Strategic Specification and human approval

The Human Leader and Strategic Analyst determine the bounded question or
action. The Strategic Analyst owns raw-evidence interpretation, quantitative
analysis, hypotheses, causal reasoning, experiment design, and creation of the
Strategic Specification. The Strategic Analyst proposes; it does not authorize
its own proposal.

The Strategic Analyst creates a task file with the canonical front matter and
the `STRATEGIC SPECIFICATION` section. It should identify:

- objective and required context;
- strategic question or hypotheses, where relevant;
- required evidence and deliverables;
- exact inputs and repository references;
- allowed and prohibited actions;
- execution constraints;
- success criteria for execution completeness;
- stop and escalation conditions;
- optional decomposition guidance;
- authorization and approved scope.

The Human Leader reviews the task and approves, modifies, or rejects it. A
normal approved task records:

```yaml
status: APPROVED
current_owner: codex
approved_by: user
```

Codex must not execute a task that is only `DRAFT`, has ambiguous scope, or
cannot be unambiguously identified.

## Step 2: Codex orchestration and prepare build/run directories

Before execution, Codex reads the approved task and decomposes only the work
inside its scope. It decides whether to work directly or use bounded
execution workers, identifies dependencies, and runs independent read-only
work in parallel when safe. Dependent work remains sequential. Shared-file
writes are serialized; workers must not edit the same file concurrently.

Codex communicates with normally OpenCode / GLM workers through transient
prompts or sessions. Worker instructions should state the task, inputs,
allowed actions, required return facts and evidence paths, and prohibited
interpretation or scope expansion. Workers probe, inspect, execute, test,
modify explicitly approved files, extract measurements, and preserve raw
evidence. They do not recommend the next strategic action.

### 2.1 Prepare builds

Use `builds/source/`, `builds/build-scripts/`, and
`builds/extra-packages/`. For a new application, inspect the cluster and
clone source there on the login node using the documented direct command before
writing build scripts, unless the application guide says an existing source
tree only needs a build directory. Do not clone application source into the
local workflow repository when project policy forbids that.

Locally create `builds/build-scripts/<build_name>/` with its README, build PBS
script, build shell script when needed, and `outputs/`. Use one stable
descriptive name. The README includes summary, compiler environment, compiler,
MPI compiler, optimization notes, metadata, and build error-patching records.
PBS stdout/stderr go directly to `outputs/`; keep every attempt.

Before remote use, review and validate locally, commit and push when required,
pull remotely, and ensure the remote output directory exists.

### 2.2 Prepare experiments

Create `experiments/<run_name>/` with README, run PBS script(s), and `outputs/`.
The README includes purpose, successful binary path, compiler environment,
compiler, MPI compiler, optimization notes, input/resource metadata, and
runtime error-patching records. Raw PBS outputs remain in `outputs/`; extracted
results go to `results/`.

Preparation must remain within the approved task scope. If a needed build,
experiment, source edit, resource change, launcher change, or package action
is not approved, stop and report the exact additional authority required.

## Step 3: Synchronize the two repositories and verify the active task

Check both clones according to `01-Git-Sync-Policy.md`. Before Codex executes:

1. synchronize the local repository according to normal policy;
2. verify the explicitly identified `tasks/TASK-XXX.md` revision is present
   locally and remotely as applicable;
3. verify the task status is `APPROVED` and its scope matches the user's
   instruction;
4. ensure Codex is not acting on a stale Strategic Specification;
5. verify reviewed scripts, application revision, and required metadata are
   the versions intended for execution.

Do not infer the active task from whichever file is newest. If fast-forward
synchronization fails, clones diverge, the task revision is stale, or the
approval state is unclear, stop and inspect or report the conflict rather than
merging, overwriting, or changing the task.

## Step 4: Execute approved work on the cluster

Submit only reviewed and approved build, run, or probe scripts through the
scheduler. Ensure output directories exist. Use new attempt labels and
filenames for retries. Within scope, Codex may issue bounded follow-up worker
tasks, perform mechanical checks, and coordinate independent work.

For every job record experiment/build ID, PBS job ID, submission and completion
timestamps, final state, exit status, allocated node, stdout/stderr paths,
compiler/MPI/module metadata, flags, inputs, resources, runtime, and output
validation.

Build success requires scheduler success, expected output files, and expected
executable presence. Run success requires scheduler success, expected output
files, normal application output, and application-specific correctness or
convergence criteria. Never classify success from exit status alone.

After a failure, classify it using
`05-Workflow-Error-Patching-Procedures.md`. Record the failed attempt before
any permitted patch or retry. Track 1 retries follow the build/experiment
README procedure. Track 2 stops the affected workflow and records
`MANUAL_INSPECTION_ERROR.md`; if it blocks the active task, Codex records the
blocked state in the task's Execution Report. Strategic interpretation of the
failure remains outside Codex's role.

If execution requires an action outside the approved task, Codex must stop.
It must not silently submit a job, change resources or launcher settings,
modify the scientific question, or start a new optimization direction.

## Step 5: Log results and append the Codex Execution Report

At the end of build/run work, record results only. Do not perform optimization
analysis or update strategic planning conclusions in this step.

Use:

```text
results/
├── README.md
├── metrics.csv
├── RESULTS.md
└── scripts/
```

Before extraction, confirm expected experiment output and required fields are
present. `metrics.csv` is the structured source of truth. Preserve rows,
append attempts, reject duplicate `(experiment_id, attempt)` records, and
support failed/incomplete attempts when metadata permits. Keep raw `.o` and
`.e` provenance. Generate or update `RESULTS.md` from the CSV. Do not add
strategic interpretation to `planning/` during results logging.

After approved execution and operational validation, Codex appends to the same
active task file:

```markdown
## 2. CODEX EXECUTION REPORT

### 2.1 Execution Status

### 2.2 Orchestration Summary

### 2.3 Work Executed

### 2.4 Operational Validation

### 2.5 Evidence and Artifacts

### 2.6 Files Changed

### 2.7 Missing / Unavailable Evidence

### 2.8 Execution Errors / Exceptions

### 2.9 Scope Compliance

### 2.10 Handoff to Strategic Analyst
```

The report records facts, completeness, provenance, operational validation,
and scope compliance. It must reference raw evidence rather than duplicate
large outputs. It must not rewrite the Strategic Specification or perform
strategic interpretation. If execution is partial, blocked, or failed, record
that state and the exact missing authority or evidence.

Codex should normally record:

- execution status: `COMPLETE`, `PARTIAL`, `BLOCKED`, or `FAILED`;
- worker count, responsibilities, parallel/sequential work, and follow-ups;
- factual work performed;
- requested evidence and run IDs verified;
- expected files, correctness criteria, and provenance checks;
- worker consistency and unresolved factual conflicts;
- evidence paths and commit identifiers;
- files changed, missing evidence, errors, and exceptions;
- whether the approved scope was respected;
- the information the Strategic Analyst needs before reading raw evidence.

After appending the report, validate the task file and references. Commit and
push when required and authorized by normal Git policy, and ensure the
Strategic Analyst can read the latest repository revision. The report is
operational context, not the analytical source of truth.

## Step 6: Analyze only with `ANALYSE_RESULTS`

Strategic analysis begins only with explicit human authorization:

```text
ANALYSE_RESULTS
analysis_id: <stable analysis name>
source: results/metrics.csv
include: <experiments, variants, or attempts to include>
grouping: <optimization direction or grouped tests>
scope: <current analysis | current session>
restrictions: <additional limits>
```

`ANALYSE_RESULTS` authorizes the Strategic Analyst to read the selected
evidence, create or update `planning/analysis/<analysis-id>.md`, update
`planning/PLANS.md` when applicable, and present analysis. It does not
authorize jobs, source changes, configuration changes, or the next experiment.
If only `ANALYSE_RESULTS` is supplied, identify the available evidence and ask
which direction or group is in scope before writing a new analysis.

The Strategic Analyst reads, as applicable:

- the Strategic Specification;
- the Codex Execution Report;
- `results/metrics.csv` and `results/RESULTS.md`;
- raw experiment, probe, profiling, PBS, and build evidence;
- relevant prior analysis and repository context.

The Strategic Analyst should read raw evidence directly whenever practical,
rather than treating the Codex report as the analytical source of truth.
Analysis must preserve technical accuracy, provenance, correctness checks,
relevant failed or invalid data, uncertainty, baseline comparisons where
required, and a clear distinction between observation and inference. Do not
invent values or turn noisy evidence into a definitive conclusion.

Every new or substantially updated analysis file uses:

```markdown
---
task_id: ...
title: ...
analysis_id: ...
status: ...
parent_task: ...
created: ...
last_updated: ...
---

# Analysis — <Title>

## 1. Summary

...

## 2. Analysis

...
```

The internal structure of `## 2. Analysis` is flexible. It may contain
quantitative comparisons, trends, hypothesis evaluation, causal reasoning,
alternative explanations, failed or inconclusive evidence, uncertainty,
technical implications, recommended next action, and provenance links.

Use technically precise but human-readable engineering prose. Prefer direct
wording over unnecessarily dense academic or agent-style language. Use jargon
only when it improves precision. Never simplify away uncertainty.

`planning/PLANS.md` may retain its established three main sections:
`## Current baseline`, `## Optimization directions`, and `## Next direction`.
The suggested next action is a recommendation for Human Leader review, not
execution permission. A new strategic action normally creates a new task file
with `parent_task` linking it to the preceding task; do not silently redefine
the purpose of an existing task.

## Step 7: Human decision, session handoff, and resume

The Strategic Analyst presents findings, uncertainty, hypothesis status, and
any proposed next action. The Human Leader decides whether to close the issue,
create a new task, approve a controlled experiment, reopen a direction, or
stop the investigation. No agent autonomously changes the campaign direction,
promotes a baseline, or turns an analysis recommendation into execution
permission.

Before ending, write a dated
`progress/YYYY-MM-DD-progress[_sN].md` file. Include the active task and
status, current workflow step, completed work, decisions, build/experiment/
attempt IDs, PBS IDs, evidence paths, blockers, authorization state, and the
exact next action.

### Case 1: execution is incomplete

Preserve evidence and failed attempts. Record the current task, IDs, PBS job,
stdout/stderr, error or blocker, and manual-inspection case. State whether the
workflow awaits user action, external change, authorization, or a fix. Record
the exact resume action. Do not retry, patch, submit, change direction, or
widen scope without authorization. Validate, commit, and push the progress
record when normal Git policy applies, and synchronize before remote work
resumes.

### Case 2: analysis is complete

Record experiments, task and analysis IDs, findings, limitations, current
baseline, and next direction. Preserve historical evidence. Treat the
Strategic Analyst's proposed next action as a recommendation and wait for the
Human Leader's confirmation or modification before preparing the next task.
After approval of a new task, resume at Step 2; do not repeat Step 1 unless
this is a genuinely new application.
