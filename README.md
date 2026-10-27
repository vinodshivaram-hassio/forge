# Forge

Forge is a backend orchestrator that turns a vague request into reviewed, tested, traceable code changes by moving work through a loop of small, single-purpose AI agents that never talk to each other and only communicate through tasks.

**Phase: design.** This repository holds the design specification, not an implementation. Everything here is **proposed** until marked decided in `docs/decisions/`.

Living design doc: https://claude.ai/artifact/R5pXQ98Lp9bc3KHDzh775H (as of 2026-10-27)

## Layout

```text
docs/
  design-spec-v1.md          overview, status, decisions needed
  glossary.md                brief section 4 terms plus additions
  decisions/                 DR-001 to DR-006
  specs/
    context-file-v1.md       header, sections, item grammar, validation, lint
    task-record-and-store.md task record, status model, TaskStore, file layout
    agents-and-gates.md      agent schema, result envelope, roster, routing, gates
  walkthroughs/
    A-clean-task.md          13 tasks, no gate failure
    B-gate-failure.md        3 review rounds, guard branch
  principles-and-open-questions.md
config/
  forge.yaml                 store path, dial, loop limits
  routing.yaml               tier to model class
  agents/                    one definition per agent (15)
  gates/                     one definition per gate (7)
lint/
  vague-terms.txt            lint list v1
examples/
  context/                   context files from the walkthroughs
  tasks.yaml                 task list snapshot from walkthrough A
```

## Not written yet

These are referenced by the configs and are the next design steps:

- `schemas/*.json`: JSON Schemas for the task record, context header, agent definition, gate definition and result envelope.
- `agents/*.md`: instruction files for each LLM agent.
- `gates/<gate>/good/*.md` and `gates/<gate>/bad/*.md`: known-good and known-bad example tasks per gate.
- The context file validator prototype.
