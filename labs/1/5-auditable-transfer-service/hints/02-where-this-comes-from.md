> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`docs/contract.md`](../../../../docs/contract.md) — shared scaffold,
  transaction, Kafka, evidence, and retry contracts.
- [`docs/lab-selection.md`](../../../../docs/lab-selection.md) — selection rationale.
- [`docs/serverless-contrast-track.md`](../../../../docs/serverless-contrast-track.md) —
  the serverless recast of this product and what it removes.
- Quirk origination: [Gray and Lamport, *Consensus on Transaction
  Commit*](https://lamport.azurewebsites.net/video/consensus-on-transaction-commit.pdf),
  the [Kafka design documentation on delivery
  semantics](https://kafka.apache.org/43/design/design/), and
  [PostgreSQL `PREPARE
  TRANSACTION`](https://www.postgresql.org/docs/current/sql-prepare-transaction.html).
  The falsified belief is that a database commit and a broker publish can be
  made one atomic step. The paper fixes that classic two-phase commit blocks
  when its coordinator fails; the Kafka documentation fixes that its
  transactions cover reading, processing, and writing across Kafka topics
  only, and do not extend to a system outside Kafka; the PostgreSQL page
  reserves prepared transactions for an external transaction manager and
  warns that leaving one open holds its locks and blocks vacuum.
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
