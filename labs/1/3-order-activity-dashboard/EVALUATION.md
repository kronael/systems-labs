> Spoilers. Open only when stuck.

# Order activity dashboard — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

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
