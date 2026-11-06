---
name: resource-aware-testing
description: Check resource headroom before model/data-heavy tests, training, inference or benchmarks; pure text, AST, hash and Git work needs no resource polling.
---
# Scope
Apply before resource-heavy tests, real image decoding/cache creation, model loading, training/inference or benchmarks. Do not add a resource-preflight loop to ordinary editing, text search, AST/JSON parsing, Git or hash-only checks. Group lightweight checks instead of repeated monitoring for each trivial command.

# Headroom
Sample CPU over a short window, system RAM, relevant GPU compute/VRAM and disk when outputs are large. Use actual measurements plus conservative known workload characteristics; do not invent projected peaks. Before starting heavy validation, if current or credible projected relevant resource usage is >=80%, reduce batch/workers/fixture size without changing semantics, wait or defer. Do not stop unrelated jobs. A previous reading does not authorize a materially heavier workload or a later changed environment.

Preserve sufficient RAM/VRAM/disk and an explicit worker/prefetch budget. CPU parallelism should reduce measured or credible I/O overhead, not oversubscribe inner BLAS/Torch/ITK pools. Avoid benchmark campaigns unless requested or necessary to select a viable configuration.

If safe equivalent validation is unavailable, record DEFERRED_RESOURCE_GUARD; do not claim it passed. If the user requested static-only, do static work and leave actual execution to its authorized stage. Report useful measurements once with the outcome, not a repetitive checklist.
