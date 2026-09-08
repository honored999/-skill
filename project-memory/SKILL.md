---
name: project-memory
description: Maintain concise repository-level project state across Codex sessions, terminals, branches, and accepted worktree tasks. Use when `.project-memory/` exists, when repository instructions require project memory, when initializing repository memory, at repository-task startup, or when synchronizing verified status/goals/next actions/recent history before completion.
---

# Project memory

## Purpose

Maintain a small, durable, repository-local summary so a new Codex session can
quickly answer:

1. What is this project currently doing?
2. What is already verified or implemented?
3. What constraints or invariants matter?
4. What should happen next?

Project memory is a navigation and handoff layer. It is not a replacement for
source code, Git history, tests, `AGENTS.md`, specifications, experiment records,
or task HANDOFF evidence.

## Default layout

Use exactly this minimal layout unless project-specific instructions require
more:

```text
.project-memory/
├─ STATUS.md
├─ GOALS.md
├─ NEXT.md
└─ LOG.md
```

Do not add extra memory files merely for completeness.

## Canonical integration branch

Repository-level project memory is canonical on the configured integration
branch, for example `main` or `master`.

Repository instructions should declare it, preferably as:

`Integration branch: <INTEGRATION_BRANCH>`

When working directly on the integration branch, read the local
`.project-memory/`.

When working on another branch or in an independent worktree, prefer the
integration-branch memory as the latest accepted repository-level state, for
example:

```text
git show <INTEGRATION_BRANCH>:.project-memory/STATUS.md
git show <INTEGRATION_BRANCH>:.project-memory/GOALS.md
git show <INTEGRATION_BRANCH>:.project-memory/NEXT.md
```

Read recent `LOG.md` entries from the same canonical branch when useful.

The `.project-memory/` checked out inside a non-integration worktree is a
branch-local snapshot. Do not automatically treat it as the latest project-wide
state.

If the integration branch is not configured or cannot be determined safely, do
not guess. Treat local memory as branch-local and report the limitation when it
matters.

If the integration branch has accepted-but-uncommitted memory changes in another
worktree, `git show` cannot see them. Do not claim cross-worktree visibility that
Git does not provide.

## Authority

When sources disagree, prefer current authoritative evidence in this order as
applicable:

1. explicit current user instruction;
2. applicable `AGENTS.md` and task/spec documents;
3. current repository/worktree contents and Git state;
4. trustworthy tests/validation/runtime evidence;
5. canonical integration-branch `.project-memory/` summary;
6. branch-local memory snapshots.

Treat project memory as potentially stale until checked against the current
checkout for correctness-sensitive work.

## Size discipline

Keep memory short enough to read at task startup without consuming substantial
context.

Recommended limits:

- `STATUS.md`: roughly <= 80 lines;
- `GOALS.md`: roughly <= 50 lines;
- `NEXT.md`: roughly <= 50 lines;
- `LOG.md`: keep only the most recent 10 task entries, normally <= 120 lines.

These are soft limits. Prefer deleting stale summary text over letting memory
become a second documentation tree. Old history remains available in Git.

Never copy full conversations, long HANDOFFs, diffs, tracebacks, test logs,
training logs, generated reports, or large tables into project memory.

## File responsibilities

### `STATUS.md`

Describe the current verified project snapshot.

Prefer:

```markdown
# Status

Updated: YYYY-MM-DD

## Current state
- ...

## Verified capabilities/results
- ...

## Important constraints
- ...
```

Include only state that helps the next agent understand the repository quickly.
Do not list every file or historical change.

### `GOALS.md`

Store the stable project direction and current milestone.

Prefer:

```markdown
# Goals

## Project goal
- ...

## Current milestone
- ...

## Success criteria
- ...
```

Do not rewrite this file after every task. Update it only when project direction,
current milestone, or success criteria materially change.

### `NEXT.md`

Store immediate operational continuation.

Prefer:

```markdown
# Next

## Current focus
- ...

## Next actions
1. ...
2. ...

## Blockers
- None.
```

Keep next actions concrete and near-term. Do not turn this into a long roadmap.

### `LOG.md`

Keep a compact rolling record of recent completed/blocked repository tasks.

Prefer one entry like:

```markdown
## YYYY-MM-DD — short task name
- Changed: ...
- Validation: ...
- Result: ...
- Next: ...
```

Commit hash is optional when already known. Never create a second commit merely
to write the current memory commit's own hash into this log.

