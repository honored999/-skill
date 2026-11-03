---
name: manual-subagent-handoff
description: Prepare task briefs and copyable launch prompts for user-dispatched execution in a new conversation, DeepSeekHarness, or another agent/harness. Use by default for delegated implementation, review, fixes, and validation; resume acceptance from returned HANDOFF evidence.
---

# Manual task dispatch and handoff

## Standing workflow

Manual dispatch is the user's persistent default, across tasks, until explicitly
changed. Prepare the execution package; the user chooses and launches the executor.

`main agent -> task brief + launch prompt -> user dispatches -> executor -> terminal HANDOFF -> main agent verifies and continues`

Do not automatically create workers, chats, worktrees, reviewers, fixers, validators,
or replacement agents. A follow-up review or fix also receives a new manual brief.
Read-only Q&A and coordination-file maintenance do not require worker dispatch.
Do not silently take over delegated production source/test implementation.

## Profile and executor selection

Default requested profile for implementation, review, fixing, and validation:
`gpt6.1sol low`. Preserve this exact user configuration label; do not invent an API
identifier or claim the model/reasoning was used without observable evidence.

The user may choose a new chat, DeepSeekHarness, another harness, or another model.
If the executor is unspecified or the profile unavailable, mark executor selection
as user-selected/pending. Do not silently substitute a model or launch an agent.
A clear task can still receive an executor-neutral package immediately.
If task requirements themselves are unclear, list the unresolved requirements,
ask only material questions, and do not authorize dependent execution until resolved.

## Deliverable

For a delegated execution task provide:

1. A self-contained Markdown task brief, saved at the project's established task
   document location when available; otherwise use `docs/tasks/` with a dated,
   descriptive filename. For tasks without a repository, provide it inline unless
   the user specifies a permitted output directory. Do not overwrite another brief.
2. A copyable launch prompt that includes the exact brief path and essential scope,
   role, profile, no-recursion rule, and terminal HANDOFF requirement.
3. User-facing file link, suggested execution context, unresolved decisions, and
   instruction to paste the final HANDOFF back to the main agent.

For an external harness that cannot access local files or skills, include the full
brief and essential rules inline. A local path alone is insufficient. Do not copy
private data, secrets, or irrelevant conversation history into portable prompts.
Do not invent harness commands, flags, available tools, or filesystem access.

## Task brief contents

Include only relevant fields, but make execution independent of parent context:

- task identifier, role, goal, and explicit exclusions;
- requested profile `gpt6.1sol low`; executor selected by user;
- verified current state, repository/worktree, branch, and starting commit;
- exact readable/writable/forbidden scope and pre-existing dirty-state constraints;
- concrete requirements, authoritative sources, unresolved decisions and stop gates;
- acceptance criteria and appropriate focused/affected validation;
- runtime/environment constraints (prefer `newconda` where applicable);
- applicable AGENTS/skills and essential invariants reproduced for inaccessible tools;
- execution permissions: obey applicable sandbox/approval rules; never infer that a
  task brief or harness selection grants unavailable access;
- raw/user-data safety, output boundary, Git/commit/push rules;
- project-memory ownership: workers do not update canonical memory unless assigned;
- exact review snapshot/commit/diff for independent reviewers;
- all executors are leaf workers: no nested agents/tasks/reviewers/fixers/validators;
- terminal HANDOFF schema below.

An isolated worktree is recommended only when materially useful. The user launches
it or delegates its creation explicitly. Review independence needs a separate
non-implementer context, not necessarily a separate worktree.

## Executor continuation

Assign one active owner per implementation lane. The executor should complete
ordinary debugging within scope; elapsed time, compaction, a failing test, or one
failed attempt is not alone a reason to abandon the task. Out-of-scope decisions,
missing permissions, unavailable resources, or scientific-protocol changes return
to the main agent. Do not recursively delegate them.

## Terminal HANDOFF

Require the final output to end with:

```text
HANDOFF
- task identifier:
- status: COMPLETED / BLOCKED / FAILED
- role:
- executor/harness:
- requested model/profile: gpt6.1sol low (or explicit user override)
- observed model/profile/reasoning: evidence or unknown
- repository/worktree, branch, starting/ending commit:
- changed files or exact reviewed snapshot/diff:
- implemented behavior or review result: PASS / BLOCKING
- tests/validation: commands, visible results, skipped/deferred checks
- resource preflight/deferred validation when relevant:
- BLOCKING findings or blockers:
- NON-BLOCKING findings when relevant:
- git status and pre-existing user changes preserved:
- raw/user-data status and generated output locations:
- project-memory changes, if authorized:
- deviations, unresolved decisions, next action:
```

No important information should follow HANDOFF. A worker's `COMPLETED` is a claim
to verify, not automatic acceptance. Independent reviewers remain read-only and
must inspect the intended final snapshot themselves.

## Main-agent resumption

Match the returned HANDOFF to its brief; inspect scope, diff/snapshot, validation,
Git state, and deviations. Verify material evidence before accepting. Do not ask the
user to restate context already supplied. Prepare a new manual package for any
necessary reviewer/fixer/validator, with a separate context for independent review.
Synchronize canonical project memory only after accepted state is established and
when authorized by the owning repository workflow.

A one-shot automatic-dispatch override ends after its stated scope; the standing
manual default remains. Only an explicit user instruction changes that default.
