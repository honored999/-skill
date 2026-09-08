---
name: subagent-orchestration
description: Coordinate scoped Codex workers for implementation, testing, auditing, review, fixes, validation, and integration. Use when selecting execution mode, defining worker ownership/scope, creating or monitoring independent tasks, collecting HANDOFFs, escalating failed worker channels, reviewing uncommitted work, or coordinating reviewer/fixer flows.
---

# Subagent orchestration

## Purpose

Use delegated workers to reduce duplicated repository reading, execution risk,
and main-agent context usage.

The main agent owns planning, execution-mode selection, integration, acceptance,
Git coordination, and final reporting.

Normal subagents use the environment's configured model/reasoning defaults.
For substantial independent work, prefer `LunaMax` unless the user requests
another profile. Explicitly request it when the platform supports model selection;
do not assume inheritance.

## Authorization and decision gates

Authorization for the underlying implementation/review/validation/audit/fixer
also covers routine worker creation and execution-mode changes needed for that
same task.

Do not ask separately to create a normal subagent, independent task, reviewer,
validator, fixer, or to escalate after worker-channel failure.

Ask the user only for a genuine decision gate:

- scope expansion;
- materially different implementation choices;
- destructive/irreversible action;
- credentials, permissions, or external access;
- experimental-protocol choice;
- explicit tool/runtime confirmation;
- manual fallback requiring user action.

Higher-priority platform/tool confirmation rules always apply.

## Execution modes

Choose one primary mode per core task:

1. **Normal subagent** — localized work, a few related files, focused bugs/tests,
   small audits/fixers, modest reading, or work that must land directly in a dirty
   worktree an isolated worker cannot safely observe.
2. **Independent Worktree Chat/task** — substantial or multi-module work,
   significant repository/paper/spec reading, repeated TDD/debug cycles, broad
   correctness review, high context pressure, or work benefiting from isolation.
3. **Manual top-level-thread fallback** — only when automatic independent-task
   creation/result retrieval is unavailable or the user explicitly prefers it.
4. **Persistent external process** — multi-hour training, simulations, long builds,
   or services.

Do not execute the same core implementation concurrently in multiple modes.
The parent remains responsible for collecting, validating, and accepting delegated
results.

Direct parent implementation is not a substitute execution mode when a supported
delegated mode has been selected and is available.

## Planning/TDD skills do not override execution-mode selection

Planning and workflow skills such as `writing-plans`, `executing-plans`, TDD,
debugging, or verification define **what** work should happen and in what order.
They do not decide **who** owns implementation.

Resolve execution mode before the first implementation write whenever practical.
A phrase such as `inline executing-plans` does not authorize the parent to bypass
the selected worker mode.

When **normal subagent** is selected:

- the parent may write or update coordination-only artifacts such as the plan,
  task brief, review notes, or validation checklist;
- one scoped normal subagent should own the implementation lane;
- RED test edits and the corresponding GREEN production edits normally belong to
  that same implementation owner;
- the parent must not directly edit source/test files in that owned lane while the
  worker is active;
- a later read-only reviewer does not retroactively satisfy implementation
  delegation.

When **independent Worktree Chat/task** is selected, the existing independent-task
rule still applies: the parent must not continue the same implementation.

If the selected worker interface is unavailable or repeatedly fails, follow the
worker-channel escalation policy instead of silently absorbing implementation
into the parent. Parent-side direct implementation is only a documented last
fallback when supported delegation paths are unavailable and the task can still
be completed safely.

If the parent already made implementation edits before detecting an orchestration
mismatch, do **not** spawn another worker merely to rewrite or duplicate the same
diff for procedural purity. Freeze further overlapping parent edits, inspect the
current ownership/state, delegate only genuinely remaining non-overlapping work
when useful, and continue review/validation. Record the orchestration deviation
in the final report.

### Normal subagent rules

Typical lifecycle:

`delegate -> monitor sparsely -> collect -> inspect -> validate`

A normal subagent must complete, return `BLOCKED`, or return `FAILED`. It must not
switch itself into another top-level execution mode. Elapsed time or context
compaction alone is not a blocker.

