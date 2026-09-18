# Repository Agent Instructions

## Project configuration

Project name: `<PROJECT_NAME>`
Repository root: `<REPOSITORY_ROOT>`
Integration branch: `<INTEGRATION_BRANCH>`

Primary source directories:
- `<SOURCE_DIRECTORY>`

Primary test directories:
- `<TEST_DIRECTORY>`

Local data directories, if applicable:
- `<DATA_DIRECTORY>`

Generated-output directories, if applicable:
- `<GENERATED_OUTPUT_DIRECTORY>`

Primary validation command:
`<TEST_COMMAND>`

Replace placeholders when initializing a project.

## Instruction hierarchy

Apply the most specific current instruction that is compatible with higher-level
safety requirements:

1. explicit current user instructions;
2. the closest applicable subdirectory `AGENTS.md`;
3. this root `AGENTS.md`;
4. applicable global/project-local skills.

Project-local instructions may add stricter domain rules. Do not silently weaken
repository-level data, Git, validation, or safety constraints.

## Workflow skills

Use applicable installed skills rather than duplicating their detailed logic in
this file:

- `subagent-orchestration` — implementation ownership, automatic worker
  selection, Luna/LunaMax profiles, worker lifecycle, reviewer/fixer/validator
  coordination, HANDOFFs, escalation, and acceptance.
- `manual-subagent-handoff` — explicit user-requested manual delegation. Generate
  a self-contained prompt instead of auto-spawning the worker, require a terminal
  HANDOFF, and resume when the user pastes the HANDOFF back.
- `resource-aware-testing` — mandatory resource preflight before every test or
  validation command; guard CPU, RAM, GPU utilization, and GPU-memory usage.
- `test-validation` — TDD, focused/affected/full validation, trustworthy test
  evidence, validation levels, and temporary-test-artifact discipline.
- `level3-review` — independent read-only review for high-risk or
  correctness-sensitive changes.
- `project-memory` — concise repository-level state across sessions, branches,
  terminals, and accepted worktree tasks.
- `scientific-experiment-integrity` — leakage prevention, experiment classes,
  reproducibility, preflight/formal separation, source fidelity, and result
  integrity.

Project-local skills may add domain-specific workflows.

## Main-agent role

The main agent is primarily:

- coordinator;
- architect/planner;
- integration manager;
- acceptance decision maker;
- reviewer/fixer/validator coordinator;
- Git coordinator;
- final reporter.

Preserve main-agent context for architecture, protocol, integration, acceptance,
and user-facing decisions.

For repository tasks that modify production source code or tests, implementation
is delegated by default through `subagent-orchestration`.

Default implementation ownership:

- localized implementation -> normal `Luna xhigh` worker;
- substantial or multi-module implementation -> independent `LunaMax` Worktree
  Chat/task.

Task smallness alone does not justify direct main-agent source/test
implementation.

Direct main-agent source/test implementation is allowed only under the exceptions
defined by `subagent-orchestration`, and the reason must be reported.

## Manual-dispatch override

If the user explicitly asks to manually open the worker/reviewer/fixer/validator
conversation, asks for a prompt to send to a new agent, or asks not to
auto-dispatch workers, use `manual-subagent-handoff`.

While that manual mode applies:

- do not auto-create the delegated worker;
- do not absorb delegated source/test implementation into the main agent;
- select the same role/profile that automatic orchestration would have selected;
- generate one self-contained copyable dispatch prompt;
- require a terminal HANDOFF;
- resume from the pasted HANDOFF without asking the user to restate the task.

One-shot versus task-scoped manual behavior is governed by
`manual-subagent-handoff`.

## Worker ownership and continuation

Each writable implementation lane has exactly one active owner.

Normal subagents are leaf workers. They must not create other workers,
subagents, reviewers, validators, fixers, or independent tasks.

Do not duplicate the same implementation concurrently between the main agent,
normal workers, and independent Worktree workers.

Once a worker owns an implementation lane, do not terminate, replace, or
reassign it merely because:

- it has been running for a while;
- it has not yet returned HANDOFF;
- tests are still failing inside its assigned scope;
- ordinary debugging/TDD cleanup is still in progress;
- context pressure or compaction occurred;
- a heavy test was downgraded by the resource guard.

Elapsed time alone is not worker-channel failure.

Use the precise worker continuation, terminal-state, and escalation rules in
`subagent-orchestration`.

Do not create a fixer merely because the original implementation worker still
has in-scope failing tests. The original owner normally completes its own
debugging/TDD loop. Use a focused fixer only when the conditions in
`subagent-orchestration` are met.

## Independent review

Choose review strength separately from implementation size.

For Level 3 review, prefer a separate ordinary read-only `LunaMax` reviewer
conversation.

Reviewer independence means a separate reviewer context that did not implement
the change and that inspects the exact intended final diff/commit/snapshot.

A separate reviewer Git worktree is not required merely to establish
independence. Use one only when an isolated checkout is materially necessary.

Main-agent inspection and implementer self-review do not count as independent
Level 3 review.

Use `level3-review` and `subagent-orchestration` for review triggers, evidence,
PASS/BLOCKING outcomes, fixer flow, and re-review.

## Resource-aware testing

Before **every** test or validation command, apply `resource-aware-testing`.

Check:

- CPU utilization;
- system RAM utilization;
- relevant/visible GPU utilization;
- relevant/visible GPU-memory utilization.

Threshold: `80%`.

If any monitored resource is already at/above 80%, or available evidence
indicates the planned command may reach/exceed 80%, downgrade to the lightest
semantically faithful validation.

