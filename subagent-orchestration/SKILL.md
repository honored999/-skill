---
name: subagent-orchestration
description: Coordinate scoped Codex workers for implementation, testing, auditing, review, fixes, validation, and integration. Use when selecting implementation mode, defining worker ownership/scope, choosing worker profiles, creating or monitoring implementation tasks, creating independent reviewer conversations, collecting HANDOFFs, escalating failed worker channels, handling manual-dispatch overrides, or coordinating reviewer/fixer flows.
---

# Subagent orchestration

## Purpose

Use delegated workers to reduce duplicated repository reading, execution risk,
and main-agent context usage.

The main agent owns planning, implementation-mode selection, integration,
acceptance, Git coordination, reviewer/fixer creation, validation coordination,
and final reporting.

Keep these decisions separate:

- **implementation mode** depends mainly on task size, scope, isolation needs,
  and context pressure;
- **review strength** depends mainly on correctness/safety risk;
- **worker transport** may be automatic or user-requested manual dispatch;
- **test intensity** may be reduced by `resource-aware-testing`, but validation
  claims must remain truthful.

A small implementation can still require strong independent review. A large
implementation does not automatically require a reviewer worktree.

## Default implementation ownership

For repository tasks that modify production source code or tests, the main agent
must select a delegated implementation owner before the first implementation
write.

Default ownership:

- localized implementation -> normal `Luna` with `xhigh` reasoning;
- substantial or multi-module implementation -> independent `LunaMax` Worktree
  Chat/task.

Task smallness alone is not a reason for direct parent implementation.

The main agent should normally remain:

- coordinator;
- architect/planner;
- integrator;
- acceptance owner;
- reviewer/fixer/validator coordinator;
- Git coordinator;
- final reporter.

Direct parent source/test implementation is allowed only when:

- the user explicitly requests parent implementation;
- no supported worker interface is available;
- supported worker channels have failed and documented fallback is justified;
- the remaining edit is tiny coordination/integration glue and delegation would
  create more risk than value.

If direct parent source/test implementation occurs, record the reason.

## Worker profiles

Delegated workers must not silently inherit the main agent's default profile.

Preferred profiles:

- normal scoped implementation worker -> `Luna xhigh`;
- focused fixer -> `Luna xhigh`;
- validator -> `Luna xhigh`;
- narrow low-risk reviewer -> `Luna xhigh`;
- substantial independent Worktree implementation -> `LunaMax`;
- Level 3 independent reviewer -> `LunaMax`.

When the platform supports explicit model/reasoning selection, request the
preferred profile directly.

When worker metadata exposes model/profile/reasoning information, verify it.
Do not claim a requested profile was used when it cannot be verified.

If the exact preferred profile is unavailable, use the closest supported
Luna-xhigh/Max profile and record the deviation.

A user-specified worker/model override takes precedence.

## Authorization and decision gates

Authorization for the underlying implementation/review/validation/audit/fixer
also covers routine worker creation and execution-mode changes needed for that
same task.

Do not ask separately to create a normal subagent, independent implementation
task, reviewer conversation, validator, fixer, or to escalate after a genuine
worker-channel failure.

Ask the user only for a genuine decision gate such as:

- scope expansion;
- materially different implementation choice;
- destructive/irreversible action;
- credentials/permissions/external access;
- experimental-protocol choice;
- explicit runtime/tool confirmation;
- manual fallback requiring user action.

Higher-priority platform/tool confirmation rules always apply.

## Explicit manual-dispatch override

If the user explicitly asks to manually open delegated worker conversations,
asks for a prompt to send to a new subagent/reviewer/fixer/validator chat, or
asks not to auto-dispatch workers, use `manual-subagent-handoff`.

This manual mode overrides automatic worker creation but does not change:

- role selection;
- model/profile choice;
- ownership;
- validation requirements;
- review requirements;
- Git/data-safety rules;
- acceptance standards.

While manual mode applies:

- do not automatically create normal subagents;
- do not automatically create Worktree Chat/tasks;
- do not automatically create reviewer conversations;
- do not automatically create fixers or validators;
- do not absorb delegated production source/test implementation into the parent;
- choose the intended worker role/profile normally;
- generate a self-contained manual dispatch prompt;
- require a terminal `HANDOFF`;
- when the user pastes the HANDOFF back, continue from that state without asking
  them to restate the task;
- if task-scoped manual mode remains active, subsequent reviewer/fixer/validator
  steps must also be emitted as manual prompts.

