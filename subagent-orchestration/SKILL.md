---
name: subagent-orchestration
description: Coordinate scoped Codex workers for implementation, testing, auditing, review, fixes, validation, and integration. Use when selecting implementation mode, defining worker ownership/scope, choosing worker profiles, creating or monitoring implementation tasks, creating independent reviewer conversations, collecting HANDOFFs, escalating failed worker channels, reviewing uncommitted work, or coordinating reviewer/fixer flows.
---

# Subagent orchestration

## Purpose

Use delegated workers to reduce duplicated repository reading, execution risk,
and main-agent context usage.

The main agent owns planning, implementation-mode selection, integration,
acceptance, Git coordination, reviewer/fixer creation, and final reporting.

Keep two decisions separate:

- **implementation mode** is chosen mainly from task size, scope, isolation needs,
  and context pressure;
- **review strength** is chosen mainly from correctness/safety risk.

A small implementation can still require strong independent review. A large
implementation does not automatically require a reviewer worktree.

## Default implementation ownership

For repository tasks that modify production source code or tests, the main agent
must select a delegated implementation owner before the first implementation
write.

Default ownership:

- localized implementation -> `Luna` with `xhigh` reasoning normal subagent;
- substantial or multi-module implementation -> independent `LunaMax` Worktree
  Chat/task.

Task smallness alone is not a reason for direct parent implementation.

The main agent should normally remain coordinator, architect, integrator,
acceptance owner, reviewer/fixer coordinator, Git coordinator, and final
reporter.

Direct parent source/test implementation is allowed only when:

- the user explicitly requests parent implementation;
- no supported worker interface is available;
- supported worker channels have failed and documented fallback is justified;
- the remaining edit is coordination/integration glue so small that delegating
  it would create more risk than value.

If direct parent implementation occurs, record the reason.

## Worker profiles

Delegated workers must not rely on the main agent's inherited/default model.

Preferred profiles:

- normal scoped implementation subagent, focused fixer, validator, or narrow
  low-risk reviewer: explicitly request `Luna` with `xhigh` reasoning;
- substantial independent Worktree Chat/task used for implementation:
  explicitly request `LunaMax`;
- Level 3 independent reviewer conversation:
  explicitly request `LunaMax`.

When the platform supports explicit model/reasoning selection, request the
preferred profile directly rather than inheriting the parent configuration.

When returned worker metadata exposes model/profile/reasoning information,
verify it. Do not claim a requested profile was used when it cannot be verified.

If the exact preferred profile is unavailable, use the closest supported Luna
`xhigh`/Max profile and record the deviation. Do not silently fall back to the
main agent's model merely because inheritance is easier.

A user-specified worker/model override takes precedence over these defaults.

## Authorization and decision gates

Authorization for the underlying implementation/review/validation/audit/fixer
also covers routine worker creation and execution-mode changes needed for that
same task.

Do not ask separately to create a normal subagent, independent implementation
task, reviewer conversation, validator, fixer, or to escalate after
worker-channel failure.

Ask the user only for a genuine decision gate:

- scope expansion;
- materially different implementation choices;
- destructive/irreversible action;
- credentials, permissions, or external access;
- experimental-protocol choice;
- explicit tool/runtime confirmation;
- manual fallback requiring user action.

Higher-priority platform/tool confirmation rules always apply.

## Implementation execution modes

Choose one primary implementation mode per core task:

1. **Normal subagent** — localized work, a few related files, focused bugs/tests,
   small audits/fixers, modest reading, or work that must land directly in a dirty
   worktree an isolated worker cannot safely observe.
2. **Independent Worktree Chat/task** — substantial or multi-module
   implementation, significant repository/paper/spec reading, repeated TDD/debug
   cycles, high context pressure, or implementation benefiting materially from an
   isolated checkout.
3. **Manual top-level-thread fallback** — only when automatic independent-task
   creation/result retrieval is unavailable or the user explicitly prefers it.
4. **Persistent external process** — multi-hour training, simulations, long
   builds, or services.

Do not execute the same core implementation concurrently in multiple modes.
The parent remains responsible for collecting, validating, and accepting
delegated results.

**Independent review is not an implementation execution mode.** By default,
independent review uses a separate ordinary reviewer conversation, not another
worktree.

Direct parent implementation is not a substitute execution mode when a supported
delegated mode has been selected and is available.

## Planning/TDD skills do not override implementation ownership

Planning and workflow skills define what work should happen and in what order.
They do not decide who owns implementation.

Resolve implementation ownership before the first production source/test write.

## Normal subagent rules

Typical lifecycle:

`delegate -> monitor sparsely -> collect -> inspect -> validate`

A normal subagent must:

- own only the lane assigned by the main agent;
- complete, return `BLOCKED`, or return `FAILED`;
- remain within declared read/write/forbidden scope;
- run self-checks/tests as needed.

A normal subagent must **not** create, delegate to, or spawn any other worker,
subagent, reviewer, validator, fixer, or independent task.

Reviewer creation, fixer creation, validation-worker creation, and
execution-mode escalation are owned by the main agent.

Self-review by an implementation worker does not count as independent review.

A normal subagent must not switch itself into another top-level execution mode.
Elapsed time or context compaction alone is not a blocker.

For localized TDD, prefer one normal subagent to carry the complete
`RED -> minimal GREEN -> focused validation` loop within its declared write scope.
The parent owns acceptance and broader validation.

## Independent Worktree implementation rules

When substantial implementation uses an independent Worktree Chat/task:

