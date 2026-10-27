---
forge_format: 1
task_id: T-0101
parent_id: none
root_id: T-0101
type: intake
title: Let customers export their invoices as CSV
version: 7
priority: 3
complexity: 3
modalities: [text, data]
tags: [backend]
---

## Original Input
Let customers export their invoices as CSV.

## Goal
A signed-in customer can download a CSV of their account's invoices from the last 12 months through a short-lived link.

## Facts
- F1 [confirmed, src: billing/models.py] Invoices belong to an account; each has number, date, amount, currency and status.
- F2 [confirmed, src: billing/views.py] Only users of an account can read its invoices today.
- F3 [confirmed, src: billing/models.py] The export includes customer name and email, which are personal data.

## Constraints
- C1 CSV is UTF-8 with a header row and comma separators.
- C2 Only users of the account may request its export.
- C3 The file is generated on request, never stored, and its download link expires after 15 minutes.

## Acceptance Criteria
- AC1 Given a signed-in user of account X, when they request an export, then they receive a CSV with one row per invoice of account X.
- AC2 [withdrawn] Given a customer with invoices, when they export, then the file contains their invoices.
- AC3 Given a user of account Y, when they request an export for account X, then the server returns HTTP 403.
- AC4 Given account X has invoices dated 13 months ago and 1 month ago, when an export is requested, then only the invoice dated 1 month ago appears.
- AC5 Given a download link created 16 minutes ago, when it is opened, then the server returns HTTP 410.

## Out of Scope
- Exports in formats other than CSV.
- Scheduled or emailed exports.

## Artifacts
None

## Open Questions
- Q4 [answered, ask: user] Which invoices does an export cover?
- Q5 [answered, ask: user] Where does the file live, and for how long?

## Clarifications
- Q4 (answered) Which invoices does an export cover? A: Invoices dated in the last 12 months. Source: user, T-0102.
- Q5 (answered) Where does the file live, and for how long? A: Generated on request, never stored, link expires after 15 minutes. Source: user, T-0102.

## Decisions
- D1 AC2 withdrawn and replaced by AC4 to remove two readings of "their invoices". Reason: review flag R2.

## Review Flags
- R1 [gate: security-and-data, round: 1, resolved] Cites: F3, AC2. Export contains customer email; no Constraint covers file storage or access. Pass when: a Constraint and an AC cover storage and who can fetch the file.
- R2 [gate: clarity, round: 1, resolved] Cites: AC2. "Their invoices" has two readings: all invoices or the current period. Pass when: the AC states the date range.
- R3 [gate: completeness, round: 2, resolved] Cites: C3. Link expiry has no acceptance criterion. Pass when: an AC tests the expiry.
