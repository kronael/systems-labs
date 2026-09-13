> Spoilers. Open only when stuck.

# Reliable record import — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule expires the lease of a named record while its work is
active, kills an invocation immediately before the durable effect of a named
record and again immediately after it, repeats named source identities, fails
one named item inside a mixed batch, injects temporary DynamoDB errors at
named records, submits a named persistent poison record, and reintroduces a
named terminal record after its fix.

Evaluation observes the SQS API, Lambda-shaped responses, DynamoDB-visible
results, public query behavior, process lifecycle, traces, metrics, and exact
delivery histories. It does not require a named idempotency pattern.
