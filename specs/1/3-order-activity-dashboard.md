---
status: draft
---

# Order activity dashboard — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule repeats named events, uses keys that challenge the stated
ordering model, kills processing immediately before the durable effect of a
named event and again immediately after it, triggers a rebalance at a named
event, skews traffic toward one customer, introduces a compatible schema
version at a named event, and shortens available history in a rebuild
fixture. One schedule
serves one study: every step presses the declared ordering scope where it is
hardest, and the duplicate, crash, rebalance, and rebuild checks assert the
supporting constraints along the way.

No internal class or consumer pattern is required. Evaluation uses public
ingestion and query APIs, Kafka-visible records and offsets, PostgreSQL-visible
state, traces, resource bounds, and rebuild evidence.