Keep only the most recent 10 entries unless project policy says otherwise.

## Initialization

Initialize `.project-memory/` only when:

- repository instructions require it; or
- the user explicitly asks for it.

For repositories using a generic `AGENTS.md` that requires project memory,
initialize it during the first repository-modifying task. Do not mutate a repo
for a purely read-only/advisory task merely to add memory files.

Before initializing:

1. read applicable `AGENTS.md`;
2. inspect `git status` and relevant recent history when available;
3. inspect the smallest set of project docs/source/tests needed to understand
   current state;
4. use verified facts only.

If important state is unknown, write `Unknown / not yet verified` or omit it.
Do not guess to make the memory look complete.

The default policy is to track `.project-memory/*.md` in Git. Do not add the
folder to `.gitignore` unless project-specific policy explicitly wants local-only
memory.

## Task startup

At task startup:

1. identify the configured integration branch;
2. determine whether the current checkout is the integration branch or another
   branch/worktree;
3. load canonical `STATUS.md`, `GOALS.md`, and `NEXT.md` from the integration
   branch when available;
4. read recent canonical `LOG.md` entries only when useful;
5. compare relevant claims with the current checkout and task request.

Do not perform a broad repository reread solely because memory exists. Its
purpose is to reduce rediscovery.

When local worktree memory differs from canonical memory, interpret the
difference as branch-local context until accepted/integrated evidence proves
otherwise.

If canonical memory is stale, continue from authoritative repository evidence and
repair it during the normal completion sync when this agent owns that sync.

## Completion sync

For a repository-modifying task, synchronize project memory after implementation
and validation state is known and before the final user report.

Update only what actually changed:

- refresh `STATUS.md` for material verified state changes;
- update `GOALS.md` only if goal/milestone/success criteria changed;
- refresh `NEXT.md` with the next concrete actions and blockers;
- add one concise `LOG.md` entry.

For `BLOCKED` or `FAILED` work, update memory only when there is a verified
partial-state change or a blocker that materially affects continuation. Do not
claim incomplete work as completed.

For pure analysis/Q&A/read-only tasks, do not modify project memory unless the
user explicitly asks for a memory update.

## Concurrency safety

Multiple Codex terminals may observe or modify the same project.

Immediately before writing memory:

1. re-read the current memory files;
2. inspect relevant Git/working-tree changes;
3. preserve newer or unexplained edits;
4. merge concise facts instead of replacing concurrent work blindly.

If another process is actively modifying the same memory file and safe merging
is unclear, report the conflict rather than overwriting it.

## Worktree ownership

A task-local HANDOFF and canonical project memory have different roles:

- HANDOFF: evidence and result from one worker/task;
- canonical project memory: accepted repository-level state for future sessions.

An independent worker/worktree created by another parent agent must not edit
canonical `.project-memory/` unless its task brief explicitly grants memory
ownership. It should return its normal concise HANDOFF instead.

The parent/main agent owns canonical memory synchronization after it validates
and accepts/integrates the worker result. This prevents multiple worktrees from
racing on status files and prevents rejected/unmerged branch-local work from
becoming current project state.

A top-level agent directly owning work on the integration branch may update
canonical project memory as part of that task.

## Git behavior

Treat project memory as ordinary small tracked documentation unless repository
policy says otherwise.

When the current agent owns the task commit, include the memory update in the
same commit when practical.

Do not:

- amend or rewrite an existing commit solely to insert memory metadata unless
  normal Git policy already permits it;
- create an extra commit solely to record the hash of the commit containing the
  memory update;
- include unrelated user changes while staging memory;
- describe uncommitted/unmerged worker changes as accepted project state.

If committing is not authorized, update memory in the working tree when required
and report that it remains uncommitted.

## Relationship to subagent orchestration

Reuse accepted worker HANDOFF facts; do not paste the full HANDOFF into memory.

A typical flow is:

```text
worker task
-> HANDOFF
-> parent inspection/validation
-> accept/integrate
-> canonical project-memory sync
-> final report
```

Canonical project-memory sync does not replace validation or independent review.

## Final quality check

Before finishing a memory sync, verify:

- files describe the current checkout/accepted state, not aspiration;
- status claims are evidence-backed;
- goals remain stable and concise;
- next actions are actionable;
- blockers are concrete;
- log contains only a compact recent entry;
- no secrets/private data/large outputs were copied;
- concurrent edits were preserved;
- independent unaccepted work was not promoted to current state.
