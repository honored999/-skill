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

Apply, in order:

1. this root `AGENTS.md`;
2. the closest applicable subdirectory `AGENTS.md`;
3. explicit user instructions.

Lower-level instructions may add stricter project-specific requirements but must
not silently weaken repository-level safety rules.

## Workflow skills

Use applicable installed skills:

- `subagent-orchestration` — implementation-mode selection, scoped normal
  subagents, independent Worktree Chat/tasks for substantial implementation,
  ordinary independent reviewer conversations, monitoring, fixer flow, and
  acceptance.
- `project-memory` — concise repository-level state for cross-session,
  cross-terminal, branch, and worktree continuity.
- `level3-review` — independent read-only review for high-risk/correctness-
  sensitive work, normally through a separate ordinary reviewer conversation.
- `test-validation` — TDD, focused/affected/full validation, test evidence, and
  temporary-test-artifact discipline.
- `scientific-experiment-integrity` — scientific experiments, leakage prevention,
  preflight/formal separation, reproducibility, and result integrity.

Project-local skills may add domain-specific workflows.

## Main-agent role

The main agent is coordinator, architect/planner, integration manager, acceptance
decision maker, reviewer coordinator when justified, Git coordinator, and final
reporter.

Production source/test implementation is delegated by default according to
`subagent-orchestration`. Task smallness alone does not justify direct main-agent
implementation.

Use `subagent-orchestration` to select implementation mode:

- normal scoped `Luna xhigh` subagent for localized/short implementation;
- independent `LunaMax` Worktree Chat/task for substantial implementation when
  isolation or context separation is useful;
- persistent external process for genuinely long execution.

Choose review strength separately from implementation mode. For Level 3 review,
prefer a separate ordinary read-only `LunaMax` reviewer conversation. Do not
create a reviewer worktree merely to satisfy independence; use one only when an
isolated checkout is materially necessary to inspect the intended final state.

## Project memory

Use `project-memory` for repository-level continuity.

Canonical integration branch:

`<INTEGRATION_BRANCH>`

Memory layout:

`.project-memory/{STATUS.md,GOALS.md,NEXT.md,LOG.md}`

Independent workers must not modify canonical project memory unless explicitly
assigned memory ownership.

## Scope and implementation economy

Do not modify unrelated files.

Prefer the smallest implementation that fully satisfies the request.

Reuse existing modules/interfaces/helpers/pipelines before creating new ones.

Avoid speculative extensibility, duplicate pipelines, broad refactors,
opportunistic cleanup/reformatting, unnecessary factories/registries/adapters,
and unrelated file changes.

## Working-tree and Git safety

Assume existing uncommitted changes may belong to the user.

Do not reset, overwrite, restore, stash, delete, or commit unrelated user work.

Do not force-push or rewrite history without authorization.

Do not push unless requested.

Before commit inspect:

- `git status`
- `git diff`
- `git diff --check`

After staging inspect:

- `git diff --cached`
- `git diff --cached --check`

Do not commit secrets, private datasets, large generated artifacts, or temporary
files.

## Data and output safety

Treat real datasets and user data as read-only by default.

Keep derived data separate from raw data.

Synthetic results must not be presented as real experimental results.

## Production and tests

Do not weaken production behavior merely to make tests pass.

Tests should exercise real production logic whenever practical.

## Final acceptance

Before completion, verify relevant acceptance criteria: requested behavior,
trustworthy validation, reviewer blockers resolved when required, no unrelated
changes, raw/user data unchanged, intended files only committed, documentation
consistent with verified behavior, and project memory synchronized when
applicable.

## Final report

Keep the final report concise and evidence-dense: what changed, changed files,
validation/results, reviewer result when required, unresolved issues, commit
hash, final Git status, and whether project memory was synchronized when
applicable.
