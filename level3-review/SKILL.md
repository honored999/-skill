---
name: level3-review
description: Perform independent read-only review for high-risk or correctness-sensitive repository changes. Use for numerical/geometric correctness, coordinate transforms, preprocessing, leakage-sensitive logic, train/validation/test splitting, metrics, serialization/checkpoint compatibility, destructive filesystem behavior, raw/user data safety, security-sensitive behavior, cross-module contracts, large difficult diffs, or external-spec reproduction with unresolved details.
---

# Level 3 review

## Purpose

Use independent review when automated tests and routine main-agent acceptance do
not provide enough confidence in correctness, safety, data integrity, or
cross-module behavior.

Review is risk-based, not mandatory for every task.

Independent review means a separate reviewer context that did not implement the
change. It does **not** require a separate Git worktree by default.

## Default reviewer form

For Level 3 review, prefer a separate ordinary reviewer conversation:

- explicitly request `LunaMax` when supported;
- keep the reviewer read-only;
- provide the exact final commit/diff/snapshot;
- keep the implementation owner separate from the reviewer;
- collect `PASS`, `BLOCKING`, and optional `NON-BLOCKING` findings.

Do not create a Worktree Chat merely to satisfy reviewer independence.

Use a reviewer worktree only when an isolated checkout is materially necessary,
for example when the reviewer must run commands against its own filesystem state,
the intended target cannot otherwise be exposed reliably, or isolation is
required for safe inspection.

Use `subagent-orchestration` for reviewer creation, profile selection, target
exposure, lifecycle, and fallback rules.

## Typical Level 3 triggers

Use Level 3 review when changes affect one or more of:

- numerical or geometric correctness;
- coordinate transforms or resampling;
- dataset construction or preprocessing;
- train/validation/test splitting;
- leakage-sensitive logic;
- metrics or evaluation semantics;
- checkpoint or serialization compatibility;
- destructive filesystem behavior;
- raw or user data safety;
- security-sensitive behavior;
- cross-module public interfaces;
- large or difficult-to-reason-about changes;
- reproduction of an external paper/specification where correctness depends on
  unresolved details.

Independent review is normally unnecessary for well-tested low-risk changes such
as small configuration additions, straightforward CLI plumbing, documentation
changes, simple helpers, localized model options using existing interfaces, or
mechanical fixture corrections.

## Reviewer scope

Reviewers are normally read-only.

Inspect the highest-risk invariants and relevant final diff rather than
re-reading the entire repository.

A reviewer must independently inspect enough evidence to support its conclusion.

Existing implementer/fixer/test evidence may be used as background but must not
substitute for checking the actual risky code path.

Main-agent inspection and implementer self-review do not count as independent
Level 3 review.

## Required outcomes

A reviewer must distinguish:

- `PASS`
- `BLOCKING`
- optional `NON-BLOCKING`

A `BLOCKING` finding must include:

- clear issue;
- concrete evidence;
- relevant file/function or reproducible location;
- why acceptance criteria are violated;
- smallest reasonable fix scope.

Do not classify style preferences, optional extra tests, documentation polish,
harmless warnings, minor optimization, or speculative concerns as blocking by
themselves.

## Validation sequence

Typical workflow:

`implementer -> focused tests -> affected tests -> ordinary independent reviewer conversation`

If `BLOCKING`:

`fixer -> focused revalidation -> affected revalidation -> focused independent re-review`

A reviewer worktree may be introduced only when evidence access or isolation
requires it; it is not part of the default Level 3 sequence.

## Main-agent acceptance after PASS

After reviewer PASS, the main agent should:

- confirm the reviewer examined the intended final diff/commit/snapshot;
- inspect Git status and changed files;
- inspect targeted high-risk areas if needed;
- verify visible validation evidence;
- confirm no blocker remains;
- perform repository commit checks.

## Suggested output

Keep the review concise.

`PASS`

or:

`BLOCKING`
- `<file/function>` — issue, evidence, violated contract, minimal fix

Optional:

`NON-BLOCKING`
- concise evidence-backed risk or improvement

Also record the exact reviewed commit/diff/snapshot identity and requested/
observed reviewer profile when verifiable.
