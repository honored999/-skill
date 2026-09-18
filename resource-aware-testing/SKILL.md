---
name: resource-aware-testing
description: Guard every test execution with CPU, system-memory, GPU-utilization, and GPU-memory preflight checks. Use before running any test, validation, benchmark-like smoke test, model-loading test, or test-suite command. If any relevant resource is projected to reach or exceed 80%, automatically downgrade to the lightest trustworthy validation and report any validation that must be deferred.
---

# Resource-aware testing

## Purpose

Prevent validation from pushing the machine into unsafe or disruptive resource
pressure.

Before **every** test or validation command, perform a resource preflight.

Monitor:

- total CPU utilization;
- system RAM utilization;
- GPU compute utilization for each visible/relevant GPU;
- GPU memory utilization for each visible/relevant GPU.

Threshold:

`80%`

If any monitored resource is already at or above 80%, or the planned test is
reasonably expected to make any monitored resource reach or exceed 80%, do not
run that test as planned. Downgrade to a lighter trustworthy validation.

This rule applies to focused tests, affected groups, full suites, reruns after a
fix, smoke tests, integration tests, model-loading tests, and validation invoked
by workers, reviewers, fixers, or the main agent.

## No guessing disguised as certainty

Do not claim precise future utilization without evidence.

Estimate resource risk using the strongest available evidence, in this order:

1. measured peak from a previous equivalent test command in the same environment;
2. known resource characteristics of the same test/model/checkpoint/data path;
3. current utilization plus a conservative estimate from a closely related test;
4. test structure and scope: full suite, parallel workers, large model loading,
   real-data inference, checkpoint-heavy tests, GPU kernels, multiprocessing,
   large temporary arrays, or broad integration paths;
5. if a heavy/unknown test cannot be shown to have enough headroom, treat it as
   unsafe and choose lightweight validation.

Never invent a numeric projected peak.

## Preflight sampling

Immediately before each test command:

1. sample CPU utilization over a short window rather than one instantaneous
   reading when practical;
2. measure used/total system RAM;
3. query every visible/relevant GPU for:
   - compute utilization;
   - used/total VRAM;
4. identify the planned test's likely CPU/RAM/GPU pressure;
5. decide `NORMAL` or `LIGHTWEIGHT`.

Do the preflight again before each later test command. A previous safe reading
does not authorize subsequent commands.

## 80% decision rule

Choose `LIGHTWEIGHT` when any of the following is true:

- current CPU utilization >= 80%;
- current system RAM utilization >= 80%;
- current GPU utilization >= 80% on any visible/relevant GPU;
- current GPU-memory utilization >= 80% on any visible/relevant GPU;
- reliable historical/equivalent evidence predicts any resource will reach or
  exceed 80%;
- the test is materially heavy and available evidence cannot establish adequate
  headroom below 80%.

Otherwise the planned test may run normally.

When a resource is already heavily occupied by another process, do not kill,
pause, reconfigure, or interfere with that process merely to make room for tests.

## Lightweight validation policy

`LIGHTWEIGHT` does not mean "skip all testing".

Choose the smallest test that still exercises the changed behavior, in roughly
this order:

1. one exact regression test/node;
2. one focused test file;
3. a narrow affected subset;
4. tiny synthetic fixtures or reduced-size inputs when they preserve the
   contract under test;
5. single-process/single-worker execution rather than parallel test execution;
6. avoid unnecessary model/checkpoint/data loading;
7. avoid full-suite, broad integration, real-data, multi-GPU, or large
   checkpoint-heavy validation unless those behaviors are exactly what must be
   tested.

Do not invent project markers or flags. Use only test-selection mechanisms known
to exist in the repository/environment.

Prefer serial execution. Disable pytest-xdist or other parallel workers when
possible and when doing so does not change the contract being tested.

## Semantic fidelity

Resource reduction must not silently change the behavior being validated.

