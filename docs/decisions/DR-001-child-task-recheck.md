# DR-001: How child tasks are re-checked

Status: Proposed · 2026-10-27 · Brief open question 1

## Question

Do child tasks produced by decomposition go back through stages 2 to 5, or are they treated as already cleared?

## Choice

Light re-check. Every child runs stage 2 (validator) and stage 3 (clarifier), then the review panel runs only the gates whose definition lists `child` in `run_on` (default: Clarity, Testability). The full panel already ran on the parent.

## Escalation rule

A child gets the full panel when either holds. Both are checked by script.

- Its complexity is 4 or 5.
- Its Facts contain an item not present in the parent.

## Reasons

- The decomposer can introduce new ambiguity, so skipping all checks breaks principle 3 (no ambiguity downstream).
- A full panel per child multiplies review cost by up to 8 (the maximum children per parent).

## Rejected alternatives

- Full panel on every child: cost.
- No re-check: lets decomposer errors reach the test writer unreviewed.
