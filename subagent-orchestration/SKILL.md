Add this section to `subagent-orchestration/SKILL.md`, preferably before
`## Implementation execution modes`:

## Explicit manual-dispatch override

If the user explicitly asks to manually open delegated worker conversations,
asks for a prompt to send to a new subagent/reviewer/fixer/validator chat, or
asks not to auto-dispatch workers, use `manual-subagent-handoff`.

This manual mode overrides automatic worker creation but does not change role
selection, model/profile choice, ownership, validation, review, or acceptance
requirements.

While manual mode applies:

- do not automatically create normal subagents, Worktree Chat/tasks, reviewers,
  fixers, or validators;
- do not absorb delegated production source/test implementation into the parent;
- choose the intended role/profile normally and generate a self-contained prompt;
- require a terminal HANDOFF;
- when the user pastes HANDOFF back, continue from that state without asking them
  to restate the task;
- in task-scoped manual mode, subsequent reviewer/fixer/validator work must also
  be emitted as prompts rather than automatically dispatched.

Manual mode may be one-shot or task-scoped. If scope is unclear, use one-shot.
