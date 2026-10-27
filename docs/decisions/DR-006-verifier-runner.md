# DR-006: How the verifier runs

Status: Decided · 2026-10-27

## Question

Should the verifier run as a script in v1, or as a model-backed agent?

## Choice

A model agent, tiers 4 to 8 with a floor of 4. Deterministic checks (test-to-criterion mapping, skip counts, test-file hashes) run as its `check_ac_coverage` tool, so the model reads facts rather than recomputing them.

## Reasons

- Some verify questions need judgement, such as whether a test still checks what its criterion says, or whether a red-check failure is for the expected reason.
- The tool keeps the mechanical parts deterministic, per principle 8.

## Rejected alternatives

- Script-only verifier: cheaper, but cannot judge whether a test was weakened in meaning rather than in count.
