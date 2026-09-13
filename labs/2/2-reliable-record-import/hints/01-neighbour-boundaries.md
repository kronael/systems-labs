> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Kafka** — replaces the per-message lease with a consumer-owned offset, so
  progress is a position the consumer moves rather than a clock the queue
  runs, and history survives acknowledgement.
- **RabbitMQ** — redelivers when a channel closes rather than when a timer
  expires, which ties recovery to connection lifetime instead of a visibility
  window.
- **AWS Step Functions** — moves retry, catch, and terminal-failure routing
  into a platform state machine, so the fate of an identity is orchestrated
  rather than designed.
