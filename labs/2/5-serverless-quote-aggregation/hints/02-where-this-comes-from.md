> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../specs/01-systems-labs.md) — shared scaffold, execution
  shape policy, cost, grading, and evidence contracts.
- [`../../docs/serverless-contrast-track.md`](../../../../docs/serverless-contrast-track.md) —
  why this pairing earns a lab and what the platform removes.
- [`../1/1-resilient-quote-service.md`](../../../../labs/1/1-resilient-quote-service/README.md) — the
  local lab this one recasts, and the source of the comparison baseline.
- [Duffel API reference — Offers](https://duffel.com/docs/api/offers) — the
  reported behaviour the product's expiry rests on: "An offer is only
  available to create an order for a limited time by the traveller before it
  expires, typically within 30 minutes", and past its stated expiry an offer
  "can no longer be used to create an order".
- [Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
  — one in-flight request per environment, and throttling once concurrency is
  exhausted, so admission is a platform setting rather than an application
  decision.
- [Execution environment lifecycle](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html)
  — initialization runs before the first request an environment serves, and
  the environment freezes when the runtime and each extension have completed
  and there are no pending events.
- [Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
  — the 900-second ceiling, the payload limits, and the default account
  concurrency.
- Implementation pointers do not exist while the spec is `draft`.