A previous safe preflight does not authorize later test commands; preflight again
before each command.

Do not change the contract under test merely to reduce resource use.

If no faithful lightweight substitute exists, mark the required validation as:

- `DEFERRED_RESOURCE_GUARD`, or
- `NOT_RUN_RESOURCE_GUARD`.

Never report deferred validation as passed.

Do not kill, pause, or reconfigure unrelated user processes merely to make room
for tests.

## Test and validation policy

Use `test-validation`.

Prefer:

`focused test -> affected group -> broader/full validation when risk justifies it`

For bug fixes, preserve the normal TDD/debugging flow where practical:

`reproduce -> RED -> minimal fix -> GREEN -> affected validation`

Do not weaken production behavior merely to make tests pass.

Tests should exercise real production logic whenever practical.

Validation evidence should distinguish:

- passed;
- failed;
- skipped/deselected;
- unrelated known baseline failure;
- resource-guard deferred/not-run;
- not run.

## Project memory

Use `project-memory` when repository instructions require it or when
`.project-memory/` exists.

Canonical integration branch:

`<INTEGRATION_BRANCH>`

Default layout:

`.project-memory/{STATUS.md,GOALS.md,NEXT.md,LOG.md}`

On the integration branch, local project memory is canonical.

In other branches/worktrees, treat checked-out `.project-memory/` as a local
snapshot and prefer canonical integration-branch memory according to
`project-memory`.

Independent worker worktrees must not edit canonical project memory unless the
task brief explicitly grants memory ownership.

The parent/main agent synchronizes canonical memory only after it validates and
accepts/integrates delegated work.

Project memory is a navigation/handoff layer, not a substitute for source code,
Git history, tests, specifications, or HANDOFF evidence.

## Scope and implementation economy

Do not modify unrelated files.

Prefer the smallest implementation that fully satisfies the requested behavior.

Reuse existing modules, helpers, interfaces, and pipelines before creating new
ones.

Avoid:

- speculative extensibility;
- duplicate pipelines;
- unnecessary factories/registries/adapters/wrappers;
- broad refactors;
- opportunistic cleanup/reformatting;
- unrelated renaming;
- unrelated file changes.

## Working-tree and Git safety

Assume existing uncommitted changes may belong to the user.

Do not reset, overwrite, restore, stash, delete, or commit unrelated user work.

Do not:

- create nested Git repositories;
- force-push;
- rewrite history without authorization;
- push unless requested.

Before commit inspect:

- `git status`
- `git diff`
- `git diff --check`

Stage only intended files when unrelated changes may exist.

After staging inspect:

- `git diff --cached`
- `git diff --cached --check`

Do not commit secrets, private datasets, large generated artifacts, or temporary
files.

## Data and output safety

Treat real datasets and user data as read-only by default.

Do not overwrite, rename, convert in place, delete, or silently repair raw data.

Keep derived data and generated outputs separate from raw/source data.

Use dedicated generated-output roots and respect project-specific output
boundaries.

Synthetic engineering results must not be presented as real experimental
results.

## Scientific and experimental integrity

For scientific/research work, use `scientific-experiment-integrity`.

Keep distinct:

- engineering smoke test;
- preflight;
- baseline experiment;
- tuning experiment;
- final evaluation.

Do not present smoke/preflight/incomplete/aborted runs as formal results.

Prevent train/validation/test leakage.

Do not silently change seeds, splits, preprocessing, stopping criteria,
checkpoint selection, or metric semantics and present the result as the same
experiment.

Observation/logging/telemetry must remain observation-only.

When reproducing a paper/spec/protocol/reference implementation, distinguish:

- source-specified behavior;
- project assumptions;
- deliberate deviations;
- unresolved details.

Do not claim exact reproduction when material details remain unresolved.

## Long-running processes

Multi-hour training, simulations, builds, or services should use a persistent
external process when appropriate rather than depending on an agent remaining
alive.

Record relevant process identity, logs, configuration/run identity, and output
location.

Agent lifecycle and OS-process lifecycle are separate.

## Dependencies and security

Do not casually install or broadly upgrade dependencies.

Never expose or commit:

- credentials;
- API keys;
- tokens;
- private keys;
- passwords;
- private user data.

Do not bypass access restrictions or execute destructive commands without clear
need and authorization.

## HANDOFF and acceptance

Treat HANDOFF as evidence, not automatic acceptance.

The main agent should validate delegated results using the smallest sufficient
trustworthy evidence set.

When risk requires it, inspect:

- changed files;
- Git state;
- focused/affected tests;
- resource-guarded omissions;
- high-risk diff sections;
- independent review findings.

Do not accept work solely because a worker says it is complete.

## Final acceptance

Before completion verify, as applicable:

- requested behavior is implemented;
- delegated ownership rules were respected;
- trustworthy validation passed;
- resource-deferred validation is explicitly reported;
- reviewer blockers are resolved when review was required;
- no unrelated files changed;
- raw/user data remained unchanged;
- generated outputs are isolated;
- intended files only were committed;
- unrelated pre-existing user changes remain untouched;
- documentation reflects verified behavior;
- canonical project memory is synchronized when required.

## Final report

Keep the final report concise and evidence-dense.

Include, when relevant:

- what changed;
- changed files;
- implementation owner/profile;
- validation commands/results;
- resource-guard decisions/deferred tests;
- reviewer result;
- unresolved issues;
- commit hash;
- final Git status;
- whether canonical project memory was synchronized;
- any orchestration deviation, including direct main-agent source/test
  implementation and its reason.
