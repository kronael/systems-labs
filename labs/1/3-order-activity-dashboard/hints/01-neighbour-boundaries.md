> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Apache Flink** — owns state, progress, and restart through checkpoints,
  so the coordination between durable effect and consumer progress becomes
  the framework's contract instead of the learner's design.
- **Kafka Streams** — materializes views from the same log with managed state
  stores and its own rebalance behavior, trading the external query store for
  state that lives inside the consumer.
- **RabbitMQ** — deletes what it has acknowledged, so the rebuild this lab
  requires is impossible by construction: history exists only while a queue
  still holds it.
