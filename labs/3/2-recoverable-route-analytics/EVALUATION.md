> Spoilers. Open only when stuck.

# Recoverable route analytics — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule kills a task immediately after a named observation's
PostgreSQL effect is acknowledged and before the checkpoint that covers it
completes, and again immediately after that checkpoint completes; removes the
newest saved state; kills a worker immediately after a named observation is
acknowledged, while the rebalance that kill triggers is still running;
introduces old and new event versions at named observations; delays events
across watermarks at a prefix whose stability duration the report has already
published; and supplies a history with an insufficient retention prefix.

Checks observe APIs, Kafka positions, Flink checkpoints and metrics,
PostgreSQL state, attempt histories, activation behavior, and reconstruction
checksums. They do not require a named connector or sink pattern.
