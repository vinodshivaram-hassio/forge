# Agents and gates

Status: Proposed · 2026-10-27

Agents and gates are config files validated against two schemas; every agent returns the same result envelope, so the orchestrator applies all results with one code path. An agent definition may set `runner: llm` or `runner: script`; in v1 every agent, the verifier included, is a model agent (DR-006).

Definitions live in `config/agents/` and `config/gates/`.

## Agent definition schema

```yaml
name: clarifier                     # unique, kebab-case
version: 3                          # bump on any change to this file or its instructions
role: Verify that every fact in the context file is accurate and complete.
runner: llm                         # llm | script
instructions_file: agents/clarifier.md
accepts: [clarify]                  # task types
select: {tags_any: []}              # used to pick among specialists
produces: [fetch, needs-human]      # task types its result may request
may_edit_sections: [Facts, Constraints, Acceptance Criteria, Out of Scope, Open Questions, Clarifications, Review Flags]
input_schema: schemas/agent-input.json
output_schema: schemas/clarify-output.json
model: {tier_min: 3, tier_max: 8, floor: null}
tools: [read_file, search_repo]
limits: {max_attempts: 3, max_cost_units: 5, timeout_s: 600}
```

Changes from the brief's example:

- The clarifier drops `web_search`: looking things up on the web is the researcher's single job, so the clarifier raises a fetch task instead.
- The clarifier gains Review Flags (to mark a flag resolved once its fix is applied) and Constraints, Acceptance Criteria and Out of Scope (add and withdraw only), because walkthrough B showed no other role could fix a clarity flag on an AC.

## Result envelope

Every agent returns this shape; `output_schema` only narrows the allowed `verdict` values and `new_tasks` types.

```yaml
task_id: T-0007
agent: clarifier
agent_version: 3
input_ref: sha256:9f1c...           # hash of the context file it read
verdict: not-accurate               # values fixed per agent
reason: F3 is unconfirmed; repo has two upload clients.
context_proposed: |                 # full new context file, or null
  ...
new_tasks:                          # requests; the orchestrator creates them
  - {type: fetch, title: Which upload client is in scope?, ask: researcher, question_id: Q2}
files_written: []                   # repo-relative paths, for build agents
decisions: []                       # lines appended to Decisions
cost: {tokens: 8120, units: 0.6}
```

## Starting roster

| Agent | Runner | Accepts | Verdicts | Tiers (min to max) | Tools |
| --- | --- | --- | --- | --- | --- |
| intake | llm | intake | drafted, invalid-input | 2 to 5 | read_file |
| clarifier | llm | clarify | accurate, not-accurate | 3 to 8 | read_file, search_repo |
| researcher | llm | fetch | answered, not-found | 2 to 6 | web_search, read_file, search_repo |
| reviewer-clarity | llm | review | pass, fail, cannot-assess | 5 to 9 (floor 5) | read_file |
| reviewer-completeness | llm | review | pass, fail, cannot-assess | 5 to 9 (floor 5) | read_file |
| reviewer-testability | llm | review | pass, fail, cannot-assess | 5 to 9 (floor 5) | read_file |
| reviewer-size | llm | review | pass, fail, cannot-assess | 4 to 8 (floor 4) | read_file, search_repo |
| reviewer-architecture | llm | review | pass, fail, cannot-assess | 5 to 9 (floor 5) | read_file, search_repo |
| reviewer-security | llm | review | pass, fail, cannot-assess | 6 to 10 (floor 6) | read_file, search_repo |
| reviewer-cost | llm | review | pass, advisory | 2 to 5 | read_file |
| decomposer | llm | decompose | decomposed, cannot-split | 4 to 9 | read_file, search_repo |
| test-writer | llm | test-spec | written | 3 to 8 | read_file, search_repo, write_tests |
| implementer-backend | llm | implement, tags backend | implemented, stuck | 3 to 9 | read_file, search_repo, write_code, run_tests |
| verifier | llm | verify | red-confirmed, red-rejected, pass, fail | 4 to 8 (floor 4) | run_tests, diff, check_ac_coverage |
| presenter | llm | present | packaged | 2 to 5 | read_file |

