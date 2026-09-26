# Codex HPC Assistant Workflow

This repository contains two versioned workflow packs for Codex-assisted HPC
optimization. The authoritative design for the new architecture is
[`AGENTIC_HPC_ARCHITECTURE.md`](AGENTIC_HPC_ARCHITECTURE.md).

## Versions

- [`workflow_v2/`](workflow_v2/) is the current recommended workflow for fresh
  and existing projects. It adds a task-based,
  layered handoff between a Human Leader, Strategic Analyst, Codex
  Orchestrator, and bounded execution workers while preserving the proven HPC
  safety and evidence rules.
- [`workflow_v1/`](workflow_v1/) is the preserved original Phase-1,
  single-agent workflow. It remains available for reference and compatibility
  and is not retroactively updated with v2 role boundaries.

The v2 flow is:

```text
Human Leader ↔ Strategic Analyst
                    ↓ draft task content
              Human review / approval
                    ↓ Strategic Analyst writes approved task directly
                      or approved fallback materializes it
              commit + push
                    ↓
              tasks/TASK-XXX.md
                    ↓
              Codex Orchestrator ↔ execution workers
                    ↓
              Codex operational validation
                    ↓
              Codex Execution Report
                    ↓
              Human-authorized Strategic Analyst analysis
                    ↓
              Human strategic decision
```

Task files contain the Strategic Specification and Codex Execution Report.
Raw benchmark and probe evidence remains in the project's canonical output
locations, while strategic analysis remains under `planning/analysis/`.
With authorized GitHub access, the Strategic Analyst writes the task after
human approval and later writes authorized analysis. If direct writing is
unavailable, the Human Leader or an authorized mechanical repository agent
materializes the exact approved content. Codex executes only synchronized,
approved repository state.

Workflow v2 uses direct non-interactive SSH as baseline connectivity, with
optional persistent SSH for performance. It preserves dirty primary clones
through clean isolated execution worktrees and resumes `EXECUTING / codex`
tasks across sessions under unchanged Section 1.11 Human approval. `SETUP`
configures these behaviors, routine synchronization permissions, and the
project's `.codex-worktrees/` ignore rule. Safe deterministic recovery stays
with Codex; meaningful authorization and strategic decisions remain human-led.

## Adoption

For a fresh project, download the recommended v2 package as a directory:

```bash
npx degit QuangPham2404/Codex-HPC-Assistant-Workflow/workflow_v2 my-project/workflow
```

Copy or adapt `workflow/AGENTS_EXAMPLE.md` as root `AGENTS.md`, then invoke
`SETUP`. The canonical task format is
[`workflow_v2/TASK-TEMPLATE.md`](workflow_v2/TASK-TEMPLATE.md); `SETUP` proposes
the required repository structure before any changes.

For an existing project, install v2 as `workflow/` and surgically adapt the
existing root `AGENTS.md` against `workflow/AGENTS_EXAMPLE.md`. Preserve its
project-specific operational rules, replace obsolete role/startup logic,
restart Codex so it reads the updated instructions, then invoke `SETUP` for
repository-wide adaptation. See the
[`workflow_v2/` guide](workflow_v2/README.md) for the bootstrap details.
Complete cluster placeholders and project permissions before remote work.

To adopt the preserved v1 workflow instead:

```bash
npx degit QuangPham2404/Codex-HPC-Assistant-Workflow/workflow_v1 my-project/workflow
```

Use v1 when reproducing or studying the original workflow; use v2 for fresh
or existing projects unless compatibility with the Phase-1 package is required.

## Repository contents

The architecture document describes the intended division of responsibility.
Each versioned directory is independently readable as a reusable workflow
pack, including the numbered rules, `AGENTS_EXAMPLE.md`, package README, and
optional cluster references. Workflow v2 also contains the canonical task
template.
