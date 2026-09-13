---
status: draft
---

# Low-latency market API — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule aligns the expirations of named keys, evicts a named
popular entry before its expiry, drives a burst on one named key through two
replicas, kills one instance at the moment a named key's fill is granted,
slows DynamoDB at a named request, makes Valkey time out at a named request,
restarts Valkey empty once a named key has been served, and changes the source
generation at a named record.

Checks observe public responses, freshness metadata, dependency requests,
Valkey state and memory, traces, metrics, and exact comparison with the
durable dataset. They do not require a named cache-aside, lock, or
stale-while-revalidate pattern.
