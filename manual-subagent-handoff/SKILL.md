---
name: manual-subagent-handoff
description: Prepare concise briefs and launch prompts for user-requested or materially useful manual delegation; not required for every source edit or follow-up fix.
---
# Manual dispatch
When delegation is appropriate, the user selects and launches the executor. Do not automatically spawn agents/chats without explicit authorization. Clear localized work may be done by the current agent; this skill does not prohibit direct source fixes.

Save one self-contained brief at the established task location, otherwise docs/tasks/. Include only:
- goal, readable/writable scope and exclusions;
- current repo/branch/commit or exact review snapshot, relevant authoritative sources;
- concrete requirements and actual unresolved blockers;
- validation proportional to risk and user instructions;
- environment/approval and raw-data/Git safety;
- leaf-worker rule and concise HANDOFF.

Do not specify a model/profile; the user chooses. Provide a copyable prompt with the exact brief path, scope, no-recursion rule and HANDOFF request. External agents without local access need the relevant brief content, not an inaccessible path or copied secrets.

HANDOFF should state status COMPLETED/BLOCKED/FAILED, changes or reviewed snapshot, validation evidence and unrun items, relevant Git/output state and next action. Do not mandate a long fixed schema or model telemetry for a tiny task.

On return, inspect material evidence and continue. Reuse the existing brief/context for ordinary corrections; do not create a new fixer/reviewer/validator package merely to preserve process. Only independent high-risk review needs a different non-implementer context. Workers do not change canonical memory unless explicitly assigned.
