# Forge Design Specification v1

2026-10-27 · Vinod Shivaram

This spec turns the Forge design brief into the five design deliverables from roadmap section 18. Every item is **proposed** until marked decided; nothing here changes a principle in brief section 2.

| # | Deliverable | File | State |
| --- | --- | --- | --- |
| 1 | Decision records DR-001 to DR-006, glossary additions | `decisions/`, `glossary.md` | Proposed |
| 2 | Context file spec v1 | `specs/context-file-v1.md` | Proposed |
| 3 | Task record, status model, TaskStore interface | `specs/task-record-and-store.md` | Proposed |
| 4 | Agent and gate definition schemas, roster, gates | `specs/agents-and-gates.md`, `../config/` | Proposed |
| 5 | Two walkthroughs written as traces | `walkthroughs/` | Proposed |

## Decisions needed

In order of impact:

1. Accept or change DR-001 (how child tasks are re-checked). It sets the cost of every decomposed task.
2. Accept or change DR-003 (YAML task list). It fixes the on-disk layout in deliverable 3.
3. Confirm the three statuses added to the lifecycle (`approved`, `blocked`, `cancelled`) and the new `needs-human` task type. They make section 15's guards expressible.

## Conventions

Ids use the brief's prefixes (T-, F, C, AC, Q, D, A) plus R for review flags. Code blocks are schemas or examples, not implementation.
