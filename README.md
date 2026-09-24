# Codex HPC Assistant Workflow

This repository contains two versioned workflow packs for Codex-assisted HPC
optimization. The authoritative design for the new architecture is
[`AGENTIC_HPC_ARCHITECTURE.md`](AGENTIC_HPC_ARCHITECTURE.md).

## Versions

- [`workflow_v2/`](workflow_v2/) is the recommended workflow for new projects
  and the current architecture under development. It adds a task-based,
  layered handoff between a Human Leader, Strategic Analyst, Codex
  Orchestrator, and bounded execution workers while preserving the proven HPC
  safety and evidence rules.
- [`workflow_v1/`](workflow_v1/) is the preserved original Phase-1,
  single-agent workflow. It remains available for reference and compatibility
  and is not retroactively updated with v2 role boundaries.

The v2 flow is:

```text
Human Leader ↔ Strategic Analyst
                    ↓ approved Strategic Specification
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

## Adoption

For a new project, select a version explicitly. The recommended v2 package can
be downloaded as a directory with:

```bash
npx degit QuangPham2404/Codex-HPC-Assistant-Workflow/workflow_v2 my-project/workflow
```

Then copy or adapt `workflow/AGENTS_EXAMPLE.md` to the project root as
`AGENTS.md`, create the project-root `tasks/` directory, and follow
`workflow/02-Repo-Structure.md` and `workflow/07-Workflow.md`. Complete the
cluster placeholders and project-specific permissions before remote work.

To adopt the preserved v1 workflow instead:

```bash
npx degit QuangPham2404/Codex-HPC-Assistant-Workflow/workflow_v1 my-project/workflow
```

Use v1 when reproducing or studying the original workflow; use v2 for new
projects unless compatibility with the Phase-1 package is required.

## Repository contents

The architecture document describes the intended division of responsibility.
Each versioned directory is independently readable as a reusable workflow
pack, including the numbered rules, `AGENTS_EXAMPLE.md`, package README, and
optional cluster references.
