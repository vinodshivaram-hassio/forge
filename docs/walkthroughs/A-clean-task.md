# Walkthrough A: clean task

Status: Proposed · 2026-10-27

The root "make uploads reliable" reaches the user's review in 13 tasks and 27 trace entries with no gate failure, at dial 3. It exercises every stage, one fetch, a user confirmation and the test-first build order.

## Root context after clarification

T-0001, complexity 3 (full file: `examples/context/T-0001-upload-retry.md`):

- F1 uploads use one HTTP PUT with no retry (src: client.py).
- F2 transient means HTTP 502, 503, 504 or a connection timeout (src: storage API docs, T-0003).
- C1 at most 5 attempts, delay doubles from 1 second, capped at 30 seconds.
- AC1 503 twice then 200 succeeds on attempt 3.
- AC2 a 400 fails at once with no retry.
- AC3 503 on all 5 attempts raises `UploadFailed` naming the last status.

## Trace

| Seq | Stage | Task | Agent, tier | Verdict | Status after | Why |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | create | T-0001 intake | user | created | open | Free text received (DR-004) |
| 2 | load | T-0001 | intake, 2 | drafted | blocked | Draft context written; T-0002 needs-human created to confirm it |
| 3 | human | T-0002 | user | confirmed | done; T-0001 open | User accepted the draft unchanged |
| 4 | clarify | T-0001 | clarifier, 5 | not-accurate | blocked | F1 confirmed from client.py; Q1 "which statuses are transient" needs a lookup; T-0003 fetch created |
| 5 | fetch | T-0003 | researcher, 3 | answered | done; T-0001 open | Storage API docs list 502, 503, 504 and timeouts as retryable |
| 6 | clarify | T-0001 | clarifier, 5 | accurate | ready-for-review | Answer applied as F2 and a Clarifications entry; validator verdict clean |
| 7 to 13 | review | T-0001 | 7 reviewers, 3 to 7 | 6 pass, 1 advisory | approved | All blocking gates pass. Cost note: about 9 units of a 40-unit ceiling |
| 14 | decompose | T-0001 | decomposer, 6 | decomposed | decomposed | T-0004 (retry with backoff, serves AC1, AC2) and T-0005 (final failure, serves AC3). D1: delay goes through an injected sleep so tests do not wait |
| 15 | load, clarify | T-0004 | intake 2, clarifier 4 | accurate | ready-for-review | Facts inherited confirmed; complexity 2 |
| 16 | review (light) | T-0004 | clarity 6, testability 6 | pass, pass | approved | DR-001 light check; T-0006 test-spec created |
| 17 | build | T-0006 | test-writer, 4 | written | done | `test_AC1_succeeds_on_third_attempt`, `test_AC2_no_retry_on_400`; T-0007 verify red created |
| 18 | verify | T-0007 | verifier, 5 | red-confirmed | done | Both fail for the expected reason: first 503 raises at once |
| 19 | build | T-0004 | implementer-backend, 4 | implemented | in-progress until verify | client.py changed; T-0008 verify green created |
| 20 | verify | T-0008 | verifier, 5 | pass | done; T-0004 done | New tests pass; suite 212 passed, 0 skipped; test-file hashes match the red check |
| 21 to 25 | same path | T-0005, T-0009 to T-0011 | as above | pass | done | Same five steps for AC3 |
| 26 | verify (parent) | T-0012 | verifier, 5 | pass | done | Every root AC has a passing test via a child's `serves` link |
| 27 | present | T-0013 | presenter, 3 | packaged | done; T-0001 awaiting-human | Package: diff by child, red then green evidence, 1 advisory, cost 8.7 units |

The user approves, and T-0001 moves to done with a trace entry naming the user as the agent.

## One trace line on disk

`tasks/T-0001/trace.jsonl`, entry 9 (the testability reviewer):

```json
{"seq": 9, "timestamp": "2026-10-27T14:02:11Z", "stage": "review", "task_id": "T-0001", "agent": "reviewer-testability", "agent_version": 1, "model": "mid", "tier": 6, "routing": {"dial": "3/10 (1 = cheapest)", "complexity": 3, "attempt": 1, "alpha": 0.5}, "input_ref": "sha256:4be0...", "output_ref": "tasks/T-0001/results/0009-reviewer-testability.json", "verdict": "pass", "next_status": null, "reason": "AC1 to AC3 each map to an automatable test; none needs judgement.", "cost": {"tokens": 5230, "units": 0.4}}
```

`next_status` is null because a single gate verdict does not move the task; the orchestrator writes the merge entry (seq 13) with `next_status: approved`.

Seq numbers in the table are counted across the whole root for readability; on disk each task's trace file numbers its own entries. Cost and token figures are illustrative, not estimates.
