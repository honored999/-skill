---
name: project-memory
description: Maintain concise repository-local project state across Codex sessions, terminals, branches, and accepted worktree tasks. Use when `.project-memory/` exists, when repository instructions require project memory, when initializing repository memory, at repository-task startup, or when synchronizing verified status/goals/next actions/recent history before completion.
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

## Authority

When sources disagree, prefer current authoritative evidence in this order as
applicable:

1. explicit current user instruction;
2. applicable `AGENTS.md` and task/spec documents;
3. current repository/worktree contents and Git state;
4. trustworthy tests/validation/runtime evidence;
5. `.project-memory/` summary.

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

When `.project-memory/` exists:

1. read `STATUS.md`;
2. read `GOALS.md`;
3. read `NEXT.md`;
4. read only the recent part of `LOG.md` when useful;
5. compare relevant claims with the current branch/worktree and task request.

Do not perform a broad repository reread solely because memory exists. Its
purpose is to reduce rediscovery.

If memory is obviously stale, continue from authoritative repository evidence
and repair memory at the normal completion sync.

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

A task-local HANDOFF and repository project memory have different roles:

- HANDOFF: evidence and result from one worker/task;
- project memory: accepted repository-level state for future sessions.

An independent worker/worktree created by another parent agent must not edit
`.project-memory/` unless its task brief explicitly grants memory ownership.
It should return its normal concise HANDOFF instead.

The parent/main agent owns memory synchronization after it validates or accepts
the worker result. This prevents multiple worktrees from racing on the same
status files and prevents rejected/unmerged work from becoming "current state".

A top-level agent directly owning a repository task may update project memory as
part of that task.

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
-> project-memory sync
-> final report
```

Project-memory sync does not replace validation or independent review.

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
