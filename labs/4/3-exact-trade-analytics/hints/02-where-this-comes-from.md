> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../specs/01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/lab-selection.md`](../../../../docs/lab-selection.md) — selection rationale,
  where ClickHouse was recorded as the strongest first addition.
- [`ReplacingMergeTree`](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/replacingmergetree)
  — deduplication happens only during a merge, merging runs in the background
  at an unknown time, and the engine offers eventual correctness rather than
  a guarantee that duplicates are absent.
- [Mutations](https://clickhouse.com/docs/en/sql-reference/statements/alter)
  — a mutation is an asynchronous background process, and the statement
  returns as soon as the entry is recorded rather than when the change
  applies.
- [MergeTree settings](https://clickhouse.com/docs/en/operations/settings/merge-tree-settings)
  — `parts_to_throw_insert` defaults to 3000, which is the ceiling small
  frequent inserts reach when they outrun merging.
- Implementation pointers do not exist while the spec is `draft`.
