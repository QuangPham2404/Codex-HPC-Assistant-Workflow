# Optional Cluster References

This directory may contain reference profiles for individual clusters. It is
not part of the mandatory workflow-pack reading pass.

The active cluster configuration must be completed in
`workflow/00-General-SSH-Rules.md`. The project root `AGENTS.md` may identify
the active cluster and add stricter project-specific rules. Those active files
are authoritative.

Reference profiles must not weaken the universal safety rules, grant project
permissions, or replace the active configuration in `00-General-SSH-Rules.md`.

Adaptations must define the required direct non-interactive read-only connectivity
check and SSH/transfer forms; persistent check/setup is optional or `none`.
`SETUP` configures primary clone, Git remote/ref and fetch capability, execution-
worktree root, routine synchronization permissions, and the downstream root
`.codex-worktrees/` ignore rule. Preserve dirty primary state through isolation;
do not reintroduce mandatory ControlMaster, unconditional dirty-tree stops,
or startup rules that prevent approved `EXECUTING / codex` tasks from resuming.
