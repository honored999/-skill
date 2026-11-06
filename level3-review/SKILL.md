---
name: level3-review
description: Independent read-only review for material unresolved correctness or safety risk, or when explicitly requested; routine small edits do not automatically require it.
---
# When independent review helps
Use a separate non-implementer context when a change presents substantial unresolved risk to numerical/physical correctness, leakage, raw data, security, serialization or complex interfaces. A medical filename, a small source edit or a fixed finding alone is not a trigger. Do not add review solely to satisfy ceremony.

Default to a short manual reviewer brief unless automatic dispatch is authorized. The user chooses the model. No reviewer worktree is needed unless isolation has a concrete benefit. All reviewers are leaves and normally read-only.

# Scope and evidence
Identify the exact final commit/diff/hash. Independently inspect the risky path; do not treat implementer self-review as independent. Reuse unchanged prior findings/evidence. After a localized low-risk blocker fix, coordinator inspection can close it; independent re-review is needed only if the fix adds substantive unresolved risk or the user requests it.

Choose static review or focused runtime evidence according to risk and user instructions. An explicit static-only request prohibits adding tests/benchmarks/model runs; label PASS STATIC, record unrun items, and do not claim runtime success. No test absence is itself a blocker under that request.

# Result
Return PASS (or PASS STATIC), BLOCKING with concrete code/evidence/impact and the smallest fix, and optional NON-BLOCKING uncertainties. Do not block on style, speculative risks, optional coverage or unrelated improvements. Keep the report short. Static acceptance does not prove real source QC, performance, experiment completion or clinical benefit.
