# Release Notes

## v1.0.1
GH#820 — migrate the 4 builtin steps (parse-csv, filter-rows, dedupe-customers, count-customers) to the node-less `local.builtin` shape (#802 D1b): `id`/`input`/`context`, no backing skill nodes/edges; chained builtins take `input: {{steps.<prev-id>.output}}` (were from_step bindings); downstream refs `{{steps.Count Customers/Dedupe Customers.output}}` → `{{steps.count-customers/dedupe-customers.output}}`. Deterministic output unchanged. contents skills 5→1.

## v1.0.0
Initial release. Aggregates a CSV deterministically with builtins — `csv-to-json` → `json-filter` (shipped) → `dedup-merge` (by customer) → `count` — then a model narrates the computed result. Demonstrates zero-token transforms: the arithmetic is exact and offline, the model only writes prose.
