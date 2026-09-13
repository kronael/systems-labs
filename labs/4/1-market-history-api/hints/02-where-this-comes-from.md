> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`docs/contract.md`](../../../../docs/contract.md) — shared scaffold,
  DynamoDB, real-data, cost, and evidence contracts.
- [`docs/lab-selection.md`](../../../../docs/lab-selection.md) — selection rationale.
- [Partition key design](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-design.html)
  — each DynamoDB partition serves a fixed per-second budget of read and
  write units, so one hot key throttles while the table sits far below its
  capacity.
- [Paginating query results](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.Pagination.html)
  — a `Query` returns at most 1 MB per call, and only the absence of
  `LastEvaluatedKey` proves a result set is complete.
- [DynamoDB local usage notes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.UsageNotes.html)
  — "Provisioned throughput settings are ignored in downloadable DynamoDB",
  "the speed of read and write operations on table data is limited only by the
  speed of your computer", and "when you run DynamoDB locally, there is no
  table partitioning", so this lab's ceiling comes from the controller and
  never from the store.
- Implementation pointers do not exist while the spec is `draft`.
