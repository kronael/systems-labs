> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`docs/contract.md`](../../../../docs/contract.md) — shared scaffold, the
  Lambda execution-shape policy, cost, grading, and evidence contracts.
- [`docs/serverless-contrast-track.md`](../../../../docs/serverless-contrast-track.md) —
  why this lab lands after the recasts and has no phase 1 partner.
- [`2-reliable-record-import.md`](../../../../labs/2/2-reliable-record-import/README.md) — the
  neighbouring queue-driven lab, whose subject is the queue's delivery contract
  rather than the execution model.
- [Execution environment lifecycle](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html)
  — the environment freezes when the runtime and each extension have completed
  and there are no pending events, which can be after the handler has
  returned, and thaws on the next invocation, so unfinished work resumes
  under another caller or never, and process state stays visible to whoever
  arrives next.
- [Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
  — the 900-second ceiling, the payload limits, and the default account
  concurrency.
- [Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
  — one in-flight request per environment, and throttling once concurrency
  is exhausted.
- Implementation pointers do not exist while the spec is `draft`.