A lightweight substitute is acceptable only when it still tests the relevant
contract.

Examples:

- a tiny synthetic tensor may be valid for shape/geometry/error-handling logic;
- metadata-only checkpoint fixtures may be valid for metadata validation;
- CPU execution may be valid only when the contract is device-independent;
- mocking a heavy dependency may be valid only when the dependency's real
  behavior is not the subject of the test.

Do **not** claim equivalent validation when:

- CUDA-specific behavior is replaced by CPU behavior;
- real checkpoint compatibility is replaced by a fake checkpoint;
- real-data integration is replaced by a synthetic unit test;
- multiprocessing/concurrency behavior is replaced by serial execution;
- numerical behavior depends materially on the omitted heavy path.

If the required behavior cannot be tested safely under the 80% guard, mark that
validation as `DEFERRED_RESOURCE_GUARD` / `NOT_RUN_RESOURCE_GUARD`.

Do not present deferred validation as passed.

## Suggested resource checks

Use already-available tools. Do not install new dependencies merely for
preflight unless the user authorizes it.

### NVIDIA GPU

Prefer `nvidia-smi`, for example querying:

- utilization.gpu
- memory.used
- memory.total

Calculate VRAM utilization as:

`100 * memory.used / memory.total`

For multiple GPUs, inspect each relevant/visible GPU.

### Windows CPU/RAM

Use available PowerShell/CIM/counter tools or an already-installed Python
library. Prefer a short CPU sampling interval rather than a single stale value.

### Linux CPU/RAM

Use available system tools such as `free`, `/proc`, `top`, `vmstat`, or an
already-installed Python library.

The exact command is environment-dependent; the policy is not.

## Planned-load classification

Use this only as a conservative qualitative aid, not as a fabricated percentage.

Typical `LIGHT` work:

- one pure unit test;
- parser/config/path validation;
- small tensor logic;
- deterministic helper tests;
- tiny synthetic fixtures.

Typical `MODERATE` work:

- one model component;
- several focused tests;
- moderate tensor allocation;
- one small checkpoint fixture.

Typical `HEAVY` work:

- full pytest suite;
- xdist/multiprocess test execution;
- real model/checkpoint loading;
- real-data inference;
- GPU end-to-end tests;
- broad integration;
- multi-GPU tests;
- repeated architecture instantiation;
- tests known to create large temporary artifacts.

For `HEAVY` work with no trustworthy peak history, require clear resource
headroom; otherwise downgrade.

## Interaction with test-validation

`test-validation` decides **what evidence is desirable**.

`resource-aware-testing` decides **whether the planned command is safe to run
now**, and if not, what lighter evidence can be run without misrepresenting
coverage.

Resource safety does not lower correctness standards. It may reduce immediate
validation scope, but any omitted required validation must remain explicitly
reported as deferred/not run.

## Worker and reviewer behavior

Any agent that intends to run tests must perform this preflight itself before
each command.

A worker HANDOFF should include, when resource guarding affected validation:

- resource preflight result;
- resource(s) near/over threshold;
- planned command that was downgraded;
- lightweight command actually run;
- validations deferred because no semantically faithful lightweight substitute
  existed.

Do not terminate a healthy implementation worker merely because a heavy test was
downgraded. The worker should continue with the lightweight path and report
remaining deferred validation.

## Reporting

For materially relevant test runs, report:

```text
RESOURCE PREFLIGHT
- cpu: <measured % or unavailable>
- ram: <measured % or unavailable>
- gpu: <per-GPU utilization or unavailable>
- vram: <per-GPU utilization or unavailable>
- planned test: <command/scope>
- decision: NORMAL / LIGHTWEIGHT
- reason: <threshold/headroom evidence>
- actual test: <command/scope>
- deferred: <none or exact validation not run>
```

Do not spam identical preflight details for every trivial rerun in the final
user-facing report; keep the underlying evidence available and summarize the
material decisions.
