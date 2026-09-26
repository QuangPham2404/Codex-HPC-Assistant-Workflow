# Git Synchronization and Execution Isolation Policy

This policy governs reviewed repository state, execution, and evidence transport
between the local clone and cluster. Project `AGENTS.md` defines exact commands,
remote/ref, paths, and restrictions; do not assume a particular remote or branch.

## Clone roles and authoritative revision

The local clone is preferred for preparation, review, commit, and push. The
cluster primary clone is a real Git clone and may contain user work, generated
evidence, or runtime artifacts. Clean execution worktrees provide isolation
without disturbing that primary working tree. Never store Git credentials on
the cluster or substitute an untracked copy for reviewed Git state.

Identify the exact approved repository commit from the explicit task handoff
and project policy, not merely the latest remote tip. Record the full commit,
task revision, and scripts used. Reviewed in-scope changes must be validated,
committed, and pushed when project policy requires before remote use; record
any resulting execution revision and verify unchanged strategic scope.

## Startup and task authority

Inspect `git status` and the revision before synchronization in either environment.
Verify the explicitly identified task has `current_owner: codex` and either
`status: APPROVED` for initial execution or `status: EXECUTING` for resume.
Section `1.11 Authorization` must still record `status: APPROVED`,
`approved_by: user`, and the unchanged approved scope. Conversation drafts are
not executable state. Human approval precedes task materialization by the
Strategic Analyst or authorized mechanical fallback and commit/push under
project policy. Never execute a stale or conflicting Strategic Specification.

## Select a safe execution tree

After the required direct SSH connectivity check:

1. Record the cluster primary clone path, HEAD, branch/ref, tracked changes,
   untracked and relevant ignored artifacts, and existing worktrees. Inspect
   enough state to determine whether synchronization could affect user material.
2. If the primary clone is sufficiently clean and safely fast-forwardable,
   fetch/synchronize using the reviewed project policy. Use only fast-forward
   synchronization, such as `git pull --ff-only <git-remote> <git-ref>`.
   Verify HEAD is the exact intended execution commit before use. A clean tree
   at a different commit is insufficient; use isolation if appropriate.
3. If the primary clone has pre-existing modified tracked files, untracked
   runtime artifacts, generated evidence, unrelated work, or cannot safely
   fast-forward to the intended commit, preserve its working tree unchanged.
   Record the state, then update remote refs non-destructively with the configured
   `git fetch <git-remote>` mechanism when authorized. Fetch and worktree
   registration may update Git metadata but must not alter primary files.
   A configured nested runtime root may gain new uniquely named worktree
   directories; preserve all pre-existing primary content and its index.
4. Resolve and verify the exact approved commit, then create a clean detached
   execution worktree inside the configured, approved root. Generic example,
   run from the primary clone with the adapted path:

   ```bash
   git worktree add --detach \
     .codex-worktrees/TASK-XXX-<shortsha> \
     <exact-approved-commit>
   ```

   Use a unique suffix if that path already exists; never overwrite it.
5. Verify the execution tree HEAD, task approval/scope, scripts, application
   revision, required files, and suitable Git state before execution. Record
   its path and revision in the Execution Report and progress handoff. Use
   paths from this tree for submissions and retrieval; do not silently fall
   back to primary scripts or artifacts. Required external inputs must be
   obtained through approved paths without overwriting user material.

A dirty primary clone alone is not a blocker. Never use `git reset --hard`,
`git clean`, automatic stash/pop, automatic merge, destructive checkout,
file deletion, or overwrite of user material to make it executable. Do not
create automatic merge commits. Inspect failed fast-forwards; isolate when
possible rather than forcing reconciliation of unrelated primary history.

## Worktree reuse, evidence, and cleanup

Reuse only a workflow execution worktree associated with the intended task,
pointing at the exact intended revision, with suitable Git state and no
unresolved or unique evidence. Otherwise create a uniquely named worktree.
Do not reset an old worktree containing evidence to reuse its directory.
Safe recreation uses a new directory unless the old tree is verified disposable.

Outputs remain authoritative evidence even in ignored runtime infrastructure.
Record exact worktree paths, retrieve/persist outputs in canonical project
locations with provenance, and preserve task reports and progress changes
through the reviewed Git policy. On resume, consult durable progress, existing
jobs, and evidence before rerunning anything; never duplicate submissions merely
because a session restarted.

Automatic `git worktree remove` is allowed only under configured project policy
for a workflow-created worktree verified to have no uncommitted tracked changes,
no unique untracked evidence, and no outputs (including ignored files) still
needing retrieval or persistence. Otherwise preserve it. Do not force removal.
Cleanup must never affect the primary clone or user-created worktrees.

## Routine authority and real blockers

Once configured commands are authorized during `SETUP`, ordinary non-destructive
inspection, fetch, revision verification, safe worktree creation/reuse, verified
disposable worktree removal, and clean-primary fast-forward synchronization
are normal task authority; no fresh approval is needed each session. They
remain bounded by root `AGENTS.md` and the approved task. Recovery stays
`EXECUTING / codex` when Codex can safely complete it under existing authority.

Stop the affected work when the intended revision cannot be fetched or identified,
histories genuinely conflict in required authoritative state, approved task
content differs between authoritative states, safe worktree creation fails,
Git authentication requires Human interaction, required files cannot be obtained,
or recovery would overwrite/delete user work. Inspect and perform deterministic
safe recovery first when available; use `BLOCKED` only when another actor
actually must act. Never infer approval for destructive conflict resolution.

## Review and evidence handoff

Before committing, inspect status, diff, and file list; validate appropriately.
Commit only reviewed, useful scripts, documentation, metadata, logs, and results.
Exclude unrelated changes and temporary files, and preserve required failure
evidence. Transfer results with the configured non-interactive file-transfer
command into matching canonical local directories, retaining provenance.

After completing the Execution Report, validate it and its evidence references,
commit/push when required and authorized, and ensure the Strategic Analyst can
read the latest repository revision before authorized analysis.
