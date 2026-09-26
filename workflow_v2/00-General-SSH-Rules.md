# General SSH and Cluster Execution Rules

This document defines the safe remote-execution policy for the workflow pack.
It contains universal rules followed by values that must be adapted for each
new cluster. Preserve the universal rules when adapting this file.

## Universal authentication rules

- Never request, read, store, transmit, echo, or log any SSH password.
- Never use `sshpass`, Expect, password files, clipboard extraction,
  environment variables, or command-line password arguments for authentication.
- Never attempt to modify SSH authentication settings.
- Never automate password entry.
- Never install tools merely to bypass interactive authentication.
- Never store cluster credentials or GitHub credentials on the cluster.

## Required connectivity and optional persistence

- Before remote work, run the configured safe, non-interactive, read-only
  connectivity check, typically:

  ```bash
  ssh -o BatchMode=yes <ssh-alias> '<read-only-command>'
  ```

- Direct non-interactive SSH is the required baseline and is sufficient.
- ControlMaster / ControlPath is optional performance infrastructure. If used,
  `ssh -O check <ssh-alias>` may check it. Failure, including
  `No ControlPath specified for "-O" command`, must not block execution when
  the required direct check succeeds. Fall back to the configured direct form.
- `ssh -MNf <ssh-alias>` is not universally required; optional setup must use
  reviewed project configuration and must not modify authentication settings.
- If direct non-interactive authentication genuinely requires human interaction,
  stop the affected remote work and report the required Human action without
  requesting or handling secrets.
- Do not initiate a normal interactive SSH login.
- Every Codex-controlled remote command must use the documented non-interactive
  SSH form.
- Every Codex-controlled file transfer must use the documented non-interactive
  SCP or rsync form.

## Remote scope and execution rules

- Keep remote work inside the approved project root unless the user explicitly
  approves another path.
- Do not use `sudo`.
- Do not install Codex, package managers, background services, daemons, proxies,
  or remote agents on the cluster.
- Do not install new software or packages without user approval.
- Do not modify shared software. Prefer available modules and site-supported
  tools.
- During workflow execution, create or edit only approved project files and
  designated output directories and configured execution-worktree infrastructure.
  Do not delete folders without user approval, except verified disposable
  workflow-created worktrees under `01-Git-Sync-Policy.md` and project policy.
- Run computation through the cluster scheduler via batch jobs.
- Do not perform computational workloads on login nodes.
- Do not poll the scheduler excessively; use bounded monitoring.

## Cluster configuration — complete before use

Replace every placeholder below. This section is the active cluster adapter.

- Cluster name: `<cluster-name>`
- SSH alias: `<ssh-alias>`
- Required non-interactive connectivity check: `<read-only-command>`
- Required SSH command form: `<ssh-command-form>`
- Optional persistent-connection check: `<command | none>`
- Optional persistent-connection setup: `<command | none>`
- Required SCP form: `<scp-command-form>`
- Required rsync form, if used: `<rsync-command-form>`
- Remote project root: `<remote-project-root>`
- Cluster primary Git clone path: `<primary-clone-path>`
- Configured Git remote/ref: `<git-remote>` / `<git-ref>`
- Remote execution-worktree root: `<remote-project-root>/.codex-worktrees/`
  (adapt and authorize the exact path during `SETUP`)
- Scheduler: `<PBS|Slurm|other>`
- Scheduler submission command: `<submission-command>`
- Scheduler monitoring command and polling limit: `<monitoring-policy>`
- MPI or application launcher: `<launcher>`
- Module policy: `<module-policy>`
- Login-node restrictions: `<login-node-policy>`
- Compute-node execution restrictions: `<compute-node-policy>`
- Approved remote paths: `<approved-paths>`

## Cluster adaptation checks

Before the first remote action, verify that:

1. all placeholders have been replaced;
2. the required direct connectivity check is read-only and correct, and
   persistence is optional (configured or `none`);
3. SSH and file-transfer commands are non-interactive;
4. the remote root is exact and sufficiently narrow;
5. scheduler commands use batch execution;
6. launcher, modules, and resource syntax match the cluster;
7. no rule asks Codex to handle or expose authentication secrets;
8. primary clone, remote/ref, execution-worktree root, and routine Git commands
   are configured, including whether cluster-side fetch works without secrets.

If any check fails, stop before remote work.
