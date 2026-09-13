---
status: draft
---

# Reliable record import — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

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
