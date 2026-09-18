---
name: manual-subagent-handoff
description: User-requested manual delegation mode. Use when the user asks to manually open a worker, reviewer, fixer, or validator chat; asks for a prompt to send to a new agent; or asks not to auto-dispatch subagents. Generate a self-contained dispatch prompt, require a terminal HANDOFF, and resume orchestration when the user pastes the HANDOFF back.
---

# Manual subagent handoff

## Purpose

Use only when the user explicitly requests manual delegation.

Manual delegation replaces automatic worker creation with:

`main agent -> generated prompt -> user opens new chat -> worker completes -> HANDOFF -> user pastes HANDOFF back -> main agent continues`

This changes worker transport only. It does not weaken ownership, model/profile,
validation, review, Git, data-safety, or acceptance rules.

## Activation and scope

Examples of activation intent:

- “这次我手动开子代理”
- “不要自动派发”
- “给我子代理提示词”
- “我自己发到新对话”
- “manual subagent / manual dispatch / manual handoff”

Two scopes:

1. One-shot: applies only to the next delegated role.
2. Task-scoped: applies to the current core task and descendant reviewer/fixer/
   validator steps until the task finishes or the user resumes automatic mode.

If ambiguous, use one-shot.

## Hard override

While manual mode applies, the main agent must not automatically create:

- normal subagents;
- Worktree Chat/tasks;
- reviewer conversations;
- fixers;
- validators;
- replacement workers.

Do not silently take over delegated source/test implementation in the parent.

Instead, choose the intended role/profile using `subagent-orchestration`, then
generate a copyable prompt for the user to send to a new chat.

## Preserve normal role/profile selection

Typical mapping:

- localized implementation -> `Luna xhigh`;
- focused fixer -> `Luna xhigh`;
- validator -> `Luna xhigh`;
- narrow low-risk reviewer -> `Luna xhigh`;
- substantial/multi-module implementation -> `LunaMax`;
- Level 3 independent reviewer -> `LunaMax`.

If the task normally needs an isolated worktree, say so in the dispatch header.
Do not create it automatically while manual mode applies.

## Required dispatch output

Output:

1. role;
2. recommended profile;
3. recommended context: ordinary new chat or Worktree Chat if materially needed;
4. manual scope: one-shot or task-scoped;
5. one self-contained prompt;
6. “完成后把最终 HANDOFF 原样粘贴回主代理”.

The prompt must include, when relevant:

- repository/worktree path;
- branch and starting commit;
- current Git/dirty-state constraints;
- task goal and verified current state;
- exact readable/writable/forbidden scope;
- project/scientific invariants;
- applicable `AGENTS.md`, skills, specs, papers, or plans;
- acceptance criteria;
- focused/affected validation;
- Git/commit/push rules;
- raw/user-data safety;
- project-memory ownership;
- no-recursion rule;
- terminal HANDOFF schema.

Do not paste irrelevant parent-conversation history.

## Worker continuation

The manual worker owns its assigned lane until a legitimate terminal state.

Do not tell it to stop merely because:

- tests fail within scope;
- debugging is in progress;
- elapsed time is long;
- context compacted;
- the first attempt failed.

It should continue normal debugging/TDD cleanup and end with exactly one status:

- `COMPLETED`
- `BLOCKED`
- `FAILED`

A normal manual worker must not create other workers/reviewers/fixers/validators.

## HANDOFF: implementation/fixer/validator

Require:

```text
HANDOFF

- status: COMPLETED / BLOCKED / FAILED
- role:
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
- failing tests or blockers:
- git status:
- raw/user data status, when relevant:
- project-memory changes, if any:
- scope deviations:
- unresolved issues:
- next recommended action:
```

## HANDOFF: independent reviewer

Require:

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
- evidence gaps:
- scope deviations:
- next recommended action:
```

No important information should appear after the HANDOFF.

## When HANDOFF is pasted back

The main agent should:

1. match it to the pending manual dispatch;
2. inspect status, role, changed/reviewed state, tests, Git state, and deviations;
3. validate repository evidence when normal acceptance rules require it;
4. continue without asking the user to restate the task;
5. if another worker/reviewer/fixer is needed and task-scoped manual mode remains
   active, generate the next manual prompt instead of auto-spawning it.

HANDOFF is evidence, not automatic acceptance.

## Ending manual mode

One-shot ends after its HANDOFF is consumed.

Task-scoped mode ends when:

- the core task is accepted/finished;
- the user asks to resume automatic dispatch;
- the user explicitly asks the main agent to take over.

Do not carry manual mode into unrelated future tasks.
