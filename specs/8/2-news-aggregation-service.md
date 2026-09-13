---
status: draft
---

# News aggregation service — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

The failure schedule replays a seeded feed schedule through the harness and
fires every fault at a named barrier, never on a timer and never at random:

- after article `wire-4711` has been grouped and served, the harness
  republishes it edited in place under the same identifier;
- the same story arrives from a second feed under the different identifier
  `syn-0114`;
- feed `F-208` replays a window of items with publication timestamps moved
  two days earlier, beginning at item `wire-3090`;
- the harness kills the ingestion service after item `wire-5000` is
  acknowledged and restarts it mid-feed.

Verification queries during ingest, immediately after a burst, and after the
system has been idle, and it drives the burst while the view is being read.

Because the grouping decision is a product decision, verification judges it
against the learner's declared policy, never against a fixed answer key. The
schedule carries anchor pairs whose grouping any coherent policy fixes — a
verbatim reprint is the same story, two unrelated events are not — and
continuum pairs where verification checks only that the outcome is consistent
with the declared policy. Whether the policy itself is defensible is a
judgment call weighed against `EVALUATION.md` by whoever checks the work.
Verification does not inspect private functions and does not require any
named similarity technique.