For a localized TDD task, prefer one normal subagent to carry the complete
`RED -> minimal GREEN -> focused validation` implementation loop within its
declared write scope. The parent owns acceptance and broader validation, not the
same source/test edits.

### Independent task rules

When supported:

- explicitly request the preferred independent-worker profile;
- request an isolated worktree when appropriate;
- record returned task/thread/worktree identifiers;
- verify returned model/profile metadata when available;
- record when model selection cannot be set or verified;
- do not also run a normal subagent for the same core task;
- do not continue the same implementation in the parent;
- do not reuse the implementation worker as its independent reviewer.

### Persistent-process rules

Record PID/process identity, logs, run/configuration identity, and output location
when relevant. A worker remaining `RUNNING` does not guarantee the underlying OS
process is persistent.

## Work ownership and scope

Each writable component or implementation lane has exactly one active
implementation owner. Planning ownership is separate from implementation
ownership: the parent may own the plan while a worker owns the implementation.

Do not assign overlapping implementation responsibility to multiple workers even
when their nominal file lists differ. Parallel workers are appropriate only when
write scopes and interface responsibilities are independent.

Reviewers/validators remain read-only unless explicitly reassigned as fixers.
Never allow two workers to modify the same file concurrently.

Every writable worker assignment must define:

1. readable files/directories;
2. exact writable files/directories;
3. forbidden files/directories;
4. expected deliverable;
5. acceptance criteria.

Workers must not expand their own scope. Do not split a small change across
multiple implementers merely for structure.

## Independent-task lifecycle

When creating an independent task:

1. define goal and acceptance criteria;
2. select the preferred profile unless overridden;
3. request an isolated worktree when appropriate;
4. record available client-task, formal-task, thread/chat, project, and worktree
   identifiers;
5. do not treat only `queued`, `setup`, or a client-side ID as proof that a usable
   worker exists;
6. resolve the formal task/thread ID when setup is asynchronous;
7. keep the parent worktree unchanged while the worker owns the task.

If creation fails because of a clear parameter/schema mismatch, retry with the
correct structure. Do not weaken reviewer independence merely because creation
failed once.

Preferred lifecycle:

`create -> record IDs -> monitor sparsely -> detect terminal state -> read result -> collect HANDOFF -> validate -> continue`

When supported, reopen/read the created task after completion and collect
`HANDOFF` or `PASS/BLOCKING` automatically. Do not require manual copy/paste when
reliable retrieval exists, and do not assume completion automatically wakes a
paused parent.

If automatic creation/result retrieval is unavailable, use the manual top-level
fallback and ask the user to return the final `HANDOFF`.

## Monitoring, escalation, and stale tasks

Healthy workers should be monitored sparsely. Do not generate repetitive
user-facing `RUNNING -> RUNNING` updates or force a healthy worker to stop for an
early summary.

Before replacing a worker/reviewer, reopen/re-list existing task/thread IDs for
the same core task and consume any valid terminal result already produced.
Do not create replacements merely because setup metadata is stale.

Worker-channel failure evidence may include:

- repeated absence of usable progress or HANDOFF;
- ignored status requests;
- usable completion only after forced termination;
- isolated changes failing to synchronize reliably;
- repeated narrowed workers failing without a repository/test/data blocker.

Normally, after two evidenced failures of the same normal-subagent channel for
the same task class, stop respawning equivalents and escalate:

`normal subagent -> independent Worktree Chat/task -> documented fallback`

The count is a heuristic, not an absolute rule. Escalate earlier/later when the
evidence justifies it.

Worker-channel failure is not automatically a repository blocker; use another
supported mode first when possible.

If an older/duplicate task later produces a valid result, consume the useful
result and avoid duplicate work. Cancel/close obsolete tasks when safely
supported. Never allow stale workers to continue modifying a scope after
ownership has moved elsewhere.

## No recursive independent-worker handoff

An independent Worktree Chat/task is the top-level execution worker for its core
task and must not hand that core task to another independent task/thread.

It may use a small number of narrowly scoped normal subagents for focused
read-only investigation, targeted tests, narrow review, or small fixer work.
Those subagents must not recursively create workers for the same core task.

