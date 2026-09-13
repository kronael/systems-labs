---
status: draft
---

# Exact trade analytics — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

The failure schedule replays a trade stream containing exact duplicates,
out-of-order arrivals, and corrections at named identities, then queries
during ingest, immediately after a burst, and after the store has been idle.
It drives insert frequency into the regime where the store rejects work,
restarts the ingestion service at a named trade inside a batch, and restarts
the store once a named correction has been accepted.

Checks do not inspect private functions or require a named table engine. They
compare every aggregate against an independently computed exact answer
derived from the accepted input.
