> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Amazon DynamoDB Streams** — a change-data stream from the store turns
  publication into someone else's problem, at the cost of a delivery order
  that no longer matches the write's atomicity, which is the same trade this
  lab makes explicit.
- **AWS Step Functions** — a workflow service makes the two-step effect a
  durable execution with its own retry and compensation semantics, moving the
  commit gap into a service contract rather than removing it.
- **PostgreSQL with a self-run Kafka** — a relational ledger with a self-run
  broker, which is the local lab, and is the comparison that makes this
  design's residual window legible.
