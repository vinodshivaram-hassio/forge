# DR-005: Where task status lives

Status: Decided · 2026-10-27

## Question

Should status stay in the context file header as well as the task record?

## Choice

No. The task record is the only home of status; `status` is removed from the context file header.

## Reasons

- The two files cannot be written atomically together, so a mirror can drift.
- Agents never act on status; only the orchestrator does, and it reads the record.
- An agent that needs it gets the task record alongside the context file.

## Rejected alternatives

- Header mirror with a warning on mismatch: adds a sync rule and a warning code for no reader.
