---
name: subagent-orchestration
description: Coordinate scoped Codex workers for implementation, testing, auditing, review, fixes, and integration. Use when choosing between a normal subagent, an automatically created independent Worktree Chat/task, a user-created top-level Codex thread fallback, or a persistent external process; defining scope; monitoring workers; collecting results; handling blockers/failures; or decomposing work.
---

# Subagent orchestration

## Purpose

Use delegated workers to reduce duplicated repository reading, execution risk,
and main-agent context usage.

Normal subagents use the user's/Codex environment's configured model and
reasoning defaults.

For substantial work, prefer an automatically created independent Codex
Worktree Chat/task using LunaMax when the platform supports it reliably.
A user-created top-level Codex conversation is only the fallback when automatic
independent-task creation or result retrieval is unavailable.

## Execution-mode selection

Choose one mode before execution begins:

1. normal subagent;
2. automatically created independent Worktree Chat/task;
3. user-created independent top-level Codex thread fallback;
4. persistent external process for genuinely long execution.

The main agent owns this decision.

### Normal subagent

Prefer for localized work, one/few closely related files, focused bugs/tests,
small audits/fixers, modest repository reading, and work reasonably expected to
finish in roughly under 10 minutes.

Use:

`delegate -> monitor sparsely -> wait -> collect -> validate`

The time estimate is a heuristic, not a hard limit.

### Automatic independent Worktree Chat/task

Prefer when work is substantial, multi-module, requires significant
repository/paper/spec reading, multiple TDD/debug cycles, a broad/deep Level 3
audit, likely 10-15 minutes or more, likely tens of minutes, or likely context
compaction.

When supported, the main agent should create this task itself and use LunaMax
unless the user requests another model.

When this mode is selected:

- do not also launch a normal subagent for the same core task;
- do not continue the same implementation concurrently in the parent;
- use an isolated worktree when appropriate;
- do not reuse an implementation worker as an independent reviewer;
- preserve task/thread/worktree identifiers returned by the platform;
- the parent remains responsible for eventually collecting and validating the
  final result.

### Manual top-level thread fallback

Use only when automatic Worktree Chat creation is unavailable, repeatedly fails
for infrastructure/tool reasons, the final result cannot be retrieved, or the
user explicitly prefers manual mode.

Generate a self-contained task brief and ask the user to run it in a new
top-level Codex conversation using LunaMax by default.

The user then returns the final `HANDOFF`.

### Persistent external process

For multi-hour training, simulations, long builds, or services, use a persistent
OS process when appropriate. Record PID/process identity, logs, run identity,
configuration, and output location.

## No mid-task execution-mode switching by normal subagents

A normal subagent must not switch itself into independent-task mode.

Once launched, it should either:

- complete the assigned task;
- return `BLOCKED` with concrete evidence;
- return `FAILED` with concrete evidence.

It must not stop merely because the task is taking longer, ask the user to open
another top-level thread, move itself to LunaMax, create another independent
Worktree Chat for the core task, or abandon work because of context compaction.

Only the main agent may change execution mode.

Elapsed time alone is not a blocker.
Context compaction alone is not a blocker.

## No recursive independent-worker handoff

An independent Worktree Chat/task is the top-level execution worker for its
assigned core task.

It must not hand the core task off again to another independent Worktree Chat or
another top-level conversation.

It may use a small number of short, narrowly scoped normal subagents for:

- focused read-only investigation;
- targeted tests;
- narrowly scoped review;
- small fixer work.

Those normal subagents must not create nested workers for the same core task.

If the independent worker cannot complete the task within scope, return
`BLOCKED` with concrete evidence.

## Automatic independent-task creation

When the main agent creates an independent Worktree Chat/task:

1. define task and acceptance criteria;
2. create it with the intended model/reasoning profile;
3. request an isolated worktree when appropriate;
4. record all returned identifiers, including client task ID, formal task ID,
   thread/chat ID, project ID, and worktree path when available;
5. do not treat "queued", "setup", or only a client-side ID as proof that a
   runnable thread exists;
6. resolve the formal thread/task identifier if setup is asynchronous;
7. keep the parent repository state unchanged while the worker owns the task.

If creation fails because of a clear parameter/schema mismatch, retry using the
correct structure. Do not silently downgrade reviewer independence or reuse the
implementation worker merely because creation failed once.

## Automatic independent-worker result collection

When the main agent created the independent task, it remains responsible for
collecting the result.

Preferred lifecycle:

`create -> record identifiers -> wait/monitor sparsely -> detect terminal state -> open/read worker thread -> collect HANDOFF -> validate -> continue`

If the platform supports listing/opening/reading the created task/thread:

- preserve its identifier;
- retrieve the final worker response after completion;
- read the final `HANDOFF` or `PASS/BLOCKING`;
- validate it against repository state and required evidence;
- continue the parent workflow.

Do not require manual HANDOFF copy/paste when the main agent can reliably
retrieve the result itself.