Tool scopes are enforced by the runner: `write_tests` may write only under the configured test paths, and `write_code` may write anywhere except them. That makes "the implementer cannot edit tests" a permission, not an instruction.

**The verifier is a model agent** (DR-006), with a floor of tier 4 as brief section 13 requires. Its mechanical checks run as the `check_ac_coverage` tool: mapping tests to criteria by name (`test_AC1_...`), counting skips and comparing test-file hashes with the red check. The model reads those results and judges the rest, such as whether a test still checks what its criterion says.

## Routing

The brief's formula is kept with alpha = 0.5:

```text
w    = alpha * (dial - 1) / 9 + (1 - alpha) * (complexity - 1) / 4
tier = clamp(round(tier_min + w * (tier_max - tier_min)) + escalation, tier_min, tier_max)
```

Worked example for the clarifier at dial 3, complexity 2: w = 0.5 × 2/9 + 0.5 × 1/4 = 0.236; tier = round(3 + 0.236 × 5) = 4. A retry adds 1 (tier 5). A reviewer at the same inputs gets round(5 + 0.236 × 4) = 6.

- `floor` is applied after the clamp and wins over the dial, never over `model_override`.
- The routing table (`config/routing.yaml`) maps tiers 1 to 3, 4 to 7 and 8 to 10 onto three model classes; no agent file names a model.

## Gate definition schema

```yaml
gate: testability
version: 1
owner_agent: reviewer-testability
applies_to: [implement, decompose]   # task types
run_on: [full, child]                # full panel, DR-001 light check, or both
pass_criteria:
  - id: TG1
    text: Every acceptance criterion maps to at least one automatable test.
  - id: TG2
    text: No criterion depends on subjective judgement, unless marked [check: human].
cannot_assess_when: An AC references a system the reviewer cannot read.
fail_output: Cite each failing AC id and the criterion id it breaks; state what would make it pass.
severity: blocking                   # blocking | advisory
examples: {good: [gates/testability/good/*.md], bad: [gates/testability/bad/*.md]}
```

## Starting gates

| Gate | Pass criteria | Severity | Runs on |
| --- | --- | --- | --- |
| Clarity | CL1 No Goal, Fact, Constraint or AC admits two materially different implementations. CL2 Every quantity has a unit. CL3 Every term used in an AC is defined in Facts or is standard in the repo's language. | Blocking | full, child |
| Completeness | CO1 Every part of the Goal is covered by at least one AC. CO2 Every Constraint is referenced by an AC or a Decision. CO3 No AC contradicts a Constraint or Out of Scope. CO4 Out of Scope is not None when complexity is 3 or more. | Blocking | full |
| Testability | TG1 and TG2 above. | Blocking | full, child |
| Size | SZ1 A task to be built has at most 3 ACs and complexity at most 2. SZ2 A task to be decomposed can be split into at most 8 children. SZ3 Depth stays at most 4. | Blocking | full |
| Architecture fit | AF1 Facts name the modules or files to be changed. AF2 Any new dependency or module has a Decision with its reason. AF3 Nothing contradicts conventions in artifacts tagged `convention`. | Blocking | full |
| Security and data | SD1 Facts state whether secrets, personal data or permissions are touched. SD2 If any are, a Constraint and an AC cover their handling. SD3 No secret value appears in the context file or artifacts. | Blocking | full |
| Cost and risk | CR1 Estimated cost units fit within the remaining root ceiling. CR2 Blast radius (modules touched) is stated. | Advisory | full |

SZ1's limits (3 ACs, complexity 2) were confirmed on 2026-10-27.

## Merging verdicts

Each reviewer sees only its own gate file, the task and its context file. The orchestrator merges verdicts:

- Any blocking fail means the round fails.
- Any cannot-assess with no fail raises a clarify pass instead of a fail.
- Advisory notes travel with the task to the user.
