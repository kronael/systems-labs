---
status: draft
---

# Internet route observatory — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

The failure schedule drops a named peer session after a named update is
acknowledged and restores it later, withdraws a prefix at one collector while
a second collector still announces it, replays a convergence burst at a named
prefix whose transient paths must not surface as routing changes, sends
malformed paths, kills processing at a named Kafka offset mid-stream, and
disconnects the optional recorder. Every fault fires at a named barrier, never
on a timer and never at random.

Checks observe source and API contracts, Kafka-visible identities, queryable
state, scope statements, traces, freshness, lag, resource bounds, and exact
accepted histories. They do not require a particular database schema or
streaming framework.
