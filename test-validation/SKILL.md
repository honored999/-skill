---
name: test-validation
description: Choose proportionate evidence for behavior changes or requested validation; static checks suffice for low-risk edits and explicit static-only work.
---
# Choose evidence, not a fixed pipeline
- Docs/rules/paths/configuration and clear localized low-risk fixes: inspect diff and relevant syntax/schema/hash. Do not automatically add tests or run pytest.
- Material behavior changes: choose the smallest meaningful existing or new focused check. Reproduce an actual bug when useful; TDD is an option, not a mandatory sequence for every fix.
- Run affected/broader tests only when shared behavior, a new failure or unresolved risk justifies it. Passing unchanged evidence is reusable; do not repeat tests simply because a HANDOFF or review stage changed.
- User static-only/no-tests instruction overrides default testing advice: no test additions, pytest, benchmark or model execution. State NOT_RUN_USER_REQUEST and distinguish static from runtime evidence.
- A real model task still needs appropriate data/resource checks and a minimal preflight before a fresh formal run, unless equivalent unchanged evidence already exists.

Before resource-heavy execution use resource-aware-testing. Ordinary text/AST/hash/Git checks do not need CPU/GPU polling. Do not weaken production or mock away the behavior being verified. Use minimal fixtures; keep large artifacts outside sources and do not load real medical data for a pure contract check.

Report commands, actual visible outcomes and important unrun checks; no invented test counts or runtime claims from parsing. Checkpoint/geometry/concurrency risks may justify focused evidence, not automatically a full suite or separate validator.
