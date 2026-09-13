> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Google Cloud Run** — serves the same scale-to-zero request shape from a
  container, and its CPU-allocation setting is this lab's boundary made
  configurable: request-based billing throttles the CPU once the response is
  sent, instance-based billing keeps it computing — for a price.
- **AWS Step Functions** — is the vendor's own answer to work larger than one
  invocation: a Standard workflow records each step durably and runs for up
  to a year, so resumption stops being the function's problem and becomes a
  second orchestration surface to own.
- **Temporal** — generalizes that answer: it records every effect in an
  event history and replays it after a crash, so code appears to run for
  months across process deaths — at the cost of operating a second stateful
  platform.
