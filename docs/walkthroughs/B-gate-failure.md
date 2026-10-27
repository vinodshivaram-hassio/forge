# Walkthrough B: a task that fails gates and loops

Status: Proposed · 2026-10-27

The root "let customers export their invoices as CSV" fails two gates in round 1 and one in round 2, passes in round 3, and never reaches decomposition with an open flag. Every loop returns to stage 1 through the task list.

## Trace

| Seq | Stage | Task | Agent | Verdict | Status after | Why |
| --- | --- | --- | --- | --- | --- | --- |
| 1 to 6 | create to clarify | T-0101 | intake, user, clarifier | accurate | ready-for-review | Clean context: F3 export includes customer name and email; AC1 to AC3 |
| 7 | review round 1 | T-0101 | reviewer-security | fail | none yet | R1: email is personal data, no Constraint covers where the file lives or who can fetch it (SD2) |
| 8 | review round 1 | T-0101 | reviewer-clarity | fail | none yet | R2: AC2 "their invoices" reads as all invoices or the current period (CL1) |
| 9 | review round 1 | T-0101 | 5 other reviewers | pass | none yet | |
| 10 | merge | T-0101 | orchestrator | round failed | open | Two blocking fails; flags written; review_rounds = 1 |
| 11 | clarify | T-0101 | clarifier | not-accurate | blocked | R1 and R2 need product decisions; one batched needs-human task T-0102 asks Q4 (date range) and Q5 (file retention and access) |
| 12 | human | T-0102 | user | answered | done; T-0101 open | Q4: last 12 months. Q5: generate on request, never store, link expires after 15 minutes |
| 13 | clarify | T-0101 | clarifier | accurate | ready-for-review | AC2 withdrawn, AC4 added with the 12-month range; C3 added for retention and expiry; R1, R2 marked resolved |
| 14 to 20 | review round 2 | T-0101 | 7 reviewers | 6 pass, 1 fail | open | R3 completeness: C3 link expiry has no AC (CO2); review_rounds = 2 |
| 21 | clarify | T-0101 | clarifier | accurate | ready-for-review | AC5 added: a link 16 minutes old returns HTTP 410; R3 resolved |
| 22 to 28 | review round 3 | T-0101 | 7 reviewers | 7 pass | approved | review_rounds = 3; continues as in walkthrough A from decompose |

## Review Flags after round 3

```markdown
## Review Flags
- R1 [gate: security, round: 1, resolved] Cites: F3, AC2. Export contains customer email; no Constraint covers file storage or access. Pass when: a Constraint and an AC cover storage and who can fetch the file.
- R2 [gate: clarity, round: 1, resolved] Cites: AC2. "Their invoices" has two readings: all invoices or the current period. Pass when: the AC states the date range.
- R3 [gate: completeness, round: 2, resolved] Cites: C3. Link expiry has no acceptance criterion. Pass when: an AC tests the expiry.
```

Full context file: `examples/context/T-0101-invoice-export.md`.

## The guard branch

Had round 3 failed, `review_rounds` would have hit the limit of 3. The orchestrator then writes a trace entry, sets T-0101 to blocked with `owner: user`, and creates one needs-human task carrying R1 to R3 and every round's verdicts. The user can answer, raise the dial, reword the root or cancel.

## What this walkthrough changed

- **Acceptance Criteria ownership:** the clarifier must be able to add and withdraw ACs after a clarification, or no role could fix R2. The ownership table in `specs/context-file-v1.md` now says so.
- **Batching:** R1 and R2 both needed the user, so the clarifier raised one needs-human task with two questions, as brief section 14 requires.
- **Flag visibility:** reviewers in round 2 saw R1 and R2 as resolved flags, which is how the completeness reviewer spotted that C3, added to fix R1, had no AC.
