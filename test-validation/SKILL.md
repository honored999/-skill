---
name: test-validation
description: Plan and execute trustworthy software validation. Use for TDD, bug reproduction, choosing focused vs affected vs full tests, evaluating test evidence, managing large temporary test artifacts, selecting validation level, or deciding how much retesting is required after a fix.
---

# Test and validation

## Principle

Choose the lightest validation that provides trustworthy evidence.

Prefer:

`focused test -> affected group -> broader/full suite when risk justifies it`

Do not run an expensive full suite merely for ceremony when unchanged
high-risk behavior is already covered by strong evidence.

## TDD bug-fix flow

For a defect:

1. reproduce it;
2. add or update a focused regression test;
3. observe RED;
4. apply the minimal production fix;
5. observe GREEN;
6. run the affected test group;
7. broaden validation only when risk/shared code justifies it.

A RED caused by a genuinely missing module/API can be valid evidence when it
proves the new test detects absent functionality.

## Production versus tests

Do not weaken production behavior merely to make tests pass.

When a production contract becomes stricter, update outdated test fixtures.

Tests should exercise real production logic whenever practical.

Do not bypass the guard under test by mocking the guard itself to always
succeed.

Prefer dependency substitution around environment-specific roots/resources
while allowing actual validation logic to execute.

## What tests should verify

Where applicable, test:

- expected input/output shape;
- failure behavior;
- deterministic behavior;
- boundary conditions;
- malformed inputs;
- numerical finiteness;
- state isolation;
- data leakage;
- split correctness;
- serialization;
- output-path safety;
- failure propagation;
- end-to-end smoke behavior.

Tests should verify behavior, not merely execution.

## Test evidence

A validation command is trustworthy only when its result is trustworthy.

Prefer both:

- successful exit code;
- visible summary such as `74 passed`.

Do not treat empty/suspicious wrapper output as strong evidence merely because
an exit code appears successful.

If wrapper behavior is suspicious, use an equivalent command in the confirmed
environment rather than inventing success.

## Temporary test artifacts

Tests must not create unnecessarily large temporary artifacts.

For metadata, serialization structure, failure handling, provenance,
configuration, and other behavior not requiring real heavy resources:

- prefer minimal synthetic fixtures;
- do not instantiate/serialize a full production model solely to satisfy a
  file-format contract;
- do not copy large checkpoints, datasets, caches, or generated outputs into
  per-test directories unless the test specifically validates real-file
  compatibility.

Full-size artifacts are appropriate only when the test genuinely validates
behavior depending on them, such as:

- real model loading;
- end-to-end inference;
- checkpoint conversion compatibility;
- architecture/state compatibility;
- integration with an external runtime format.

Use a controlled temporary root when practical.

Avoid creating a new persistent temporary root for each rerun.

Temporary test artifacts are disposable and are not formal experiment outputs.

## Validation levels

### Level 1 — localized low-risk change

Typical flow:

`scoped implementation -> focused tests -> changed-file/status check -> targeted acceptance`

Independent review is not required by default.

### Level 2 — moderate change

Typical flow:

`scoped implementation -> focused tests -> affected group -> targeted diff inspection -> integration validation`

Use independent review only if the change crosses a correctness-sensitive
boundary or validation evidence is weak.

### Level 3 — high-risk/correctness-sensitive change

Typical flow:

`implementation -> focused tests -> affected group -> independent read-only review -> fixer if needed -> revalidation -> full validation when appropriate -> acceptance`

Use `level3-review` for the independent review itself.

Do not automatically promote a task to a higher validation level merely because
more testing is possible.

## Retesting after fixes

After a narrow fix:

1. rerun the focused regression;
2. rerun the directly affected group;
3. broaden only if shared/high-risk code changed or evidence is still weak.

Do not repeatedly rerun an unchanged full suite when nothing relevant changed.

## Reporting

Report exact commands and visible results when they materially support
acceptance.

Distinguish:

- passed;
- failed;
- deselected/skipped;
- known unrelated baseline failure;
- not run.

Do not silently hide failures or present partial validation as full validation.
