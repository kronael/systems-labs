> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../specs/01-systems-labs.md) — shared scaffold,
  portability, cost, IaC, platform, and evidence contracts.
- [`../../docs/lab-selection.md`](../../../../docs/lab-selection.md) — selection rationale.
- [Execution environment lifecycle](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html)
  — Lambda freezes the environment between invocations and recycles it within
  hours, so unfinished background work and buffered state survive only if the
  same environment happens to thaw, and `/tmp` outlives a freeze but not a
  replacement.
- [Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
  — an invocation runs at most 900 seconds, `/tmp` offers 512 MB to 10 GB,
  and concurrent executions default to 1,000 per account per Region.
- Implementation pointers do not exist while the spec is `draft`.
