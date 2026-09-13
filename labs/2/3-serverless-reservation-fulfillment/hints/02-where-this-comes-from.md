> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`docs/contract.md`](../../../../docs/contract.md) — shared scaffold, execution
  shape policy, cost, grading, and evidence contracts.
- [`docs/serverless-contrast-track.md`](../../../../docs/serverless-contrast-track.md) —
  why this pairing earns a lab and what the platform removes.
- [`labs/1/2-reservation-fulfillment`](../../../../labs/1/2-reservation-fulfillment/README.md) — the
  local lab this one recasts.
- [Execution environment lifecycle](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html)
  — the environment freezes when the runtime and each extension have completed
  and there are no pending events, and thaws on the next invocation, so
  unfinished work resumes under another caller or never.
- [Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
  — one in-flight request per environment, and throttling once concurrency is
  exhausted.
- [DynamoDB transactions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transaction-apis.html)
  — the store's transaction and condition contract, and the limits that bound
  both. Solution-bearing: this belongs in `hints/`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
