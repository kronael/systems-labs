> Spoilers. Open only when stuck.

# What the neighbours do differently

- **RabbitMQ** — redelivers on consumer liveness rather than on a clock:
  unacknowledged work returns when the channel or connection drops, so a
  merely slow worker is not handed a duplicate, and its dead-letter exchange
  moves the message for you instead of signalling exhaustion.
- **Kafka** — gives the consumer a position in a retained log rather than a
  per-unit lease, so there is no redelivery timer at all, one bad unit blocks
  its partition until the position moves past it, and useful parallelism is
  capped by partition count.
- **Temporal** — owns retry, state transitions, and worker recovery as
  platform features, which moves the recovery rule out of the learner's
  design and into a workflow engine's event history.
