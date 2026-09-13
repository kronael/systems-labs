> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Kafka Streams** — keeps its processing state in changelog topics on the
  broker it already reads from, so restoring saved state and reprocessing
  history are one mechanism rather than two paths to reconcile.
- **Spark Structured Streaming** — binds a query to its checkpoint location
  and permits only limited query changes across restarts, so an upgrade is a
  rebuild by default rather than a compatibility decision.
- **Materialize** — keeps the derived views inside the same system that
  computes them, so there is no external query store to keep consistent
  across recovery — and no independent second recovery path to compare
  against.
