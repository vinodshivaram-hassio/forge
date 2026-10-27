# Principle check, deferred questions, next steps

Status: Proposed · 2026-10-27

No proposal in this design breaks a principle in brief section 2; four places extend the brief and are listed so each can be rejected on its own.

## Principle check

| Principle | Where the design enforces it |
| --- | --- |
| 1 Tasks are the only channel | Agents return a result envelope; only the orchestrator writes the store and creates tasks (`new_tasks` are requests) |
| 2 One agent, one job | Web lookup moved from clarifier to researcher |
| 3 No ambiguity downstream | Validator `clean` verdict is a precondition for ready-for-review; DR-001 re-checks children |
| 4 Test first | Build order eligibility rules plus tool scopes (`write_tests` vs `write_code`) |
| 5 Traceable to the root | `create` rejects orphans; `serves` on every child AC; parent-level verify checks coverage |
| 6 Everything documented | `append_trace` before any status change; raw results kept by seq |
| 7 Loops return to step 1 | Every failure transition targets open or blocked; no stage calls another |
| 8 Strict formats | Item grammar per section; YAML with schema; JSON Lines trace; validator never auto-fixes |
| 9 Pluggable by interface | TaskStore contract, `runner: llm \| script`, routing table as config |
| 10 Human decides at the end | awaiting-human to done only on user approval; no merge operation exists |

## Extensions to the brief, awaiting decision

- Statuses `approved` and `cancelled` (plus `blocked`, which the brief mentions but does not list).
- Task type `needs-human`.
- `append_trace` in the TaskStore interface.
- Clarifier may add and withdraw Constraints, Acceptance Criteria and Out of Scope items (found by walkthrough B).

## Deferred questions 5 to 10

These do not change any schema, so they wait until execution design, per brief section 17.

| # | Question | Default carried forward | Decide before |
| --- | --- | --- | --- |
| 5 | Target environments | Python, pytest, one repo layout | Writing the verifier |
| 6 | How agents run | Behind an `AgentRunner` interface; spike direct API vs agent SDK vs headless CLI | First agent prototype |
| 7 | Sequential or parallel workers | Sequential; claims already atomic with expiry | Throughput becomes a problem |
| 8 | Gates in parallel | Parallel, all blocking gates must pass, no voting (already assumed in the merge rule) | Orchestrator design |
| 9 | Artifact storage | Content-addressed under `artifacts/sha256/` (already in the layout) | First multi-modal task |
| 10 | Task selection order | Priority, then children of in-flight work, then oldest | Orchestrator design |

## Next steps

- [ ] Decide DR-001 to DR-004 and the extensions above
- [ ] Write the JSON Schemas: task record, context header, agent definition, gate definition, result envelope
- [ ] Build the context file validator as a prototype to test the grammar against both walkthroughs
- [ ] Write two known-good and two known-bad example tasks per gate
- [ ] Run the design gate: both walkthroughs replayed against the final schemas
