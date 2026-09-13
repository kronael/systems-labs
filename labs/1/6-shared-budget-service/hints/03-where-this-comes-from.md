> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../docs/contract.md) — shared scaffold,
  PostgreSQL, evidence, and retry contracts.
- [PostgreSQL transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
  — the page carries both halves of the quirk. A transaction refused for a
  dependency between what it read and what a concurrent transaction wrote
  reports `ERROR: could not serialize access due to read/write dependencies
  among transactions`; its guidance on a serialization failure is that an
  application "should abort the current transaction and retry the whole
  transaction from the beginning", and that "applications must not depend on
  results read during a transaction that later aborted; instead, they should
  retry the transaction until it succeeds". It also fixes that a sequential
  scan "will always necessitate a relation-level predicate lock", which raises
  the rate of refusals. Solution-bearing: this belongs in `hints/`, never in
  `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
