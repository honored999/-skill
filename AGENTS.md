# Repository Agent Instructions

## Project configuration

Project name: `<PROJECT_NAME>`
Repository root: `<REPOSITORY_ROOT>`

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

- `subagent-orchestration` — normal-subagent delegation, automatic independent
  Worktree Chat/task creation, LunaMax long-task workers, result collection,
  manual top-level-thread fallback, monitoring, fixer flow, and acceptance.
- `project-memory` — concise repository-local state for cross-session and
  cross-terminal continuity; initialization, startup reading, completion sync,
  worktree-safe ownership, and recent project history under `.project-memory/`.
- `level3-review` — independent review for high-risk/correctness-sensitive work.
- `test-validation` — TDD, focused/affected/full validation, test evidence, and
  temporary-test-artifact discipline.
- `scientific-experiment-integrity` — scientific experiments, leakage prevention,
  preflight/formal separation, reproducibility, and result integrity.

Project-local skills may add domain-specific workflows.

## Project memory

Use `project-memory` for durable repository-local continuity.

The default repository memory is:

```text
.project-memory/
├─ STATUS.md
├─ GOALS.md
├─ NEXT.md
└─ LOG.md
```

At the start of a repository task, when `.project-memory/` exists, read
`STATUS.md`, `GOALS.md`, and `NEXT.md`; inspect only the recent part of `LOG.md`
when useful. Treat memory as a concise navigation aid, not as authority over the
current checkout, Git state, source code, tests, `AGENTS.md`, or explicit specs.
Verify stale or correctness-sensitive statements against authoritative sources.

When this template is active and `.project-memory/` does not yet exist, initialize
it during the first repository-modifying task after enough inspection to avoid
guessing. Do not create or modify project memory for a purely read-only/advisory
task unless explicitly requested.

Before the final report of a completed repository-modifying task, synchronize
project memory after implementation and validation state is known:

- `STATUS.md` — current verified project state and important constraints;
- `GOALS.md` — stable project goal and current milestone; change only when the
  goal or milestone actually changes;
- `NEXT.md` — immediate next actions and concrete blockers;
- `LOG.md` — one short recent-task entry with changes and validation.

Keep memory short and operational. Do not copy full conversations, diffs,
tracebacks, command logs, generated reports, secrets, private data, or large
artifacts into it. Re-read the memory files immediately before writing and
preserve newer or concurrent edits instead of blindly overwriting them.

The main agent may update `.project-memory/*.md` directly as coordination
metadata even when implementation changes are delegated. This is a narrow
exception to implementation delegation, not permission to edit unrelated docs.

An independent worker/worktree created by another parent agent must not modify
`.project-memory/` unless the task brief explicitly assigns memory ownership.
It should return a concise HANDOFF; the parent updates project memory only after
validating/accepting the worker result. Do not describe unmerged, rejected, or
unverified worker changes as current project state.

Project memory is intended to be tracked in Git unless project-specific policy
says otherwise. When the current agent owns the task commit, include the memory
sync in the same commit when practical; do not create an extra commit solely to
record the hash of the commit that already contains the memory update.

## Main-agent role

The main agent is coordinator, architect/planner, integration manager, acceptance
decision maker, reviewer coordinator when justified, Git coordinator, and final
reporter.

Preserve main-agent context for architecture, protocol, integration, acceptance,
and user-facing decisions.

Use `subagent-orchestration` to select the execution mode:

- normal scoped subagent for localized/short work;
- automatically created independent Worktree Chat/task for substantial or
  long-running implementation/review work when supported;
- user-created top-level LunaMax thread only as fallback when automatic creation
  or result retrieval is unavailable;
- persistent external process for genuinely long execution.

Do not routinely reconstruct a trusted worker's full local investigation.

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

Do not create nested Git repositories, force-push, rewrite history without
authorization, or push unless requested.

Before commit inspect:

- `git status`
- `git diff`
- `git diff --check`

Stage explicit intended files when unrelated changes may exist.

After staging inspect:

- `git diff --cached`
- `git diff --cached --check`

Do not commit secrets, private datasets, large generated artifacts, or temporary
files.

## Data and output safety

Treat real datasets and user data as read-only by default.

Do not overwrite, rename, convert in place, delete, or silently repair raw data.

Keep derived data separate from raw data.

Synthetic results must not be presented as real experimental results.

Store generated artifacts under dedicated generated-output directories.

Enforce output-root boundaries using normalized resolved paths.

## Production and tests

Do not weaken production behavior merely to make tests pass.

Tests should exercise real production logic whenever practical.

Use `test-validation` for detailed validation strategy.

## Dependencies and security

Do not casually install or broadly upgrade dependencies.

Never expose or commit credentials, tokens, keys, passwords, or private user
data.

Do not bypass access restrictions or execute destructive commands without clear
need and authorization.

## Documentation and source fidelity

Documentation must reflect verified behavior.

When implementing from a paper/spec/API/protocol/reference project, distinguish
source-specified behavior, project assumptions, deliberate deviations, and
unresolved details.

Do not silently replace ambiguity with common practice.

## Final acceptance

Before completion, verify relevant acceptance criteria: requested behavior,
trustworthy validation, reviewer blockers resolved when required, no unrelated
changes, generated outputs isolated, raw/user data unchanged, intended files
only committed, documentation consistent with verified behavior, and project
memory synchronized when the task changed repository state.

## Final report

Keep the final report concise and evidence-dense: what changed, changed files,
validation/results, reviewer result when required, unresolved issues, commit
hash, final Git status, and whether project memory was synchronized when
applicable.
