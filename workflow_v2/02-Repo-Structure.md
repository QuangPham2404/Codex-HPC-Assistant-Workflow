# Repository Structure and Skeleton

Codex must use this document when creating, validating, or explaining the
workflow repository skeleton. Create the directories and documentation locally
first. Do not create a different structure merely because another layout seems
convenient.

## Canonical skeleton

```text
<project-root>/
├── AGENTS.md
├── APPLICATION.md
├── README.md
├── workflow/
├── tasks/
│   ├── README.md
│   └── TASK-XXX.md
├── builds/
│   ├── README.md
│   ├── source/
│   │   └── README.md
│   ├── build-scripts/
│   │   └── <build-name>/
│   │       ├── README.md
│   │       ├── build_<name>.pbs
│   │       ├── build_<name>.sh
│   │       └── outputs/
│   └── extra-packages/
│       └── README.md
├── experiments/
│   ├── README.md
│   ├── _template/
│   │   ├── README.md
│   │   └── run.pbs
│   └── <run-name>/
│       ├── README.md
│       ├── run_<name>.pbs
│       └── outputs/
├── planning/
│   ├── README.md
│   ├── PLANS.md
│   └── analysis/
├── results/
│   ├── README.md
│   ├── metrics.csv
│   ├── RESULTS.md
│   └── scripts/
├── progress/
│   ├── README.md
│   └── YYYY-MM-DD-progress.md
└── scripts/
    ├── README.md
    └── outputs/
```

Application-specific source and build trees, compiled artifacts, and approved
extra packages may be generated under designated paths, but generated trees
must follow the project's ignore policy and must not replace the documented
workflow directories.

## Directory roles

- `workflow/`: reusable workflow rules and templates. Read every numbered file
  before workflow action.
- `tasks/`: persistent handoff between the Strategic Analyst and Codex. Each
  bounded strategic action has one task file, such as `TASK-001.md`.
- `builds/source/`: application source cloned or copied for building different
  compilation methods. Follow the project rule for where source is acquired.
- `builds/build-scripts/`: reusable build scripts, build READMEs, PBS outputs,
  and build evidence. Each build variant gets its own stable directory.
- `builds/extra-packages/`: explicitly approved local dependencies or vendored
  packages. Never add packages without approval.
- `experiments/`: run-specific experiment directories. Each directory is one
  optimization run or run family and owns its raw PBS outputs.
- `planning/`: optimization hypotheses, priorities, tracker, and authorized
  analysis. Do not put optimization conclusions in `APPLICATION.md`.
- `results/`: extracted structured results and the generated human-readable
  report. It is not the place for optimization interpretation.
- `progress/`: dated session handoff and technical diary files.
- `scripts/`: general reusable scripts that do not naturally belong to builds,
  experiments, or result extraction. Probe outputs normally belong in
  `scripts/outputs/`.

Historical `records/` directories may be retained, but new workflows must not
depend on or create them. New PBS `.o` and `.e` evidence belongs in the
relevant `outputs/` directory.

## Build directory contract

Each new build directory must contain a concise README, the corresponding PBS
build script, the build shell script when applicable, and `outputs/`. The
README records the build purpose, compiler environment, compiler, MPI wrapper,
flags, metadata, and build error-patching attempts. Raw PBS stdout and stderr
are authoritative evidence and must not be overwritten on retries.

## Experiment directory contract

Each new experiment directory must contain a concise README, one or more run
PBS scripts, and `outputs/`. The README records the run purpose, successful
binary path, compiler/MPI environment, input and resource metadata, validation,
and runtime error-patching attempts. Extracted values go to `results/`, not
inside the experiment output directory.

## Results and planning contract

`results/metrics.csv` is the structured source of truth. Preserve existing
rows, append compatible attempts, reject duplicate `(experiment_id, attempt)`
records, and support failed or incomplete attempts when metadata permits.
Generate `RESULTS.md` from the CSV. `planning/PLANS.md` and detailed analysis
files are created or updated only in the applicable planning or authorized
analysis workflow.

## Task and analysis contract

One bounded strategic action corresponds to one task file under `tasks/`.
The task file is the persistent handoff between the Strategic Analyst and
Codex and contains only:

1. `STRATEGIC SPECIFICATION`, written by the Strategic Analyst and approved by
   the Human Leader;
2. `CODEX EXECUTION REPORT`, appended by Codex after execution.

Raw benchmark, probe, profiling, PBS, and result evidence stays in its
existing canonical location. Task files reference that evidence rather than
duplicating it. Worker communication inside the execution layer is transient;
per-worker task and report Markdown files are not part of this workflow.

Strategic analysis remains under `planning/analysis/`. Analysis files use this
simplified contract:

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

The internal structure of `## 2. Analysis` is flexible and should fit the
question and evidence.

## Naming contract

- Directories use kebab-case.
- Shell and PBS filenames use lowercase snake_case.
- Use one stable build or experiment name across README, scripts, job names,
  output names, and binary paths.
- Attempts use `<name>_v1`, `<name>_v1.1`, and `<name>_v1-final`.
- PBS evidence uses `<name>_v1.o` and `<name>_v1.e`, with new names for retries.

## Skeleton initialization behavior

When asked to create the skeleton, Codex should create missing directories,
README files, templates, and placeholders locally, while preserving existing
files and user changes. It must not clone source, submit jobs, install
packages, modify shared software, or invent application-specific correctness
criteria without the relevant project information and authorization.
