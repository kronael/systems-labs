> Spoilers. Open only when stuck.

# What the neighbours do differently

- **ActiveMQ** — as a JMS broker, can join an XA transaction with the
  database, so one coordinator prepares the commit on both sides — and
  inherits the coordinator's failure mode: an in-doubt transaction that holds
  its locks until recovery resolves it.
- **Temporal** — makes the cross-system progress itself durable state owned
  by a workflow engine, so what survives a crash between the two effects
  becomes that engine's contract rather than the application's design.
- **CockroachDB** — holding ledger and audit in one store, removes the
  boundary entirely: both commit in a single transactional domain, at the
  price of the audit product sharing the ledger's failure and load domain.