- explicitly request `LunaMax`;
- request an isolated worktree when appropriate;
- record returned task/thread/worktree identifiers;
- verify returned model/profile metadata when available;
- record when model selection cannot be set or verified;
- do not also run a normal subagent for the same core implementation;
- do not continue the same implementation in the parent;
- do not reuse the implementation worker as its independent reviewer.

An independent Worktree Chat/task is the top-level implementation worker for its
core task. It must not hand that same core task to another independent task/thread.

It may use a small number of narrowly scoped normal subagents for focused
read-only investigation, targeted tests, narrow review, or small fixer work.
Those normal subagents should use `Luna` with `xhigh` reasoning and inherit the
normal-subagent no-recursion rule.

If the independent worker cannot complete within scope, return `BLOCKED` with
concrete evidence.

## Independent reviewer conversations

For correctness-sensitive Level 3 review, prefer a **separate ordinary reviewer
conversation** with `LunaMax` and read-only scope.

Independent review normally means:

- a separate reviewer context that did not implement the change;
- the intended final diff/commit/snapshot is explicitly identified;
- the reviewer stays read-only;
- the reviewer independently checks the risky code path and evidence;
- the reviewer returns `PASS`, `BLOCKING`, and optional `NON-BLOCKING` findings.

A reviewer does **not** need a separate worktree merely to count as independent.
Do not create a Worktree Chat only for procedural independence.

Use a reviewer worktree only when an isolated checkout is materially necessary,
for example when:

- the reviewer must run commands against an isolated filesystem state;
- the target commit cannot otherwise be inspected reliably;
- platform snapshot/diff/patch access cannot expose the intended state;
- repository state or tooling makes a separate checkout safer than read-only
  inspection in an ordinary reviewer conversation.

For a narrow, low-risk read-only review that does not require Level 3,
`Luna` with `xhigh` reasoning is sufficient by default.

Main-agent inspection and implementer self-review do not count as independent
Level 3 review.

## Review target and uncommitted work

An independent reviewer must inspect the actual intended change set, not merely
the baseline branch.

For committed work, prefer the exact commit/diff plus relevant tests and
contracts.

For uncommitted work, expose the intended review target without mutating or
cleaning the source worktree. Preferred approaches:

1. platform-supported read-only working-tree snapshot/diff;
2. review-only patch/snapshot containing intended tracked/untracked changes plus
   the exact baseline commit;
3. when appropriate and authorized, a temporary review commit/ref preserving the
   exact intended change set without rewriting unrelated history.

The reviewer remains read-only unless reassigned as a fixer. Do not stash, reset,
clean, overwrite, or silently commit unrelated user work to make review easier.

## Fixer and validator profiles

For a focused fixer or validation worker:

- explicitly request `Luna` with `xhigh` reasoning;
- keep its scope narrow and derived from the concrete defect or validation need;
- do not allow it to expand into a second implementation owner for unrelated
  work.

If the fix becomes substantial or multi-module, run implementation-mode
selection again and normally escalate to an independent `LunaMax` Worktree task.

After a correctness-sensitive fix, repeat independent review against the new
final state. A separate ordinary `LunaMax` reviewer conversation remains the
default; a reviewer worktree is optional and evidence-driven.

## Monitoring and escalation

Healthy workers/reviewers should be monitored sparsely. Do not generate
repetitive user-facing `RUNNING -> RUNNING` updates or force a healthy worker to
stop for an early summary.

Normally, after two evidenced failures of the same normal-subagent implementation
channel for the same task class, stop respawning equivalents and escalate:

`normal Luna xhigh subagent -> independent LunaMax Worktree Chat/task -> documented fallback`

Review-channel failure does not imply that a reviewer worktree is required.
Retry or recreate an ordinary independent reviewer conversation when
appropriate; use a reviewer worktree only when evidence access or isolation
materially requires one.

## Worker/reviewer briefs

Include, when relevant:

- repository/worktree path;
- current branch/commit and Git state;
- requested profile (`Luna xhigh` or `LunaMax`);
- goal and relevant existing implementation state;
- authoritative `AGENTS.md`, specs, plans, or source documents;
- exact read/write/forbidden scope;
- important invariants;
- acceptance criteria and required validation;
- exact review commit/diff/snapshot/patch identity;
- Git/commit instructions for implementers;
- required `HANDOFF` or review-result format.

Do not paste the full parent conversation or make workers rediscover already
settled architecture unnecessarily.

## HANDOFF

Implementation/fixer/validation workers should return concise evidence including:

- status: `COMPLETED` / `BLOCKED` / `FAILED`;
- worker identity;
- requested and observed model/profile when verifiable;
- branch/worktree or exact snapshot;
- starting commit(s);
- changed files;
- implemented behavior;
- tests/validation;
- commit hash if committed;
- git status;
- unresolved issues;
- scope deviations;
- next recommended action.

Independent reviewers should return concise `PASS` / `BLOCKING` / optional
`NON-BLOCKING` findings plus the exact reviewed commit/diff/snapshot identity and
requested/observed reviewer profile when verifiable.

## Evidence-based acceptance

Use the smallest sufficient trustworthy evidence set. Escalate validation when
evidence is weak, tests fail, uncertainty is reported, unexpected files changed,
or work affects sensitive numerics, data integrity, geometry, serialization,
training/evaluation semantics, public interfaces, destructive behavior, or
security.

## Context compaction

After compaction, re-read authoritative files/spec sections required for
correctness-sensitive conclusions.

Compaction does not authorize abandoning the task, weakening validation, or
recursively changing execution mode.
