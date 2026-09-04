---
name: scientific-experiment-integrity
description: Protect scientific validity and reproducibility in experimental or research code. Use for dataset splits, leakage-sensitive preprocessing, formal experiments, preflight/smoke runs, tuning, final evaluation, reproducibility metadata, paper/spec reproduction, instrumentation, aborted runs, or reporting scientific results.
---

# Scientific experiment integrity

## Experiment classes

Distinguish:

- engineering smoke test;
- preflight;
- baseline experiment;
- tuning experiment;
- final evaluation.

A smoke test or preflight is not a formal result.

Do not aggregate incomplete or aborted runs into formal results unless the
protocol explicitly defines resumable aggregation.

Do not cherry-pick favorable runs.

Do not silently rerun poor outcomes with changed seeds/settings and present them
as the same experiment.

If a formal run aborts:

- preserve it when practical;
- mark it aborted;
- exclude it from final aggregation;
- record the reason when known.

## Data leakage

Avoid train/validation/test leakage.

Do not use test data for:

- training;
- validation;
- hyperparameter selection;
- normalization statistics;

unless the explicit protocol requires it.

When group metadata exists, preserve grouping.

Do not describe sample-level evaluation as subject-independent,
writer-independent, user-independent, patient-independent, or equivalent unless
the split actually enforces that property.

Project-specific leakage constraints belong in project-local `AGENTS.md` or
project-local skills.

## Preflight

When a formal workflow needs a preflight mode, implement it explicitly.

Do not simulate preflight by silently overriding formal configuration.

Preflight should use the same production paths/logic where appropriate and
differ only in documented engineering limits.

Mark preflight outputs as:

- non-formal;
- ineligible for formal aggregation;
- not a reported experimental result.

## Observation-only instrumentation

Logging, visualization, telemetry, and monitoring must be observation-only.

They must not change:

- model parameters;
- optimizer behavior;
- learning rate;
- random seed;
- data split;
- preprocessing;
- epochs;
- stopping criteria;
- checkpoint selection;
- final metric semantics.

Optional telemetry failure must not block computation.

Do not swallow genuine computation failures such as CUDA OOM, forward/backward
errors, optimizer failures, or invalid data.

## External-source fidelity

When reproducing a paper, benchmark, protocol, or reference implementation:

1. inspect the primary source;
2. inspect official documentation/code/data when available;
3. distinguish source-specified settings from project assumptions;
4. document deliberate deviations;
5. mark unresolved details as unresolved.

Do not silently replace ambiguity with common practice.

Do not claim exact reproduction when unresolved implementation details remain.

## Reproducibility

When reproducibility matters, record enough information to reconstruct a run.

Useful metadata may include:

- Git commit;
- configuration snapshot;
- random seed;
- environment;
- device;
- dataset version;
- exclusions;
- split policy;
- timestamps;
- run identifier.

Generated run metadata must not influence the experiment itself.

## Result integrity

Do not claim:

- numerical reproduction;
- benchmark success;
- deployment readiness;
- data validation;
- protocol confirmation;

until the relevant milestone was actually completed.

Clearly distinguish:

- verified;
- partially verified;
- blocked;
- unresolved;
- not started.

Synthetic engineering results must not be presented as real experimental
results.

## Failure handling

When an experiment fails, collect evidence before changing anything.

Record as relevant:

- command;
- exit code;
- traceback;
- affected stage;
- process state;
- generated artifacts;
- last completed unit;
- configuration;
- Git state.

Do not immediately retry with changed parameters.

Do not hide failed attempts.

Do not classify infrastructure interruption as a model defect without evidence.

## Long-running experiments

Multi-hour training, simulations, or services should not depend on a subagent
remaining alive unless process persistence is guaranteed.

When appropriate, launch a persistent OS process and record:

- PID/process identity;
- stdout/stderr;
- run identity;
- configuration;
- output location.

Monitor the process separately from agent lifecycle.