Manual mode may be one-shot or task-scoped. If scope is unclear, use one-shot.

## Implementation execution modes

Choose one primary implementation mode per core task:

1. **Normal subagent** — localized implementation, a few related files, focused
   bugs/tests, small audits/fixers, modest reading, or work that must land
   directly in a dirty worktree an isolated worker cannot safely observe.
2. **Independent Worktree Chat/task** — substantial or multi-module
   implementation, significant repository/paper/spec reading, repeated
   TDD/debug cycles, high context pressure, or implementation benefiting
   materially from an isolated checkout.
3. **Manual dispatch** — only when the user explicitly requests manual worker
   launching, or when automatic worker creation is unavailable and manual
   top-level handoff is the documented fallback.
4. **Persistent external process** — multi-hour training, simulations, long
   builds, or services.

Do not execute the same core implementation concurrently in multiple modes.

The parent remains responsible for collecting, validating, integrating, and
accepting delegated results.

Independent review is not an implementation execution mode.

## Planning/TDD skills do not override implementation ownership

Planning and workflow skills define **what** work should happen and in what
order. They do not decide **who** owns implementation.

Resolve implementation ownership before the first production source/test write.

When normal-subagent implementation is selected:

- one scoped normal subagent owns the implementation lane;
- RED tests and corresponding GREEN production edits normally belong to that
  same owner;
- the parent may create plans, task briefs, validation checklists, review notes,
  or other coordination artifacts;
- the parent must not directly modify source/test files in that owned lane while
  the worker is active;
- a later reviewer does not retroactively satisfy implementation delegation.

When independent Worktree implementation is selected:

- explicitly request `LunaMax`;
- request isolation when materially useful;
- the parent must not continue the same implementation.

## Normal subagent rules

A normal subagent is a leaf worker.

Typical lifecycle:

`delegate -> monitor sparsely -> collect -> inspect -> validate`

A normal subagent must:

- own only the lane assigned by the main agent;
- remain within declared read/write/forbidden scope;
- run self-checks/tests as needed;
- continue ordinary debugging/TDD work until reaching a legitimate terminal
  state;
- return exactly one terminal state: `COMPLETED`, `BLOCKED`, or `FAILED`.

A normal subagent must **not** create, delegate to, or spawn any other:

- worker;
- subagent;
- reviewer;
- validator;
- fixer;
- independent task.

Reviewer creation, fixer creation, validation-worker creation, and
implementation-mode escalation are owned by the main agent.

Self-review by an implementation worker does not count as independent review.

A normal subagent must not switch itself into another top-level execution mode.

## Worker continuation and termination

Once an implementation worker owns a lane, preserve that ownership until the
worker reaches a legitimate terminal state.

The main agent must not terminate, replace, or reassign a running worker solely
because of:

- elapsed time;
- silence between status polls;
- lack of an immediate HANDOFF;
- a desire to avoid waiting indefinitely;
- current-turn completion pressure;
- context pressure or compaction;
- known failing tests still within the assigned implementation scope;
- partial implementation that still requires ordinary debugging/TDD cleanup;
- one failed attempt;
- a heavy test being downgraded by resource-aware validation.

Failing tests produced during the worker's normal RED -> GREEN cycle are
work-in-progress, not worker-channel failure evidence.

If the worker remains reachable and within scope, keep the same owner and let it
continue.

Ownership may move only after one of the following:

- worker returns `COMPLETED`;
- worker returns `BLOCKED`;
- worker returns `FAILED`;
- the worker/task channel reports a concrete terminal/tool failure;
- the worker materially violates scope;
- unsafe behavior requires termination;
- repeated progress/status checks provide concrete evidence that the channel is
  unusable rather than merely slow;
- the user explicitly requests cancellation or reassignment.

Elapsed time by itself is not evidence of channel failure.

## Fixer policy

Do not create a fixer merely because the original implementation worker has
failing tests.

The original worker should normally complete its own debugging/TDD cleanup.

A focused fixer is appropriate when:

- the implementation owner has reached a terminal state;
- accepted/reviewed work later reveals a bounded defect;
- an independent reviewer returns a concrete `BLOCKING` finding;
- integration exposes a narrow defect after the original implementation phase.

Focused fixer profile:

`Luna xhigh`

If the fix becomes substantial or multi-module, rerun implementation-mode
selection and normally use `LunaMax` Worktree implementation.

## Independent Worktree implementation rules

When substantial implementation uses an independent Worktree Chat/task:

