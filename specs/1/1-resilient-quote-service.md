---
status: draft
---

# Resilient quote service — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule applies a sustained latency increase to one provider at a
named request, injects provider errors at named requests, drives a burst above
measured capacity beginning at a named request, cancels the client of a named
in-flight request, and sends SIGTERM once a named request has been
acknowledged and while its work is still active. It runs both open-loop and
closed-loop traffic.

Checks do not inspect private functions or require a named concurrency
pattern. They observe API behavior, the expiry stamped on each returned quote,
process lifecycle, traces, metrics, queue or admission state exposed by the
design, and resource use.
