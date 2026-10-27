# Task record and TaskStore

Status: Proposed · 2026-10-27

The task record holds everything the orchestrator needs to pick and route a task; the context file holds everything an agent needs to do it. Status lives only in the record, and every status change must point at a trace entry already written.

## Task record

```yaml
- id: T-0007
  type: implement
  title: Add retry with backoff to the upload client
  parent_id: T-0001
  root_id: T-0001
  children: [T-0011, T-0012, T-0013]
  status: approved            # see status model
  stage: build                # next stage to run: load, clarify, review, decompose, build, present
  priority: 2
  complexity: 2
  tags: [backend]
  owner: implementer-backend  # agent name, or "user" for needs-human tasks
  claim: null                 # or {worker: w-1, expires_at: "2026-10-27T14:05:00Z"}
  counters: {clarify: 1, review_rounds: 1, attempts: 0}
  blocked: null               # or {reason: ..., owner: user, waiting_on: T-0015}
  context_ref: tasks/T-0007/context.md
  context_version: 4
  trace_ref: tasks/T-0007/trace.jsonl
  last_trace_seq: 9
  result_ref: null
  cost: {units: 2.4, tokens: 41210}
  created_at: "2026-10-27T13:40:02Z"
  updated_at: "2026-10-27T13:58:47Z"
  version: 12                 # record version, for optimistic concurrency
```

Record rules:

- `id`, `type`, `parent_id`, `root_id` and `created_at` never change.
- `children` only grows.
- `version` increments on every update.
- Task types gain one addition: `needs-human`, a task owned by the user that holds an escalation question.

## Status model

The brief's statuses stay; three are added. `approved` names the state between passing the panel and being decomposed or built. `blocked` makes section 15's guards expressible. `cancelled` lets the user stop work without deleting records.

```text
                 a child changed this task's facts
   +------------------------------------------------------+
   v                                                      |
 [ selectable at stage 1 ]   claim    +-------------+  split   +--------------+
 |  open                 | ---------> | in-progress | -------> | decomposed   |
 |  ready-for-review     | <--------- | one stage,  |          +--------------+
 |  approved             |   result   | one agent   |                | children done
 +-----------------------+            +-------------+                v
                                             | verified        +----------------+
                                             +---------------> | awaiting-human |
                                             |                 +----------------+
                                             v                       | user approves
                                          [ done ] <-----------------+

 Side states, reachable from any active status:
   blocked    entered when a guard fires or a human is needed; returns to prior status when waiting_on is done
   cancelled  entered only when the user cancels; descendants cancelled too; records kept
```

| Status | Meaning | Selectable at stage 1 |
| --- | --- | --- |
| open | Waiting for its next stage (load or clarify) | Yes |
| in-progress | Claimed; one stage running | No |
| ready-for-review | Clean context file; waiting for the panel | Yes |
| approved | Every applicable gate passed; waiting for decompose or build | Yes, if its build preconditions hold |
| decomposed | Children exist and are being worked | No |
| awaiting-human | Review package or escalation is with the user | No |
| blocked | Waiting on a named task or owner, with a reason | No |
| done | Finished; kept forever for lineage | No |
| cancelled | Stopped by the user; kept forever | No |

### Transitions

| From | To | Trigger |
| --- | --- | --- |
| open, ready-for-review, approved | in-progress | Orchestrator claims it at stage 1 |
| in-progress | open | Clarification applied, a gate failed, or an attempt failed under its limit |
| in-progress | ready-for-review | Clarifier verdict accurate and validator verdict clean (stage 4) |
| in-progress | approved | Every applicable gate returned pass |
| in-progress | decomposed | Decomposer result applied; children created |
| in-progress | done | Leaf: green verify child passed. Fetch, test-spec, verify: result applied |
| decomposed | open | A child's clarification changed this task's facts; siblings go to blocked |
| decomposed | awaiting-human | Root only: all children done, parent-level verify passed, present task done |
| any active | blocked | A guard fired (brief section 15) or a needs-human task was raised |
| blocked | prior status | The task named in `waiting_on` is done |
| awaiting-human | done | User approves |
| any active | cancelled | User cancels; descendants are cancelled with it |

A request for changes is not a transition on the root: it creates a new task whose parent is the present task, as the brief requires.

## Build order for an implement task

The orchestrator enforces test-first with eligibility rules a script can check, not with agent instructions.

1. When an implement task becomes approved, the orchestrator creates child `test-spec`.
2. When test-spec is done, it creates child `verify` with `mode: red`.
3. The implement task is selectable for `stage: build` only if a red verify child is done with verdict `red-confirmed`.
4. When the implementer result is applied, the orchestrator creates child `verify` with `mode: green`.
5. The implement task moves to done only when the green verify child is done with verdict `pass`.

## TaskStore interface

| Operation | Contract | Fails with |
| --- | --- | --- |
| list(filter) | Records matching status, type, priority, root or owner; never returns content of context files | none |
| get(id) | One record plus its context_ref and trace_ref | not-found |
| create(record) | Assigns the next id; rejects a missing or unknown parent unless type is root intake | orphan, schema |
| update(id, patch, expected_version) | Applies the patch if version matches; a status change must carry `trace_seq` equal to the latest trace entry | version-conflict, no-trace, illegal-transition, schema |
| claim(id, worker, ttl) | Atomic; sets claim and status in-progress | already-claimed, not-selectable |
| release(id, worker) | Clears claim; status returns to the one in the closing trace entry | not-owner |
| expire_claims(now) | Releases claims past expires_at and writes a trace entry for each | none |
| append_trace(id, entry) | Appends one entry; returns its seq | schema |
| children(id), lineage(id) | Walk down, or up to the root | not-found, broken-lineage |

The store has no delete operation. `append_trace` is the addition to the brief's interface: making it a store operation lets `update` refuse a status change that has no trace behind it.

## Trace entry

One JSON object per line in `trace.jsonl`:

| Field | Content |
| --- | --- |
| seq | Integer, per task, starting at 1 |
| timestamp | ISO 8601 UTC |
| stage, task_id | What ran and on what |
| agent, agent_version | Which configuration produced the result |
| model, tier, routing | The model class, the tier, and the inputs that chose it (dial, complexity, attempt, alpha) |
| input_ref | sha256 of the context file the agent read |
| output_ref | Path of the raw result or the task produced |
| verdict, next_status | Outcome; next status or null when this entry does not move the task |
| reason | One or two sentences |
| cost | tokens and units |

## File-based layout

```text
<store.path>/
  tasks.yaml                 # all task records, canonical YAML (DR-003)
  .lock                      # single-writer lock for v1
  tasks/
    T-0001/
      context.md
      trace.jsonl            # one JSON object per line, append-only
      results/
        0003-clarifier.json  # raw agent result, named by trace seq
  artifacts/
    sha256/ab/abcd...ef.png  # content-addressed, referenced from Artifacts by hash
```

- Writes to `tasks.yaml` and `context.md`: write to a temp file in the same folder, fsync, rename.
- Trace files are JSON Lines, not YAML, because appending a line is atomic and needs no rewrite.
- Raw agent results are kept beside the trace so any stage can be replayed from its `input_ref`.
- Rewriting one `tasks.yaml` on every update is fine for v1. Above about 5,000 records, the layout should switch to one record file per task; the interface does not change.
