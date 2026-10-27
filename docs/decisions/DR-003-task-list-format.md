# DR-003: Task list file format

Status: Proposed · 2026-10-27 · Brief open question 3

## Question

What format is the task list file: YAML, JSON or markdown table?

## Choice

YAML, validated against a JSON Schema (`schemas/task-record.schema.json`) on every read and write. The writer emits a canonical form: fixed key order, block style, no anchors or aliases, no flow collections except short id lists.

## Safety rules

- Values that look like booleans, dates or numbers but are strings (`no`, `2026-10-27` as a label) are quoted by the writer.
- The loader uses a safe loader and rejects unknown keys.
- Trace files are JSON Lines, not YAML, because appending a line is atomic and needs no rewrite.

## Reasons

- Hand-editable and diff-friendly, and the context file header is already YAML.
- Canonical output and a schema remove YAML's ambiguity risks (implicit types, anchors).

## Rejected alternatives

- JSON: safe, but painful to hand-edit and has no comments.
- Markdown table: cannot hold nested fields and is fragile to parse.