Do not assume worker completion automatically wakes a paused parent unless that
behavior has been verified in the current environment.

If automatic wake-up does not occur, the saved task/thread ID should be used to
open/read the completed worker when the parent is resumed.

If the platform cannot reliably retrieve the final result, explicitly fall back
to:

`Please paste the worker's final HANDOFF back into this conversation.`

Manual handoff is a fallback, not the preferred path when automatic retrieval is
available.

## Independent-worker task brief

Whether created automatically or manually, the task brief must be self-contained
but compact.

Include:

- repository/worktree path;
- current branch or exact commit snapshot;
- current Git state when relevant;
- optional shared Deep Work/conversation link;
- goal;
- authoritative `AGENTS.md`, specs/plans/source documents;
- relevant existing implementation state;
- exact read scope;
- exact write scope;
- forbidden scope;
- important invariants;
- acceptance criteria;
- required validation;
- Git/commit instructions;
- required final `HANDOFF` format.

Do not paste the entire parent conversation history.
Do not make the worker rediscover already-decided architecture unnecessarily.
Do not reset, stash, clean, overwrite, amend, or rewrite unrelated user work
unless authorized.

## Shared context link

A shared conversation/Deep Work link is optional background only.

Current authoritative sources are:

1. current task brief;
2. current repository/worktree or exact snapshot;
3. applicable `AGENTS.md` and project-local instructions;
4. authoritative task spec/design document.

If shared context conflicts with current authoritative sources, follow the
current sources and report the discrepancy.

The task must remain executable if the link is unavailable.

## Independent worker HANDOFF

Before finishing, return:

```text
HANDOFF

- status: COMPLETED / BLOCKED / FAILED
- reviewer/worker identity and independence, when relevant:
- branch/worktree or exact snapshot:
- starting/reviewed commit(s):
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

Keep it concise and evidence-dense.

The parent must not accept the task solely from HANDOFF text when risk requires
repository inspection, targeted validation, or independent review.

## Normal-subagent scope

Every normal-subagent assignment must define:

1. readable files/directories;
2. exact writable files/directories;
3. forbidden files/directories;
4. expected deliverable;
5. acceptance criteria.

Do not allow two subagents to modify the same file concurrently.
Do not split a small change across multiple implementers merely for structure.

## Evidence-based acceptance

Use the smallest sufficient evidence set.

For routine work this may include:

1. scoped completion report;
2. successful validation evidence;
3. Git status/changed-file inspection;
4. targeted inspection of risky/integration-sensitive diff sections.

Escalate when evidence is weak, tests fail, uncertainty is reported, unexpected
files changed, or the task affects sensitive numerics, data integrity, geometry,
serialization, training/evaluation semantics, public interfaces, destructive
behavior, or security.

## Normal-subagent lifecycle

`delegate -> monitor -> wait -> collect -> inspect -> validate -> continue`

State handling:

- `RUNNING` — continue monitoring;
- `COMPLETED` — collect and validate;
- `FAILED` — collect failure evidence;
- `BLOCKED` — collect blocker evidence and apply a decision gate.

Launching a subagent does not complete the parent task.

## No premature conclusion

Do not force a running worker to return early while it is still collecting
required evidence.

Do not send instructions such as:

- `return immediately`;
- `stop now and summarize`;
- `give PASS/BLOCKING now`;
- `finish with current evidence`.

## Monitoring cadence

For a healthy normal subagent, prefer sparse monitoring. Roughly 5-10 minute
checks are normally sufficient for moderate or Level 3 work unless new output,
expected command completion, failure, or infrastructure issues justify earlier
inspection.

Routine `RUNNING -> RUNNING` polls should remain internal and should not produce
repetitive user-facing commentary.

For automatically created Worktree Chats/tasks, prefer platform task-state and
result retrieval over conversational polling.

## Decision gates

Ask the user only when approval, scope expansion, destructive action,
experimental protocol, credentials/permissions, materially different choices,
or manual top-level-thread fallback is genuinely required.

Do not ask the user to perform manual handoff if the platform can create and
retrieve the independent task automatically.

## Failure and blocker policy

Collect evidence before changing anything.

Record as relevant: command, exit code, traceback, affected stage, process/task
state, generated artifacts, last completed unit, configuration, Git state, and
task/thread IDs.

Do not hide failed attempts or classify infrastructure interruption as a
code/model defect without evidence.

A blocker must be concrete, such as reproducible test failure, missing required
input, permission failure, incompatible interface, contradictory validated data,
unsafe output behavior, or an undefined required protocol decision.

## Fixer flow

Use a narrowly scoped fixer for a small defect.

If the fix itself is substantial, run execution-mode selection again and prefer
an automatically created independent Worktree Chat/task when available.

After a fix:

`relevant test -> affected test group -> broader validation when appropriate`

Repeat independent review when correctness or safety was involved.

## Context compaction

After compaction, re-read authoritative files/spec sections required for a
correctness-sensitive conclusion.

Compaction does not authorize abandoning the task or recursively changing
execution mode.
