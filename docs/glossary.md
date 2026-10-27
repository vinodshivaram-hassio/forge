# Glossary

Terms are used exactly as defined. The first eight come from brief section 4; the rest are added by this design.

## From the brief

| Term | Definition |
| --- | --- |
| Task | The unit of work and the only message between agents. Has an id, a type, a status, a parent, an owner stage and a result. Stored in the Task Store. |
| Root task | The task created from the user's original input. Every lineage ends here. |
| Task Store | The backend that holds task records and lists open ones. File-based first; switchable. |
| Context file | A strictly formatted markdown file attached to a task, holding the facts, constraints, acceptance criteria and clarifications. |
| Agent | A configured worker: instructions, allowed task types, model tier, input and output schema. Stateless between tasks. |
| Gate | A named evaluation with explicit pass criteria, owned by one review agent. |
| Trace | The append-only record of every step taken on a task. |
| Lineage | The chain of parent links from any task back to the root. Always unbroken. |

## Added by this design

| Term | Definition |
| --- | --- |
| Result | The structured output an agent returns for one task, validated against its output schema. The orchestrator applies it; the agent never writes the store. |
| Transition | A recorded change of one task's status, made only by the orchestrator, preceded by a trace entry. |
| Claim | A time-limited lock a worker holds on a task while it runs. |
| Review round | One run of the applicable gates on one task version. Counted per task. |
| Change record | A file listing the files touched by an implement task, with the task id and commit reference. |
| Escalation | A `needs-human` task created when a guard fires or a question only the user can answer. |

## Task types

intake, clarify, review, decompose, test-spec, implement, verify, present, fetch (from the brief), and needs-human (added: a task owned by the user that holds an escalation question).
