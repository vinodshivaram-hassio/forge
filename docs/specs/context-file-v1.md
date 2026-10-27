# Context file spec v1

Status: Proposed · 2026-10-27

A context file is **valid** when a deterministic script can parse its header, its 11 sections in order and every item line against the grammar below. It is **clean** when, in addition, nothing is unconfirmed, open or vague. File name: `context.md` in the task's folder.

## Header

| Key | Type | Required | Rule |
| --- | --- | --- | --- |
| forge_format | integer | yes | Equals 1 for this spec |
| task_id | string | yes | Pattern `T-[0-9]{4,}`; equals the task record id |
| parent_id | string or `none` | yes | `none` only when task_id equals root_id |
| root_id | string | yes | Pattern `T-[0-9]{4,}` |
| type | enum | yes | intake, clarify, review, decompose, test-spec, implement, verify, present, fetch, needs-human, plus configured types |
| title | string | yes | One line, 1 to 99 characters |
| version | integer | yes | Starts at 1; +1 on every write of this file |
| priority | integer | yes | 1 to 5, 1 is most urgent |
| complexity | integer | yes | 1 to 5; set by intake; 0 means not yet scored |
| modalities | list | yes | Subset of text, image, audio, document, data; at least one |
| tags | list | no | Free labels used to select specialised agents (for example backend, data) |
| model_override | map | no | Agent name to tier 1 to 10; beats the dial |

**Status is not a header key** (DR-005). The brief put status in both the header and the task record. Two files cannot be written atomically together, so the task record owns status. Agents never act on status; if one needs it, the orchestrator passes the task record with the file.

## Sections and item grammar

Sections are H2 headings with these exact names, in this order. An empty section holds the single line `None`. Each item is one markdown bullet on one line.

| # | Section | Id | Item grammar | Owner (may edit) | Edit mode |
| --- | --- | --- | --- | --- | --- |
| 1 | Original Input | none | Free text, verbatim | Intake, on root only | Immutable after first write |
| 2 | Goal | none | One sentence, ends with a full stop | Intake | Replace |
| 3 | Facts | F | `- F1 [confirmed\|unconfirmed\|withdrawn, src: <source>] <text>` | Clarifier | Edit status and text; never delete |
| 4 | Constraints | C | `- C1 <text>` or `- C1 [withdrawn] <text>` | Intake, Clarifier | Add, withdraw |
| 5 | Acceptance Criteria | AC | `- AC1 [serves: T-0001/AC2] Given <x>, when <y>, then <z>.` | Intake (root draft), Clarifier (add or withdraw after a clarification or flag), Decomposer (children) | Add, withdraw |
| 6 | Out of Scope | none | `- <text>` | Intake, Clarifier | Add |
| 7 | Artifacts | A | `- A1 path: <rel path>, media: <mime>, modality: <m>, sha256: <hex64>` | Intake | Add, withdraw |
| 8 | Open Questions | Q | `- Q1 [open\|answered\|withdrawn, ask: user\|researcher] <text>` | Clarifier | Add, change state |
| 9 | Clarifications | Q ref | `- Q1 (answered) <question> A: <answer>. Source: <src>, <task id>.` | Clarifier | Append only |
| 10 | Decisions | D | `- D1 <choice>. Reason: <reason>.` | Any role, appended from its result | Append only |
| 11 | Review Flags | R | `- R1 [gate: <gate>, round: <n>, open\|resolved] Cites: <ids>. <reason>. Pass when: <condition>.` | Review agents (add), Clarifier (resolve) | Add, change state |

- Source values: `user`, a repo-relative path, a URL, or a task id.
- `serves` is required on every AC of a non-root task and forbidden on the root.
- An AC that cannot be automated carries `[check: human]` after `serves`, which the testability gate must have accepted.
- "Owner" means the role whose result may change that section (see "Who writes the file").

## Validation rules

The validator returns one of three verdicts: `invalid` (any error), `valid` (no errors, not clean), `clean` (valid and every clean rule holds).

| Code | Rule | Level |
| --- | --- | --- |
| E-HEADER | Header missing, unparsable, unknown key, or a key breaks its type rule | Error |
| E-SECTION | Section missing, renamed, duplicated or out of order | Error |
| E-ITEM | An item line does not match its section grammar | Error |
| E-ID-DUP | An id appears twice in one file | Error |
| E-ID-REUSE | An id that existed in the previous version now names a different item, or an id disappeared (must be withdrawn instead) | Error |
| E-ID-GAP | A new id is not the next integer after the highest ever used | Error |
| E-APPEND | Original Input, Clarifications or Decisions changed other than by appending | Error |
| E-OWNER | The diff touches a section the writing role may not edit | Error |
| E-LINEAGE | parent_id or root_id does not match the task record, or `serves` points at a missing parent AC | Error |
| E-ARTIFACT | An artifact file is missing or its sha256 differs | Error |
| C-FACT | A fact is unconfirmed | Not clean |
| C-QUESTION | An open question exists | Not clean |
| C-GWT | An AC does not parse as Given, when, then | Not clean |
| C-LINT | A lint term appears in Goal, Facts, Constraints or AC without a measurable value | Not clean |
| C-FLAG | A review flag is open | Not clean |

## Lint list v1

A term is flagged unless a number with a unit, a named standard or an explicit enumeration follows within the same clause. The list lives in `lint/vague-terms.txt`, so adding a term is a config change.

## Validator behaviour

- Two entry points: `validate(file)` for a single version, and `validate_change(old, new, role)` which adds the E-ID-REUSE, E-ID-GAP, E-APPEND and E-OWNER diff rules.
- Output is JSON: verdict, a list of findings each with code, section, item id and line number, and the sha256 of the file read (used as the trace `input_ref`).
- The orchestrator calls `validate_change` on every agent result before applying it. An error rejects the result and counts as a failed attempt (brief section 15, invalid agent output).
- The validator never fixes a file. Auto-formatting would hide what the agent actually wrote.

## Who writes the file

Agents never write it directly. An agent returns a result containing a proposed new context file; the orchestrator runs `validate_change(old, new, role)` and, if it passes, writes it atomically.

## Example

See `examples/context/T-0001-upload-retry.md` and `examples/context/T-0101-invoice-export.md`.
