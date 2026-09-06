---
name: subagent-orchestration
description: Coordinate scoped Codex workers for implementation, testing, auditing, review, fixes, and integration. Use when choosing between a normal subagent, an automatically created independent Worktree Chat/task, a user-created top-level Codex thread fallback, or a persistent external process; defining scope; monitoring workers; collecting results; handling blockers/failures; escalating execution mode; reviewing uncommitted work; or decomposing work.
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

## Authorization inheritance

Once the user has authorized the underlying implementation, review, validation,
audit, or fixer work, that authorization also covers routine worker creation
needed to execute the same already-authorized task.

Creating or switching between a normal subagent, independent Worktree Chat/task,
reviewer, fixer, or validation worker is an execution-mode decision owned by the
main agent. It is not, by itself, a new user-facing task.

Do not ask the user for separate approval merely to:

- create a normal subagent;
- create an independent Worktree Chat/task;
- create an independent reviewer;
- create a narrowly scoped fixer;
- create a validation worker;
- switch execution mode after worker-channel failure.

Ask again only when a genuine decision gate is reached, such as scope expansion,
a materially different implementation choice, destructive action, credentials or
permissions, experimental-protocol choice, or a tool/runtime that itself requires
explicit user confirmation.

If a platform or tool has a higher-priority rule that requires explicit user
consent before creating a task/chat, state that concrete tool-level restriction
at the first point where it blocks the required execution mode. Do not first
spawn repeated equivalent workers and only later introduce the approval gate.

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

A normal subagent may also be appropriate when the user explicitly requires
changes to land directly in an existing dirty worktree and an independent
worktree cannot safely observe or modify those uncommitted changes. This is a
technical placement constraint, not a general reason to avoid independent
workers for later review or validation.

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

## Main-agent execution-mode escalation

The main agent must distinguish a slow healthy worker from a failing worker
channel.

Do not switch modes merely because one healthy worker has been running for a
while. First collect evidence such as task state, output/progress, returned
errors, synchronized files, or missing HANDOFF behavior.

However, if the same normal-subagent channel fails to produce usable progress or
a terminal HANDOFF twice for the same task class, stop spawning equivalent normal
subagents for that core task.

Typical evidence of a worker-channel failure includes:

- repeated workers remain `RUNNING` without usable progress/status beyond the
  expected monitoring window;
- workers ignore status requests and never return a HANDOFF;
- completion occurs only after forced termination;
- isolated-worker changes fail to synchronize back reliably;
- multiple narrowly reduced tasks fail in the same way despite no repository,
  test, permission, or data blocker.

When this pattern is established, automatically escalate:

`normal subagent -> independent Worktree Chat/task -> documented fallback`

Do not require the user to choose or separately approve this escalation when the
underlying work was already authorized.

Do not keep reducing and respawning equivalent normal subagents indefinitely.
Record the infrastructure evidence and change execution mode.

If automatic independent-task creation itself fails repeatedly for
infrastructure/tool reasons, use the manual top-level-thread fallback or another
documented supported mode.

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

## Independent review of uncommitted work

An independent reviewer must review the actual intended change set, not merely
the baseline branch.

A newly created worktree normally cannot see uncommitted or untracked changes
from another worktree. Therefore, when the review target is dirty or contains
untracked intended files, the main agent must explicitly make the review target
visible without mutating the source worktree.

Preferred approaches, in order:

1. use a platform-supported read-only working-tree snapshot/diff handoff;
2. create a review-only patch/snapshot artifact containing all intended tracked
   and untracked changes plus the exact baseline commit SHA;
3. when appropriate and authorized by the task workflow, create a temporary
   review commit/ref that preserves the exact change set without rewriting user
   history.

The review brief must identify:

- baseline commit SHA;
- intended changed/untracked files;
- snapshot/patch identity or location;
- whether the reviewer is inspecting a commit, worktree, or patch;
- any files intentionally excluded from review.

The independent reviewer must remain read-only unless explicitly assigned fixer
work.

Do not:

- assume another worktree can see dirty changes;
- silently review only `HEAD` when the intended diff is uncommitted;
- force-stash, reset, clean, overwrite, or commit unrelated user work;
- fall back indefinitely to shared normal reviewers merely because the target is
  uncommitted.

If no supported mechanism can transfer the uncommitted review target safely,
report that concrete technical blocker. Ask the user only if the remaining
fallback itself requires user action or explicit tool-level authorization.

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
- review snapshot/patch identity when reviewing uncommitted work;
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

This rule applies to healthy workers. It does not prohibit the main agent from
terminating a worker after concrete infrastructure-failure evidence has been
established under the execution-mode escalation policy.

## Monitoring cadence

For a healthy normal subagent, prefer sparse monitoring. Roughly 5-10 minute
checks are normally sufficient for moderate or Level 3 work unless new output,
expected command completion, failure, or infrastructure issues justify earlier
inspection.

Routine `RUNNING -> RUNNING` polls should remain internal and should not produce
repetitive user-facing commentary.

For automatically created Worktree Chats/tasks, prefer platform task-state and
result retrieval over conversational polling.

If multiple normal workers have already shown the same non-responsive behavior,
do not restart the monitoring clock indefinitely for each replacement worker.
Apply the execution-mode escalation policy.

## Decision gates

Ask the user only when approval, scope expansion, destructive action,
experimental protocol, credentials/permissions, materially different choices,
manual top-level-thread fallback requiring user action, or an explicit
tool/runtime confirmation requirement is genuinely present.

Do not ask the user merely to approve internal orchestration choices for work
that is already authorized.

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
unsafe output behavior, an undefined required protocol decision, or a verified
worker/task infrastructure failure that prevents the selected execution mode.

Worker-channel failure is not automatically a repository blocker. When another
supported execution mode exists, escalate first.

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