- explicitly request `LunaMax`;
- request an isolated worktree when appropriate;
- record returned task/thread/worktree identifiers;
- verify returned profile metadata when available;
- record when model selection cannot be set or verified;
- do not also run a normal subagent for the same core implementation;
- do not continue the same implementation in the parent;
- do not reuse the implementation worker as its independent reviewer.

The independent Worktree worker is the top-level implementation owner for its
core task.

It must not hand that same core task to another independent task/thread.

It may use a small number of narrowly scoped normal subagents for:

- focused read-only investigation;
- targeted tests;
- narrow review;
- small fixer work.

Those normal subagents remain leaf workers and should use `Luna xhigh`.

If the independent worker cannot complete within scope, return `BLOCKED` with
concrete evidence.

## Independent reviewer conversations

For correctness-sensitive Level 3 review, prefer a separate ordinary reviewer
conversation using `LunaMax` with read-only scope.

Independent review means:

- separate reviewer context;
- reviewer did not implement the change;
- exact final diff/commit/snapshot is identified;
- reviewer independently checks risky code paths and evidence;
- reviewer returns `PASS`, `BLOCKING`, and optional `NON-BLOCKING`.

Reviewer independence does **not** require a separate Git worktree.

Do not create a reviewer Worktree Chat merely to satisfy independence.

Use a reviewer worktree only when an isolated checkout is materially necessary,
for example when:

- reviewer must run commands against an isolated filesystem state;
- target commit cannot otherwise be inspected reliably;
- platform snapshot/diff/patch access cannot expose the intended state;
- repository/tooling conditions make separate checkout materially safer.

For narrow low-risk review that does not require Level 3, `Luna xhigh` is
sufficient by default.

Main-agent inspection and implementer self-review do not count as independent
Level 3 review.

## Review target and uncommitted work

An independent reviewer must inspect the actual intended change set.

For committed work, prefer:

- exact commit;
- exact diff;
- relevant test evidence;
- applicable contracts/specs.

For uncommitted work, expose the intended review target without mutating or
cleaning the source worktree.

Preferred approaches:

1. platform-supported read-only working-tree snapshot/diff;
2. review-only patch/snapshot containing intended tracked/untracked changes plus
   exact baseline commit;
3. when appropriate and authorized, temporary review commit/ref preserving the
   intended change set without rewriting unrelated history.

Do not stash, reset, clean, overwrite, or silently commit unrelated user work
just to make review easier.

## Resource-aware testing coordination

Any worker, fixer, validator, reviewer, or main agent that intends to run a test
or validation command must apply `resource-aware-testing` before that command.

This includes:

- focused regression tests;
- affected test groups;
- full suites;
- smoke tests;
- integration tests;
- checkpoint/model-loading tests;
- reruns after fixes.

If CPU, RAM, GPU utilization, or GPU-memory utilization is already at/above 80%,
or available evidence indicates the planned test may reach/exceed 80%, downgrade
to the lightest semantically faithful validation.

If a faithful lightweight substitute does not exist, report:

- `DEFERRED_RESOURCE_GUARD`, or
- `NOT_RUN_RESOURCE_GUARD`.

Do not present deferred validation as passed.

A resource-driven test downgrade is not a reason to terminate a healthy worker.

## Work ownership and scope

Each writable implementation lane has exactly one active owner.

Planning ownership and implementation ownership are separate.

Do not assign overlapping implementation responsibility to multiple workers,
even when nominal file lists differ.

Parallel workers are appropriate only when write scopes and interface
responsibilities are genuinely independent.

Reviewers/validators remain read-only unless explicitly reassigned as fixers.

Every writable worker assignment should define:

1. readable files/directories;
2. exact writable files/directories;
3. forbidden files/directories;
4. expected deliverable;
5. acceptance criteria.

Workers must not expand their own scope.

Do not split a small change across multiple implementers merely for structure.

## Monitoring and escalation

Healthy workers/reviewers should be monitored sparsely.

Do not generate repetitive `RUNNING -> RUNNING` updates.

Before replacing a worker/reviewer:

- reopen/re-list existing task/thread identifiers when available;
- consume any valid terminal result already produced;
- distinguish "slow" from "unusable".

Worker-channel failure evidence may include:

- explicit terminal tool/channel error;
- repeated inability to retrieve progress/result;
- repeated narrowed workers failing without repository/test/data blocker;
- isolated worker state that cannot be synchronized reliably;
- confirmed unusable task/thread state.

