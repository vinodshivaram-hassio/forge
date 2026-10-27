# DR-004: How a root task is created

Status: Proposed · 2026-10-27 · Brief open question 4

## Question

Is a root task created from free text, or from a filled template?

## Choice

Free text. The intake agent drafts the context file; the draft goes to the user as a `needs-human` confirmation task before the clarifier runs. The user may also submit a filled context file, which skips the draft step but not validation.

## Reasons

- Low friction for the user.
- The confirmation step stops intake from silently inventing scope.

## Rejected alternatives

- Mandatory template: high friction, and users fill templates with vague wording the lint list would reject anyway.