If the independent worker cannot complete within scope, return `BLOCKED` with
concrete evidence.

## Independent review of uncommitted work

An independent reviewer must inspect the actual intended change set, not merely
the baseline branch.

Because a new worktree normally cannot see another worktree's dirty/untracked
changes, expose the intended review target without mutating the source worktree.
Preferred approaches:

1. platform-supported read-only working-tree snapshot/diff;
2. review-only patch/snapshot containing intended tracked/untracked changes plus
   the exact baseline commit;
3. when appropriate and authorized, a temporary review commit/ref preserving the
   exact intended change set without rewriting unrelated history.

The review brief must identify baseline commit, intended changed/untracked files,
snapshot/patch/commit identity, what the reviewer is actually inspecting, and
intentionally excluded files.

The reviewer remains read-only unless reassigned as a fixer. Do not stash, reset,
clean, overwrite, or silently commit unrelated user work to make review easier.
If no supported mechanism can expose the real change set safely, report the
concrete technical blocker.

## Worker task brief

Keep independent-worker briefs self-contained but compact. Include, when relevant:

- repository/worktree path;
- current branch/commit and Git state;
- goal and relevant existing implementation state;
- authoritative `AGENTS.md`, specs, plans, or source documents;
- exact read/write/forbidden scope;
- important invariants;
- acceptance criteria and required validation;
- review snapshot/patch identity for uncommitted review;
- Git/commit instructions;
- required HANDOFF format.

Do not paste the full parent conversation or make workers rediscover already
settled architecture unnecessarily.

Authoritative context priority:

1. current task brief;
2. current repository/worktree or exact snapshot;
3. applicable `AGENTS.md` and project-local instructions;
4. authoritative task specification/design/source documents.

Shared conversation links are optional background only. If they conflict with
current authoritative sources, follow the current sources and report the
discrepancy.

## HANDOFF

Independent workers should return:

```text
HANDOFF

- status: COMPLETED / BLOCKED / FAILED
- worker/reviewer identity and independence, when relevant:
- branch/worktree or exact snapshot:
- starting/reviewed commit(s):
- review snapshot/patch, when relevant:
- changed files:
- implemented behavior:
- important design decisions:
- tests/validation:
- visible results:
- reviewer result, if any:
- commit hash, if committed:
- git status:
- raw/user data status, when relevant:
- real training/server validation, when relevant:
- unresolved issues:
- scope deviations:
- next recommended action:
```

Keep it concise and evidence-dense. The parent must not accept a task solely from
HANDOFF text when risk requires repository inspection, targeted validation, or
independent review.

## Evidence-based acceptance

Use the smallest sufficient trustworthy evidence set. For routine work this may
include:

1. scoped completion report;
2. successful validation evidence;
3. Git status/changed-file inspection;
4. targeted inspection of risky/integration-sensitive diff sections.

Escalate validation when evidence is weak, tests fail, uncertainty is reported,
unexpected files changed, or work affects sensitive numerics, data integrity,
geometry, serialization, training/evaluation semantics, public interfaces,
destructive behavior, or security.

## Failure, blockers, and fixer flow

Collect evidence before changing anything. Record as relevant: command/exit code,
traceback, affected stage, worker/task/process state, generated artifacts, last
completed unit, configuration, Git state, and task/thread IDs.

Do not hide failed attempts or classify infrastructure interruption as a
repository/model defect without evidence.

A blocker must be concrete, such as reproducible test failure, missing required
input, permission failure, incompatible interface, contradictory validated data,
unsafe output behavior, undefined required protocol decision, or verified
infrastructure failure preventing every supported execution mode.

Use a narrowly scoped fixer for a small defect. If the fix is substantial, run
execution-mode selection again.

After a fix:

`focused test -> affected test group -> broader validation when appropriate`

Repeat independent review when correctness or safety was involved.

## Context compaction

After compaction, re-read authoritative files/spec sections required for
correctness-sensitive conclusions.

Compaction does not authorize abandoning the task, weakening validation, or
recursively changing execution mode.
