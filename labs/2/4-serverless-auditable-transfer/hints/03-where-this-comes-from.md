> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`docs/contract.md`](../../../../docs/contract.md) — shared scaffold, execution
  shape policy, cost, grading, and evidence contracts.
- [`docs/serverless-contrast-track.md`](../../../../docs/serverless-contrast-track.md) —
  why this pairing earns a lab and what the platform removes.
- [`labs/1/5-auditable-transfer-service`](../../../../labs/1/5-auditable-transfer-service/README.md)
  — the local lab this one recasts.
- [Execution environment lifecycle](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html)
  — the environment freezes when the runtime and each extension have completed
  and there are no pending events, so work still unfinished at that point may
  never run.
- [DynamoDB transactions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transaction-apis.html)
  — an atomic write's changes propagate gradually to indexes and streams, so
  records from one transaction may appear at different times and interleave
  with records from others. Solution-bearing: this belongs in `hints/`,
  never in `README.md`.
- [Queue event source mapping](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)
  and [batch failure reporting](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-errorhandling.html)
  — batches arrive at least once, and the outcome of a batch containing a
  failed item is governed by a documented reporting contract.
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
