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

## Evidence completeness

Do not return PASS before required evidence is collected.

If a required specification, file, runtime artifact, or authoritative source is
missing:

1. try to locate it within allowed scope;
2. if unavailable, report the evidence gap;
3. do not guess its contents;
4. do not silently convert the gap into PASS.

If context compaction occurs during review, re-read the authoritative spec
sections and high-risk code required for the final conclusion.

## No premature reviewer termination

The main agent must allow the reviewer to finish its assigned evidence
collection.

While the reviewer is:

- reading required files/specifications;
- tracing a relevant call path;
- inspecting the final diff;
- running required focused regression;
- resolving an evidence gap;

do not force a terminal conclusion.

Use `subagent-orchestration` for lifecycle and sparse-monitoring rules.

A premature reviewer conclusion is not valid final review evidence for a
correctness-sensitive task.

## Validation sequence

Typical workflow:

`implementer -> focused tests -> affected tests -> independent reviewer`

If `BLOCKING`:

`fixer -> focused revalidation -> affected revalidation -> focused independent re-review`

Run a full suite when changed shared/high-risk code justifies it. Do not rerun
expensive full validation mechanically when focused/affected evidence is
sufficient.

## Main-agent acceptance after PASS

After reviewer PASS, the main agent should:

- confirm the reviewer examined the intended final diff;
- inspect Git status and changed files;
- inspect targeted high-risk areas if needed;
- verify visible validation evidence;
- confirm no blocker remains;
- perform repository commit checks.

A post-commit audit may validate an existing commit, but do not claim it was a
pre-commit review if the actual order was different.

## Suggested output

Keep the review concise.

Example:

`PASS`

or:

`BLOCKING`
- `<file/function>` — issue, evidence, violated contract, minimal fix

Optional:

`NON-BLOCKING`
- concise evidence-backed risk or improvement
