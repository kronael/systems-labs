> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../specs/01-systems-labs.md) — shared scaffold, Flink,
  Kafka, evidence, and failure contracts.
- [`../../docs/lab-selection.md`](../../../../docs/lab-selection.md) — selection rationale.
- Quirk origination: the Flink documentation on [checkpoints versus
  savepoints](https://nightlies.apache.org/flink/flink-docs-stable/docs/ops/state/checkpoints_vs_savepoints/),
  [fault-tolerance guarantees of sources and
  sinks](https://nightlies.apache.org/flink/flink-docs-stable/docs/connectors/datastream/guarantees/),
  and [window
  lateness](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/datastream/operators/windows/).
  The falsified belief is that a checkpoint is a backup and an exactly-once
  switch makes results exactly-once everywhere. The pages fix that checkpoints
  are Flink-owned recovery state, dropped by default when a job terminates;
  that the exactly-once guarantee covers state inside Flink, while what an
  external system observes across recovery depends on that system's own
  coordination with the checkpoint; and that an element arriving after the
  watermark has passed its window is dropped by default. Solution-bearing:
  this belongs in hints/, never in README.md.
- Domain grounding, judged neutral — it reports the numbers a real churn
  report publishes and touches no recovery design: [Geoff Huston, *BGP
  updates in 2025*, APNIC Blog, 9 January
  2026](https://blog.apnic.net/2026/01/09/bgp-updates-in-2025/), measured
  from a single vantage point (AS131072). Most update messages "come from a
  pool of between 30,000 to 80,000 prefixes" of the roughly 1.2 million
  advertised; the "daily average time for an unstable prefix to reach
  stability is now between 20 and 45 seconds"; "Less than 5% of the unstable
  prefixes caused half of all BGP updates during December 2025", and "Fifty
  origin Autonomous System Numbers (ASNs) accounted for one-third of all BGP
  IPv4 updates in this period". These measurements are the lab's three views
  with their published values. The article's explanation of the convergence
  timescale is the observation lab's hint territory and never publishes into
  this lab's learner-facing text; the figures ground the product.
- Implementation pointers do not exist while the spec is `draft`.