Normally, after two evidenced failures of the same normal-subagent channel for
the same task class, stop respawning equivalents and escalate:

`normal Luna xhigh -> independent LunaMax Worktree -> documented fallback`

This count is heuristic, not absolute.

Do not use elapsed time alone as failure evidence.

Review-channel failure does not imply a reviewer worktree is required. Retry or
recreate an ordinary independent reviewer conversation first when appropriate.

## Manual HANDOFF coordination

When manual mode is active, use `manual-subagent-handoff`.

A manually opened worker must return a terminal `HANDOFF`.

When the user pastes it back:

1. match it to the pending manual dispatch;
2. inspect status/role/changed or reviewed state/tests/Git state/deviations;
3. validate repository evidence when required;
4. continue from that state without asking the user to restate the task;
5. if task-scoped manual mode remains active, generate the next reviewer/fixer/
   validator prompt instead of auto-spawning.

HANDOFF is evidence, not automatic acceptance.

## Worker/reviewer briefs

Keep briefs self-contained but compact.

Include, when relevant:

- repository/worktree path;
- current branch/commit and Git state;
- requested profile;
- goal and verified current state;
- authoritative `AGENTS.md`, specs, plans, papers, or source docs;
- exact read/write/forbidden scope;
- important invariants;
- acceptance criteria;
- required validation;
- exact review commit/diff/snapshot/patch identity;
- Git/commit instructions;
- resource-aware test requirements;
- required HANDOFF/review-result format.

Do not paste the full parent conversation unless truly necessary.

Authoritative context priority:

1. current task/review brief;
2. current repository/worktree or exact review snapshot;
3. applicable `AGENTS.md` and project-local instructions;
4. authoritative task specification/design/source documents.

## HANDOFF

Implementation/fixer/validator workers should return:

```text
HANDOFF

- status: COMPLETED / BLOCKED / FAILED
- role:
- worker identity:
- requested model/profile:
- observed model/profile/reasoning, when verifiable:
- repository/worktree:
- branch:
- starting commit:
- ending commit, if any:
- changed files:
- implemented behavior:
- important design decisions:
- tests/validation:
- resource preflight/deferred validation, when relevant:
- failing tests or blockers:
- git status:
- raw/user data status, when relevant:
- project-memory changes, if any:
- scope deviations:
- unresolved issues:
- next recommended action:
```

Independent reviewers should return:

```text
HANDOFF

- status: COMPLETED / BLOCKED / FAILED
- role: independent reviewer
- requested model/profile:
- observed model/profile/reasoning, when verifiable:
- reviewed repository/worktree or snapshot:
- reviewed commit/diff/patch identity:
- result: PASS / BLOCKING
- BLOCKING findings:
- NON-BLOCKING findings:
- validation/evidence inspected:
- resource preflight/deferred validation, when relevant:
- evidence gaps:
- scope deviations:
- next recommended action:
```

## Evidence-based acceptance

Use the smallest sufficient trustworthy evidence set.

Routine acceptance may include:

1. scoped completion/HANDOFF;
2. successful focused/affected validation;
3. Git status and changed-file inspection;
4. targeted inspection of risky/integration-sensitive diff sections.

Escalate validation/review when:

- evidence is weak;
- tests fail;
- uncertainty is reported;
- unexpected files changed;
- numerics/geometry/data integrity are affected;
- serialization/checkpoint behavior changes;
- training/evaluation semantics change;
- public interfaces change;
- destructive behavior/security is involved.

Resource-limited deferred validation must remain explicitly visible in
acceptance decisions.

## Failure, blockers, and fixer flow

Collect evidence before changing ownership or creating a fixer.

A blocker must be concrete, such as:

- reproducible test failure that cannot be resolved within scope;
- missing required input;
- permission failure;
- incompatible interface;
- contradictory validated data;
- unsafe output behavior;
- undefined required protocol decision;
- verified infrastructure failure preventing supported execution modes.

Do not classify "worker has been running for a while" as a blocker.

After a focused fix:

`focused test -> affected group -> broader validation when appropriate`

Each test command still requires resource preflight.

When correctness/safety is involved, repeat independent Level 3 review using a
separate ordinary `LunaMax` reviewer conversation by default.

## Context compaction

After compaction, re-read authoritative files/spec sections required for
correctness-sensitive conclusions.

Compaction does not authorize:

- abandoning a healthy worker;
- weakening validation;
- changing ownership without evidence;
- recursively changing execution mode;
- bypassing manual-dispatch mode;
- ignoring resource-guarded deferred validation.
